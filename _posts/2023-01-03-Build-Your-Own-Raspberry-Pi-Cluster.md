---
published: true
---

Have you ever wondered how to combine multiple Raspberry Pi boards into one single super computer. In this post I will guide you step by step to accomplish the above goal.

## Requirements
	1. One or more Raspberry Pi boards.
	2. Raspbian OS or any other Linux distribution of your choice).

## Let’s get started

First of all, you need to write a Raspbian OS image on **micro-SD** card, and an image writer application. To do so follow these steps:

- Plug in the micro-SD into your computer.
- Lunch Raspberry Pi Image.
- Click on `Choose OS` then navigate to `Raspberry Pi OS Other` select the desired operating system from the menu (in our case Raspberry Pi OS light).
- Select the available storage device, which is our micro-SD.
- You may want to tweak the configuration of the boot variables, via the settings button.
- Once you have done, click the **write** button.
- After completion, go to the micro-SD directory and search for file called `cmdline.txt`, then open it for editing.
- Jump to the end of the line and append the following:
`cgroup_memory=1 cgroup_enable=memory ip=<your ip address>::<gateway>:<subnet mask>:<hostname>:eth0:off`, then save and close.
- Then search for a file labeled `config.txt`, and open it.
- Scroll to the bottom of the file and append the following:
`arm_64bit=1`, then save and close.
- Create a new file called `ssh`, to enable the secure shell connection.
`$ touch ssh`
- Unplug the micro-SD from your computer, and plug it back to the Raspberry Pi adapter, then connect your Ethernet, the power the board.
- Wait couple seconds, until the boot process finishes, check for connectivity by typing this command:
```
ping -c5 x.x.x.x
```
- Woo-hoo! You are almost done, now it is time to connect to our lovely Pi via secure shell, type the following command:
```
$ ssh username@ip address
```
Then accept the RSA footprint for the first time, then type the default password `raspberry`
- Elevate your credential to root, `sudo -i`, then type the password.
