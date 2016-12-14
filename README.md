# (WIP) Data Science On WSL

This guide contains notes on setting up a development environment for data science in Windows 10's new ***Windows Subsystem for Linux (WSL)*** aka ***Bash on Ubuntu on Windows***.

![wsl](https://github.com/joshpeng/Data-Science-On-WSL/raw/master/imgs/wsl.png)

# Table of Contents

### [Bash and Python](#bash-and-python-1)

**Prerequisite: Windows 10 Anniversary Build 14393 or later**

1. [(Optional) Install Cmder](#1-optional-install-cmder)
2. [Install WSL](#2-install-wsl)
3. [Install Anaconda](#3-install-anaconda)
   1. [Apply fix for Jupyter notebooks](#fix-jupyter-notebooks)
   2. [Apply fix for MKL](#fix-mkl)
4. Install X Server for Windows
   1. Apply fix for X
   2. Apply fix for dbus
5. (Optional) Install Command Line power user features
   1. Install Oh My Zsh
   2. Configure .bashrc
   3. Configure .zshrc
   4. Install Powerline compatible fonts
   5. Configure Cmder startup tasks


### Node and Web Dev

1. Install nvm
2. Install latest node LTS
3. Known issues/limitations





# Bash and Python

### 1. (Optional) Install Cmder

Cmder is an excellent console emulator for Windows offering many features beyond what cmd provides. To name just a few, you'll benefit from color themes, tabs, and git integration.

1. Go to [cmder.net](http://cmder.net)
2. Select ```Download full```
3. Extract somewhere convenient
4. (Optional) Add the Cmder directory to your PATH variable to allow for quick launch from the Start Menu
   1. Press <kbd>Win</kbd>
   2. Type ```environment``` and select ```Edit the system environment variables```
   3. Click ```Environment Variables```
   4. Select ```Path``` entry under the ```System variables``` section and press ```Edit...```
   5. Click ```New``` to add your Cmder directory and press ```OK``` when done
5. Apply fix for <kbd>Shift</kbd>+<kbd>Up</kbd> hot key
   1. Go to ```<Cmder Directory>\vendor\clink```
   2. Open ```clink_inputrc_base``` in Notepad
   3. Find and replace all ```M-C-u``` with ```"\033`b"```. There should be three instances.
   4. Save and restart Cmder. Now press <kbd>Shift</kbd>+<kbd>Up</kbd> will navigate up in your directories

### 2. Install WSL

Follow Microsoft's installation guide [here](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide).

- On the "Create a UNIX user" step, if you want root access you can create a user with ```root``` as the name. This will create your user as a superuser and will preclude you from needing to use ```sudo``` for any commands. 
- If you mess up your WSL and wish to do a clean install, use the following commands in Cmder (not Bash):
   ```
   lxrun /uninstall /full
   lxrun /install
   ```

- File permissions are handled separately by both Windows and Linux. See [here](https://msdn.microsoft.com/en-us/commandline/wsl/user_support) for more information. You may want to run Cmder as administrator. If you want to always run Cmder as admin, you can do so by right-clicking its .exe file and selecting the option in its properties.
- When in Windows, you can find the Linux file system at ```%AppData%\Local\lxss```. When in Bash, you can find the Windows file system at ```/mnt/c```.
  - Despite this, file interoperability is not supported. Linux files have information stored in their NTFS Extended Attributes that Windows can't create. If you try to edit them in Windows, those attributes might get stripped and become unusable in Linux also. For more information see the "Emulating Linux features" section in this [MSFT blog post](https://blogs.msdn.microsoft.com/wsl/2016/06/15/wsl-file-system-support/).

### 3. Install Anaconda

Anaconda is the industry standard Python data science platform used by most people. It comes with a great selection of preinstalled packages as well as features to help you manage environments for multiple Python versions. Downloads are located at [https://www.continuum.io/downloads](https://www.continuum.io/downloads)

1. Open  Cmder and start bash

2. Choose if you want to use Python 2.7 or 3.5 with one of the following commands (Anaconda2 for 2.7, Anaconda3 for 3.5):

   ```
   wget https://repo.continuum.io/archive/Anaconda2-4.2.0-Linux-x86_64.sh
   wget https://repo.continuum.io/archive/Anaconda3-4.2.0-Linux-x86_64.sh
   ```

3. After the download completes, install via one of the following commands:
   ```
   bash Anaconda2-4.2.0-Linux-x86_64.sh
   bash Anaconda3-4.2.0-Linux-x86_64.sh
   ```


#### Fix Jupyter notebooks

In Windows 10 Build 14393, there is a bug with libzmq that is apparently fixed in later Insider Builds. For now though, we will need to do the following command in bash:

```
conda install -c jzuhone zeromq=4.1.dev0
```

This patches zeromq to work with WSL. For more information about the issue see [here](https://github.com/Microsoft/BashOnWindows/issues/185).

#### Fix MKL

On Windows 10 Build 14393, the Ubuntu version is 14.04 which unfortunately doesn't support MKL optimizations yet. If you upgrade to Insider Builds that are using Ubuntu 16.04, those do support MKL, but for now, we will need to uninstall MKL and reinstall regular versions of the following packages:

- NumPy
- NumExpr
- SciPy
- Scikit-Learn

Use the following command in bash:

```
conda install nomkl numpy scipy scikit-learn numexpr
conda remove mkl mkl-service
```

For more information about MKL, see [here](https://docs.continuum.io/mkl-optimizations/#uninstalling-mkl).