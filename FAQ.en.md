####General FAQ:
- _Do I need Shiki?_  
If you have Intel Ivy CPU or newer, iTunes DRM playback does not work for you, and your GPU works with HDCP, you may try it.  
Sometimes it might help even improperly configured or problematic Sandy CPUs.

- _How to disable Shiki?_  
Add -shikioff argument to boot-args. It will also not load when -x or -s are set.

- _How to turn debug printing on in DEBUG builds of Shiki?_  
Add -shikidbg argument to boot-args.

- _What OS X version does this work with?_  
It is meant to work with 10.9 and newer. However, it is not known whether it works with all the minor releases.  
Note that Shiki needs a relatively recent version of iTunes. Tested mostly with 12.3 and newer.  
You should also know that you might get random boot panics with OsxAptioFix(2) and 10.10, a solution is in the works.

- _How do I load Shiki?_  
You are supposed to load it early when your system boots. It is recommended to use bootloader kext injection.  
`kextload` is known not to work and /S/L/E or /L/E might fail.

- _Is it dangerous to use?_  
Any instrument can be dangerous in improper hands. Shiki is stable at this point but it is still considered to be somewhat experimental.

- _Does Shiki modify files on my disk?_  
Recent versions of Shiki affect nothing on your disk, however, certain I/O commands will report files as modified when Shiki is loaded due to the specifics of API hooks.

- _Do I need to alter System Integrity Protection (rootless) settings?_  
No, they are irrelevant.

- _How do I check if my system is configured correctly before trying the kext?_  
There is a ticklist to check against for all sorts of configurations. See configuration checklist section.

- _Why am I supposed to properly configure VDA before using Shiki?_  
You may theoretically not do this but the consequences will be unspecified.

- _Which GPUs this solution is known to glitch with?_
   - Azul GPUs (e.g. HD 4400, HD 4600) when used with a connector-full platform-id without a discrete GPU cause a system freeze due to not working HDCP playback. The issue is unrelated to Shiki, and happens regardless of iTunes and QuickTime.  
   - Certain GPUs (e.g. HD 4000) cause iTunes crashes when playing paid HD movies for unspecified reasons if hardware video decoding is enabled. The issue is likely caused by some Apple bug and supposedly involves HDCP handling. A fresh system or a DRM cleanup (see below) usually help to solve the issue otherwise try using the guide for unsupported CPUs.

- _Is my computer banned?_  
If you are able to view the trailers but bought movies do not play even after computer authorisation your NIC MAC might be banned. Sometimes it is possible to log out and reauthorise your computer after a short while. Otherwise you must change your LAN MAC address.
Check System configuration FAQ for libHookMac usage details. If it helps you consider flashing your ethernet card with a different MAC (or using other ways to change it).

