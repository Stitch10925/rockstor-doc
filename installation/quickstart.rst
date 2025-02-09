
.. _quickstartguide:

Quick start
===========

Please take note of the information in our :ref:`pre_install` guide, though
not strictly required it may help in avoiding problems encountered during
install.

.. _minsysreqs:

Minimum system requirements
---------------------------

Rockstor is a complete Linux distribution "Built on openSUSE" intended for direct hardware installation.
Virtual Machine installs can work but are not recommended without full drive or preferably whole drive controller pass-through.
Hardware raid, or md/dm software raid, underneath btrfs, Rockstor's chosen filesystem, will weaken data integrity assurances.
Multi-layered raid/drive-management arrangements, including any LVM, are not supported.
Raid controllers, if used, should thus be configured to HBA / JBOD operation.

* x86_64 celeron/AMD equivalent, or ARM64 A53 processor (2+ cores i3+ or A72+ recommended).
* 2 GB RAM (4 GB+ recommended).
* 16 GB drive dedicated to the Operating System (32 GB+ SSD recommended, 5 TB+ ignored by installer). See :ref:`usbwarning`.
* One or more additional drives dedicated to data use (less than 5 GiB ignored, 16 GiB+ recommended). See :ref:`drivewarning`.
* Ethernet interface (with internet access for updates).
* All drives must have unique serial numbers (real drives do); not all VM
  systems default to, or are capable of this. See: :ref:`vmwarewarning`.
* A UPS (recommended) that is supported by `NUT <https://networkupstools.org/>`_.
* USB port and 1 GB+ USB key for the x86_64 installation media; or a DVD drive.
  Raspberry Pi 4 / RPi 400 / ARM64EFI .raw.zx installers are self expanding system disk images,
  and so do not require a separate install media.

.. _usbwarning:

USB advisory
^^^^^^^^^^^^

.. warning::

    Rockstor cannot work reliably when installed on regular USB keys.
    Only fast variants, or preferably HDD and SSD class devices are appropriate.
    If USB is used then 2.0 is bare minimum and 3.0 and better is recommended.

.. _drivewarning:

Data drives advisory
^^^^^^^^^^^^^^^^^^^^

.. warning::

    Rockstor v4.1.0-0 and older have a 1 GiB 'ignore' threshold.
    See `if your device is small <https://btrfs.wiki.kernel.org/index.php/FAQ#if_your_device_is_small>`_
    in the btrfs wiki regarding our 16 GiB recommended minimum.
    Note: Rockstor does not use the "--mixed" format option; irrespective of drive size.

.. note::

    Raid requires compliant drive members: NAS capable drives are strongly advised.
    I.e. drives that can at least support a 7 second max:

        - Time Limited Error Recovery (TLER) - Wester Digital.
        - Error Recovery Control (ERC) - Seagate.
        - Command Completion Time Limit (CCTL) Hitachi.

    Non NAS drives often default to unlimited error recovery time,
    causing the entire drive to be reset via the linux kernel's 30s block layer timeout.
    N.B. non NAS drives may also not persist error timeout reconfiguration over a power cycle.

.. _vmwarewarning:

VMware advisory
^^^^^^^^^^^^^^^

.. warning::

    For VMware ensure you have :code:`disk.EnableUUID="true"` in your .vmx file.

Download Rockstor
-----------------

Rockstor `download options <https://rockstor.com/download.html>`_. Create a
bootable USB installer, or system disk, from a downloaded or via our self build
installer option and proceed to the :ref:`installation` section.

.. _makeusbinstalldisk:

Making a Rockstor USB install disk
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The downloaded iso must first be *restored* to a physical device to create the
install media for a physical machine install. Depending on you desktop/laptop
OS(Linux, Windows or Mac), there are GUI as well as Command Line programs using
which the install disk can be created.

.. _makeusbinstalldiskgui:

GUI options
~~~~~~~~~~~

On a Linux Unity or Gnome Desktop see our :ref:`gnome_disks_howto`.

