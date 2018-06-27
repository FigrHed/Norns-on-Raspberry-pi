nornsinstall.txt


# 1. Download raspbian stretch lite 

# 2. write to sdcard 
I use, etcher.io


# 3 Boot PI

# 4 follow norns setup
https://github.com/monome/norns-image/blob/master/build-dev-image.md

## 4.1 do raspi-config section


## 4.2 do user  including sudo
(I installed all locations, but it took forever, id be tempted to skip this, and just select your own region :)  )


at this stage, I usually change to using ssh to log in remotely we@norns/sleep
easier to cut n paste things ;) 

## 4.5 updates
do as instructed :) 
till you get to the kernel , then stop : )


## 4.4 kernel (OPTIONAL!) (UPDATED)

you cannot use the 'norns' kernel, and actually there is no real need too.
so you have two choices 

### 4.4 A - keep the kernel as is, i.e. do nothing
for most this is the 'preferred' option, I've run with the stock kernel as it was fine.

### 4.4 B build your own kernel
so norns usually runs with a preemptive kernel, so you can build a similar one if you wish.
(which is what I did, since I already had a means to do this quickly/easily - and not on the PI which takes a very long time!)

we use a 4.14 kernel and just change a couple of settings 
follow instructions on raspbian website, building a kernel, you can either cross compile, or build locally, take ~90min?

https://www.raspberrypi.org/documentation/linux/kernel/building.md
https://www.raspberrypi.org/documentation/linux/kernel/configuring.md

menuconfig :
to get into the menuconfig run make menuconfig from the linux folder
Kernel Features—>Timer Frequenecy 1000hz (optional, as suggested by norns dev, but doesnt make much difference to me)
Full Preemption:
Kernal Features —> Preemption Model —> Preemptive kernel (low-latency dektop)
(I think thats what technobear meant by full-preemption - FigrHed)
CPU Power Management —> CPU Frequency scaling —>
default CPUFreq governor (performance)

you do NOT need dt-blob.bin (shouldn't hurt though)

you may want to alter /boot/config.txt e.g. if your using touchscreens


once installed, rebooted, 
back to the norn-image setup readme 

#4.5 setup image (DON’T RUN setup.sh YET!)

(do clone from git clone https://github.com/monome/norns-image.git )

do not run setup.sh YET

norns-image/config/norns-init.service - comment out (put # in front, we dont want any i2cset)
#ExecStart=-/usr/sbin/i2cset -y 1 0x28 0x00
#ExecStart=-/usr/sbin/i2cset -y 1 0x28 0x40

norns-image/config/norns-jack.service - change to your soundcard.
(-d alsa -d hw:1,0 )
 if you’re using a usb soundcard, you'll probably need to change buffer size to (-p 256 or -p 512) 

norns-image/config/norns-maiden.service - change exec to maiden (not maiden.arm) 
ExecStart=/home/we/maiden/maiden -fd 3 -app ./app/build -data /home/we/dust -doc /home/we/norns/doc


norns-image/scripts/init-norns.sh 
comment out i2cset 




! now run setup.sh



#6  install norns

follow instructions for apt-get, accept all defaults


cd ~
git clone https://github.com/monome/norns (or alternative)

optional push2 : https://github.com/thetechnobear/norns
sudo apt-get install libusb-1.0-0-dev 
wget https://raw.githubusercontent.com/TheTechnobear/MEC/master/resources/79-push2.rules
sudo mv 79-push2.rules /etc/udev/rules.d/



sudo vi /lib/systemd/system/systemd-udevd.service  , change MountFlags=slave to MountFlags=shared, as instructed


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
rm -rf linux-armv7 *.tar.gz



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

