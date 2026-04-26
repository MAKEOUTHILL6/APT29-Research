
Also noticed that the operators avoid using the same C&C network infrastructure between different victim organizations. This kind of compartmentalization is generally only seen by the most meticulous attackers. It prevents the entire operation from being burned when a single victim discovers the infection and shares the related network IoCs with the security community.

The Dukes have employed several interesting tactics to hide the communications between the implants and their C&C servers, including the use of social media platforms and steganography

MiniDuke and HammerDuke leveraged Twitter to host their C&C URLs. In addition, they use a Domain Generation Algorithm (DGA) to generate new Twitter handles. Each time the malware generates a new handle, it fetches the Twitter page corresponding to that handle and searches the page for a specific pattern, which is the encrypted C&C URL

In CloudDuke, the operators leveraged cloud storage services such as OneDrive as their C&C channels. They were not the first group to use this technique, but it is generally effective for the attackers as it is harder for defenders to spot hostile connections to legitimate cloud storage services than to other “suspicious” or low-reputation URLs.

Moreover, the Dukes like to use steganography to hide data, such as additional payloads, in pictures. It allows them to blend into typical network traffic by transferring valid images while its true purpose is to allow the backdoor to communicate with the C&C server

### **Example How They Hide C2C URLs in Reddit**

![[c2c-apt29.png]]


**C&C server address retrieval from public webpages**

Strings from PolyglotDuke are decrypted using two different algorithms. The string is either RC4 encrypted using the CryptDecrypt API where the key is derived from the system directory path with the drive letter removed, or using the custom encryption algorithm shown in Figure 6. An IDA Python script to decrypt these strings is provided in our GitHub repository. The C&C server address is retrieved and decoded from various public webpages such as Imgur, ImgBB or Fotolog posts, tweets, Reddit comments, Evernote public notes, etc. Several encrypted public webpage URLs are hardcoded in each sample (from three to six URLs in a single sample) and it will iterate over the hardcoded list of C&C server addresses until it is able to decode a valid C&C URL successfully. An example of a public webpage containing an encoded C&C URL is shown:

![[apt29-c2c-x.png]]

### **To Analyze**

- How they leveraged Twitter to hide traffic, OneDrive and steganography.