- _Is Shiki opensource?_  
Not completely at this moment. Certain parts like tools or [kernel patcher](https://github.com/vit9696/AppleALC/tree/master/Common) are.

- _How can I download a DRM protected trailer for a local test?_  
Enable coremedia tracing: `defaults write com.apple.coremedia cfbyteflume_trace 1`  
Afterwards you will see a trailer URL in Console.app if you filter the output by 'iTunes' keyword.  
`... <<< CFByteFlume >>> FigCFHTTPCheckCacheValidator: Comparing dictUrl = http://.....m4v, url = http://......m4v`  
Paste it into your browser, and you will be able to download the trailer file ([example](https://applelife.ru/threads/chernye-trejlery-itunes.42290/page-11#post-583709)).  

- _DRM playback starts to produce garbled frames after a while, why does it happen?_  
From the tests it happens to be a bug in Apple DRM decoder, and the issue supposedly exists on Apple hardware as well.  
Normally this does not happen anywhere but 1080p movies with a large bitrate. Try rebooting your PC, resetting DRM configuration as prescribed in the FAQ, checking your CPU power management. 

- _Can I play HTML5 Netflix videos with Shiki?_  
No, you cannot since Netflix limited their 1080p content for very few mac models even excluding most Mac Pros.

- _Can I use Shiki to apply other changes?_  
It may be possible and can be discussed.

####System configuration FAQ:
- _How can I check that hardware video decoding works?_  
Run [VDADecoderChecker](https://github.com/cylonbrain/VDADecoderCheck/releases/download/0.20/VDADecoderChecker) and check its output:  
`GVA info: Successfully connected to the Intel plugin, offline Gen75`  
`Hardware acceleration is fully supported`

- _How can I check that IMEI/IGPU device is present in IOReg?_  
Run `ioreg | grep IMEI` in Terminal and make sure something alike is printed:  
`    | |   +-o IMEI@16  <class IOPCIDevice, id 0x100000209, registered, matched, active, busy 0 (6 ms), retain 11>`

- _How can I check my platform-id?_  
Run `ioreg -l | grep platform-id` in Terminal and make sure something alike is printed:  
`    | |   | |   "AAPL,ig-platform-id" = <04001204>`  
`04 00 12 04` is the current platform-id.

- _How can I enable AppleGVA debugging?_  
Run:  
`defaults write com.apple.AppleGVA gvaDebug -boolean yes`  
`defaults write com.apple.AppleGVA enableSyslog -boolean yes`

- _How can I set hardware video decoder preferences?_  
Run one of the following lines:  
`defaults write com.apple.AppleGVA forceNV -boolean yes`  — forces NVIDIA decoder  
`defaults write com.apple.AppleGVA forceATI -boolean yes` — forces ATI decoder  
`defaults write com.apple.AppleGVA forceIntel -boolean yes` — forces Intel decoder  
`defaults write com.apple.AppleGVA forceSWDecoder -boolean yes` — forces software decoder  
`defaults write com.apple.coremedia hardwaveVideoDecoder disable` — disables hardware decoder  
`defaults write com.apple.coremedia hardwaveVideoDecoder force` — forces hardware decoder  

- _How do I reset my DRM configuration (may be needed to fix the crashes)?_  
Run the following commands in terminal:  
`defaults delete com.apple.coremedia`  
`defaults delete com.apple.AppleGVA`  
`sudo rm -rf /Users/Shared/SC\ Info`  
`sudo defaults delete com.apple.coremedia`  
`sudo defaults delete com.apple.AppleGVA`  
Afterwards make sure AppleGVA sigature is valid (the command should output nothing):  
`codesign --no-strict --verify /System/Library/PrivateFrameworks/AppleGVA.framework`  
If it is not, restore AppleGVA.framework from a newly installed system with the correct permissions.  
Reboot twice.

- _How can I change my NIC MAC address via [libHookMac.dylib](https://github.com/vit9696/Shiki/raw/master/HookMac/libHookMac.dylib)?_  
   - Disable SIP (System Integrity Protection);  
   - Run in Terminal (specifying your own random MAC):   
    `DYLD_INSERT_LIBRARIES=/full/path/to/libHookMac.dylib MAC=00:11:22:33:44:55 /Applications/iTunes.app/Contents/MacOS/iTunes`;
   - You will see a corresponding print if the address was changed.

- _How can I check that my dyld shared cache is ok?_  
Run the following commands and compare their output:  
`DYLD_PREBIND_DEBUG=1 DYLD_SHARED_CACHE_DONT_VALIDATE=1 "/Applications/QuickTime Player.app/Contents/MacOS/QuickTime Player"`  
`DYLD_PREBIND_DEBUG=1 "/Applications/QuickTime Player.app/Contents/MacOS/QuickTime Player"`  
If their output differs quite a bit (the one without DYLD_SHARED_CACHE_DONT_VALIDATE=1 prints a lot of paths), then your shared cache is corrupted.  
To fix it up disable Shiki, reboot, and run `sudo update_dyld_shared_cache -force` command in Terminal.

- _Which mac models support hardware acceleration?_  
To check that read `/System/Library/PrivateFrameworks/AppleGVA.framework/Info.plist`, if your mac model or board id is present there, then this model does support hardware video decoding acceleration. You are to select a closest configuration to the one you own. For example, iMac13,1 uses an IGPU/discrete GPU combo whereas iMac13,3 only has an IGPU. If you use a model meant to work with a discrete GPU without a graphical card installed VDA will not work and you are likely to get an error from VDADecoderChecker. To correct this either choose an accurate model or edit the `forceOfflineRenderer` property in the Info.plist, it will need to be set to NO.

- _How can I inject IOVARendererID/IOVARendererSubID in certain NVIDIA GPUs?_  
NVIDIA drivers do not properly add these values necessary for VDA decoding for Maxwell GPUs in their Web drivers. You could add them with a plist-only kext. The correct values for VP4 GPUs are:  
IOVARendererID    → `<08 00 04 01>`  
IOVARendererSubID → `<03 00 00 00>`  
VP3 ones want a different IOVARendererID → `<04 00 04 01>`.  
Thanks to igork for noticing it.

- _I cannot get VDA decoder work with my AMD GPU, what could I try?_  
Prioritising Intel and using connector-full platform-id (e.g. `<03 00 66 01>` for HD 4000, `<03 00 22 0D>` for HD 4600) seems to help with certain AMD GPUs (e.g. HD 7750).

- _I get hardware accelerated decoding working on my AMD with forced ATI decoder but DRM decoding still does not work, what is up?_  
Certain AMD GPUs, e. g. HD 7750, do support hardware accelerated video decoding but fail to decode DRM video. The cause is unknown. Use Shiki normally.

- _What is [BoardHash](https://github.com/vit9696/Shiki/raw/master/BoardHash/BoardHash) tool for?_  
BoardHash tool can generate mac board id hashes similar to the ones present in CoreAUC.framework (_PsZXJ2EK7ifxrtgc function).  
For example, Mac-F221BEC8 (MacPro5,1) stands for 5f571162ce99350785007863627a096bfa11c81b.  
It seems to have hashes of the macs with special HDCP permissions. E. g. it is known that MacPro5,1 model makes HD movies work on HD 4000 regardless of decoder state. 

####Configuration checklist

- _Non-Shiki based solution if you have:_  
   - AMD with working DRM VDA (e.g. HD 7870, HD 6670, HD 7970);
   - NVIDIA with working DRM VDA (supposedly 2xx series and some others).  

  This solution is not well explored but it is known to work for some people. Kext installation is not needed.
   - IGPU device is enabled, set to preferred in BIOS, and present with a connector-full AAPL,ig-platform-id prop (e.g. `<03 00 66 01>`) or removed and disabled completely without a trace in IOReg (e. g. via [D2EN register](https://applelife.ru/threads/chernye-trejlery-itunes.42290/page-14#post-584519));
   - Hardware video decoder preferences are forced to ATI or NVIDIA depending on the GPU installed;
   - VDADecoderChecker confirms VDA decoder working with VP3 (NVIDIA) or AMD decoders;
   - Mac model set to the one supporting hardware acceleration.

- _Shiki-based solution for non-freezing Intel and/or any discrete GPU:_  
   - IGPU device is enabled, and present with a connector-less AAPL,ig-platform-id prop (e.g. `<04 00 12 04>`, `<07 00 62 01>`), use connector-full framebuffers in solo mode;
   - Discrete GPU is properly configured with IOVARendererID present in IOReg;
   - VDADecoderChecker confirms Intel Offline (Online) VDA decoder working;
   - IMEI device is present in IOReg;
   - No override preferences are used;
   - Mac model set to the one supporting hardware acceleration.

- _Shiki-based solution for an unsupported CPU (e.g. Haswell Celeron E):_  
  Hardware video decoding acceleration does not work with these CPUs and to boot you need to fake your CPUID.  
  Disable IGPU completely or rename it to some random name (e.g. IGFX) and install Shiki, it should work for you.
  It is not fully explored what preferences are needed but it is known that disabled hardware acceleration by AppleGVA plist editing/MacPro5,1 model setting helps to view HD movies sometimes.  
  

_Thanks to: 07151129, Andrey1970, Сашко666, chrome, family1232009, garcon, iDark Soul, igork, m-dudarev, Mieze, tatur_sn, Quadie, and certain others._