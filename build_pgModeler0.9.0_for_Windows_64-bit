# Build pgModeler 0.9.0 for Windows 64-bit

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
> pacman -S mingw-w64-x86_64-qt5 mingw-w64-x86_64-postgresql mingw-w64-x86_64-libxml2
```
This will install Qt5, PostgreSQL and libxml2 as MSYS2/MinGW libraries.

Alternately, you can install them system wide with the windows graphical installers
from [Qt](https://www1.qt.io/download-open-source/)
and [PostgreSQL](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)
(libxml2 is included in the PostgreSQL installer). 

Both methods works, the first one is just more easy to update thanks to `pacman`.

If you use the graphical installer for postgreSQL, be careful to not install PostgreSQL in the default
directory (`C:\Program Files\PostgreSQL`) because there is space in `Program Files` and this will break
the building scripts. Install it directly under `C:\PostgreSQL`. For example, for PostgreSQL 9.6.5,
install it under `C:\PostgreSQL\9.6`. 

Finally download and install [InnoSetup](http://www.jrsoftware.org/isinfo.php) who will be used to create
an offline installer for pgModeler (there is not yet a MSYS2 package for InnoSetup).


## Build pgModeler

### Download pgModeler

Launch the MinGW 64-bit shell from the start menu and get the pgModeler sources with the help of `git`:

```
> git clone https://github.com/pgmodeler/pgmodeler.git
> cd pgmodeler
> git checkout tags/v0.9.0
```
We will build the 0.9.0 version of pgModeler. If a new version is available since this How-To,
you just need to change the tag accordingly. You can also try to build from `git checkout master`
or `git checkout devel` for more up-to-date releases.


### Setup

If you have installed PostgreSQL and libxml2 with `pacman`, you need to edit the `pgmodeler.pri` file to point
to the needed libraries like this:
```
windows {
  !defined(PGSQL_LIB, var): PGSQL_LIB = C:/msys64/mingw64/bin/libpq.dll
  !defined(PGSQL_INC, var): PGSQL_INC = C:/msys64/mingw64/include
  !defined(XML_INC, var): XML_INC = C:/msys64/mingw64/include/libxml2
  !defined(XML_LIB, var): XML_LIB = C:/msys64/mingw64/bin/libxml2-2.dll
  ...
}

```

If you have used the graphical installers instead, then the file is probably already correct. 
It must look like something like:
```
windows {
  !defined(PGSQL_LIB, var): PGSQL_LIB = C:/PostgreSQL/9.6/lib/libpq.dll
  !defined(PGSQL_INC, var): PGSQL_INC = C:/PostgreSQL/9.6/include
  !defined(XML_INC, var): XML_INC = C:/PostgreSQL/9.6/include
  !defined(XML_LIB, var): XML_LIB = C:/PostgreSQL/9.6/bin/libxml2.dll
  ...
}

```
<small>Note: `vim` was installed alongside `git`, so you can use it to edit the files.
You can also install other editors like `nano` from MSYS2.</small>


### Build

For building the application do the following commands:
```
> qmake pgmodeler.pro
> make
> make install
> cd build
> windeployqt --compiler-runtime pgmodeler.exe
```

This will compile the software and create a `/build` directory with half of the necessary file for running pgModeler.


### Copy the required DLL files

The `/build` directory is not yet complete, we need to add manually the DLLs of the libraries than the `windeployqt`
command have missed. Most of the missing dependencies are related to postreSQL.

It is required to manually add them to the the `/build` directory. To know the missing libraries, just double-click
on `/build/pgModeler.exe` and it will give you messages about the missing DLLs.

As the time of this writing, do the following commands inside the `/build` directory to copy the missing DLLs:
```
> cp /mingw64/bin/libeay32.dll .
> cp /mingw64/bin/libfreetype-6.dll .
> cp /mingw64/bin/libglib-2.0-0.dll .
> cp /mingw64/bin/libgraphite2.dll .
> cp /mingw64/bin/libharfbuzz-0.dll .
> cp /mingw64/bin/libiconv-2.dll .
> cp /mingw64/bin/libicudt58.dll .
> cp /mingw64/bin/libicuin58.dll .
> cp /mingw64/bin/libicuuc58.dll .
> cp /mingw64/bin/libintl-8.dll .
> cp /mingw64/bin/liblzma-5.dll .
> cp /mingw64/bin/libpcre-1.dll .
> cp /mingw64/bin/libpcre2-16-0.dll .
> cp /mingw64/bin/libpng16-16.dll .
> cp /mingw64/bin/libpq.dll .
> cp /mingw64/bin/libxml2-2.dll .
> cp /mingw64/bin/libbz2-1.dll .
> cp /mingw64/bin/Qt5Networkd.dll .
> cp /mingw64/bin/Qt5PrintSupportd.dll .
> cp /mingw64/bin/ssleay32.dll .
> cp /mingw64/bin/zlib1.dll .
```
<small>Note than the DLLs names may change since the writing of this tutorial.</small>


### Create the Windows installer

By now, you can already run pgModeler directly by double-clicking it inside the `/build` directory,
but if you want to create an installer, and have nice short-cuts in Windows you nees to create the installer
with the help of InnoSetup.

First go back inside the `pgmodeler` directory:
```
> cd ~/pgmodeler
```
and then launch the following command in the minGW 64-bit shell:
```
> "/c/Program Files (x86)/Inno Setup 5/ISCC.exe" ./installer/windows/pgmodeler.iss
```

Alternately you can also build the package with the graphical Inno Setup Compiler from your start menu.
You will have to point to the `pgmodeler.iss` file witch will be somewhere like
`C:\msys64\home\yourusername\pgmodeler\installer\windows\pgmodeler.iss`


## Install

Inno Setup should have create a `pgmodele.exe` file inside  `C:\msys64\home\myusername\pgmodeler`.
This is the installer executable. Just run it to properly install PgModeler.

