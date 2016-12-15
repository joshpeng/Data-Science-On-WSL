# Data Science On WSL

This guide contains notes on setting up a development environment for data science in Windows 10's new ***Windows Subsystem for Linux (WSL)*** aka ***Bash on Ubuntu on Windows***.

<p align="center"><img src="https://github.com/joshpeng/Data-Science-On-WSL/raw/master/imgs/wsl.png"></p>

# Table of Contents

### [Bash and Python](#bash-and-python-1)

1. [(Optional) Install Cmder](#1-optional-install-cmder)
2. [Install WSL](#2-install-wsl)
3. [Install Anaconda](#3-install-anaconda)
   1. [Apply fix for Jupyter notebooks](#fix-jupyter-notebooks)
   2. [Apply fix for MKL](#fix-mkl)
4. [Install X client for Windows](#4-install-x-client-for-windows)
   1. [Configure X and apply fix for dbus](#configure-x-and-fix-dbus)
5. [(Optional) Install Command Line power user features](#5-optional-install-command-line-power-user-features)
   1. [Install Oh My Zsh](#install-oh-my-zsh)
   2. [Install Powerline compatible fonts](#install-powerline-compatible-fonts)
   3. [Configure Cmder startup tasks](#configure-cmder-startup-tasks)


### [Node and Web Dev](#node-and-web-dev-1)

1. [Install node](#1-install-node)
2. [Known issues/limitations](#2-known-issueslimitations)





# Bash and Python

**Prerequisite: Windows 10 Anniversary Build 14393 or later**

### 1. (Optional) Install Cmder
<p align="center"><img src="https://github.com/joshpeng/Data-Science-On-WSL/raw/master/imgs/cmder.png"></p>

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
   4. Save and restart Cmder. Now pressing <kbd>Shift</kbd>+<kbd>Up</kbd> will navigate up in your directories

### 2. Install WSL

<p align="center"><img src="http://www.monsterblog.biz/wp-content/uploads/2016/10/windows-vs-ubuntu.png" width="400"></p>

Follow Microsoft's installation guide [here](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide).
Below are additional notes on WSL you should know about.

- On the "Create a UNIX user" step, if you want root access you can create a user with ```root``` as the name. This will create your user as a superuser and will preclude you from needing to use ```sudo``` for any commands. 
- If you mess up your WSL and wish to do a clean install, use the following commands in Cmder (not Bash):
   ```
   lxrun /uninstall /full
   lxrun /install
   ```

- File permissions are handled separately by both Windows and Linux. See [here](https://msdn.microsoft.com/en-us/commandline/wsl/user_support) for more information. You may want to run Cmder as administrator. If you want to always run Cmder as admin, you can do so by right-clicking its .exe file and selecting the option in its properties.
- When in Windows, you can find the Linux file system at ```%AppData%\Local\lxss```. When in Bash, you can find the Windows file system at ```/mnt/c```.
  - Despite this, file interoperability is not supported. Linux files have information stored in their NTFS Extended Attributes that Windows can't create. If you try to edit them in Windows, those attributes might get stripped and become unusable in Linux also. **Very important read** on what you can and cannot do [here](https://blogs.msdn.microsoft.com/commandline/2016/11/17/do-not-change-linux-files-using-windows-apps-and-tools/).

### 3. Install Anaconda

<p align="center"><img src="https://www.continuum.io/sites/all/themes/continuum/assets/images/logos/logo-horizontal-large.svg" width="500"></p>


Anaconda is a widely used Python-based data science platform. It comes with a large selection of preinstalled packages as well as features to help you manage environments for multiple Python versions. Downloads are located at [https://www.continuum.io/downloads](https://www.continuum.io/downloads).

1. Open Cmder and type ```bash```
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

In Windows 10 Build 14393, there is an issue with libzmq that is apparently fixed in later Insider Builds. For now though, we will need to do the following command in bash:

```
conda install -c jzuhone zeromq=4.1.dev0
```

This patches zeromq to work with WSL. If this step is not done, the kernel will keep dying whenever you view a Jupyter notebook resulting in an inability to execute code cells. For more information about this issue, see [here](https://github.com/Microsoft/BashOnWindows/issues/185).

#### Fix MKL

In Windows 10 Build 14393, the Ubuntu version is 14.04 which unfortunately doesn't support [MKL](https://software.intel.com/en-us/intel-mkl) optimizations yet. If you upgrade to Insider Builds that are using Ubuntu 16.04, those do support MKL, but for now, we will need to uninstall MKL and reinstall regular versions of the following packages:

- NumPy
- NumExpr
- SciPy
- Scikit-Learn

Use the following command in bash:

```
conda install nomkl numpy scipy scikit-learn numexpr
conda remove mkl mkl-service
```

To test if this is working:

```
python
import scipy
scipy.test()
```

It should no longer generate the following errors:

```
OMP: Error \#100: Fatal system error detected.
OMP: System error \#22: Invalid argument
```

For more information about MKL, see [here](https://docs.continuum.io/mkl-optimizations/#uninstalling-mkl).

### 4. Install X client for Windows 
<p align="center"><img src="https://chocolatey.org/content/packageimages/vcxsrv.1.18.3.0.png"></p>

Linux uses [X Window System](https://en.wikipedia.org/wiki/X_Window_System) which uses a server-client model to display GUI applications. In order for us to view these applications on Windows, we will need a X client capable of receiving the communication coming from our Linux's X server.

There are two main options we can choose from: [VcXsrv](https://sourceforge.net/projects/vcxsrv/) and [Xming](https://sourceforge.net/projects/xming/). Choose whichever one you like. If you run into issues displaying applications in one, try the other. We will use VcXsrv in this guide since some users are reporting Xming to be a bit slower.

1. Download [VcXsrv](https://sourceforge.net/projects/vcxsrv/)
2. Install and run VcXsrv
3. (Optional) Add VcXsrv as a service so you don't have to manually start it every time

#### Configure X and fix dbus

We will need to configure Linux to send X communication to where our Windows' X client is listening at.

1. Open Bash
2. Type the following command:

   ```
   echo "export DISPLAY=localhost:0.0" >> ~/.bashrc
   ```
   This is enough to launch X applications to our X client, but many applications will still perform poorly or crash. This happens because Windows 10 Build 14393 does not fully support Unix sockets yet. Later Insider Builds are said to have this fixed already as well.
3. Type the following to change dbus into using TCP instead of Unix sockets:

   ```
   sudo sed -i 's$<listen>.*</listen>$<listen>tcp:host=localhost,port=0</listen>$' /etc/dbus-1/session.conf
   ```
4. To test if this is working we can install and run [Sublime Text](https://www.sublimetext.com/) from Linux. You may need to restart your bash if it doesn't work right away.

   ```
   sudo add-apt-repository ppa:webupd8team/sublime-text-3
   sudo apt-get update
   sudo apt-get install sublime-text-installer
   subl
   ```

### 5. (Optional) Install Command Line power user features

Bash is great, but there are quite a few annoying nuances to it like tab completion being case sensitive. To improve our productivity we can upgrade our CLI away from Bash to Zsh. To learn about some of the great Zsh features see [this blog post](http://code.joejag.com/2014/why-zsh.html). Those can then be further improved by using [Oh My Zsh](http://ohmyz.sh/).

#### Install Oh My Zsh

<p align="center"><img src="http://ohmyz.sh/img/OMZLogo_BnW.png"></p>

1. Open Bash and install the prerequisites for Oh My Zsh with the following commands
   ```
   sudo apt-get update
   sudo apt-get install zsh
   sudo apt-get install git
   ```

2. Install Oh My Zsh with curl

   ```
   sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
   ```

   If you prefer wget you can use this command:

   ```
   sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
   ```

3. (Optional) Configure your Bash to use Zsh as the default shell on startup

   1. Open ```.bashrc``` in your choice of editor

      ```
      vi ~/.bashrc
      ```

   2. Add this as a new line at the very bottom

      ```
      zsh
      ```

      Note: In the future if you end up further modifying your ```.bashrc```, you will want to check that the ```zsh``` line remains at the bottom. This ensures your full ```.bashrc``` file is processed before starting Zsh.

   3. Restart Bash and it should automatically start Zsh

4. (Optional) Configure your Oh My Zsh

   1. Open ```.zshrc``` in your choice of editor

      ```
      vi ~/.zshrc
      ```

   2. To change the Zsh theme, change the ```ZSH_THEME``` line. I recommend the "agnoster" theme. More themes can be found [here](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes).

      ```
      ZSH_THEME="agnoster"
      ```

   3. To add additional plugins, scroll down to the ```plugins=(git)``` line. This line is space-delimited so you can add whichever plugins you want inside the parenthesis. The list of default plugins can be found [here](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins).

      ```
      plugins=(git python sudo)
      ```

#### Install Powerline compatible fonts

Many Zsh themes use custom characters to display symbols like git branches. To have these displayed properly in Cmder you will need to use a Powerline compatible font.

1. Choose a font from this [GitHub repo](https://github.com/powerline/fonts). Font samples can be seen [here](https://github.com/powerline/fonts/blob/master/samples/All.md). I recommend "DejaVu Sans Mono".
2. Download and install the font's .ttf files into Windows
3. Open Cmder
4. Press <kbd>Win</kbd>+<kbd>Alt</kbd>+<kbd>P</kbd> to bring up the ```Settings``` screen
5. Change ```Main console font``` to your Powerline font.
6. Press ```Save settings```

#### Configure Cmder startup tasks

1. From Cmder's ```Settings``` screen, go to the Startup --> Tasks tab

2. Press ```+```

3. Name the group ```Bash on Ubuntu```

4. (Optional) Check the box ```Default task for new console```

5. Fill in the following for ```Task parameters```:

   ```
   -icon %USERPROFILE%\AppData\Local\lxss\bash.ico"
   ```

6. Fill in the following for the task command:

   ```
   %windir%\system32\bash.exe
   ```

7. (Optional) Go to Startup tab and change ```Specified named task``` to ```{Bash on Ubuntu}```



# Node and Web Dev

### 1. Install node
<p align="center"><img src="https://nodejs.org/static/images/logos/nodejs-new-pantone-black.png" width="300"></p>

With how dominant node.js is in web development these days, it is important to get this working on WSL. We can use [node version manager (nvm)](https://github.com/creationix/nvm) to handle differing version requirements we may have too.

1. Open Bash and install the prerequisites for nvm with the following commands

   ```
   sudo apt-get update
   sudo apt-get install build-essential
   sudo apt-get install libssl-dev
   ```

2. Install nvm with curl

   ```
   curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.32.1/install.sh | bash
   ```

   If you prefer wget you can use this command:

   ```
   wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.32.1/install.sh | bash
   ```

3. Install and use latest node LTS

   ```
   nvm install --lts
   nvm use --lts
   ```

### 2. Known Issues/Limitations

Not everything in the web development world works properly with WSL yet. The following is an incomplete list of some items I've personally ran into.

| Item                                   | Version | Issue                                    |
| -------------------------------------- | ------- | ---------------------------------------- |
| [Meteor](https://www.meteor.com/)      | 1.4.2.3 | Unable to establish connections to MongoDB when starting projects |
| [docpress](http://docpress.github.io/) | 0.7.1   | Unable to build static site. Issue discussion found [here](https://github.com/docpress/docpress/issues/169#issuecomment-257766560). |