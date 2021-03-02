# espressobin2021
Configure an EspressoBin v7 as an Ubuntu 20.04 Server

Right now, this project is NOT functional.

The EspressoBin is a pretty sweet little SBC at a good price, but suffers from very limited support.  It's CPU and RAM are competitive with the Raspberry Pi, and has a couple advanteges over the Ras Pi.  It has a SATA3 port, mini PCIe, 1x USB2, 1x USB3, available GPIO ports competitve with the RasPi, and three Gigabit ethernet ports which *are not* using the same controller chip, which noticeably increases throughput from or to a USB drive.  I've had an EspressoBin operating as an Ubuntu 16.04 or 18.04 file server for more than three years, and it has never bogged down as I abuse it.

This project is a restart of a previous project, attempted with Ubuntu 16.04 and then 18.04.  The first goal is to easily jump through the hoops to make a fully functional Ubuntu 20.04 file and LAMP server.  If this is successful, the second goal is to elevate the EspressoBin as a Primary Domain Controller.  Goal 1 seems pretty easy to accomplish, but I've struggled with getting the kernel configured to support POSIX ACLs, which is needed for PDC functionality.

I find it easier to understand how a new system works by beating it up, rather than learning a whole lot of commands to make something that I don't understand "turn on."  This project is built considering the most common configuration options for some software, and asking the right questions in more human terms.  This gives the user the chance to quickly create a configuration that can be used for learning why the questions matter, and the script itself shows how to accomplish the desired result.  Since my main goal for this project is so that I have a functional system I can rely on for a few more years, it streamlines the configuration for more advanced users, too.

This project is a collection of Bash scripts that will ask the user the right questions so that once the scripts have finished, the EspressoBin is ready to rock.  There are several scripts:

### osbuilder.sh
This script will build the kernel for the EspressoBin and will create the OS image that will be put on the microSD card.  This script needs to be run on a Linux Desktop or Laptop.  I've tried building the kernel on a Raspberry Pi 3B+, and it just wasn't strong enough.  I'm developing and testing the script on an Ubuntu 20.04 Desktop image.

### sd-image.sh
This script makes it easy and fast to put the OS image onto the microSD card without the need to know any Bash commands or install any software.  It can be run on the Desktop/Laptop, but can also be run on a RasPi.

### firstboot.sh
This script is added to the OS image created by osbuilder.sh, and then runs as soon as the EspressoBin first boots.  
1. It enables networking, which isn't functional on the EspressoBin until it is configured.
2. It performs apt update and apt upgrade to ensure the OS is as current as possible.
3. It installs wget so it can download the current version of ebin-config.sh
4. It downloads and then runs ebin-config.sh

### ebin-config.sh
This script is the "meat and potatos" of the project.  It navigates the user through a series of options so the user can choose from a wide variety of final configuration options.  Wherever possible, the questions are asked in the beginning of the process, so the process (which can take as long as an hour or so) can be started, and the user can walk away for a while without delaying the configuration process.

# Process Steps:

## osbuilder.sh

This process can take several hours, depending upon the configuration you choose for the EspressoBin.
1. Read osbuilder.md for explanations of the options presented by the script and be ready to answer the questions asked by the script.
2. On your Linux Desktop/Laptop, open a terminal session.
3. Paste the following into the terminal:

```
wget https://github.com/ChameleonGeek/espressobin2021/raw/master/osbuilder.sh
chmod +x osbuilder.sh
sudo bash osbuilder.sh
```
4. Answer the questions asked by the script according to the decisions you made while reading osbuilder.md. 
5. Once you have answered all the questions, the system will begin building the kernel (if selected), download the Ubuntu 20.04 ISO image and build the OS image for the microSD card.  The script will tell you when it's asked all of its questions and you can walk away until it is finished.
6. Once the script has told you that you can step away, you could use a couple of those minutes to configure the EspressoBin to boot from the microSD card (see below, minus the first step).  If you do so, the EspressoBin will need to be unplugged before you install the microSD card.

