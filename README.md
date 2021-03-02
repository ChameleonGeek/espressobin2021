# espressobin2021
Configure an EspressoBin v7 as an Ubuntu 20.04 Server

The EspressoBin is a pretty sweet little SBC at a good price, but suffers from very limited support.  It's CPU and RAM are competitive with the Raspberry Pi, and has a couple advanteges over the Ras Pi.  It has a SATA3 port, mini PCIe, 1x USB2, 1x USB3, available GPIO ports competitve with the RasPi, and three Gigabit ethernet ports which *are not* using the same controller chip, which noticeably increases throughput from or to a USB drive.  I've had an EspressoBin operating as an Ubuntu 16.04 or 18.04 file server for more than three years, and it has never bogged down as I abuse it.

This project is a restart of a previous project.  The first goal is to easily run through the hoops to make a fully functional Ubuntu 20.04 file and web server.  If this is successful, the second goal is to elevate the EspressoBin as a Primary Domain Controller.  Goal 1 seems pretty easy to accomplish, but I've struggled with getting the kernel configured to support POSIX ACLs, which is needed for PDC functionality.

I find it easier to understand how a new system works by beating it up, rather than learning a whole lot of commands to make something that I don't understand "turn on."  This project is built considering the most common configuration options for some software, and asking the right questions in more human terms.  This gives the user the chance to quickly create a configuration that can be used for learning why the questions matter, and the script itself shows how to accomplish the desired result.  Since my main goal for this project is so that I have a functional system I can rely on for a few more years, it streamlines the configuration for more advanced users, too.

This project is a collection of Bash scripts that will ask the user the right questions so that once the scripts have finished, the EspressoBin is ready to rock.  There are several scripts:
1. A desktop/laptop is needed to build the kernel and first-boot image for the EspressoBin
2. A desktop/laptop or a Ras Pi can image the microSD card
3. A script will be placed on the EspressoBin OS image which will start as soon as the EspressoBin first boots, which enables networking and downloads the configuration script
4. A final configuration script that will ask all the right questions, giving the user a number of options to get the EspressoBin up and running as quickly as possible

Right now, this project is NOT functional.  Since the EspressoBin is a very lightweight system, I'm taking advantage of wget to make the retreival of scripts ultra-simple.  Once this repo is functional, wget will be similarly used to ensure the most up-to-date scripts are available, even if the original microSD image has been sitting on your desk for a while.

# osbuilder.sh
This script will build the kernel for the EspressoBin and will create the OS image that will be put on the microSD card.  This script needs to be run on a Linux Desktop or Laptop.  I've tried building the kernel on a Raspberry Pi 3B+, and it just wasn't strong enough.  I'm developing and testing the script on an Ubuntu 20.04 Desktop image.

# sd-image.sh
This script makes it easy and fast to put the OS image onto the microSD card without the need to know any Bash commands or install any software.  It can be run on the Desktop/Laptop, but can also be run on a RasPi.

# firstboot.sh
This script is added to the OS image created by osbuilder.sh, and then runs as soon as the EspressoBin first boots.  
1. It enables networking, which isn't functional on the EspressoBin until it is configured.
2. It performs apt update and apt upgrade to ensure the OS is as current as possible.
3. It installs wget so it can download the current version of ebin-config.sh
4. It downloads and then runs ebin-config.sh

# ebin-config.sh
This script is the "meat and potatos" of the project.  It navigates the user through a series of options so the user can choose from a wide variety of final configuration options.  Wherever possible, the questions are asked in the beginning of the process, so the process (which can take as long as an hour or so) can be started, and the user can walk away for a while without delaying the configuration process.
