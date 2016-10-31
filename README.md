# Bootstrapping Package Managers on ExaCloud

Unless you're a Superuser, setting up a development environmenton ExaCloud requires a bit of massaging. Below is a set of instructions to install [Miniconda](http://conda.pydata.org/miniconda.html) and [Linuxbrew](http://linuxbrew.sh) to manage your Python and system level software installations, respectively. This means, if you need to install most any software package, all you'll need to do is type into your terminal:
```
brew install PACKAGE
```

Pretty nifty, eh?

And as an added bonus, if you've ever had errors compiling packages on ExaCloud due to unsupported (old) compilers, the setup has been designed to resolve these issues as well.

### Step 1
First, log on to ExaCloud, then start an interactive session (i.e. don't use the head node) and setup a `packages` directory:
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

The first command downloads an installation script from the [Continuum Analytics website](http://conda.pydata.org/miniconda.html), while the second step runs the downloaded bash script. The bash script will have you agree to a license ('yes') and then setup the root directory for Miniconda, which should be `/home/users/USERNAME/packages/miniconda` (where packages is the directory we created in the first step). Note that I removed the `2` from the default Miniconda installation path (`/home/users/USERNAME/miniconda2`), this is optional but something to be mindful of in the following steps if you opt not to do this. Finally, Miniconda will ask whether you would like to include Miniconda in your `~/.bash_profile`, be sure to say `yes` here (the default is `[no]`).

### Step 3
Third, we need to install updated compilers, an updated Git, & cURL:
```
conda install gcc
conda install libgcc
conda install libgfortran
conda install git
conda install curl
```

We now have updated compilers, Git, and cURL. We need Ruby to install Linuxbrew too, but ExaCloud has an installation that appears to work for this purpose. If you'd like to install your own updated version later, you can always do this using Linuxbrew and the command `brew install ruby`.

### Step 4
Fourth, we need to update the build references in our `~/.bash_profile` to use the new compilers:
```
echo 'export CC="$HOME/packages/miniconda/bin/gcc"' >>~/.bash_profile
echo 'export CXX="$HOME/packages/miniconda/bin/g++"' >>~/.bash_profile
echo 'export OBJC="$HOME/packages/miniconda/bin/gcc"' >>~/.bash_profile
echo 'export OBJCXX="$HOME/packages/miniconda/bin/g++"' >>~/.bash_profile
```
Note: *If you used an alternate path for your miniconda installation, be sure to update the paths above to reflect that.*

Exit the interactive session and log off of ExaCloud. Log back on, and the changes to your `~/.bash_profile` will take effect.

Again, start a new interactive session on a child node:
```
condor_submit --interactive
```

### Step 5
Fifth, copy the command below to install Linuxbrew:
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install)"
```

### Step 6
Sixth, edit your $PATH variable in your `~/.bash_profile` to point to your Linuxbrew installation:
```
echo 'export PATH="$HOME/.linuxbrew/bin:$PATH"' >>~/.bash_profile
echo 'export PATH="$HOME/.linuxbrew/sbin:$PATH"' >>~/.bash_profile
```

Again, exit the interactive session and log off of ExaCloud. Log back on, and the changes to your `~/.bash_profile` will take effect.

### Step 7
Finally, check that Linuxbrew installed correctly:
```
brew doctor
```
Note: *When working with Linuxbrew, it's always a good idea to run `brew doctor` before you install anything (i.e. any day you log on to ExaCloud and decide to install or update your packages). This ensures that you won't install packages that break or fail to install due to an incorrect Linuxbrew configuration or package dependency. If you do see any new Warning messages, its a good idea to copy them each into a search engine and make sure that none will cause you any trouble.*

You may get some warnings. Most of these warnings shouldn't cause you (or your installed packages!) any trouble. The ones I get are (you may see a few more, to understand why see Step 8):
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

If you got any other warnings, check Stackoverflow to make sure they're not important and, if you're feeling ambitious, follow the steps to clear them – but be careful; if you're not starting fresh you could upset software that you already have installed. I should also note that some warnings can't and shouldn't be cleared as they require file and directory privileges not available to non-administators on ExaCloud. For example:
```
Having additional scripts in your path can confuse software installed via
Homebrew if the config script overrides a system or Homebrew provided
script of the same name. We found the following "config" scripts:
  /opt/ganglia/bin/ganglia-config
  /opt/rocks/bin/python-config
  /opt/rocks/bin/xml2-config
  /opt/rocks/bin/python2.6-config
```

### Step 8 (Optional Cleanup)
Strictly speaking, once you have Linuxbrew you don't need Miniconda and can delete it. That said, I've found certain dependencies available on Miniconda that aren't explicitly available on Linuxbrew. Of course, it's also great for managing virtual environments and installing Python packages (i.e. it works in place of `pip`):
```
conda install numpy
conda install scipy
conda install scikit-learn
# ...etc.
```

To just use Linuxbrew (and pip for Python package management), you'll need to install new compilers, Git, and (optionally) cURL with Linuxbrew, then delete Miniconda:
```
brew install gcc
brew install git
brew install curl
~/packages && rm -rf miniconda
```

Next, you should modify the Miniconda build references we exported in Step 4 from your `~/.bash_profile`. To do this, use vim, emacs, nano, or your preferred CLI-based editor. The lines in your `~/.bash_profile` should look like this:
```
export CC="$HOME/packages/miniconda/bin/gcc"
export CXX="$HOME/packages/miniconda/bin/g++"
export OBJC="$HOME/packages/miniconda/bin/gcc"
export OBJCXX="$HOME/packages/miniconda/bin/g++"
```
Note: *Again, if your Miniconda installation is somewhere else, these paths may look different (but you probably already know that).*

They should be changed to:
```
export CC="$HOME/.linuxbrew/bin/gcc"
export CXX="$HOME/.linuxbrew/bin/g++"
export OBJC="$HOME/.linuxbrew/bin/gcc"
export OBJCXX="$HOME/.linuxbrew/bin/g++"
```

A few notes on Step 8: 
* The `.` in of the folder name .linuxbrew means the folder is a system folder. Folders following this naming convention are generally hidden from the user. That said, you can still `cd` into them (e.g. to access the location of your compilers, you can always type `cd ~/.linuxbrew/bin`)
* It may not be strictly required to set the build variables above unless you're compiling your own source code (i.e. in C/C++, etc.), but it's nonetheless a good idea to set them up so you won't encounter any future problems should the occasion arise. 

#### Reap The Rewards!
Finally, some packages that may be of interest to install (because that's what we're really after, right?)
```
# R Language
brew install science/r
# Python 2.7.x
brew install python
# Python 3.5.x
brew install python3
# Java 1.8
brew install jdk
# Java 1.7
brew install jdk7
# Ruby
brew install ruby
Node.js
brew install node
```

Not sure which installation recipe you need? The command below has you covered:
```
brew search PACKAGE
```

Hope this has been helpful! If you run into any issues or have questions, please create an [Issue](https://github.com/greenstick/bootstrapping-package-management-on-exacloud/issues/new) and I'll try and address them promptly – the aim is to make this guide as straightforwad and easy as possible. 

Cheers!
