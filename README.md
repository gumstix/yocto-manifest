Gumstix Repo Manifests for Yocto Build System
=============================================
This repository provides Repo manifests to setup the Yocto build system for
the Gumstix Overo COM.

Yocto allows the creation of custom linux distributions for embedded systems
including Gumstix-based systems.  It is a collection of git repositories known
as *layers* each of which provides *recipes* to build software packages as well
as configuration information.

Repo is a tool that enables the management of many git repositories given a
single *manifest* file.  Tell repo to fetch a manifest from this repository and
it will fetch the git repositories specified in the manifest and, by doing so,
setup a Yocto build environment for you!  Sweet deal!

This document is a work-in-progress based on

 * http://www.sakoman.com/cgi-bin/gitweb.cgi?p=meta-sakoman.git;a=blob_plain;f=README;hb=denzil
 * https://github.com/adam-lee/Gumstix-YoctoProject-Repo/blob/master/README.md

Getting Started
---------------
1.  Install Repo.

    Download the Repo script.

        $ curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > repo

    Make it executable.

        $ chmod a+x repo

    Move it on to your system path.

        $ sudo mv repo /usr/local/bin/

2.  Initialize a Repo client.

    Create an empty directory to hold your working files.

        $ mkdir yocto
        $ cd yocto

    Tell Repo where to find the manifest

        $ repo init -u git://github.com/ashcharles/gumstix-yocto-manifest.git

    A successful initialization will end with a message stating that Repo is
    initialized in your working directory. Your client directory should now
    contain a .repo directory where files such as the manifest will be kept.
    ***
    **Note**
    You can use the **-b** switch to specify the branch of the repository
    to use.  We develop on the guaranteed-to-break *dev* branch.  The *master*
    branch should at least compile.
    The **-m** switch selects the manifest file (default is *default.xml*).
    Our default.xml is designed to be stable as it *pins* particular commits.
    The current.xml tracks the head of all the manifest repositories.

    To test out the bleeding edge, type:
        $ repo init -u git://github.com/ashcharles/gumstix-yocto-manifest.git -b dev -m current.xml
    This can be done in the current repo checkout.
    ***

3.  Fetch all the repositories.

        $ repo sync

    Now go put on the coffee machine as this may take 20 minutes depending on
    your connection.

4.  Initialize the Yocto Environment.

        $ TEMPLATECONF=meta-gumstix/conf source ./poky/oe-init-build-env
    This copies default configuration information into the *poky/build/conf*
    directory and sets up some environment variables for Yocto.  You may
    wish to edit the configuration options at this point.

5.  Build an image.

    This process downloads several gigabytes of source code and then proceeds to
    do an awful lot of compilation so make sure you have plenty of space (15GB
    minimum), and expect a day or so of build time depending on your network
    connection.  Don't worry---it is just the first build that takes a while.

        $ bitbake overo-console-image

    If everything goes well, you should have a compressed root filesystem
    tarball as well as kernel and bootloader binaries available in your
    *work/deploy* directory.  If you run into problems, the most likely
    candidate is missing packages.  Check out <angstrom/yocto page> for the
    latest dependency information for your operating system.

Staying Up to Date
------------------
To pick up the latest changes for all source repositories, run:

    $ repo sync

Enter the Yocto environment:

    $ source poky/oe-init-build-env

    If you forget to setup these environment variables prior to bitbaking,
    your OS will complain that it can't find bitbake on the path.  Don't try
    to install bitbake using a package manager, just run the command.

You can then rebuild as before:

    $ bitbake overo-console-image

Starting from Fresh
-------------------
You borked it.  You're not really sure how.  But it doesn't work any more.
There are several degrees of *starting fresh*.

 1. clean a package
 2. clean build tree
 3. clean sstate
 4. destroy it all (not recommended)

Customize
---------
Sooner or later, you'll want to customize some aspect of the image either
adding more packages, picking up some upstream patches, or tweaking your kernel.
To this, you'll want to customize the Repo manifest to point at different
repositories and branches or pull in additional meta-layers.

Clone this repository (or fork it on github):

    $ git clone git://github.com/ashcharles/gumstix-yocto-manifest.git

Make your changes (and contribute them back if they are generally useful :) ),
and then re-initialize your repo client

    $ repo init -u <file:///path/to/your/git/repository.git>

