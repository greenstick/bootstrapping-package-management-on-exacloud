# Bootstrapping Package Managers on ExaCloud

Unless you're a Superuser, setting up a development environmenton ExaCloud requires a bit of massaging. Below is a set of instructions to install [Miniconda](http://conda.pydata.org/miniconda.html) and [Linuxbrew](http://linuxbrew.sh) to manage your Python and system level software installations, respectively. This means, if you need to install most any software package, all you'll need to do is type into your terminal:
```
brew install PACKAGE
```

Pretty nifty, eh?

### Step 1
First, log on to ExaCloud, then start an interactive session (not using the head node) and setup a packages directory:
```
condor_submit --interactive
cd ~/ && mkdir packages
```

### Step 2
Second, download and install Miniconda (x64, Python 2 version):
```
wget https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh
bash Miniconda2-latest-Linux-x86_64.sh
```

The first command downloads an installation script from the [Continuum Analytics website](http://conda.pydata.org/miniconda.html), while the second step runs the downloaded bash script. The bash script will have you agree to a license ('yes') and then setup the root directory for Miniconda, which should be `/home/users/USERNAME/packages/miniconda` (where packages is the directory we created in the first step). Note that I removed the 2 from the default Miniconda installation path (`/home/users/USERNAME/miniconda2`), this is optional but something to be mindful of in the following steps if you opt not to do this. Finally, Miniconda will ask whether you would like to include Miniconda in your .bash_profile, be sure to say 'yes' here (the default is [no]).

### Step 3
Third, we need to install updated compilers, an updated Git, & cURL:
```
conda install gcc
conda install libgcc
conda install libgfortran
conda install git
conda install curl
```

We now have updated compilers, Git, and cURL. We need Ruby to install Linuxbrew too, but ExaCloud has an installation that appears to work for this purpose. If you'd like to install your own update version later, you can always do this using Linuxbrew and the command `brew install ruby`.

### Step 4
Fourth, we need to update our `.bash_profile` to make sure it knows where to find the new compilers:
```
echo 'export CC="$HOME/packages/miniconda/bin/gcc"' >>~/.bash_profile
echo 'export CXX="$HOME/packages/miniconda/bin/g++"' >>~/.bash_profile
echo 'export OBJC="$HOME/packages/miniconda/bin/gcc"' >>~/.bash_profile
echo 'export OBJCXX="$HOME/packages/miniconda/bin/g++"' >>~/.bash_profile
```
Note: *If you used an alternate path for your miniconda installation, be sure to update the paths above to reflect that.*

Exit the interactive session and log off of ExaCloud. Log back on, and the changes to your `.bash_profile` will take effect.

Again, start a new interactive session on a child node:
```
condor_submit --interactive
```

### Step 5
Fifth, install Linuxbrew. Copy the command below to install Linuxbrew:
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install)"
```

### Step 6
Sixth, edit your $PATH variable in your .bash_profile to point to your Linuxbrew installation:
```
echo 'export PATH="$HOME/.linuxbrew/bin:$PATH"' >>~/.bash_profile
```

Again, exit the interactive session and log off of ExaCloud. Log back on, and the changes to your `.bash_profile` will take effect.

### Step 7
Finally, check that Linuxbrew installed correctly:
```
brew update
brew doctor
```
Note: *When working with Linuxbrew, it's always a good idea to run the commands above before you install anything (i.e. any day you log on to ExaCloud and decide to install or update your packages). This ensures that you won't install packages that break or fail to install due to an incorrect Linuxbrew configuration or package dependency. If you do see any new Warning messages, its a good idea to copy them each into Google and make sure that none will cause you any trouble.*

You may get some warnings. Most of these warnings shouldn't cause you (or your installed packages!) any trouble. The ones I get are:
```
Warning: Setting LD_* vars can break dynamic linking.
Set variables:
  LD_LIBRARY_PATH: /usr/local/lib

Warning: "config" scripts exist outside your system or Homebrew directories.
`./configure` scripts often look for *-config scripts to determine if
software packages are installed, and what additional flags to use when
compiling and linking.

Having additional scripts in your path can confuse software installed via
Homebrew if the config script overrides a system or Homebrew provided
script of the same name. We found the following "config" scripts:
  /home/users/cordier/packages/miniconda/bin/python-config
  /opt/ganglia/bin/ganglia-config
  /opt/rocks/bin/python-config
  /opt/rocks/bin/xml2-config
  /opt/rocks/bin/python2.6-config
```

If you got any other warnings, check Stackoverflow to make sure they're not important and, if you're feeling ambitious, follow the steps to clear them. If having these warnings drives you crazy, look to Step 8 for a known way to clear some of them. Note that some warnings cannot be cleared as they require file and directory privileges not available to non-administators on ExaCloud, specifically:
```
Having additional scripts in your path can confuse software installed via
Homebrew if the config script overrides a system or Homebrew provided
script of the same name. We found the following "config" scripts:
  /opt/ganglia/bin/ganglia-config
  /opt/rocks/bin/python-config
  /opt/rocks/bin/xml2-config
  /opt/rocks/bin/python2.6-config
```

### Step 8 (Optional)
Strictly speaking, once you have Linuxbrew you don't need Miniconda you can delete it. That said, I've found certain dependencies available on Miniconda that aren't explicitly available on Linuxbrew and it's great for managing virtual environments and installing Python packages (replaces `pip`):
```
conda install numpy
conda install scipy
conda install scikit-learn
# ...etc.
```

If you'd rather just use Linuxbrew (and pip for Python package management) you can install new compilers, Git, and cURL with Linuxbrew and delete Miniconda:
```
brew install gcc
brew install git
brew install curl
cd ~/packages && rm -rf miniconda
```
Note: *Again, if your miniconda installation is somewhere else, be sure to `cd` to where you installed it and remove the directory from there.*

#### Reap The Rewards!
Finally, some packages that may be of interest to install (because that's what we're really after, right?)
```
# R Language
brew install science/r
# Python 3.5
brew install python3
# Java 1.8
brew install jdk
# Java 1.7
brew install jdk7
# Ruby
brew install ruby
```

Hope this has been helpful! If you run into any issues or have questions, feel free to send a pull request and I'll try and address them & hammer out this guide to make the commands more robust. 

Cheers!
