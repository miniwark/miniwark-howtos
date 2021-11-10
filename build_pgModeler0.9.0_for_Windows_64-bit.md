# Build pgModeler 0.9.0 for Windows 64-bit

## Intro

This are the steeps tu build `pgModeler` for Windows 10 64-bit on Windows itself.
The produced `pgModeler` software will also be in 64-bit.


## Dependencies install

Before building `pgModeler` we need to install the build tool-chain and dependencies.
We will use the [MSYS2](http://www.msys2.org/) building platform.

MSYS2 will provide MinGW, gcc and the Qt library.
For all this tools we will use their 64-bit version.


### Install MSYS2

[MSYS2](http://www.msys2.org/) is a build tool-chain forked from [Cygwin](https://www.cygwin.com/). MSYS2 come with
the [Pacman](https://www.archlinux.org/pacman/) package manager from Arch Linux and the MinGW fork
from [MinGW-w64](https://mingw-w64.org/).

To install MSYS2 you need to download the 64-bit version installer from  [http://www.msys2.org/](http://www.msys2.org/)
and follow the install Wizard.

When done you will have 3 new available shells.

* the MSYS shell
* the MinGW 32-bit shell
* the MinGW 64-bit shell

The MSYS2 shell is used only to perform install and update tasks and the MinGW shels are used to build software.

At the last steep of the Wizard, keep checked the *"Run MSYS2 Now"* check-box. This will start the MSYS shell
(you can also launch it from the start menu). 

In this shell, first update MSYS2 itself with the welp of `pacman`:
```
> pacman -Syuu
```

Then restart the MSYS shell and redo the above command until `pacman` tell you than there is nothing more to update.

<small>Note: all of the MSYS2 stuff, including things installed with the `pacman` command will be installed under `C:\msys64\`.
It emulate a unix file system, including a `home` directory at `C:\msys64\home\`.</small>


### Install the build tools

To buid pgModeler we need a few build tools and libraries.

To install the build tools, in the MSYS shell run:
```
> pacman -S pacman base-devel mingw-w64-x86_64-toolchain git
```
When asked, just install all the members.

To install the libraries required to build pgModeler run:
```
> pacman -S mingw-w64-x86_64-qt5 mingw-w64-x86_64-qt-installer mingw-w64-x86_64-postgresql mingw-w64-x86_64-libxml2
```
This will install Qt5, QT Installer Framework, PostgreSQL and libxml2 as MSYS2/MinGW binaries.

Alternately, you can install them system wide with the windows graphical installers
from [Qt](https://www1.qt.io/download-open-source/)
and [PostgreSQL](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)
(libxml2 is included in the PostgreSQL installer). Don't forget to include the QT Installer Framework.

Both methods work, the first one is just easier to update thanks to `pacman`.

If you use the graphical installer for postgreSQL, be careful to not install PostgreSQL in the default
directory (`C:\Program Files\PostgreSQL`) because there is space in `Program Files` and this will break
the building scripts. Install it directly under `C:\PostgreSQL`. For example, for PostgreSQL 9.6.5,
install it under `C:\PostgreSQL\9.6`. 


## Build pgModeler

### Download pgModeler

Launch the MinGW 64-bit shell from the start menu and get the pgModeler sources with the help of `git`:

```
> git clone https://github.com/pgmodeler/pgmodeler.git
> cd pgmodeler
> git checkout tags/v0.9.3
```
We will build the 0.9.3 version of pgModeler. If a new version is available since this How-To,
you just need to change the tag accordingly. You can also try to build from `git checkout master`
or `git checkout devel` for more up-to-date releases.


### Setup

Since version 0.9.2-alpha1, pgModeler is now configured and built for Windows through the `windeploy.sh`
script using the QT Installer Framework. While most of the procedure is done automatically, you still 
have to check for a few parameters to be right.

The script assumes that you installed PostgreSQL and QT through MSYS. If that is not the case,
edit the script and alter the values of `QT_ROOT`, `PGSQL_ROOT` and `QMAKE_ARGS` (which assumes
that the PostgreSQL include files and libraries are installed inwide MINGW). And as you will notice,
you will also have to alter `MINGW_ROOT` if you didn't install MSYS/MINGW in the default locations.

<small>Note: `vim` was installed alongside `git`, so you can use it to edit the files.
You can also install other editors like `nano` from MSYS2.</small>


### Build

For building the application just send the following command:
```
> /bin/bash ./windeploy.sh -x64-build
```

This will compile the software and create a `/build` directory with the necessary files for running 
pgModeler, and then a `/dist` directory where you will find the installer. You can hunt for errors
in the `windeploy.log` file that the script will generate through the process, and another good measure
is to add the command `set -x` right at the beginning of the script to debug the whole execution, if you
are still having trouble.

If the script ran just fine, you can skip to the Install section of this HOWTO.


### In case there are missing DLL files

You will find the `pgmodeler.exe` file in the `/build` directory. Try to execute it, and if the application 
does not come up, complaining instead about missing libraries, try to copy those to the `/build` directory 
from `/mingw64/bin` and keep trying to run the program and copying the missing libraries util it comes up.
Most of the missing dependencies are related to postreSQL.

This is just in case `windeploy.sh` missed any DLLs, but it shouldn't be.

### Manually recreate the Windows installer

To regenerate the installer without restarting the whole `windeploy.sh` process, just run:

```
> binarycreator -v -c installer/template/config/config.xml -p installer/template/packages dist/pgmodeler-0.9.3-windows64.exe
```

Otherwise, invoking `windeploy.sh` again will force a cleanup and run a rebuild.


## Install

QT Installer Framework should have created a `pgmodeler-0.9.3-windows64.exe` file inside  `/dist`.
This is the installer executable. Just run it to install PgModeler.

