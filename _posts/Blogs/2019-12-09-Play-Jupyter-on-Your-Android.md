---
layout: post
title: Play Jupyter on Your Android
categories: Android
description:  Install Jupyter on Android device
keywords: Jupyter, Python, Android, Termux
---
## Introduction to Termux

[Termux](https://termux.com/) is an Android terminal emulator and Linux environment app that works directly with no rooting or setup required. A minimal base system is installed automatically - additional packages are available using the APT package manager.

You can install Python on it, which means that basicly you can do everything with it! In this post I will record how to play Jupyter on your Android device.

## Get Prepared

1. Install Termux from Google Play, open it.

2. Install APT package and ungrade:

```
pkg install apt
apt update && apt upgrade
```

3. Install Python3, git, clang, FFTW

```
apt install python
apt install git
apt install clang
apt install fftw
```

## Installing the Scipy Stack

1. First, install numpy using pip:

```
LDFLAGS=" -lm -lcompiler_rt" pip install numpy
```

If this is unsuccessful(or some packages in the future), you can try to install them using apt-get install: 

```python
apt-get install python-numpy
```



1. For matplotlib, we'll need to install a few more dependencies:

```
apt install freetype freetype-dev libpng libpng-dev pkg-config
LDFLAGS=" -lm -lcompiler_rt" pip install matplotlib
```

3. And for Jupyter we need to install the zmq library as well:


```
apt install libzmq libzmq-dev
LDFLAGS=" -lm -lcompiler_rt" pip install jupyter
```

4. Install Pandas:

```
LDFLAGS=" -lm -lcompiler_rt" pip install pandas
```

You can install them one by one and maybe the -dev packages are not necessary.

You can also ignore the 'LDFLAGS=" -lm -lcompiler_rt"' part and use pip install directly.

And the installation of numpy, matplotlib, pandas and jupyter will take a while.

## Run Jupyter Notebook

To get a Jupyter notebook server running, you need to do same thing you would on any other computer:

```
jupyter notebook
```

Paste one of the links shown in the terminal into Chrome or Firefox (or other browsers like Duckduckgo) and enjoin the trip!
