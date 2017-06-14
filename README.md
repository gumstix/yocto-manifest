Gumstix Repo Manifests for the Yocto Project Build System
=============================================
This repository provides Repo manifests to setup the Yocto build system for 
supported Gumstix products.

***
**Note:**
If you already have a Yocto Project setup and want only the Gumstix BSP layer, 
use the meta-gumstix repository found here: 
git://github.com/gumstix/meta-gumstix.git.
***

The Yocto Project allows the creation of custom linux distributions for embedded
systems, including Gumstix-based systems.  It is a collection of git
repositories known as *layers* each of which provides *recipes* to build
software packages as well as configuration information.

Repo is a tool that enables the management of many git repositories given a 
single *manifest* file.  Tell repo to fetch a manifest from this repository and
it will fetch the git repositories specified in the manifest and, by doing so,
setup a Yocto Project build environment for you!

Getting Started
---------------
**1.  Install Repo.**

Download the Repo script:

    $ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > repo

Make it executable:

    $ chmod a+x repo

Move it on to your system path:

    $ sudo mv repo /usr/local/bin/

If it is correctly installed, you should see a Usage message when invoked
with the help flag.

    $ repo --help

**2.  Initialize a Repo client.**

Create an empty directory to hold your working files:

    $ mkdir yocto
    $ cd yocto

Tell Repo where to find the manifest:

    $ repo init -u git://github.com/gumstix/yocto-manifest.git 

A successful initialization will end with a message stating that Repo is
initialized in your working directory. Your directory should now
contain a .repo directory where repo control files such as the manifest are
stored but you should not need to touch this directory.

***
**Note:**
You can use the **-b** switch to specify the branch of the repository
to use.  We develop on the guaranteed-to-break **morty** branch.  Most people should use
the **fido** branch, which should at least compile.

The **-m** switch selects the manifest file (default is *default.xml*).
Our default.xml on **fido** is designed to be stable as it *pins*
particular commits.

To test out the bleeding edge, type:

    $ repo init -u git://github.com/gumstix/yocto-manifest.git -b morty
    $ repo sync

Note that the default settings for bblayers.conf and local.conf may change
between branches.  If the environment was originally setup with e.g.
_TEMPLATECONF=meta-gumstix-extras/conf_, check the _*.sample_ files in that
directory for any corresponding changes needed to the settings in
 _build/conf/_.

To get back to the known stable version, type:

    $ repo init -u git://github.com/gumstix/yocto-manifest.git -b fido
    $ repo sync

Also you can get a specific version of Yocto Project:

For example,

    $ repo init -u git://github.com/gumstix/yocto-manifest.git -b refs/tags/danny
    
