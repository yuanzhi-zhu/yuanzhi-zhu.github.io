---
layout: post
title: Play Jupyter on Your Android
categories: Android
description: 
keywords: OOMMF, Android, Termux
---
## Introduction to OOMMF

[Object Oriented MicroMagnetic Framework (OOMMF)](https://math.nist.gov/oommf/) is a project in the Applied and Computational Mathematics Division (ACMD) of ITL/NIST, in close cooperation with µMAG, aimed at developing portable, extensible public domain programs and tools for micromagnetics. 

## Get an Ubuntu in Your Android

1. The first thing you need to do is making sure to get Termux installed in your phone as I have introduced in [my last Blog](https://yuanzhi-zhu.github.io/2019/12/09/Play-Jupyter-on-Your-Android/). 

2. Then you need to install [AnLinux](https://github.com/EXALAB/AnLinux-App) from PlayStore and follow the structions in this app to download an Ubantu OS under Termux. Then follow the instructions in AnLinux to install a Desktop Environment, which will install a VNC server at the same time (set your password and remenber your VNC port).

3. To use this Destop Environment, you need to download a [VNC client](https://www.realvnc.com/en/connect/download/viewer/) on your phone. If your want to view the previous Ubuntu localhost on the same smart phone you download the VNC, the IP address to visit the Ubuntu is '127.0.0.1'. For the following steps, you can run the VNC on your PC to control your phone more conveniently.

## Installation of OOMMF

1. Since the Ubuntu on your phone is the simplest one, you need to have TCL/TK on the Ubuntu, which are necessary for the installation of OOMMF, by typing (by default you are the root user, which will make the installation process easier):

```
apt-get update && apt-get upgrade
apt-get install tcl tcl-dev
apt-get install tk tk-dev
```

2. After getting TCL and TK, we need to get the source code of OOMMF. Here I choose the version [OOMMF 1.2 beta 3 (27-Sep-2019)](https://math.nist.gov/oommf/software-12.html). You can get it from the graphic interface(VNC) using a web browser of from the terminal.

2.1 Since there seem to be no web browser in the Ubantu, you can using the following code to surfing:

```
apt-get install w3m
apt-get install w3m-img
w3m google.com
```

2.2 Another way to get the source code is derectly using terminal by typing the following command on the terminal in Ubuntu:

```
wget https://math.nist.gov/oommf/dist/oommf12b3_20190927.tar.gz
tar -zxvf oommf12b3_20190927.tar.gz
```

By Enter 'ls' we can see the unzipped oommf folder.

3. Since we are on an Android phone, the oommf can not recognize our platform, we need to look up [here](https://math.nist.gov/oommf/doc/userguide12b3/userguide/Advanced_Installation.html#sec:platformNames) for solution.

3.1 Add a new file [linux.tcl](./documents/blog/linux001.pdf) in the folder 'oommf/config/names'.

3.2 Add a new file [linux.tcl](./documents/blog/linux002.pdf) in the folder 'oommf/config/platforms' (you should rename they with the same name 'linux.tcl' even they have different content)

3.3 Now you can test it in the /oommf folder at terminal by enter:

```
tclsh oommf.tcl +platform
```

You will see the Platform Name is linux.

3.4 Then you should re-make the OOMMF in the /oommf folder at terminal by enter:

```
tclsh oommf.tcl pimake distclean
tclsh oommf.tcl pimake
```

This will take about 30 minutes.

## Run OOMMF

To get OOMMF running, you need to do same thing you would on any other computer by entering the following in the /oommf folder:

```
tclsh oommf.tcl
```
