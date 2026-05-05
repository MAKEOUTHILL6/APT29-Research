
### Intro
Weaponization is where recon output becomes a deployable weapon, in this part of the cyber kill chain, APT29 focuses on the social engineering factor - meaning how would they make a spear-phishing email look legit and also the technical part of the malware sent. It's important to note that for every target they have a different C&C network infrastructure, this compartmentalization prevents an entire campaign from being exposed when one of the victims discovers that the adversaries have gotten into the environment of the organization and share network IoCs. It was observed that the threat actors generally worked from 9 AM to 5PM UTC+3, the idea of their operational times generated from these sources based from **ESET**:

• The time at which they uploaded C&C pictures to the Dropbox account used by RegDuke
• The time at which they posted encoded C&C URLs on the social media accounts used by PolyglotDuke
• The compilation timestamps of dozens of samples. We believe they were not tampered with, as they are consistent with what we see in ESET telemetry data.

### Stage 1 Malware Deployment - PolyglotDuke

**PolyglotDuke** this is the initial dropper, this is what gets dropped to the affected system, which uses Twitter or other websites such as Reddit and Imgur to get its C&C URL. It also relies on steganography in pictures for its C&C communication and has its own custom string encryption implementation, the goal is to download **MiniDuke** onto the system. We will cover both method of delivery - via **PE file** and **macro-embedded Word documents**.

**Step by step malware delivery guide for PE file:**

The dropper is the first executable - the **PE file** that gets dropped via the spear-phishing email and hidden inside is the **PolyglotDuke DLL**, the dropper's job is to decrypt **Polyglot Duke**, you can think of the dropper as a matryoshka doll.

### The dropper is a PE File. You may ask yourself: "What is a PE file?"

Every Windows executable — whether a `.exe` program or a `.dll` library — is packaged in a format called Portable Executable (PE). Think of it as a structured container with labeled compartments. One of those compartments is called the `.rsrc` section — a filing cabinet where programs store embedded assets like icons, images, and dialog templates. Windows treats this section as passive storage. APT29 used it as a hiding place for **PolyglotDuke's DLL**.

#### The dropper's disguise — naming and placement

The dropper executable itself was named to mimic legitimate software components already expected to exist on a corporate workstation. Across documented campaigns, APT29 used names associated with Adobe, Apple, Dell, HP, and Skype — software present in virtually every enterprise environment. The dropped file was placed in a corresponding directory under `%APPDATA%\Roaming\[LegitSoftwareName]\`, and persistence was established via an HKCU Run key using a name matching the same software family — entries like `WebCache`, `ToolboxFX`, `ConnectionCenter`, and `Communicator`.

#### The signed payload verification

Before PolyglotDuke executes any payload retrieved from its C2 infrastructure, it performs a cryptographic integrity check. The downloaded file — delivered disguised as a JPEG or PNG image — contains an encrypted payload blob with a signed SHA-1 hash appended. PolyglotDuke verifies this hash signature against an RSA public key hardcoded in the binary. Only payloads signed with APT29's corresponding private key pass the check and execute. Any unsigned or incorrectly signed payload is silently rejected.

The idea is that if a security researcher or law enforcement agency identifies and seizes a C2 domain, they cannot weaponize that access against existing infections. Feeding a tracking payload or a neutralizing update to live implants is architecturally impossible without the private signing key — which never leaves APT29's infrastructure. This is sinkholing prevention built directly into the malware's design, not an afterthought. It reflects an operational maturity that assumes partial infrastructure exposure as a risk worth engineering against.

### You may also wonder "What a DLL is and why it matters here"

A DLL (Dynamic Link Library) is a PE file that cannot run on its own. It must be loaded into another process. When loaded, Windows calls its entry point — a function called `DllMain` — automatically. This means the moment a DLL is loaded, its code executes. APT29 weaponized this behavior by hiding a DLL inside `.rsrc` of a `.exe file` that looked like a GIF image, extracting it at runtime, and loading it through a legitimate Windows binary so the malicious code executed without ever appearing as a standalone suspicious file.

### The GIF resource — hiding in plain sight

Inside the dropper's `.rsrc` section, APT29 embedded an entry typed as `GIF` with resource ID `129`. To any static scanner inspecting the file, this appeared to be an embedded GIF image — the resource type matched, and the raw bytes began with `GIF89`, the standard five-byte magic header that identifies valid GIF files. What followed those five bytes was not image data. It was the encrypted PolyglotDuke DLL.

This technique exploits a simple assumption: security tools inspecting PE resources check the type label and the file header magic bytes. If both look legitimate, the resource is typically ignored. APT29 satisfied both checks deliberately. The magic bytes served double duty — they authenticated the fake GIF to scanners while simultaneously providing the encryption key for the payload hidden beneath them.

### How did the decryption of the encrypted **Polyglot Duke DLL** in the `.exe' .rsrc` works

To decrypt the payload, APT29 used a custom algorithm that required no Windows cryptography APIs — no calls to `CryptDecrypt`, no imported crypto libraries. This mattered operationally because API-level hooks used by security tools to monitor cryptographic operations would see nothing.

The algorithm operated on three values for each byte position `i`:

```
plainByte = (i / 5) XOR encryptedByte XOR GIF89[i % 5]
```

The first value — `GIF89[i % 5]` — cycles through the five magic bytes (`G`, `I`, `F`, `8`, `9` in hex: `0x47 0x49 0x46 0x38 0x39`) as a repeating XOR key. The second value — `i / 5` — is the byte's position divided by five using integer division, producing a counter that increments every five bytes. The third value is the encrypted byte from the resource itself. XOR combines all three.

