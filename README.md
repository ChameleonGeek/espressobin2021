# espressobin2021
Configure an EspressoBin 7v as an Ubuntu 20.04 Server

This is a restart of previous attempts to configure an EspressoBin v.7 SBC as an Ubuntu Domain Controller.  I've successfully configured the EspressoBin as an Ubuntu Server with 16.04 and 18.04, but have been unable to enable POSIX ACLs, which are needed for a Domain Controller.  I'm hoping the kernel and OS updates since 18.04 will fix some of the issues I encountered in previous attempts.  My preferred final project should elevate the EspressoBin to a Primary Domain Controller, but it has been difficult to get the kernel just right.  Even if the EspressoBin can't function as a PDC, it is still *very* useful as a file server and can be a domain member.  I've been successfully running an EspressoBin v5 as an Ubuntu fileserver for more than three years.

Right now, this project is NOT functional.  Since the EspressoBin is a very lightweight system, I'm taking advantage of wget to make the retreival of scripts ultra-simple.  Once this repo is functional, wget will be similarly used to ensure the most up-to-date scripts are available, even if the original microSD image has been sitting on your desk for a while.
