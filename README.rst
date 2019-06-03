F Prime Development Via Vagrant VMs
===================================

Tutorial and Configuration for developing "F Prime"_ FSW projects in a VM.


Quick Start
-----------

1. Install VirtualBox (or another VM supported by Vagrant)
2. Install Vagrant
3. Copy your favorite config file to a file called ``Vagrantfile``
4. Clone into your F Prime repo in a folder adjacent to this one
5. Run ``vagrant up``
6. Run ``vagrant ssh``
7. Develop on "F Prime"_


Getting Started Tutorial
------------------------

This repository is designed to help make it easy to get a development environment set up for working with F Prime.
The assumptions are that you will use Oracle's VirtualBox_ for the VM tool and HashiCorp's Vagrant_ to manage the VM images.

 - VirtualBox_ Download Page: https://www.virtualbox.org/wiki/Downloads
 - Vagrant_ Download Page: https://www.vagrantup.com/downloads.html

Folder Structure
^^^^^^^^^^^^^^^^

The following directory layout is recommended for your host computer when developing an "F Prime"_ project with this repository::

    fprime-devel
        |-- fprime
        |-- fprime-vagrant-config
        |-- tools
        \-- other_folder_you_want_synced

fprime-devel
  Strictly speaking, this can be named anything you want.
  This folder gets synced wholesale into your Vagrant instance that is defined
  within the ``fprime-vagrant-config/Vagrantfile``.
  Inside the VM, it will be mounted as ``~/src``.
  Some other subfolders will be mounted into other locations in the VM's filesystem,
  so understand that messing with system cache folders may have unintended consequences.

fprime
  This is your "F Prime"_ source, either from the NASA public git, or your own fork.
  You can safely make edits and changes within this folder.

fprime-vagrant-config
  This repository!  The ``wheels`` subfolders and ``dev-archive`` subfolder serve as caches
  for the VM.
  This helps avoid needlessly downloading files many times over.
  The name of this folder is not sensitive, you could have multiple versions of this
  for e.g.: each different OS version you want to develop within.
  Start with one, and more advanced usage will come with time.

tools
  This folder is expected to be the "Rpi toolchain"_ mentioned in the Rpi tutorial.
  The folder must exist, but only needs to contain that repository if you intend to
  cross-compile for Rpi.
  If not, you can just create the empty folder.

other_folder_you_want_synced
  Any other folders that you want to exist on both your local machine and the VM.
  This could be the ``build`` folder for your ``cmake`` output, if desired.


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
The following two packages cannot be assumed to be available, and maybe should be added to this list::

    xterm
    cmake

Also helpful for editing cmake settings and debugging: cmake-curses-gui which provides the ccmake tool.
It can be used in place of cmake, directly.

.. _"F Prime": https://github.com/nasa/fprime
.. _VirtualBox: https://www.virtualbox.org/wiki/Downloads
.. _Vagrant: https://www.vagrantup.com/downloads.html
.. _"Rpi Toolchain": https://github.com/raspberrypi/tools