The result of this design is that the encryption key is never stored anywhere in the binary as a separate value. It is derived from the file's own header — the GIF magic bytes that also serve as camouflage. A reverse engineer examining the dropper finds no suspicious key material because the key is hiding as what appears to be a normal file format identifier.

### What happens after decryption?

Once decrypted, the payload buffer begins with the bytes `4D 5A` — the PE magic bytes identifying a valid Windows executable. The dropper writes this DLL to the current working directory using standard Windows file APIs, then executes it using `rundll32.exe` — a legitimate, Microsoft-signed Windows binary whose purpose is loading and executing DLL functions.

This execution method is a Living Off the Land (LOTL) technique. Because `rundll32.exe` is a trusted system binary, its presence in a process list raises no immediate flags. Many legitimate applications use it. The malicious DLL inherits that legitimacy by being loaded through it.

By the time PolyglotDuke's `DllMain` executes, the dropper has done its job. It can delete itself. The payload operates independently, and the only artifact connecting it to the dropper is the dropped DLL file in the working directory — which itself looks like a legitimate file due to its naming conventions mirroring real software components.


### The Big Picture

![[Pasted image 20260505010004.png]]


### Macro-enabled PowerDuke
#### The PowerDuke variant — macro documents and Alternate Data Streams

Running parallel to the PolyglotDuke dropper approach, APT29 deployed a second weaponization method in campaigns documented by Volexity in 2016. Rather than a standalone executable, the payload was delivered through macro-embedded Microsoft Word and Excel documents — `.docm` and `.xls` files whose visible content was legitimate, repurposed material from the institutions being impersonated.

When the victim opened the document and enabled macros, the embedded VBA code executed a dropper that wrote two files to disk. The first was a small DLL with a legitimate-looking name and extension — `.lock`, `.dat`, or `.idx` — placed in an `%APPDATA%\Roaming\` subdirectory. The second was an Alternate Data Stream attached to that file, named with a `:schemas` suffix. This stream contained the actual PowerDuke backdoor component, encrypted using the Tiny Encryption Algorithm (TEA).

Alternate Data Streams are a feature of the NTFS filesystem that allow data to be attached to a file invisibly. A file named `kxwn.lock` can have a hidden stream `kxwn.lock:schemas` that does not appear in Windows Explorer, does not show up in a standard directory listing, and contributes nothing to the visible file's size. Only tools explicitly querying for ADS — such as `dir /r` on the command line or Sysinternals Streams — reveal its existence. The legitimate-looking host file provides cover; the payload lives in a compartment most investigators would never open.

TEA was chosen for this variant rather than the custom XOR algorithm used in PolyglotDuke for a consistent reason — no dependency on Windows cryptography APIs. TEA is implementable in a handful of lines of code with no external library requirements, leaving no API call signature for security tools monitoring cryptographic function imports.

Persistence was established via an HKCU Run key invoking `rundll32.exe` against the visible host file — for example `rundll32.exe %APPDATA%\Roaming\Microsoft\kxwn.lock, #2` — which loaded the host DLL, which in turn read and decrypted the ADS stream containing the real payload. The chain maintained the LOTL principle throughout: every executing binary was either a legitimate Windows component or a file that appeared to belong to installed software.

#### Design decision analysis — why these specific choices

The technical choices across both weaponization variants follow a consistent design philosophy that is worth making explicit. Every decision optimizes for the same objective: remaining undetected long enough to establish persistent access.

Custom encryption over standard algorithms avoids the API call signatures that endpoint security tools monitor. A call to `CryptDecrypt` or `BCryptDecrypt` is a known indicator — custom XOR and TEA implementations call nothing that a hook can intercept. The GIF resource type and magic bytes satisfy two scanner assumptions simultaneously — the file type label and the header magic — at the cost of zero additional complexity. The ADS hiding technique exploits a filesystem feature so rarely examined by routine investigation that it provides effective concealment without any active evasion. The signed payload verification protects the most valuable asset — the final stage implant — from being neutralized through infrastructure interdiction.

Taken together these choices describe a development team that understood defensive tooling well enough to design around it specifically, rather than broadly. Each evasion targets a particular detection mechanism rather than applying generic obfuscation.

Tutorial on how this works in practice is in the **APT29 Simulation** part of the research.

### From The Defender's Perspective

**What to look for:**

- `.rsrc` sections with unusually high entropy — legitimate image resources compress easily; encrypted content does not
- Resource Hacker or PE-bear: open the dropper and inspect the GIF-typed resource — if the bytes don't decompress to a valid image, investigate further
- `rundll32.exe` spawning from an unexpected parent process (cmd.exe, a document application, a shortcut handler)
- Sysmon Event ID 7 (image loaded) flagging unsigned DLLs being loaded by `rundll32.exe`
- New DLL files appearing in `%APPDATA%` or the working directory of a document application
- Sysmon Event ID 11 (file created) showing DLL creation immediately after a document opens
- An analyst reviewing a process list or startup entries sees what appears to be a legitimate background component of installed software. The anomaly is only detectable by cross-referencing the file path, checking the DLL's signature (absent), and comparing its hash against known-good versions. Each step requires active investigation — the default assumption favors the attacker.
#### MITRE ATT&CK mapping

|Technique|ID|APT29 usage|
|---|---|---|
|Software packing|T1027.002|Encrypted PolyglotDuke DLL in PE resource|
|Steganography|T1027.003|TEA-encrypted payload in ADS stream|
|DLL side-loading|T1574.002|rundll32 loading malicious DLL|
|Code signing|T1553.002|RSA signed payload verification|
|Visual Basic macro|T1059.005|Macro documents dropping PowerDuke|
|Deobfuscate/decode|T1140|Runtime GIF89 XOR and TEA decryption|
|Registry run keys|T1547.001|HKCU Run key persistence across both variants|
