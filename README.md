# [plutoSDR tutorial](https://wiki.analog.com/university/tools/pluto/users/quick_start)
## [1. Installing the HoRNDIS driver on Apple Silicon, macOS 12](https://wiki.analog.com/university/tools/pluto/drivers/osx)
> HoRNDIS is a driver for Mac OS X that allows you to use your RNDIS to get network access to Pluto. It is required for Remote Network Driver Interface Specification (RNDIS) which is a USB protocol to provides a virtual Ethernet link.
### Switching to "Reduced Security" mode
- Shut down your Apple Silicon Mac.
- Press and hold down the power button until the text under the Apple logo says "Loading startup options…", then let go.
- Select "Options".
- You are now in recoveryOS — enter your password if it asks.
- Go to Utilities → Startup Security Utility.
- Select "Reduced Security" and enable Allow user management of kernel extensions from identified developers".
- Shut down your Apple Silicon Mac.
### Disabling SIP (System Integrity Protection)

**IMPORTANT:** Disabling SIP in any capacity, even partially, will also disable Apple Pay, as well as any iOS-on-macOS apps you may have downloaded from the App Store. This is a strange (and annoying) decision that Apple has decided to make specifically on Apple Silicon, as Apple Pay actually works fine even when SIP is disabled on x86_64 (Intel) Macs.

- Follow steps 2〜4 from above.
- Go to Utilities → Terminal.
- Type in the following to fully disable SIP: `csrutil disable`

<br> **Note:** It is possible to only partially disable the part of SIP that enforces kext signature verification `csrutil enable --without kext`, but according to Apple, this is apparently an "unsupported configuration". Use it if you wish (as many do already), but please make sure to read and fully understand the warning that csrutil gives if you try.
Reboot your Apple Silicon Mac.
### Compiling HoRNDIS for Apple Silicon (arm64e)

- Download and install Xcode.
- Install Xcode Command Line Tools by opening your Terminal and running `xcode-select --install`
- Switch Active Developer Directory `sudo xcode-select --reset`
- Agree to Xcode Licence `sudo xcodebuild -license`
- Now we can clone the repo. Run the following in Terminal:
```
cd ~
git clone --recursive https://github.com/jwise/HoRNDIS.git
cd HoRNDIS
xcodebuild -sdk macosx -configuration Release
sudo cp -rv build/Release/HoRNDIS.kext /Library/Extensions/
```
- Go to System Preferences → Security & Privacy and approve the HoRNDIS kernel extension.


## 2. SSH config
> Describing the USB device

- First, we need to install the Homebrew package managerL
    - `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` and follow the installation commands
    - Then install wget using `brew install wget`

- Now run `wget https://raw.githubusercontent.com/analogdevicesinc/plutosdr_scripts/master/ssh_config -O ~/.ssh/config`
- SSH into the Pluto:
```adi-mm:tests analogdevices$ ssh plutosdr
Warning: Permanently added 'pluto' (ECDSA) to the list of known hosts.
root@pluto's password: analog
# uname -a
Linux pluto 4.6.0-08511-gc1315e6-dirty #247 SMP PREEMPT Mon Oct 24 16:46:25 CEST 2016 armv7l GNU/Linux
# exit
Connection to 192.168.2.1 closed.
```

- If you have `sshpass` installed, you can use that so you dont need to type in a password:

```
analog@imhotep:~/pluto$ sshpass -panalog ssh plutosdr
Warning: Permanently added 'pluto' (ECDSA) to the list of known hosts.
Welcome to:
______ _       _        _________________
| ___ \ |     | |      /  ___|  _  \ ___ \
| |_/ / |_   _| |_ ___ \ `--.| | | | |_/ /
|  __/| | | | | __/ _ \ `--. \ | | |    /
| |   | | |_| | || (_) /\__/ / |/ /| |\ \
\_|   |_|\__,_|\__\___/\____/|___/ \_| \_|

http://wiki.analog.com/university/tools/pluto
# 
```