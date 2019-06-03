F Prime Development Via Vagrant VMs
===================================

Tutorial and Configuration for developing F Prime FSW projects in a VM.

Quick Start
-----------

1. Install VirtualBox (or another VM supported by Vagrant)
2. Install Vagrant
3. Copy your favorite config file to a file called ``Vagrantfile``
4. Clone into your F Prime repo in a folder adjacent to this one
5. Run ``vagrant up``
6. Run ``vagrant ssh``
7. Develop on F Prime

Getting Started Tutorial
------------------------

This repository is designed to help make it easy to get a development environment set up for working with F Prime.
The assumptions are that you will use Oracle's VirtualBox for the VM tool and hassio's Vagrant to manage the VM images.

16.04 Xenial Workarounds Handled by This Config
-----------------------------------------------

Somewhere along the way, running Gds/wxgui/tools/gds.py requires wx version 4+.
This is not available from the ubuntu package manager for 16.04, which is fine because we can use pip to install it.
Pip doesn’t want to install it without removing wx 3.X, which is also OK.
We can work around this by uninstalling the Ubuntu packaged version [python-wxgtk3.0-dev and python-wxgtk3.0],
then doing pip2 install wxpython, which takes 5-ever to compile and install wx 4.X.

Then we find that somewhere we have a tool that uses wxversion to determine a version compatibility of wx,
but fails because that tool is only available in wx 3 (as Andrew mentioned previously).
We can work around that by re-installing the packaged wx 3, because pip can’t complain that we did that;
and now everything appears to work correctly in python land.

An additional detail is that the mk/os-pkg/ubuntu-packages.sh script is very helpful,
but didn’t quite get all the required packages to compile with cmake and runs the Gds GUI.
The following two packages cannot be assumed to be available, and maybe should be added to this list:

    xterm
    cmake

Also helpful for editing cmake settings and debugging: cmake-curses-gui which provides the ccmake tool.
It can be used in place of cmake, directly.
