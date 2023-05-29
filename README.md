# plutoSDR tutorial
## Installing the HoRNDIS driver on Apple Silicon
> HoRNDIS is a driver for Mac OS X that allows you to use your RNDIS to get network access to Pluto. It is required for Remote Network Driver Interface Specification (RNDIS) which is a USB protocol to provides a virtual Ethernet link.
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

- Follow steps 2〜4 from above.
- Go to Utilities → Terminal.
- Type in the following to fully disable SIP: `csrutil disable`

<br> **Note:** It is possible to only partially disable the part of SIP that enforces kext signature verification `csrutil enable --without kext`, but according to Apple, this is apparently an "unsupported configuration". Use it if you wish (as many do already), but please make sure to read and fully understand the warning that csrutil gives if you try.
Reboot your Apple Silicon Mac.
### 3. Compiling HoRNDIS for Apple Silicon (arm64e)

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


## Serial Configuration
- Connect the Pluto, it should show up under System Preferences > Network
- Run `ls -l /dev/tty.* ` in Terminal, your connected devices should show up:
    ``` 
    crw-rw-rw-  1 root  wheel  0x9000004 28 May 17:00 /dev/tty.Bluetooth-Incoming-Port
    crw-rw-rw-  1 root  wheel  0x9000002 28 May 17:00 /dev/tty.samarsBeatsStudio
    crw-rw-rw-  1 root  wheel  0x9000006 29 May 09:56 /dev/tty.usbmodem1104
    crw-rw-rw-  1 root  wheel  0x9000000 28 May 16:59 /dev/tty.wlan-debug
    ```
- Find the USB port where the Pluto is connected to, and run `screen /dev/tty.usbmodem1104`
- Now we can login:
    ```
    Welcome to Pluto
    pluto login: root
    Password: 
    Welcome to:
    ______ _       _        _________________
    | ___ \ |     | |      /  ___|  _  \ ___ \
    | |_/ / |_   _| |_ ___ \ `--.| | | | |_/ /
    |  __/| | | | | __/ _ \ `--. \ | | |    /
    | |   | | |_| | || (_) /\__/ / |/ /| |\ \
    \_|   |_|\__,_|\__\___/\____/|___/ \_| \_|

    v0.30
    http://wiki.analog.com/university/tools/pluto
    # uname -a
    Linux pluto 4.14.0-41915-gc2041af #279 SMP PREEMPT Mon Jan 14 13:13:47 CET 2019 armv7l GNU/Linux
    ```

 ## SSH Configuration
 > We can also SSH into the Pluto instead of through the console port
- First, we need to install the Homebrew package manager
    - `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` and follow the installation commands
    - Then install wget using `brew install wget`

- Now run `wget https://raw.githubusercontent.com/analogdevicesinc/plutosdr_scripts/master/ssh_config -O ~/.ssh/config`

```
samarqureshi@MacBook-Air-2 ~ % ssh plutosdr
Warning: Permanently added 'plutosdr' (ECDSA) to the list of known hosts.
root@plutosdr's password: analog
Welcome to:
______ _       _        _________________
| ___ \ |     | |      /  ___|  _  \ ___ \
| |_/ / |_   _| |_ ___ \ `--.| | | | |_/ /
|  __/| | | | | __/ _ \ `--. \ | | |    /
| |   | | |_| | || (_) /\__/ / |/ /| |\ \
\_|   |_|\__,_|\__\___/\____/|___/ \_| \_|

v0.30
http://wiki.analog.com/university/tools/pluto
# uname -a
Linux pluto 4.14.0-41915-gc2041af #279 SMP PREEMPT Mon Jan 14 13:13:47 CET 2019 armv7l GNU/Linux
# exit
Connection to 192.168.3.3 closed.
samarqureshi@MacBook-Air-2 ~ % 
```


## IIO-Scope Installation
>The ADI IIO Oscilloscope supports plotting of the captured data in four different modes (time domain, frequency domain, constellation and cross-correlation). The application also allows to view and modify several settings of the evaluation board's devices.
- Make sure Homebrew is installed
    - `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` and follow the installation commands
```
brew update
brew upgrade
brew tap tfcollins/homebrew-formulae
brew install --verbose --build-from-source libiio
brew install --verbose --build-from-source libad9361-iio
brew install --verbose --build-from-source iio-oscilloscope
```


## GNU Radio Installation for Apple Silicon
>GNU Radio is a free & open-source software development toolkit that provides signal processing blocks to implement software radios or other generic processing.

### XQuartz