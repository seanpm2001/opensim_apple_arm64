# opensim_macos_arm64

Opensim Support for Apple Silicon M1/M2

v1.9 / 16 March 2023

This project provides instructions and files to run Opensimulator server software
(http://opensimulator.org) on Apple Silicon (M1/M2) computers fully native,
and to install a more recent version of the Bullet physics engine on macOS
or Linux computers.

This project is in active development and this page will be updated as the project evolves.

If you just want to run Opensim on Apple Silicon and are not interested in details for developers,
you can skip down to the General Requirements section below .

*What's New*

This release brings a major update with universal binaries for macOS, which are used 
in the dotnet6 branch of Opensimulator. The arm64-only libraries in this repository have been working
fine and no updates are needed on those. However, there is a long-term goal of migrating to
universal binaries instead of architecture-specific libraries for macOS. These universal
binaries support Intel and Apple Silicon CPUs in a single file, and provide a simpler path
to future rebuilding of libraries when updates and bug fixes become available. They also
simplify the configuration files in Opensimulator which determine which
libraries should be loaded for various CPU and platform combinations.

The BulletSim universal library previously posted here was working across both CPU architectures but
did not support macOS versions older than Monterey. Today's release includes an update to the Bullet
universal library that includes support for [all macOS versions that dotnet6 supports](https://learn.microsoft.com/en-us/dotnet/core/install/macos),
Catalina (10.15) through Ventura (13.x). In addition, the other two unmanaged libraries in
Opensimulator --  ubODE and openJpeg -- are now available as universal libraries as well, with 
the same CPU and macOS version support. All libraries are code signed with Apple. 

  - libBulletSim-2.86-20230316-universal.dylib
  - libopenjpeg-2-1.5.0-20230316-universal.dylib
  - libubode.5-20230316-universal.dylib

At the time of this writing, these universal binaries have not yet been merged into the
Opensimulator trunk, and are available from this repository. Please try them and provide feedback
on your success.

The Bullet universal binary listed above includes all the current
bugfix patches (including [this patch](https://bitbucket.org/opensimulator/opensim-libs/src/master/trunk/unmanaged/BulletSim/0001-Call-setWorldTransform-when-object-is-going-inactive.patch)
and has functional parity with Bullet libraries for other platforms. 

The following files in this repository mirror the latest versions
in the Opensim repository (dotnet6 branch), but will probably be replaced
by the universal binaries at some point.
These are code-signed with Apple and were built with the process detailed in a later section.
  - libopenjpeg-arm64.dylib
  - libubode-arm64.dylib


*Bullet Physics*

There is ongoing work in developing a new version of Bullet for all Opensim platforms, 
based on the latest version of Bullet, 3.25, and other bug fixes. 
Misterblue is leading the code development using a new build process that
I helped develop. I'm also developing a number of in-world test cases that can be used to
validate physics engine functionality and identify and fix bugs.

The long term plan is to have the same version, patches, and feature parity across all platforms.
Misterblue is also working on a major update to the Bullet wrapper that Opensim uses
with the Bullet engine. The future wrapper version will report the actual Bullet physics version and will be
mostly 64-bit clean, so is also likely to improve performance.

We identified some reproducible bugs that still need looking into:

1) Some of the console commands to change Bullet parameters do not appear to be taking hold

2) Physics objects that go into physics "sleep" state do not wake up properly in some use cases

Eventually, after appropriate testing, discussions and fixes,
Bullet libraries in Opensim trunk will be replaced with the new version.
Until then, I will try to keep this page updated with the latest status.

I'm providing experimental versions of the Bullet library using the
updated 3.25 version of Bullet physics and the 0001 patch described above.
These should be at least as good as the current version, adding the latest
Bullet engine, although they still use the old (current) Bullet wrapper and
don't yet resolve the bugs described above.

  - libBulletSim-3.25-20230315-universal.dylib  (x86\_64 and arm64, macOS 10.15-13.x)
  - libBulletSim-3.25-x86_64-20230211.so (64-bit Linux, built on Ubuntu)

To install,
first place the file in opensim/bin/lib64/. Back up an original copy of the file
	
	/bin/OpenSim.Region.PhysicsModule.BulletS.dll.config

then edit that file to point to the new dylib file.

For macOS, you would change the line:

	<dllmap os="osx" dll="BulletSim" target="lib64/libBulletSim-2.86.1-20221213-universal.dylib" />

to:

	<dllmap os="osx" dll="BulletSim" target="lib64/libBulletSim-3.25-20230315-universal.dylibb" />

For 64-bit Linux, you would change the line:

	<dllmap os="!windows,osx" cpu="x86-64" dll="BulletSim" target="lib64/libBulletSim.so" />

to:

	<dllmap os="!windows,osx" cpu="x86-64" dll="BulletSim" target="lib64/libBulletSim-3.25-x86_64-20230211.so" />


And lastly, I am providing an experimental version of Bullet 3.25, the same as the one
above but with the Bullet sleep feature disabled using the patch _Bullet-disable-sleeping.patch_
from this repository. This replaces the 0001 patch and effectively 
disables the switching between object sleep and wake states.

This is a hack
and not the best way to accomplish shutting off the physics sleep feature. But it is an 
easy way to determine if an observed bug is related to the physics sleep feature.
This hack resolves bugs for some use cases with "stuck" prims, but regresses the wandering-prim
issue on other use cases. A cleaner long-term solution is needed.
The change may have the effect of using more CPU on physics-enabled objects.
  - libBulletSim-3.25-20221215-universal-NOSLEEP.dylib

We are looking for feedback from people who are testing this. Please let us know
about your successes or issues!

This work is licensed under Creative Commons BY-NC-SA 3.0:
https://creativecommons.org/licenses/by-nc-sa/3.0/

-------------
**General Requirements to Run Opensim on Apple Silicon**

An Apple computer with an M1 or M2 chip. Currently the entire line of Apple Macs
are using these. Most of the setup process described here works the same on Intel macs. I will note
where any steps are different for Intel.

You will need to install libgdiplus. I recommend using the package manager Brew for this.

To install Brew, run the following at the terminal:
	/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

Next, use Brew to install libgdiplus, which is required for Opensimulator.\
	brew install mono-libgdiplus

This will install about 30 packages which include prerequisites. On Big Sur through Ventura this
process takes a few minutes.

If you are on Catalina, the installation works, but it can take several hours
since many of the components need to be compiled. Issue the above command and then 
bring a nice book to read at the coffee house while you wait for it to complete.

On an Apple Silicon computer, create a symbolic link:
	sudo ln -s /opt/homebrew/Cellar/mono-libgdiplus/6.1_1/lib/libgdiplus.dylib /usr/local/lib/libgdiplus.dylib

This link is required because on Apple Silicon, brew installs into
/opt/homebrew/ instead of /usr/local. Dotnet uses /usr/local as a hard-coded path 
and ignores shell environments.

Download and install Dotnet6 SDK Installer from the following site.
Select arm64 or x86_64 depending on your architecture:
	https://dotnet.microsoft.com/en-us/download/dotnet/6.0

Download and install the latest stable Mysql Community Server:
	https://dev.mysql.com/downloads/mysql/

On older macOS versions you will need to use a MySQL download from the past releases.
Use the latest version that is compatible with your operating system.

During MySQL installation, you will be prompted to choose Legacy or Secure encryption.
Select "Use Legacy Password Encryption” unless you know what you’re doing.
Opensim does support Secure Encryption but this option is more complex to set up
and is not needed for standalone or test systems.
    
Mysql will work fine with the default settings, but if you want to change the 
settings, you can install a file at /private/etc/my.cnf and use MySQL Pref Pane
to Apply the changes.

Once the Mysql server is up and running, then either import your existing opensim database or
create a new empty database for a fresh install of opensim. 

Set up a Mysql user account that has all privileges granted for the opensim database. 
Confirm the account can access the database by logging in to it with a database client.

-------------
**Installing Opensim**

Use git to pull in the latest source with

	git clone git://opensimulator.org/git/opensim

You should place the folder where it will permanently reside; problems will occur
if you move it's path later. On my system I place it in my top level user folder, at
/Users/myname/opensim, or ~/opensim.

Switch to the dotnet6 branch with

	git checkout dotnet6
	
Follow the instructions in BUILDING.md to build opensim. On macOS this typically is:

	./runprebuild.sh
	cp -f bin/System.Drawing.Common.dll.linux bin/System.Drawing.Common.dll
	dotnet build --configuration Release OpenSim.sln

You will also need to create some configuration files before Opensim will run.
If you are migrating an existing system, just copy over your config files. 
If you are new to opensim, config file requirements are explained in BUILDING.md.

-------------
**Start Up Opensim, the Dotnet Way**

Once the config files are set up, you are ready to start the server:

	cd /path/to/opensim/bin; ./opensim.sh


-------------
**To build arm64 libraries from source**

The following instructions are for reference only and document how the
macOS libraries in this repository were built. You can use them if you want
to build the libraries yourself for some reason. 
I have updated the instructions for universal binaries.

You will need to have a shell environment with environment 
variables appropriately set for the development work. Bash (Bourne-again shell)
is the preferred shell when working with older unmanaged software.

Apple sets the default shell to zsh, so this will need to be changed.
Open the Users
and Groups preference pane. Right-click on your username and select Advanced Options.
This opens a window where you can change your default shell to /bin/bash. 

I am including in this repository  a sample
bashrc file which is a good replacement for the default that Apple provides.
This has a configuration that merges paths for
installed apps and libs from Brew (in /opt/homebrew) and from independently
installed packages like Mysql (in /usr/local). 

To install this, backup or rename the file at /private/etc/bashrc, then
move the file _bashrc_ from this repository to /private/etc/bashrc.

Once this is installed it will take effect in any new terminal windows
that you open. If you want these environment variables to be applied
to a terminal window already open, you can apply the changes by typing:

	source /etc/bashrc

On most systems, installing Brew will also trigger the installation of
Apple's "Command Line Developer Tools". If for some reason it was not installed, 
that needs to be installed at this time. There is no harm if you re-install it. 
You can install it by typing the following at the command line:

	xcode-select --install

Download the repository for opensim-libs, which contains projects needed for the
physics engines. 

	git clone git://opensimulator.org/git/opensim-libs

Please note, when building your own libraries, they will work on your system,
but will need to be code signed with Apple before they will work on other macOS systems.

-------------
**Installing Libopenjpeg**

Visit https://bitbucket.org/opensimulator/libopenmetaverse/src/master/openjpeg-dotnet/

Go to Downloads -> Download Repository. Download and unpack.

	cd opensimulator-libopenmetaverse-*/openjpeg-dotnet
	
Download the file _openjpeg-mac-arm64.patch_ from this repository and place it in this directory.
This is required for the arm64 architecture or for a universal binary containing that.
Apply it with the following:

	patch -p1 < openjpeg-mac-arm64.patch

Then build and install the shared library:	

	make -f Makefile.osx
	cp -f libopenjpeg-dotnet-2-1.5.0-dotnet-1.dylib /path/to/opensim/bin/lib64/libopenjpeg-[version]-[date]-universal.dylib



-------------
**Installing ubODE**

	cd /path/to/opensim-libs/trunk/unmanaged/ubODE-OpenSim
	brew install libtool automake
	PATH="/opt/homebrew/opt/libtool/libexec/gnubin:$PATH"
	./bootstrap
	./configure --enable-shared --enable-double-precision 

The build system doesn't support multi-CPU options without hand-editing Makefiles.	
At this point, for building a universal binary, you need to hand-edit the following three Makefiles:

- [path to ubODE-OpenSim]/Makefile
- [path to ubODE-OpenSim]/ou/Makefile
- [path to ubODE-OpenSim]/ou/src/ou/Makefile

In each of these add the following compiler flags to any occurrence of CFLAGS, CPPFLAGS and CXXFLAGS:

	-arch arm64 -arch x86_64

Then you should be able to run the following commands at the top level of the project to finish the build:
	make
	cp -f ode/src/.libs/libubode.5.dylib /path/to/opensim/bin/lib64/libubode-[version]-[date]-universal.dylib

	
-------------
**Installing Bullet** 

Building Bullet requires cmake. If not already installed, install it with brew:

	brew install cmake
	
Bullet installation is done in 2 steps. First, compile a Bullet distribution and
install library archive files (.a) and include files into temporary directories. Then, compile the
"Bullet Glue" which packages the Bullet libraries, along with connector
libraries, into a single static library.

The build process described here works the same on Bullet version 2.86, the version currently
used in Opensim trunk, and the latest version 3.25. The only differences are the
patches that need to be applied before compiling.

For version 2.86, download and unpack the tarball from:
	https://codeload.github.com/bulletphysics/bullet3/tar.gz/2.86.1

Or, if you want to try the latest Bullet version 3.25, download a ZIP file from:
	https://github.com/bulletphysics/bullet3

Apply this [patch](https://bitbucket.org/opensimulator/opensim-libs/src/master/trunk/unmanaged/BulletSim/0001-Call-setWorldTransform-when-object-is-going-inactive.patch)
provided by Misterblue to fix an issue involving wandering prims. This patch has been used on all the
architectures of Bullet libraries in the Opensim distribution.

Bullet version 2.86 also requires a patch if it is being compiled for macOS x86\_64 architecture
or a universal binary containing x86\_64. This patch is not required for Bullet 3.25 as it is already applied. 
The patch is provided in the file _bullet-2.86-mac-arm64-x86\_64.patch_ from this repository. 
Place it in the top level directory and apply it with the following:

	patch -p1 < bullet-2.86-mac-arm64-x86_64.patch

At the terminal, change directory to the top level or your
Bullet distribution. Depending on your version this will be _bullet3-2.86.1_ or _bullet-master_.
The following instructions reference bullet3-2.86.1, but the steps are the same for
either version.

Note that the cmake command listed below merges flags needed for either Bullet version. You can use the
same command for either version; flags not recognized will just be ignored.

	cd bullet3-2.86.1
	mkdir bullet-build
	cd bullet-build
	cmake .. -G "Unix Makefiles" -DBUILD_BULLET2_DEMOS=off -DBUILD_BULLET3=on -DBUILD_CLSOCKET=off -DBUILD_CPU_DEMOS=off -DBUILD_ENET=off -DBUILD_EXTRAS=on -DBUILD_DEMOS=off -DBUILD_OPENGL_DEMOS=off -DBUILD_PYBULLET=off -DBUILD_SHARED_LIBS=off -DBUILD_BULLET_ROBOTICS_GUI_EXTRA=off -DBUILD_BULLET_ROBOTICS_EXTRA=off -DBUILD_OPENGL3_DEMOS=off -DBUILD_UNIT_TESTS=off -DINSTALL_LIBS=ON -DINSTALL_EXTRA_LIBS=ON -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS="-fPIC -arch arm64 -arch x86_64" -DCMAKE_INSTALL_PREFIX=./install
	make
	make install

This installs
  - Bullet .a files to bullet3-2.86.1/bullet-build/install/lib
  - Bullet includes into bullet3-2.86.1/bullet-build/install/include/bullet


Step 2 requires building the Bullet glue:

	cd /path/to/opensim-libs/trunk/unmanaged/BulletSim/

A patch must be applied to build this on arm64 or on a universal binary containing arm64.

Download the file _bulletsim-glue-mac-arm64-x86\_64.patch_ from this repository and place it in this directory.
This patch file contains changes needed only for building the arm64 architecture.

Apply the patch with the following:

	patch -p1 < bulletsim-glue-mac-arm64-x86_64.patch

Edit the Makefile and set IDIR and LDIR to the path for your .a and include files for Bullet.

	LDIR = /full/path/to/bullet3-2.86.1/bullet-build/install/lib
	IDIR = /full/path/to/bullet3-2.86.1/bullet-build/install/include/bullet

To build a universal binary, also replace any occurrence of "-arch arm64" with 
"-arch arm64 -arch x86_64".

Then build and install:

	make
	cp -f libBulletSim.dylib /path/to/opensim/bin/lib64/libBulletSim-[version]-[date]-universal.dylib


-------------

This is a work in progess. Please notify me of any bugs or feature requests.

Cuga Rajal (Second Life and Opensim)

cuga@rajal.org
