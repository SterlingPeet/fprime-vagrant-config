F Prime Development Via Vagrant VMs
===================================

Tutorial and Configuration for developing `F Prime`_ FSW projects in a VM.

NOTE
""""

This repository uses `git-lfs`_ for GitHub.
If you use GitHub's git client, you may already have it installed, if not then you should install it.
Once that is installed, run this to install the hooks in your local git repo::

    git lfs install

After you clone into the repository, you need to convert (smudge) the file pointers
to transform them to the actual files you need::

    git lfs checkout


Quick Start
-----------

1. Install VirtualBox (or another VM supported by Vagrant)
2. Install Vagrant
3. Copy your favorite config file to a file called ``Vagrantfile``
4. Clone into your F Prime repo in a folder adjacent to this one
5. Run ``vagrant up``
6. Run ``vagrant ssh``
7. Develop on `F Prime`_


Getting Started Tutorial
------------------------

This repository is designed to help make it easy to get a development environment set up for working with F Prime.
The assumptions are that you will use Oracle's `VirtualBox`_ for the VM tool and HashiCorp's `Vagrant`_ to manage the VM images.

- `VirtualBox`_ Download Page: https://www.virtualbox.org/wiki/Downloads
- `Vagrant`_ Download Page: https://www.vagrantup.com/downloads.html

Folder Structure
^^^^^^^^^^^^^^^^

The following directory layout is recommended for your host computer when developing an `F Prime`_ project with this repository::

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
  This is your `F Prime`_ source, either from the NASA public git, or your own fork.
  You can safely make edits and changes within this folder.

fprime-vagrant-config
  This repository!  The ``wheels`` subfolders and ``dev-archive`` subfolder serve as caches
  for the VM.
  This helps avoid needlessly downloading files many times over.
  The name of this folder is not sensitive, you could have multiple versions of this
  for e.g.: each different OS version you want to develop within.
  Start with one, and more advanced usage will come with time.

tools
  This folder is expected to be the `Rpi toolchain`_ mentioned in the Rpi tutorial.
  The folder must exist, but only needs to contain that repository if you intend to
  cross-compile for Rpi.
  If not, you can just create the empty folder.

other_folder_you_want_synced
  Any other folders that you want to exist on both your local machine and the VM.
  This could be the ``build`` folder for your ``cmake`` output, if desired.


Choosing Your Vagrantfile
^^^^^^^^^^^^^^^^^^^^^^^^^

This project has several options available for how to go about developing on F Prime.
The first part is just to tell you it is a ``Vagrantfile``.
The provided Vagrantfiles have three-part names, where the second part denotes
the VM OS, such as ``xenial`` for Ubuntu 16.04, Xenial Xerus.
The third part hints at the provisioning strategy for the VM.

If you are the pedantic sort of person who wants to download and compile everything
from source yourself, then you should choose a file that ends with ``-vanilla``.
If you don't want to wait 5-ever for things to compile and just want a working
system (and trust that didn't do anything nasty to the prebuilt python wheels);
then you should pick the ``-prebuiltwheels`` file.

From within the base folder of **this** repository, you can copy the Vagrantfile you chose::

    cp Vagrantfile-xenial-prebuiltwheels Vagrantfile

Or my preference on unix sytems; symbolically link it::

    ln -s Vagrantfile-xenial-prebuiltwheels Vagrantfile

At this point, you may be interested in what exactly is going on within the ``Vagrantfile``.
That is totally cool, skip down to the `How This Config Works`_ section and read about it.

All of the Vagrantfiles require a plugin to resize the disk.
This is easy, just run::

    vagrant plugin install vagrant-disksize


Creating Development VM
^^^^^^^^^^^^^^^^^^^^^^^

Now that the ``Vagrantfile`` exists, bring up the VM::

    $ vagrant up

This may take a long time if you are using a ``-vanilla`` variant, or if it is the first
time that the ``vagrant up`` command has been run.
If you are on a sloooow internet connection (e.g.: at a hotel during a seminar), you
may see significant improvement by using the *sneakernet*, aka: a thumb drive to borrow
the download cache from your neighbor.
Subsequent VMs should be much less time consuming to create (unless you chose the ``-vanilla`` variant).

Now you just have to log in and use the VM::

    $ vagrant ssh

Using the VM
^^^^^^^^^^^^

As a sanity check, we can compile the ``Ref`` application to run locally on the linux VM.
Here we use ``cmake`` in the standard way, and then start the ``Gds`` GUI to interact with ``Ref``::

    vagrant@ubuntu-xenial:~$ cd ~/src/fprime/Ref
    vagrant@ubuntu-xenial:~/src/fprime/Ref$ mkdir build
    vagrant@ubuntu-xenial:~/src/fprime/Ref$ cd build
    vagrant@ubuntu-xenial:~/src/fprime/Ref/build $ cmake ..
    vagrant@ubuntu-xenial:~/src/fprime/Ref/build $ make
    vagrant@ubuntu-xenial:~/src/fprime/Ref/build $ ./bin/Linux/Ref

Now that the ``Ref`` software is running, open a new terminal window to log in and run the Gds.
This will require an X server running on the host system; installing the X server
is beyond the scope of this tutorial.
Start by navigating to the ``fprime-devel/fprime-vagrant-devel`` folder and run this:

    $ vagrant ssh
    vagrant@ubuntu-xenial:~$ ./src/fprime/Ref/scripts/run_ref_gds.sh

NOTE:
  If you don't have an X server, you can launch the VM in a visual mode, and invoke the
  ``run_ref_gds.sh`` script from within the VM window.  This is also beyond the scope of
  this tutorial.

Once the Gds is running, send a ``CMD_NO_OP`` command to see if the system is working.


How This Config Works
---------------------

Somewhere along the way, running ``Gds/wxgui/tools/gds.py`` requires ``wx`` version 4+.
This is not available from the ubuntu package manager for 16.04 and does not seem to work
correctly for 18.04 (in F Prime).
This is fine because we can use ``pip`` to install it.
Pip doesn’t want to install it without removing ``wx`` 3.X, which is also OK.
We can work around this by uninstalling the Ubuntu packaged version [``python-wxgtk3.0-dev`` and ``python-wxgtk3.0``],
then doing ``pip2 install wxpython``, which takes 5-ever to compile and install ``wx`` 4.X.

Then we find that somewhere we have a tool that uses ``wxversion`` to determine a version compatibility of ``wx``,
but fails because that tool is only available in ``wx`` 3.
We can work around that problem by re-installing the packaged ``wx`` 3, because ``pip`` can’t
complain that we installed two different versions of ``wxpython`` if ``apt`` installs the second version;
and now everything appears to work correctly in python land.

An additional detail is that the ``mk/os-pkg/ubuntu-packages.sh`` script is very helpful,
but didn’t quite get all the required packages to compile with ``cmake`` and run the Gds GUI.
The following two packages cannot be assumed to be available, and maybe should be added to this list::

    xterm
    cmake

Also helpful for editing cmake settings and debugging: ``cmake-curses-gui`` which provides the ``ccmake`` tool.
It can be used in place of ``cmake``, directly.

.. _`F Prime`: https://github.com/nasa/fprime
.. _VirtualBox: https://www.virtualbox.org/wiki/Downloads
.. _Vagrant: https://www.vagrantup.com/downloads.html
.. _`Rpi Toolchain`: https://github.com/raspberrypi/tools
.. _git-lfs: https://git-lfs.github.com/