On a Windows Desktop consider using a fairly intuitive solution like
`balenaEtcher <https://www.balena.io/etcher/>`_.

**Please note** the following USB image writing programs have been found to
produce **NON working USB install disks** when used with the Rockstor iso and
their default settings.

* Unetbootin
* Rufus (N.B. does work in DD image mode, see our :ref:`rufus_howto`)

Please also see the :ref:`usbinstall` and the :ref:`bootorderchanges` sections
of the :ref:`sohoguide` document for more information on making and using the
USB install disk.


Command line options
~~~~~~~~~~~~~~~~~~~~

To create a USB install disk on Linux or Mac one can use the dd command.
For example if your USB device is /dev/sdc then from within the directory
containing your downloaded or self-built
(see:`rockstor-installer <https://github.com/rockstor/rockstor-installer>`_)
file (iso for X86_64's) the single line command would be:-

    dd if=Rockstor-Leap15.2-generic.x86_64-4.0.7-0.install.iso of=/dev/sdc

Note that the installer file name will vary depending on the Rockstor
installer profile used: i.e. the base OS version, the general or machine
specific nature, i.e.: "generic", "ARM64EFI", or specific Pi4; and the
target architecture. If the suggested edits were not performed during the DIY
installer builder method then the file name & required single line command
will be more like:

    dd if=Rockstor-NAS.x86_64-4.0.7-0.install.iso of=/dev/sdc

**The Pi4 specific installer**, when downloaded, is a raw.zx image of a self
expanding system disk.
To transfer this file to the example proposed system disk of /dev/sdc the
following single line command could be used:

    xzcat Rockstor-Leap15.2-RaspberryPi4.aarch64-4.0.7-0.raw.xz | dd bs=4M of=/dev/sdc iflag=fullblock conv=notrunc status=progress

If you **built your own Pi4 installer** via our
`rockstor-installer <https://github.com/rockstor/rockstor-installer>`_
instructions then you can forgo the initial xzcat extraction step and use
(single line command):

    dd bs=4M if=Rockstor-Leap15.2-RaspberryPi4.aarch64-4.0.7-0.raw of=/dev/sdc iflag=fullblock conv=notrunc status=progress

For the more technically interested, we create our xz download image files from
the raw installer created by the `kiwi-ng <https://github.com/OSInside/kiwi>`_
system that our
`rockstor-installer <https://github.com/rockstor/rockstor-installer>`_ is a
configuration for via "xz --threads=4 --memlimit-compress=80% Rockstor-...raw"
to enable multi-threaded decompress.

**The ARM64EFI generic images**, when downloaded, are available in both the
raw.zx file format, like the Pi4 images, and in pre-sized (16 GB) qcow2 formats.
For the raw.zx downloaded files, the single line command is identical to the Pi4
raw.zx example above, except for the filename (single line command):

        xzcat Rockstor-Leap15.2-ARM64EFI.aarch64-4.0.7-0.raw.xz | dd bs=4M of=/dev/sdc iflag=fullblock conv=notrunc status=progress

*N.B.* The qcow2 images of the ARM64EFI profile do NOT self expand. They are
set at 16 GB. However our
`rockstor-installer <https://github.com/rockstor/rockstor-installer>`_
can configure this via the *<size unit="G">16</size>* parameter.

When built via the DIY
`rockstor-installer <https://github.com/rockstor/rockstor-installer>`_ the
resulting images are the qcow2 type. These files can be booted directly on
most common Hypervisors.

Another option on linux systems is the ddrescue command which gives
more reassuring feedback whilst the USB key is being written. On an
openSUSE/SuSE system install via *zypper in ddrescue*. On Debian and
Ubuntu systems install via *sudo apt-get install gddrescue* and on
Fedora/RehHat systems install via *sudo dnf install ddrescue*. Use is similar
to dd above only using the following single command:-

    sudo ddrescue -d -D --force Rockstor-Leap15.2-generic.x86_64-4.0.7-0.install.iso /dev/sdc

Note that there are 2 "-" characters next to each other before the "force"
switch.


Mac OS X
~~~~~~~~

For Mac OS X (tested on El Capitan). You can also use dd and the diskutil
program to create the USB stick.

Insert the USB stick and open a terminal window (Open LaunchPad and type
terminal and click on the icon). Determine the device name below, make sure
you specify the USB stick and not your OS X disk.  If you are unsure which is
which, don't go any further.

    diskutil list

Under the IDENTIFIER column, you should see a disk# (you may see a disk#s# but
just note the disk# since we need to format the whole USB Stick). Unmount and
burn the Rockstor ISO to the USB drive using the following commands, replacing
disk# with your IDENTIFIER name (this will DESTROY all data on the USB drive).

    diskutil unmountDisk /dev/disk#
    sudo dd if=~/Downloads/Rockstor-Leap15.2-generic.x86_64-4.0.7-0.install.iso of=/dev/rdisk# bs=1m

Note the 'r' is placed in front of the disk# and 'bs=1m' is for blocksize.
There is no progress bar, you will return to the command prompt once the
command finishes. Once that happens, eject the disk and you are done.

    diskutil eject /dev/disk#


Windows
~~~~~~~

There is also `dd for Windows <http://www.chrysocome.net/dd>`_ but this is
untested, please see our :ref:`makeusbinstalldiskgui`.

.. _osinstall:

Installation
------------

Rockstor 4
^^^^^^^^^^

Installing Rockstor 4 is particularly quick and straight forward.
See the following dedicated doc section for details :ref:`installer_howto`.

Rockstor 3
^^^^^^^^^^

Since Rockstor 3 is based on CentOS and uses it's anaconda installer the
installation looks similar to that of a CentOS or Fedora. Note however that not
all non-default configurations within this installer are supported by the
resulting Rockstor 3 install. So it is advised to stick to the defaults where
possible.

You can also read (for a Rockstor 3 example)
:ref:`vmmrockstorinstall` section of our :ref:`kvmsetup` for more information
about our older Rockstor 3 installation.

.. warning::
   **Important!** Installing Rockstor deletes existing data on the system
   drive(s) selected as installation destination.

.. note::
   If you need further assistance during or post install, you can post a topic
   on our `Forum <https://forum.rockstor.com>`_ or send an email to
   support@rockstor.com

1. Boot your machine with the Rockstor CD or USB and the splash screen will
   appear. Press enter and the graphical installer will start momentarily and
   display the **Installation Summary screen**

2. **Installation Summary screen**

   On this screen, multiple parameters can be configured together.

   a. Click on the **Date & Time** to change the default timezone.

   b. A network connection is required and the installation will not proceed
      otherwise. The default is DHCP which the installer automatically picks
      up. You can configure the network manually, but make sure your system has
      a working ip address for the installation to proceed.

   c. Under the **Installation Destination** there may be further action
      required if there are partitions on sda. By default the sda hard drive is
      selected and set to be auto partitioned but only if blank. If not then an
      exclamation icon indicates the need for attention. Please see our
      :ref:`wiping_disk` for more details.

      For the default automatic partitioning, just click **DONE**.

      If you are an advanced user, you can go with a custom partitioning
      scheme. However, note that Rockstor only supports **BTRFS** for its root
      filesystem.

   .. warning::
      **Important!** Installing Rockstor deletes existing data on the system
      drive(s) selected as installation destination.

   d. Once the installation configuration is complete and there are no amber
      icons, click on **Begin Installation** button to start the package
      installation.

3. **Package Installation**

   On the next screen, package installation begins in the background and you
   must set the root password. You can **optionally** create an additional
   user.

4. **Boot into Rockstor**

   Package installation takes a few minutes and once it's complete you can
   reboot, remove the install cd and boot into Rockstor. Once the system boots
   up, the url for web-ui is displayed above the login prompt. The url is
   simply :code:`https://<IP_ADDRESS_OF_THE_SYSTEM>`.

5. **Setup Rockstor**

   Go to Rockstor's web-ui from your web browser and complete the initial
   setup.
