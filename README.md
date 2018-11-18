# 1 Install Raspbian Stretch Lite

• Download lastest verison of Raspbian Stretch Lite here: <https://www.raspberrypi.org/downloads/raspbian/>  
• Follow the guide here <https://www.raspberrypi.org/documentation/installation/installing-images/README.md>  
• Insert SD card and power up your Raspberry Pi  

# 2 Setup Norns

• Follow the guide here: <https://github.com/monome/norns-image/blob/master/build-dev-image.md>  
Stop once you get to the kernel section.

## 2.1 Build your own kernel (Optional)

Follow these two gudies to configure and build a 4.14 kernel:  
<https://www.raspberrypi.org/documentation/linux/kernel/building.md>  
<https://www.raspberrypi.org/documentation/linux/kernel/configuring.md>  

### 2.1.1 Configuration

• Run `make menuconfig` from the linux folder

• Change the following options:  
  1. `Kernel Features—>Timer Frequenecy 1000hz` (Optional)  
  2. `Kernal Features —> Preemption Model —> Preemptive kernel (low-latency dektop)`  
  3. `CPU Power Management —> CPU Frequency scaling —> default CPUFreq governor (performance)`  

You may also want to alter /boot/config.txt e.g. if your using touchscreens

• Once installed, reboot the Raspberry Pi

### 2.1.2 Build

• Complete the the rest of the building guide

## 2.2 Configure Norns image

### 2.2.1 Clone norns-image repo

• Run `git clone https://github.com/monome/norns-image.git`

### 2.2.2 Remove I2C setup

• Run `nano norns-image/config/norns-init.service`

• Change `line:ExecStart=/usr/bin/amixer set Master 255 on` to `ExecStart=/usr/bin/amixer set PCM 255 on` (Depending on your audio interface, you can find out if you need this by using the command `amixer`)

• Comment out the following by placing a `#` at the start of the following lines:  
  1. `ExecStart=-/usr/sbin/i2cset -y 1 0x28 0x00`  
  2. `ExecStart=-/usr/sbin/i2cset -y 1 0x28 0x40`

Press `Ctrl+X` and then `Y` to save changes and exit.

• Run `nano norns-image/scripts/init-norns.sh`  
• Comment out the `i2cset` lines by placing a `#` at the start of each line• 

### 2.2.3 Set audio device

• Run `nano norns-image/config/norns-jack.service`  
• Change `hw:1` to your audio device  

Press `Ctrl+X` and then `Y` to save changes and exit.  

### 2.2.4 Disable network overrides (Optional)

Complete the following steps to stop Norns overwriting your network settings and configuring the 'norns' access point.  

• Run `nano norns-image/setup.sh`  
• Comment out the WiFi setcion by placing a `#` at the start of each line:  

Press `Ctrl+X` and then `Y` to save changes and exit.  

### 2.2.5 Configure Maiden

• Run `nano norns-image/config/norns-maiden.service`  
• Remove `.arm` from the end of `/home/we/maiden/maiden.arm`  

Press `Ctrl+X` and then `Y` to save changes and exit.  

## 2.3 Complete Norns images setup

• Run `./norns-image/setup.sh`

# 3 Install Dust
• Run `cd ~`  
• Run `git clone https://github.com/monome/dust`  

# 4 Install Crone, Maitron and Maiden

• Run `cd ~`  
• Run `git clone https://github.com/monome/norns`  

• Run`sudo nano /lib/systemd/system/systemd-udevd.service`  
• Change `MountFlags=slave` to `MountFlags=shared`  

• Run `cd ~/norns`  
• Run `./waf configure`  
• Run `./waf`  

• Run `pushd sc`  
• Run `./install.sh`  
• Run `popd`  

## 4.1 Install Crone
• Run `touch ~/.jackdrc`  
• Run `cat ~/norns-image/config/jackdrc >> ~/.jackdrc`  

## 4.2 Install Matron
• Run `sudo usermod -a -G video we` to ensure that the user can access the framebuffer

## 4.3 Install Maiden
• Download latest .tgz release from <https://github.com/monome/maiden/releases>  
• Extract it to `~/maiden`  

# 5 Finish
Reboot and enjoy

# Troubleshooting


