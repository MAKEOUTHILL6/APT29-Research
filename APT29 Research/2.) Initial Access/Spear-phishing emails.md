
### **Conclusions so far, prone to updates when I have more info**

- APT29 uses either personally crafted phishing emails, originating from trusted or long-term domains to look more secure and legit, the contents inside the emails are either - direct .lnk downloads or embedded macros in documents, all of this is to somehow deliver their malware PowerDuke\PolygotDuke\RegDuke backdoor to the compromised system in order to initiate a connection to their C2 server.

### **Intro**
The group’s main initial tactic to breach a network is to send spear-phishing emails that contain a link or an attachment. In order to increase the attackers’ chances, it is designed to be a subject of particular interest of the recipient. These spear phishing messages were spoofed and made to appear to have been sent from real individuals at well-known think tanks in the United States and Europe.

### **For Analysis**
The Dukes are known for launching their attacks by sending links to ZIP files, that contain malicious executables, hosted on legitimate compromised web servers. However, each of the e-mail messages from the August attacks contained a Microsoft Office Word (.doc) or Excel (.xls) attachment. These attachments, when viewed, contained legitimate report content from each of the organizations they appeared to have been sent from. However, the attackers inserted macros into the documents designed to install a malware downloader on the system. Successful exploitation would result in the download of a PNG image file from a compromised webserver. These attack campaigns leveraged steganography in the PNG files by hiding components of a backdoor that would exist only in memory after being loaded into rundll32.exe. Volexity has dubbed this backdoor **PowerDuke**. Similar attack campaigns using documents with macros dropping PowerDuke were further observed through October, where Universities, and not think tanks appear to have been the primary targets. Details of these attacks have been provided to Volexity customers. Concerned NGO’s and Universities that may have been targeted by these attacks campaigns are welcome to reach out for additional details.

**Personal Comments and Thoughts**
- So basically the initial process of compromise contained stagers/stages as seen with PowerDuke which is an embedded macro in a document used for setting the first stage of the compromise - investigate further what PowerDuke is doing, and how it works, also keep in my different stages of the attack.
- There were also time when they just point links to files hosted on web servers, other times they have macro embedded - investigate how this is done via examples below
- Also how do they get info on the target about what exactly to send as a phishing mail


### **Example Attack With File Download**

The first attack wave is similar to much older attacks from the Dukes that purport to be an electronic Fax. This message claims to have been sent from Secure Fax Corp. and has a link to a ZIP file that contains a Microsoft shortcut file (.LNK). This shortcut file contains PowerShell commands that conduct anti-VM checks, drop a backdoor, and launch a clean decoy document. The e-mail message was sent from the attacker controlled e-mail account **industry.faxsolution@gmail.com**. The screen shot below shows the e-mail that was sent.

![[cozy-efax-link-apt29.png]]
The e-mail contained links pointing to the following URL:

hxxp://efax.pfdweek[.]com/eFax/message0236.ZIP

Inside of this password (1854) protected ZIP file is a Microsoft shortcut file named:

**37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk**

Note that **pfdweek[.]com** appears to be under the control of the attackers but may be a hijacked domain.

Details on each of the files are included below.

**Filename:** message0236.ZIP  
**File size:** 643843 bytes  
**MD5 hash:** bea0a6f069bd547db685698bc9f9d25a  
**SHA1 hash:** ee09bec09388338134d47fa993d5e0f86efe5bd4  
**Notes:** Password protected ZIP file containing malicious Microsoft shortcut file (37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk)

**Filename:** 37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk  
**File size:** 724003 bytes  
**MD5 hash:** c272aebc661c54cc960ba9a4a3578952  
**SHA1 hash:** 52d62213c66a603e33dab326bf4fa29d6ac681c4  
**Notes:** Microsoft shortcut file with embedded PowerShell, PowerDuke backdoor (hqwhbr.lck), and clean decoy document.

