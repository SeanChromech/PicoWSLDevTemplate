# PicoWSLDevTemplate
Go from zero to a working RP2040/Pico development environment using WSL quickly.

## Update 6/2023:
As of now, Raspberry Pi has released an official Windows installer for the Pico SDK and dev environment.  I haven't used it personally, but it's probably a better solution than this.
More information at: https://www.raspberrypi.com/news/raspberry-pi-pico-windows-installer/

## Note:
While this method seems to work (or did last time I tested it) I've found that I prefer using JetBrains CLion as an IDE for the Pico.  Setup for that is much easier, and it's a cleaner experience overall (no funky WSL required).  It's not free, which is a bummer, but I'd recommend doing that instead if you have any way of getting it (it's pretty reasonable compared to a Keil, etc.).

## Why, What, and How
### Why
I needed to quickly get a working development environment for my RP2040/Pico work, as I imagine you do, too.  The problem is, I run Windows.  The guides avilable, both though the quickstarts and the official documentation, provide a fairly inconvenient method using either the Visual C++ build tools (and whole multi-GB Windows SDK) or MinGW.

I couldn't get them to work, but I found a better solution anyway.  This is that.

### What
Using WSL (Windows Subsystem for Linux) we can install a full Linux distro seamlessly in Windows and use that for development via VS Code.  That way, we can use the superior and better-supported Linux toolchain without the hassle of a dual-boot.

I prefer Alpine Linux for this task because it's a lot smaller and more lightweight than Ubuntu.  The official Alpine Edge repository has, at the time of writing, a more up-to-date version of the compiler.  Ubuntu's latest package is a few years out of date at this point.

Beyond that, the normal Linux tools are available in WSL consistent with the Raspberry Pi docs.  That means we'll use the following:

On your computer:
- VS Code, a text editor and development environment.
   - C/C++ Extension Pack by Microsoft.
     - CMake and CMake Tools.
     - C/C++ Extension for Intellisense and Debug.
     - A few other useful things.
- WSL and the Alpine Linux distribution from the Microsoft Store.
  
On the WSL Distro:
- WSL Stuff that VS Code installs automatically.
- OpenSSH and OpenSSH-Keygen for generating SSH keys and pulling code from GitHub.
- Git, for pulling code and version control.
  - You're reading this on GitHub, so this is probably a given.
- Python3, required by other stuff.
- gcc-arm-none-eabi, The ARM Compiler and Toolchain.
- newlib, The C Library.
- cmake, The CMake Tool for generating Makefiles.
- build-base, the Alpine equivalent of the Ubuntu build-essential package.  Needed by the RP2040 SDK to build tools that run on your computer, not the RP2040 itself.
- The RP2040 SDK, which is what your code will be based on.
- My template, which is a template for a new project with a minimal CMakeLists.txt to get you started.

Simple, right?  Yeah...  I'll walk you through it, it's actually not that bad.

### How
1. Install WSL 2 and get it working.  This is outside the scope of this guide, but on an updated Windows install, type wsl --install in an Admin console.

    See https://docs.microsoft.com/en-us/windows/wsl/install

2. Install the Alpine Linux distribution from the Microsoft Store.  It's called "Alpine WSL" by agowa338.  Hit Open when it's done, and follow the prompts to set username and password.

3. Alpine doesn't ship with sudo, so just do `su` and put in your password to get an admin shell.

4. **OPTIONAL** Switch to Alpine Edge for the most up-to-date packages. This may cause issues if a package is buggy, but this is currently needed for the newest version of GCC.  
   1. `apk add nano` to get a better text editor.  Vi is included with Alpine, but vi sucks, fight me.
   2. `nano /etc/apk/repositories`
   3. Edit the version numbers to `edge`, so it should look like 
   ```
   https://dl.alpinelinux.org/alpine/edge/main
   https://dl.alpinelinux.org/alpine/edge/community
   ```
   4. `ctrl-x`, `y`, then `enter` to save the file.

5. `apk update && apk upgrade` to make sure everything is up to date before we start.
   
6. `apk add openssh git python3 gcc-arm-none-eabi newlib-arm-none-eabi cmake build-base`

7.  **OPTIONAL** Set up sudo.
    1. `apk add sudo`
    2. `passwd [username]`, put in a password for your user account.
    3. `echo '%wheel ALL=(ALL) ALL' > /etc/sudoers.d/wheel`
    4. `adduser [username] wheel`
    5. `exit` to drop out of the admin shell.

    (Yes, I know you can use visudo, but ugh vi.)

8.  **OPTIONAL** Generate SSH Key and set up Git.

    1. `exit` to drop out of the admin shell, if you haven't already.
    2. `ssh-keygen -t ed25519 -C "[your_email@example.com]"` and follow prompts.
    3. `git config --global user.email "[your_email@example.com]"`
    4. `git config --global user.name "[Your Name]"`
    5. `cat .ssh/id_ed25519.pub` and copy it to GitHub.
   
9. Grab the template from GitHub.

   1.  `git clone git@github.com:SeanChromech/PicoWSLDevTemplate.git`
   2.  `mv PicoWSLDevTemplate [Clever Project Name]`
   3.  `cd [Clever Project Name]`
   4.  `git submodule update --init` to grab the SDK.
   5.  `cd pico-sdk`
   6.  `git submodule update --init` to grab tinyusb.  *Note: Don't use --recursive, it will grab a lot of stuff from tinyusb that isn't needed.*

10. On Windows, install VS Code from https://code.visualstudio.com/Download.
    
11. Click the two arrow icon on the bottom left of the VS Code window.  Select "New WSL Window Using Distro" and select "Alpine".
    
12. In the new window, File->Open Folder, select your project folder, and trust it if prompted.

13. Install the C/C++ extension pack and Remote - WSL from the VS Code Extensions panel in the Alpine WSL window.

14. Enable CMake Tools (*Note* You may need to right-click and Enable Workspace) and reconfigure.  Select GCC[version] arm-none-eabi installed inside Alpine.  Probably in /usr/bin.

15. **OPTIONAL** Edit CMakeLists.txt and rename anything with "template" in it to your project name.

16. Click "Build" in the bottom bar.  If everything goes well, congratulations!  You're ready to go.  Binaries are in the "build" folder.  
    
*TIP* You can open a terminal ( Ctrl+Shift+\` ) in VS Code and type `explorer.exe .` to see the files in the project folder from within Windows.  This is handy for copying the *.uf2 file to your RP2040 for flashing.
