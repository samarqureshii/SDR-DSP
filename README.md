# GNU Radio on Apple Silicon, macOS
- Install homebrew `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

- Make sure you update Xcode and CLI tools if you’re getting x86 compilation errors

- `xcode-select --install` and follow the instructions on the Update window 

- `softwareupdate --list` then softwareupdate --install "Command Line Tools" if Command Line Tools shows up under updates

- Then run `brew install gnuradio`
- 
## GNU Radio With Asahi (Arch) Linux)
- `sudo pacman -Sy`
- `sudo pacman -S gnuradio`
    - If this gives errors with numpy packages, try:
        `sudo rm /var/lib/pacman/db.lck` to unlock the pacman database
        `sudo pacman -S --overwrite '*' gnuradio`
- Confirm installation `gnuradio-config-info --version`
- Install package `sudo pacman -S gr-osmosdr`
    - This will most likely give an error since its expecting x86 architecture while we're on arm64, and the package isn't directly available in the Arch User Repository (AUR):
    - Run the following:
    ```
    sudo pacman -S base-devel git
    git clone https://aur.archlinux.org/gr-osmosdr-git.git
    cd gr-osmosdr-git
    ```    
    - Now lets add support for aarch64: `nano PKGBUILD`, and modify the `arch=` line to: `arch=('i686' 'x86_64' 'aarch64')`
    - Install additional missing dependencies that are not already in AUR:
        - 
    - Try building again `makepkg -si`

# bladeRF tutorial on Apple Silicon processors (M1)
`brew install libbladerf`
- To get it working with GNU Radio, build from source:
```
git clone https://github.com/Nuand/gr-bladeRF.git
cd gr-bladeRF
mkdir build
cd build
brew install pybind11
cmake ..
make -j4
sudo make install
```


---
## bladeRF software with ith Asahi Linux
- Run `curl https://alx.sh | sh` in your Terminal to begin installation of [Asahi Linux's Alpha Release](https://asahilinux.org)
    - Follow the prompts and boot into the new OS
- Make sure you have the most recent version `sudo pacman -Syu`
- Install dev tools/libraries `sudo pacman -S base-devel cmake libusb`
- Clone the repo `git clone https://github.com/Nuand/bladeRF.git`
- Enter bladeRF directory `cd bladeRF`
- Make a build directory `mkdir build`, enter in `cd build`
- Build software `make`
- Install software `sudo make install`
- Run `bladeRF-cli -p`

## Troubleshooting
If you get the error `bladeRF-cli: error while loading shared libraries: libbladeRF.so.2: cannot open shared object file: No such file or directory` after running `bladeRF-cli -p`, try the following:
- `sudo nano /etc/ld.so.conf`
- Add this line to the end of the file: `/usr/local/lib`, save and exit
- `sudo ldconfig`

If you get `Found a bladeRF via VID/PID, but could not open it due to insufficient permissions, or because the device is already open`, try setting up udev rules:
- `sudo nano /etc/udev/rules.d/88-nuand.rules`
- Add these lines to the file:
    ```
    SUBSYSTEM=="usb", ATTR{idVendor}=="2cf0", ATTR{idProduct}=="5246", MODE="0660", GROUP="plugdev"
    SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="6066", MODE="0660", GROUP="plugdev"
    ```
    - Replace `plugdev` with the actual user (i.e., `samarqureshi`)
- Save and exit, the reload the rules with:
    ```
    sudo udevadm control --reload-rules
    sudo udevadm trigger
    ```
If that still is not working, reset the permissions:
- Open each of the following files:
    ```
    sudo nano /etc/udev/rules.d/88-nuand-bladerf1.rules
    sudo nano /etc/udev/rules.d/88-nuand-bladerf2.rules
    sudo nano /etc/udev/rules.d/88-nuand-bootloader.rules
    ```
- Change the `MODE` in each of the files to `0666`
- Save and exit, then run
    ```
    sudo udevadm control --reload-rules
    sudo udevadm trigger
    ```
- Disconnect and connect the bladeRF device


## Basic Commands
- `bladeRF-cli -i` will start the CLI
- `bladeRF-cli -l` followed by a path to a file will load the FPGA image
    - You can view all FPGA images [here](https://www.nuand.com/fpga_images/)
- `bladeRF-cli -f` followed by a path to a file will update the firmware    

# plutoSDR tutorial for Apple Silicon 

## Installing the HoRNDIS driver 
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
    Password: analog
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

# GNU Radio Installation for Apple Silicon (without Homebrew)
>GNU Radio is a free & open-source software development toolkit that provides signal processing blocks to implement software radios or other generic processing.

## XQuartz

XQuartz is needed to run X11 applications in the Docker container. The steps to set it up are:

1. Install XQuartz
2. In the XQuartz preferences, go to the "Security" tab and enable "Allow connections from network clients"
3. Reboot the machine

We found that it was important to reboot the machine for the "Allow connections from network clients" setting to be applied.

On each boot, it is necessary to launch XQuartz and then run
```
xhost + 127.0.0.1
```

in a terminal. This will allow connections from the Docker container to XQuartz. **Note:** something more involved
is recommended in the README for gnuradio-docker-env, which involves using `tcpdump` to check which IP the X11 packets
come from. We found that these always seem to come from `localhost`, so using `127.0.0.1` seems a good bet, but your
luck may vary. To debug, it's possible to use `xhost +`, which will allow connections from any IP (this is potentially
insecure).