**Filename:** kxwn.lock  
**File size:**  10752 bytes  
**MD5 hash:** 28b95a2c399e60ee535c32e73860fbea  
**SHA1 hash:** bf4ce67b6e745e26fcf3a2d41938a9dff1395076  
**Notes:** Primary PowerDuke backdoor (DLL) loader (leverages kxwn.lock:schemas) dropped to “%APPDATARoamingMicrosoft” with persistence via HKCU Run Key “WebCache” (rundll32.exe %APPDATARoamingMicrosoftkxwn.lock , #2). Connects directly to **173.243.80.6:443** for command and control.

**Filename:** kxwn.lock:schemas  
**File size:**  609853 bytes  
**MD5 hash:** 4e1dec16d58ba5f4196f6a76a0bca75c  
**SHA1 hash:** a7c43d7895ecef2b6306fb00972c321060753361  
**Notes:** Alternate data stream (ADS) PNG  file with the PowerDuke backdoor component hidden and encrypted within using Tiny Encryption Algorithm (TEA).

### **Example Attack With Embedded Macro**

The second attack wave that Volexity observed leveraged a Microsoft Word document with a malicious embedded macro. This appears to be consistent with several previous Dukes attack campaigns, such as those on August 25, 2016. The Macros contain several anti-VM checks designed to avoid executing in virtualized environments. The e-mail message was sent from the attacker controlled e-mail account **securefaxsolution@gmail.com**.

![[cozy-efax-doc-apt29.png]]

Details on the malware components of this attack wave are included below.

**Filename:** election-headlines-FTE2016.docm  
**File size:** 835072 bytes  
**MD5 hash:** a8e700492e113f73558131d94bc9ae2f  
**SHA1 hash:** b5684384c8028f0324ed7119f6abf379f2789970  
**Notes:** Document containing malicious macro that drops

**Filename:** fywhx.dll  
**File size:** 10752 bytes  
**MD5 hash:** ad6723f61e10aefd9688b29b474a9323  
**SHA1 hash:** dd766876b3be5022bfb062f454f878abfbc670b8  
**Notes:** PowerDuke backdoor file dropped to “%APPDATARoamingHP” with persistence via HKCU Run Key “ToolboxFX” (rundll32.exe %APPDATARoamingHPfywhx.dll #2). Connects directly to **185.132.124.43:443** for command and control.

**Filename:** fywhx.dll:schemas  
**File size:**  608854 bytes  
**MD5 hash:** 8c53ee9137a7d540fcff0d523f7d0822  
**SHA1 hash:** ab32c09c46e0c9dbc576fefee68e5a2f57e0482e  
**Notes:** Alternate data stream (ADS) PNG  file with the PowerDuke backdoor component hidden and encrypted within using Tiny Encryption Algorithm (TEA).

### **Example of Wide Spread Attack**

Volexity believes the following e-mail received the widest distribution among the targeted organizations. The e-mail purports to have been sent from Harvard’s “PDF Mobile Service” or “PFD Mobile Service”. The spelling of this non-existent service is inconsistent in the e-mail.  The latter spelling appears to be a typographical error that is consistent with the domain names registered by the attackers. The screen shot below shows the e-mail that was sent.

![[coz-link1-1 1.png]]

The e-mail contained links pointing to the following URL:

hxxp://efax.pfdresearch[.]org/eFax/RWP_16-038_Norris.ZIP

Inside of this password (8734) protected ZIP file is an executable named:

**RWP16-038_Norris.exe**

Note that **pfdresearch[.]org** appears to be under the control of the attackers but may be a hijacked domain.

Details on the malware components of this attack wave are included below.

**Filename:** RWP_16-038_Norris.ZIP  
**File size:** 854996 bytes  
**MD5 hash:** 8b3050a95e3ce00424b85f6e9cc3ccec  
**SHA1 hash:** d5dcf445830c54af145c0dfeaebf28f8ec780eb5  
**Notes:** Password protected ZIP file with malicious executable inside (RWP16-038_Norris.exe).

**Filename:** RWP16-038_Norris.exe  
**File size:** 1144832 bytes  
**MD5 hash:** 3335f0461e5472803f4b19b706eaf4b5  
**SHA1 hash:** 5cc807f80f14bc4a1d6036865e50d576200dfd2e  
**Notes:** Dropper for PowerDuke backdoor and clean decoy document

**Filename:** gwV46iIc.idx  
**File size:**  10752 bytes  
**MD5 hash:** ae997d2047705ff46a0c228f7b5d7052  
**SHA1 hash:** 1067ddd5615518e0cbac7389a024b32f119a3229  
**Notes:** Primary PowerDuke backdoor (DLL) loader (leverages gwV46iIc.idx:schemas) dropped to “%APPDATARoamingApple” with persistence via HKCU Run Key “ConnectionCenter” (rundll32.exe %APPDATARoamingApplegwV46iIc.idx, #2). Connects directly to **185.124.86.121:443** for command and control.

**Filename:** gwV46iIc.idx:schemas  
**File size:**  580968 bytes  
**MD5 hash:** 7b9b51cb44cd6a7af1cd28faeeda04a7  
**SHA1 hash:** e3bd7bdfe0026cf4ee39fd75a771eac52ffea095  
**Notes:** Alternate data stream (ADS) PNG  file with the PowerDuke backdoor component hidden and encrypted within using Tiny Encryption Algorithm (TEA).


### **Example Attack of .DOC Macro**

The fourth attack wave that Volexity observed leveraged a Microsoft Word document with a malicious embedded macro. This appears to be consistent with several previous Dukes attack campaigns, such as those on August 25, 2016. The Macros contain several anti-VM checks designed to avoid executing in virtualized environments. The screen shot below shows the e-mail that was sent.

![[cozy-doc-1.png]]
![[coz-doc-bottom-1.png]]
Details on the malware components of this attack wave are included below.

**Filename:** harvard-iop-fall-2016-poll.doc  
**File size:** 2808832 bytes  
**MD5 hash:** ead48f15ebc088384a4bd6190c2343fa  
**SHA1 hash:** 0b9dccfcb2cc8bced343b9d930e475f1d0e5d966  
**Notes:** Document containing malicious macro that drops impku.dat and impku.dat:shemas.

**Filename:**  impku.dat  
**File size:** 10752 bytes  
**MD5 hash:** 9f420779c90e118a0b5fd904380878a1  
**SHA1 hash:** 11523d859e9a818c2628d7954502cbdb5eeb2199  
**Notes:** PowerDuke backdoor file dropped to “%APPDATARoamingDell” with persistence via HKCU Run Key “Communicator” (rundll32.exe %APPDATARoamingDellimpku.idat, #2). Connects directly to **185.26.144.109:443** for command and control.

**Filename:** impku.dat:schemas  
**File size:**  608854 bytes  
**MD5 hash:** b774f39d31c32da0f6a5fb5d0e6d2892  
**SHA1 hash:** ae3ff39c2a7266132e0af016a48b97d565463d90  
**Notes:** Alternate data stream (ADS) PNG  file with the PowerDuke backdoor component hidden and encrypted within using Tiny Encryption Algorithm (TEA).

### **Another Example Attack of Malicious Attachment**

![[cozy-link2-bottom.png]]

As seen in the screen shot above, the e-mail contained links pointing to the following URL:

hxxp://efax.pfdregistry[.]net/eFax/37486.ZIP

Inside of this password (6190) protected ZIP file a Microsoft Shortcut file named:

**37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk**

Note that **pfdregistry[.]net** appears to be under the control of the attackers but may be a hijacked domain.

Details on the malware components of this attack wave are included below.

**Filename:** 37486.ZIP  
**File size:** 580688 bytes  
**MD5 hash:** f79caf27a99c091e6c1775b306993341  
**SHA1 hash:** a76c02c067eae26d78f4b494274dfa6aedc6fa7a  
**Notes:** Password protected ZIP file containing malicious Microsoft shortcut file 37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk.

**Filename:** 37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk  
**File size:** 661782 bytes  
**MD5 hash:** f713d5df826c6051e65f995e57d6817d  
**SHA1 hash:** 68ce4c0324f03976247ff48803a7d988f9f9f43f  
**Notes:** Microsoft shortcut file with embedded PowerShell, PowerDuke backdoor (hqwhbr.lck), and clean decoy document.

**Filename:** hqwhbr.lck  
**File size:** 10752 bytes  
**MD5 hash:** 57c627d68e156676d08bfc0829b94331  
**SHA1 hash:** 4bcbf078a78ba0e842f78963ba9dd71240ab6a6d  
**Notes:** PowerDuke backdoor file dropped to “%APPDATARoamingSkype” with persistence via HKCU Run Key “IAStorIcon” (rundll32.exe %APPDATARoamingApplehqwhbr.lck, #2).  Connects directly to **177.10.96.30:443** for command and control.

**Filename:** hqwhbr.lck:schemas  
**File size:** 547636 bytes  
**MD5 hash:** cbf96820dc74a50a91b2b8b94376682a  
**SHA1 hash:** 5f105801a1abb398dadc756480713f9bd7a4aa73  
**Notes:** Alternate data stream (ADS) PNG  file with the PowerDuke backdoor component hidden and encrypted within using Tiny Encryption Algorithm (TEA).