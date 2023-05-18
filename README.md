# [pySDR] (https://pysdr.org/index.html)
## [Installing the HoRNDIS driver on Apple Silicon, macOS 12] (https://wiki.analog.com/university/tools/pluto/drivers/osx)
> This guide was mainly developed from [this thread](https://github.com/jwise/HoRNDIS/issues/146) with a few additions
### 1. Switching to "Reduced Security" mode
- Shut down your Apple Silicon Mac.
- Press and hold down the power button until the text under the Apple logo says "Loading startup options…", then let go.
- Select "Options".
- You are now in recoveryOS — enter your password if it asks.
- Go to Utilities → Startup Security Utility.
- Select "Reduced Security" and enable Allow user management of kernel extensions from identified developers".
- Shut down your Apple Silicon Mac.
### 2. Disabling SIP (System Integrity Protection)

**IMPORTANT:** Disabling SIP in any capacity, even partially, will also disable Apple Pay, as well as any iOS-on-macOS apps you may have downloaded from the App Store. This is a strange (and annoying) decision that Apple has decided to make specifically on Apple Silicon, as Apple Pay actually works fine even when SIP is disabled on x86_64 (Intel) Macs.

-Follow steps 2〜4 from above.
-Go to Utilities → Terminal.
-Type in the following to fully disable SIP: `csrutil disable`
**Note:** It is possible to only partially disable the part of SIP that enforces kext signature verification `csrutil enable --without kext`, but according to Apple, this is apparently an "unsupported configuration". Use it if you wish (as many do already), but please make sure to read and fully understand the warning that csrutil gives if you try.
Reboot your Apple Silicon Mac.
### 3. Compiling HoRNDIS for Apple Silicon (arm64e)

-Download and install Xcode.
-Install Xcode Command Line Tools by opening your Terminal and running `xcode-select --install`
-Switch Active Developer Directory `sudo xcode-select --reset`
-Agree to Xcode Licence `sudo xcodebuild -license`
-Now we can clone the repo. Run the following in Terminal:
```
git clone --recursive https://github.com/jwise/HoRNDIS.git
cd HoRNDIS
xcodebuild -sdk macosx -configuration Release
sudo cp -rv build/Release/HoRNDIS.kext /Library/Extensions/
```
-Go to System Preferences → Security & Privacy and approve the HoRNDIS kernel extension.