## sd-image.sh
This process can be run on nearly any Linux system (including a Raspberry Pi).  While developing this project, I had a Raspberry Pi standing by to reimage the microSD card, which I had to do quite a few times as I was creating ebin-config.sh.  The actual imaging process only takes a couple of minutes.
1. Ensure the OS image created by osbuilder.sh (???) is in the current user's home directory
2. Ensure the microSD card isn't connected to the Linux system
3. Open a terminal session
4. Paste the following into the terminal:
```
cd ~
wget https://github.com/ChameleonGeek/espressobin2021/raw/master/sd-image.sh
chmod +x sd-image.sh
sudo bash sd-image.sh
```
5. Follow the script's instructions to complete the image process
6. The microSD will be automatically unmounted when the script is complete, so it can be safely disconnected when the script has finished
7. Once you have successfully imaged one microSD card, you can image again by simply typing 'sudo bash sd-image.sh' into a terminal session.

## Configure the EspressoBin to boot from the microSD card
### This process only has to be performed once.
1. Connect an ethernet cable from your network to the EspressoBin's WAN port (the separate ethernet connector).
2. Connect a micro USB cable to the EspressoBin and your computer.
    1. The green power LED will turn on, and your computer will detect that a device has been connected.  The EspressoBin *is not* powered via USB and *can not* be configured without 12v power.
3. Connect 12v 2a power to the EspressoBin power connector.
4. Identify the serial port that the EspressoBin is connected to.
5. Open a serial terminal program and connect to the EspressoBin (previously identified port), 115200, 8, n, 1.
    1. I have had best success in Windows with PuTTY.
6. If the EspressoBin's "Marvell>>" prompt is not displayed in the terminal, press the EspressoBin reset button.
    1. If you have tried to configure the EspressoBin to boot from other media, reset the EspressoBin and press a key when prompted to stop autoboot.
7. Paste the following into the terminal program **_one command at a time_**:
```
setenv image_name boot/Image
setenv fdt_name boot/armada-3720-community.dtb
setenv bootmmc 'mmc dev 0; ext4load mmc 0:1 $kernel_addr $image_name;ext4load mmc 0:1 $fdt_addr $fdt_name;setenv bootargs $console root=/dev/mmcblk0p1 rw rootwait; booti $kernel_addr - $fdt_addr'
setenv bootcmd 'mmc dev 0; ext4load mmc 0:1 $kernel_addr $image_name;ext4load mmc 0:1 $fdt_addr $fdt_name;setenv bootargs $console root=/dev/mmcblk0p1 rw rootwait; booti $kernel_addr - $fdt_addr'
saveenv
```
8. Type "reset" and hit enter.  The EspressoBin will now reboot.  The EspressoBin will, of course, fail to boot into Ubuntu if the microSD is not installed
9. Before you insert the microSD card, the 12v power and micro USB cable must be disconnected so the EspressoBin is fully powered down.

## 4. firstboot.sh
This script will automatically run when you first boot into Ubuntu.  It performs some initial setup which is necessary to begin the larger configuration script.
It won't ask you any questions, but will display what it is doing as it is doing it.
As soon as it is finished, it will run ebin-config.sh.
1. Install the imaged microSD card into the EspressoBin MicroUSB slot.
2. Follow steps 1 through 5 of `Configure the EspressoBin to boot from the microSD card` above
3. Once the EspressoBin has completed booting into Ubuntu, log in.
    1. The username is `root`
    2. There is no password
    3. Don't worry - ebin-config.sh will ask you for a new root password and assign it to root.  It will also ask for a username and password for an interactive user.

The script will prevent itself from automatically running on future boots.

## 5. ebin-config.sh
This script will be downloaded by firstboot.sh and will automatically run after firstboot.sh finishes.  This script is set up to ask as many questions as possible as early as possible.  I hate being tethered to a computer while it chugs away asking the occasional question as much as you do...
Unless you are very well-versed in Linux commands over SSH, I strongly recommend installing Webmin.  Webmin is a web-based GUI that can perform most of the final configuration steps in a much friendlier interface.  I have found that its small overhead is well worth it!
1. Please read ebin-config.md for explanations of the options presented by the script and be ready to answer the questions asked by the script.
2. Once you have answered all the questions, the EspressoBin will download, install and configure the software you have selected per your answers.  This process can take an hour or so based on the configuration you select.
3. The script will tell you when it has asked all its questions and you can walk away for a while.

## 6. Final configuration
These scripts install the selected software and perform basic configuration so that all software is functional.  Each user needs to ensure that they perform additional configuration to better suit their usage model and ensure proper security.  If you installed it, you can use webmin to perform a lot of these steps by connecting to the EspressoBin at https://&lt;ebin-ip-adress&gt;:10000, and loggin in as the interactive user.
