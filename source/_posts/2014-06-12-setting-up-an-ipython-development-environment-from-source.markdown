---
layout: post
title: "Setting up an IPython Development Environment from Source"
date: 2014-06-12 15:32:18 -0700
comments: true
categories: [IPython, Python, PySide, ZeroMQ] 
---

I recently decided to start hacking on the excellent [IPython](http://ipython.org/)
project. I wanted
to have full control over the versions of all the software involved, which
meant compiling Python from source. This guide is intended to take one through
the entire process of setting up a custom Python build with virtualenv in the
least number of steps possible, with the final goal of building a virtualenv
specifically for IPython dev work.

For this guide I'm using Mint 17 (based on Ubuntu 14.04). Most of the commands
should be very similar for most modern Linux systems. The biggest things that
will be different is installing build dependencies. Usually on debian based
system that will involve something along the lines of:

    sudo apt-get install build-essential

And maybe a few other packages.

## Building and Installing Python

For this example I will be installing to /opt/python277. First, create the
directory:

    sudo mkdir -p /opt/python277

Now we'll get the python source.
I want to use the latest Python 2.7. As of this writing it's 2.7.7.
get it [here](https://www.python.org/ftp/python/2.7.7/Python-2.7.7.tar.xz).
You should be able to follow these instructions with any 2.7.x version. 3.x
should work as well, but might be a little different.

Extract the downloaded tarball and go into the directory:

    tar -xvf Python-2.7.7.tar.xz
    cd Python-2.7.7

We will now configure Python source:

    export LD_RUN_PATH=/opt/python277/lib
    ./configure --prefix=/opt/python277 --enable-shared

What we're doing here is telling python to install to /opt/python277
and to be available as a shared library. This is important for
certain packages such as PySide, which we'll install later. The LD_RUN_PATH tells it which
library our python executable should link against at runtime. If
we didn't set that environment variable, it would link against the
system's python library, which causes all sorts of confusion.

Now make and install:
    make
    sudo make install

This will install a fresh python into /opt/python277. You can test it by running

    /opt/python277/bin/python

You should get a python 2.7.7 prompt.

## Setting up Package Management and virtualenv

The next thing we want to do is get pip up and running as quickly as possible,
so that we can use it for all our package management. We'll download pip
directory from pypi. It depends on setuptools, so download that
[here](https://pypi.python.org/packages/source/s/setuptools/setuptools-4.0.1.tar.gz#md5=190b1d4470de9bae0b4414353e14700d)

Then extract and install it:
    tar -xzvf setuptools-4.0.1.tar.gz
    cd setuptools-4.0.1/
    sudo /opt/python277/bin/python setup.py install

Now do the same thing with pip. The one I used is
[here](https://pypi.python.org/packages/source/p/pip/pip-1.5.6.tar.gz#md5=01026f87978932060cc86c1dc527903e)
    tar -xzvf pip-1.5.6.tar.gz
    cd pip-1.5.6/
    sudo /opt/python277/bin/python setup.py install

Alright, we should now be able to install most python packages from pypi simply
with pip now. The first thing we need is virtualenv. If you're not familiar with
virtualenv, it's awesome. Check it out [here](http://virtualenv.readthedocs.org/en/latest/).
Install it with:
    sudo /opt/python277/bin/pip install virtualenv

## Set up IPython virtualenv

We'll now create a virtualenv just for ipython development. I like to keep my
virtualenvs in ~/virt_python.
    mkdir ~/virt_python
    cd ~/virt_python

Create the virtualenv. I'll call it "ipython-dev":
    /opt/python277/bin/virtualenv ipython-dev

Activate it:
    source ipython-dev/bin/activate

Now when we run python or pip it will use the executables in
~/virt_python/ipython-dev, and any packages we install with pip
will only affect our ipython virtualenv.

## Install Dependencies

The IPython dependencies we need will depend on which parts of IPython you want
to work on. For example, to run the notebook we'll want numpy, ZeroMQ, jinja,
and tornado. It's now simply a
matter of using pip:
    pip install numpy pyzmq jinja2 tornado

Alternatively you can install the dependencies for a specific IPython console
automatically as explained below.

I want to run the IPython QT console, which depends on QT. I like the [PySide](http://qt-project.org/wiki/pyside)
python bindings. First install QT. On my system I needed:
    sudo apt-get install qt4-default

Then install PySide:
    export PYTHON_INCLUDE_DIRS=/opt/python277/lib
    pip install pyside

We need to set PYTHON_INCLUDE_DIRS so that qmake knows what to build against.



## Get the IPython Source

Clone the repository from github into ~/ipython-dev:
    cd
    git clone https://github.com/ipython/ipython ipython-dev

Install dependencies for the IPython notebook with the following:
    pip install -e ".[notebook]"

This also creates an IPython executable in your virtualenv so as long as it is
active you can simply run
    ipython
to run IPython from your development source.

You should now be set to start hacking!
