---
layout: post
title: Run JOOMMF on Your Android
categories: Android
description: none
keywords: JOOMMF, Android, Termux, Jupyter
---
## Introduction to JOOMMF

Assume you already have knowledge about OOMMF and Python, the [JOOMMF](https://joommf.github.io/) project aims to integrate the popular micromagnetic simulation package OOMMF with Python and Jupyter Notebook.

## Follow [My Last Blog](https://yuanzhi-zhu.github.io/2019/12/13/Run-OOMMF-on-Your-Android/) to Add OOMMF on You Device

Notice: JOOMMF is nothing but a user-friendly GUI by conbining OOMMF with Jupyter, it is not necessary to install it. However, if you decide to use jupyter interface to play with OOMMF, you do not need to get a desktop environment, which may save your storage.

## Installation of JOOMMF

1. You need Python3 and pip3 installed in your device:

```
apt-get update && apt-get upgrade
apt-get install python3
apt-get install python3-pip
```

2. Simply run:

```
pip3 install joommf
```
will not work because you need python module scipy and numpy first. And I can not install scipy with pip3. But the following works:

```
apt-get install libblas3 liblapack3 liblapack-dev libblas-dev
apt-get install python3-numpy python3-scipy python3-matplotlib ipython jupyter python3-pandas python3-sympy python3-nose
pip3 install joommf
```

*I used to get into trouble in installing scipy in another old linux system(Redhat), I guess this can solve the problem.*


3. Test

3.1 Set jupyter so you can access it from your computer or other device.

Follow instruction from this [link](https://jupyter-notebook.readthedocs.io/en/latest/public_server.html#notebook-server-security)

```diff

This can be helpful if you want to set your device as a jupyter server.
```

3.2 python3 -c "import oommfc; oommfc.test()"

I get an error information: ' module 'matplotlib.cbook' has no attribute 'is_string_like' '. After search for the internet, I find I need to downgrade the matplotlib:

```
pip3 install matplotlib==2.2.3
```

another error: 'ContextualVersionConflict: (ipywidgets 6.0.0...'. Run:

```
pip3 install matplotlib==2.2.3
```
to upgrade ipywidgets.

I can import oommfc now.

3.3 New problem when key 'python3 -c "import oommfc; oommfc.test()"' is that OSError: cannot find OOMMF.

According to the author of JOOMMF, we need to set the environment varible:

```
vim  /etc/profile
```
add a new line:

```
export OOMMFTCL='the absolute path of the oommf.tcl file'
```
then save the profile and run:

```
source /etc/profile
printenv | env
jupyter notebook
```
Now you can see the envvar of OOMMFTCL and run oommfc on the new jupyter(close previous one).

3.4 Successful Test

Now you can run 'python3 -c "import oommfc; oommfc.test()"' and (it will takes a while to) see most of the tests PASSED.

4 Run Your OOMMFC Using Jupyter

## Other Problems

1 When test [domain related problem](https://oommfc.readthedocs.io/en/latest/ipynb/05-getting-started-exercise-dw-pair-conversion.html), I get the Error: 'Error in OOMMF run.'.
By using oommf to load the generated mif file, I get the error information:'unrecognized Specify Block: Oxs_DMICnv:'. Then git clone the [relative file](https://github.com/joommf/oommf-extension-dmi-cnv) and re-pimake my oommf and it works this time.

## useful links
[https://github.com/palash1992/GEM/issues/51](https://github.com/palash1992/GEM/issues/51)
[https://stackoverflow.com/questions/33368261/what-is-the-easiest-way-to-install-blas-and-lapack-for-scipy/33369334](https://stackoverflow.com/questions/33368261/what-is-the-easiest-way-to-install-blas-and-lapack-for-scipy/33369334)
[https://stackoverflow.com/questions/30919393/failed-with-error-code-1-while-installing-scipy](https://stackoverflow.com/questions/30919393/failed-with-error-code-1-while-installing-scipy)
[https://jupyter-notebook.readthedocs.io/en/latest/public_server.html#notebook-server-security](https://jupyter-notebook.readthedocs.io/en/latest/public_server.html#notebook-server-security)
[https://gitter.im/joommf/support](https://gitter.im/joommf/support)