## Docker Desktop

1. Install Docker Desktop following the [instructions](https://docs.docker.com/docker-for-mac/install/)
2. It is recommended to increase the number of CPUs and RAM available to Docker. This can be done in the preferences of
   the Docker Desktop GUI. We decided to use 8 CPUs and 6 GB of RAM for the M1 laptops (the machine has a total of 8
   cores and 8 GB of RAM).
   
On each boot, the Docker Desktop application needs to be run. Then it is possible to interact with Docker by using
the different `docker` command in a terminal.

## Testing XQuartz + Docker

Use the following to test that Docker is able to run GUI applications that display on the MacOS screen:
```
docker run -e DISPLAY=host.docker.internal:0 gns3/xeyes
```

This should show up a window with a pair of eyes that follow the mouse. If there are problems, besides the
gnuradio-docker-env README, another good resource can be
[this note](https://gist.github.com/cschiewek/246a244ba23da8b9f0e7b11a68bf3285).


## Ubuntu Docker image for GNU Radio

The [gnuradio-docker-env](https://github.com/igorauad/gnuradio-docker-env) comes with a `Dockerfile` of an Ubuntu
image with the build dependencies of GNU Radio installed. There is also a `docker-compose.yml` file, as this image is
intended to be used with docker-compose. We decided not to use docker-compose, to keep things simpler should problems
arise (mainly my fault, since I'm not experienced with docker-compose).

The steps to build the image and launch the container manually are as follow.

1. Clone the gnuradio-docker-env repo:
```
git clone https://github.com/igorauad/gnuradio-docker-env
```

2. Build the image. Inside the `gnuradio-docker-env` directory run:
```
docker build -t gnuradio .
```

(here `gnuradio` will only be used to give a name to the image).

3. Run the image to create a container:
```
docker run --name gnuradio -it gnuradio
```

(here `gnuradio` is just to give a name to the container, and `-it` are the parameters used to get a terminal session
in the container).

**Note:** when installing this with the interns we didn't give the image or container a name (my fault!). The existing
images can be renamed by doing `docker tag IMAGE_ID gnuradio` (the `IMAGE_ID` can be seen by running `docker images`).
The existing container can be renamed with `docker rename current_name gnuradio` (the current name can be seen by
running `docker container ls -a`).

## Running the container again

When we exit the container (because we close the terminal, reboot the machine or any other reason), we can run again
the existing container (plus all modifications we have made since its creation) with
```
docker start -i gnuradio
```

## Building and installing GNU Radio

1. First we install some packages that we will need:
```
sudo apt install nano swig libczmq
```

2. We decided to build GNU Radio 3.8 from source. The basic instructions are
[here](https://wiki.gnuradio.org/index.php/InstallingGR#For_GNU_Radio_3.8_or_Earlier). Installing a more modern
version of GNU Radio 3.9 is similar (although SWIG is not required for GNU Radio 3.9 and later).

3. Clone the GNU Radio repo
```
git clone https://github.com/gnuradio/gnuradio
```

4. Checkout the 3.8 tag (at this time v3.8.3.1 is the latest release, so that's what we installed)
```
cd gnuradio
git checkout v3.8.3.1
git submodule update --init --recursive
```

4. Run cmake
```
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DPYTHON_EXECUTABLE=/usr/bin/python3 ..
```

5. At this point watch out for the output of `cmake`. In particular, any GNU Radio components that might have
been disabled because of unmet dependencies. In fact we had to install `swig` to have Python support (which
includes `gnuradio-companion`).

6. Build with
```
make -j4
```
It takes perhaps about 30 minutes to build on the 2020 MacBook with `-j4`. Watch out, since
it's possible to get out of RAM while building, which will most likely cause the C compiler to be killed (and the
build will stop). With `-j8` you're very likely to get out of RAM with only 8 GB, so something around `-j4` seems
safer.

7. Install with
```
make install
ldconfig
```
(since we're running as root inside the Docker container, it is not necessary to use `sudo`).

## Final configuration

We needed to add a few environment variables to `/root/.bashrc`. For this, we do
```
nano /root/.bashrc
```
and add the following lines at the end of the file:
```
export DISPLAY=host.docker.internal:0
export GRC_BLOCKS_PATH=/usr/local/share/gnuradio/grc/blocks
export PYTHONPATH=/usr/local/lib/python3/dist-packages/
```
After modifying the `.bashrc` file, the easiest way to apply the settings is to log out of the
container with Ctrl+D and log in again with `docker start` as described above.

## Running GNU Radio Companion

If everything went well, at this point we can run
```
gnuradio-companion
```
inside the docker container and have a functional GNU Radio companion window on our Mac OS screen.

If something doesn't work well and the X11 + XQuartz is suspect, it's possible to install and run `xterm`, which prints
more debug information than `gnuradio-companion`.