To learn more about repo, look at [Repo Command Reference](https://source.android.com/source/using-repo "Using repo")
***

**3.  Fetch all the repositories:**

    $ repo sync

Now go turn on the coffee machine as this may take 20 minutes depending on
your connection.

**4.  Initialize the Yocto Project Build Environment.**

    $ export TEMPLATECONF=meta-gumstix-extras/conf 
    $ source ./poky/oe-init-build-env

This copies default configuration information into the **poky/build/conf**
directory and sets up some environment variables for the build system.  This configuration
directory is not under revision control; you may wish to edit these configuration
files for your specific setup. In particular, change the `MACHINE` variable in **conf/local.conf** if you are
not building for the Overo (default).

**5.  Build an image:**

This process downloads several gigabytes of source code and then proceeds to
do an awful lot of compilation so make sure you have plenty of space (25GB
minimum), and expect a day or so of build time depending on your network
connection.  Don't worry---it is just the first build that takes a while.

    $ bitbake gumstix-console-image

If everything goes well, you should have a compressed root filesystem
tarball as well as kernel and bootloader binaries available in your
**tmp/deploy/images/{ overo | duovero | pepper }** directory.  If you run into problems, the most likely
candidate is missing software packages.  Check out
[The Build Host Packages](http://www.yoctoproject.org/docs/current/yocto-project-qs/yocto-project-qs.html#resources "Yocto quick start guide")
for the list of required packages for operating system. Also, take
a look to be sure your operating system is supported:
[Distribution Supported List](https://wiki.yoctoproject.org/wiki/Distribution_Support "Yocto wiki")


**6. Create a bootable micro SD card:**

You are one step closer to booting your Gumstix with the new image you built! 
Usually you have to create two partitions in your uSD: `boot` and `root`, and copy the bootloader and the root file system.
Optionally you may want to create a swap partition.
Yocto Project comes with a small utility called `wic` to help you out with this process.
Here is a generic workflow using the Gumstix kickstart file:

    $ wic create sdimage-gumstix -e gumstix-console-image

    Info: The new image(s) can be found here:
     /var/tmp/wic/build/sdimage-gumstix-201506231742-mmcblk.direct

The kickstart file `sdimage-gumstix.wks` describes the layout of the storage. It is located here:

    meta-gumstix-extras/scripts/lib/image/canned-wks/

By default, it creates a DD-able image (`.direct`) with boot, root and swap partitions. It fits into a 2GB large uSD card.
You can tweak the size parameter (--size) in the kickstart file to match the size of the image to your micro-SD card.

**7. Flash your image:**

Note that your uSD will have to be at least 2GB large. Pop in your micro SD card to your card writer, and find out the location of
the block device by running `dmesg`.

Be sure to install `bmap-tools`:

    $ sudo apt-get install bmap-tools

Create a bmap file:

    $ bmaptool create /var/tmp/wic/build/sdimage-gumstix-201506231742-mmcblk.direct > image.bmap

Flash uSD:

    $ sudo bmaptool copy --bmap image.bmap /var/tmp/wic/build/sdimage-gumstix-201506231742-mmcblk.direct /dev/sdb

If you get an error due to the size of the image being bigger than the uSD card, adjust the size parameter (`--size`) in the kickstart file.

Hooray you are done!

Staying Up to Date
------------------
To pick up the latest changes for all source repositories, run:

    $ repo sync

Enter the Yocto Project build environment:

    $ source poky/oe-init-build-env

    If you forget to setup these environment variables prior to bitbaking, your 
    OS will complain that it can't find bitbake on the path.  Don't try to
    install bitbake using a package manager, just run the above command.

You can then rebuild as before:

    $ bitbake gumstix-console-image

Starting Fresh
-------------------
So something broke... what do you do now?

There are several degrees of *starting fresh*: individual packages can be 
rebuilt or the whole system can be reconstructed.

 1. clean a package: bitbake <package-name> -c cleansstate
 2. re-download package: bitbake <package-name> -c cleanall
 3. destroy everything but downloads: rm -rf build/sstate-cache build/tmp (or wherever your sstate and work directories are)
 4. destroy it all (not recommended): rm -rf build

***
**Note:**
If you've made a change to a recipe and want the package to be rebuilt, just 
increment the recipe version (the PR variable); cleaning is not necessary.

To understand better how bitbake processes recipes, look at the excellent 
documentation:
[Yocto Project Reference Manual](http://www.yoctoproject.org/docs/current/poky-ref-manual/poky-ref-manual.html)
***

To make sense of the differences between these cleaning methods, it is useful to
 understand that Yocto caches both the downloaded source files for all the 
packages it tries to build (the DL_DIR configuration parameter) and the packages
once built (the SSTATE_DIR configuration parameter). Typically, deleting the
downloaded source is a bad idea---this just means re-fetching gigabytes of code
which wastes network bandwidth. Cleaning the sstate cache for a particular
package ensures that it actually gets rebuilt from source rather than simply
restored from the cache.

Customize
---------
Sooner or later, you'll want to customize some aspect of the image either adding
 more packages, picking up some upstream patches, or tweaking your kernel. To
this, you'll want to customize the Repo manifest to point at different
repositories and branches or pull in additional meta-layers.

Clone this repository (or fork it on github):

    $ git clone git://github.com/gumstix/yocto-manifest.git

Make your changes (and contribute them back if they are generally useful), and
then re-initialize your repo client

    $ repo init -u <file:///path/to/your/git/repository.git>


Additional Resources
--------------------
Please checkout [Gumstix Yocto Project Wiki](https://github.com/gumstix/yocto-manifest/wiki) for tips on building and using the Gumstix images.
