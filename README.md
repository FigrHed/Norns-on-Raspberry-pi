# 1.0 Download Raspbian Stretch Lite

Available here: <https://www.raspberrypi.org/downloads/raspbian/>

# 2.0 Write Raspbian image to SD card

Follow the guide here: <https://www.raspberrypi.org/documentation/installation/installing-images/README.md>

# 3.0 Insert SD card and power up your Raspberry Pi

# 4.0 Norns setup

Follow the guide here: <https://github.com/monome/norns-image/blob/master/build-dev-image.md>
Stop once you get to the kernel section.

## 4.1 Build your own kernel (Optional)

Follow these two gudies to configure and build a 4.14 kernel:
<https://www.raspberrypi.org/documentation/linux/kernel/building.md>
<https://www.raspberrypi.org/documentation/linux/kernel/configuring.md>

### 4.1.1 Configuration

to get into the menuconfig run make menuconfig from the linux folder

`Kernel Features—>Timer Frequenecy 1000hz` (Optional)

For full preemption:
`Kernal Features —> Preemption Model —> Preemptive kernel (low-latency dektop)`
`CPU Power Management —> CPU Frequency scaling —> default CPUFreq governor (performance)`

You may also want to alter /boot/config.txt e.g. if your using touchscreens

### 4.1.0 Build

Once built and installed, rebooted,
back to the norn-image setup readme

## 4.2 Setup Norns image

### 4.2.1 Clone norns-image repo

Run `git clone https://github.com/monome/norns-image.git`

### 4.2.2 Remove i2c setup

Run `nano norns-image/config/norns-init.service`
Comment out the following by placeing a `#` at the start of each line:
`ExecStart=-/usr/sbin/i2cset -y 1 0x28 0x00`
`ExecStart=-/usr/sbin/i2cset -y 1 0x28 0x40`

Press `Ctrl+X` and then `Y` to save changes and exit.

Run `nano norns-image/scripts/init-norns.sh`
comment out i2cset

### 4.2.3 Set audio device

Run `nano norns-image/config/norns-jack.service`
Change `hw:1` to your audio device

Press `Ctrl+X` and then `Y` to save changes and exit.

### 4.2.4 Configure Maiden

Run `nano norns-image/config/norns-maiden.service`
Remove `.arm` from the end of `/home/we/maiden/maiden.arm`

Press `Ctrl+X` and then `Y` to save changes and exit.

## 4.3 Complete norns-image setup

Run `./norns-image/setup.sh`

#5 Install Crone, Maitron and Maiden

follow instructions for apt-get, accept all defaults

cd ~
git clone https://github.com/monome/norns (or alternative)

optional push2 : https://github.com/thetechnobear/norns
sudo apt-get install libusb-1.0-0-dev
wget https://raw.githubusercontent.com/TheTechnobear/MEC/master/resources/79-push2.rules
sudo mv 79-push2.rules /etc/udev/rules.d/

sudo vi /lib/systemd/system/systemd-udevd.service , change MountFlags=slave to MountFlags=shared, as instructed

cd norns
./waf configure
./waf

! run sclang :
sclang

./install.sh

more details at: https://github.com/monome/norns/blob/master/readme-setup.md

#7 install dust
cd ~
git clone https://github.com/monome/dust

#8.1 install crone
Crone handles the sound
if this fails change the norns-image/config/jackdrc to match the line we we edited in norns-jack.service then copy it to your home dir
touch ~/.jackdrc
cat pathto/norns-image/config/jackdrc >> ~/.jackdrc
and you may need to change the line: ExecStart=/usr/bin/amixer set Master 255 on
to:
ExecStart=/usr/bin/amixer set PCM 255 on
(at least i did on mine - FigrHed
you can find out which command you need by just typing ‘amixer’ and it will show up at the topic the format:
Simple mixer control ‘IT_HERE’, 0)

#8.2 install Matron
This should automatically run if you have an HDMI screen plugged in. If it doesn’t, it may be because you do not have access privileges to the framebuffer device.
You can add your self to the video group with: sudo usermod -a -G video we

#8.3 install maiden
(see https://github.com/monome/maiden/blob/dev/README.md)

NOTE: unless you are planning to actively change/develop Maiden, it may be better to just download the latest build from the repo.
https://github.com/monome/maiden/releases
make sure it ends up in: ~/maiden

wget https://storage.googleapis.com/golang/go1.9.linux-armv6l.tar.gz
sudo tar -C /usr/local -xzf go1.9.linux-armv6l.tar.gz

wget https://github.com/Masterminds/glide/releases/download/v0.13.1/glide-v0.13.1-linux-armv7.tar.gz
tar xvzf glide-v0.13.1-linux-armv7.tar.gz linux-armv7/glide
sudo mv linux-armv7/glide /usr/local/go/bin/
rm -rf linux-armv7 \*.tar.gz

export PATH=$PATH:/usr/local/go/bin (add to .profile too)
export GOPATH=$HOME/go (add to .profile too)

go get -d github.com/monome/maiden
cd ~/go/src/github.com/monome/maiden
glide install
go build

cd ~
ln -s ~/go/src/github.com/monome/maiden

UI, I built on a desktop, much faster
then copy into ~/maiden/app/build
