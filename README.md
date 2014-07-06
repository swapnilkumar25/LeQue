Auditory Analyzer
=====================

The intention of this *web-application* is to allow the visitor to provide auditory commands, which get executed on the server.  The languages used for this project includes:

- PhP
- Python
- Bash-Script
- JavaScript
- Sass (Ruby)

**Note:** Since this project is open to the public, feel free to contribute, or email jeff1evesque@yahoo.com regarding any questions.

##Auditory Analyzer, written in Python & PhP

###Definition:

Speech recognition (SR) is the translation of spoken words into text.

- http://en.wikipedia.org/wiki/Speech_recognition

###Overview:

This project utilizes *Flash* within the web-browser in order to access the users microphone.  When a recording is saved, it is reconfigured to a *16 bit, 16 kHz, mono* wav file.  This reconfigured file, is then converted to text using CMUSphinx, specifically *PocketSphinx*, which allows our Python scripts to parse the converted text into executable commands.

##Requirement

###Development:

Since we running OSX 10.6.8 for the host Operating System, we installed VirtualBox 4.1.10 r76795 using Ubuntu 11.04 for the Guest Operating System.

The following sources allow for the above configuration:

- http://old-releases.ubuntu.com/releases/11.04
- https://www.virtualbox.org/wiki/Download_Old_Builds_4_1_pre14

###Pre-Installation:

####Internet Connection

Change the contents of the `/etc/resolv.conf` file by adding only Google's *Nameserver 8.8.8.8* to the following:

```
Generated by NetworkManager
domain guate.net.gt
search guate.net.gt
# Add google DNS (edited by JL)
nameserver 8.8.8.8
# Keep the remaining nameservers
nameserver 10.52.56.1
nameserver 10.4.24.1
nameserver 10.26.24.1
# NOTE: the libc resolver may not support more than 2 nameservers.
$ The nameservers listed below may not be recognized
nameserver 10.63.64.1
nameserver 68.238.96.12
nameserver 68.238.112.12
```

Since `/etc/resolv.conf` file is regenerated to default settings after each reboot, we need to ensure the above modifications are persistent.  This can be done by issuing the command:

```
sudo chattr +i /etc/resolv.conf
```

which changes the file attributes, by making it *immutable*.

####Source List

Since *Ubuntu 11.04* is not supported, create a backup of the  `/etc/apt/sources.list` file:

```
cp /etc/apt/sources.list /etc/apt/sources.list.backup
```

Then, change the contents of the original file to the following:

```
## EOL upgrade sources.list
# Required
deb http://old-releases.ubuntu.com/ubuntu/ natty main restricted universe multiverse
deb http://old-releases.ubuntu.com/ubuntu/ natty-updates main restricted universe multiverse
deb http://old-releases.ubuntu.com/ubuntu/ natty-security main restricted universe multiverse

# Optional
deb http://old-releases.ubuntu.com/ubuntu/ natty-backports main restricted universe multiverse
deb http://old-releases.ubuntu.com/ubuntu/ natty-proposed main restricted universe multiverse
```

###Installation:

The following packages need to be installed through terminal in Ubuntu:

```
sudo apt-get update
sudo apt-get install inotify-tools
sudo apt-get install ffmpeg
sudo apt-get install firefox
sudo apt-get install flashplugin-installer
sudo apt-get install git-core
sudo apt-get install lamp-server^ phpmyadmin
```

**Note:** Be sure to check the *README.md** file from each submodule being cloned into this project.  Additional packages may be required.

###Configuration:

####LAMP server

Recall earlier that we installed our LAMP server with phpMyAdmin.  We need to modify `apache2.conf` file in order to access phpMyAdmin:

```
sudo pico /etc/apache2/apache2.conf
```

by adding the following to the end of the file:

```
# Allow access to phpMyAdmin (edited by JL)
Include /etc/phpmyadmin/apache.conf
```

Then, restart apache:

```
sudo service apache2 restart
```

####GIT

Since we installed GIT earlier, we have to remember to configure our GIT user.  Only change the values within the double quotes (remove the quotes for the email):

```
git config --global user.email "YOUR-EMAIL@DOMAIN.COM"
git config --global user.name "YOUR-NAME"
```

Fork this project in your GitHub account, then clone your repository of this project within Ubuntu VM:

```
sudo git clone https://[YOUR-USERNAME]@github.com/[YOUR-USERNAME]/audio-analyzer.git [PROJECT-NAME]
```

Then, add the *Remote Upstream*, this way we can pull any merged pull-requests:

```
cd /var/www/audio-analyzer
git remote add upstream https://github.com/[YOUR-USERNAME]/[REPOSITORY-NAME].git
```

#####GIT Submodule

We need to initialize our git *submodules*:

```
sudo git submodule init
sudo git submodule update
```

**Note:** We have to use the *sudo* prefix, since we haven't taken care of file permissions yet.

The above two commands will update submodules.  If they are already initialized, then the latter command will suffice. Then, we need to pull the code-base into the initialized submodule directory:

```
cd /var/www/audio-analyzer
git checkout -b NEW_BRANCH master
cd [YOUR_SUBMODULE]
git checkout master
git pull
cd ..
git status
```

Now, commit and merge the submodule changes.

####File Permission

Change the file permission for the entire project by issuing the command:

```
cd /var/www
sudo chown -R jeffrey:admin audio-analyzer
```

**Note:** change 'jeffrey' to the user account YOU use.

Then, with the exception of the `.gitignore` file, ensure `/var/www/audio-analyzer/audio` is an empty directory, so that we can change it's ownership:

```
cd /var/www/audio-analyzer
sudo chown www-data:admin audio
```

####Bootable bash script

`/etc/rc.local` allows us to run bash-scripts during apache2 boot.  Since `bash_loader` loads all our required bash-scripts, we will run this script.  So, add the following lines:

```
# run 'bash_loader' at start-up for '/var/www/audio-analyzer' application (edited by JL)
cd /var/www/audio-analyzer/bash && ./bash_loader > /dev/null 2>&1 &
```

To ensure the above modification is working:

```
sudo /etc/init.d/rc.local start
```

####Local Ignore Rules

We do not want to commit files, or directories within our git *submodules*.  For this reason, we created a bash-script that will apply git *local ignore rules*, which will ignore anything contained within the specified directories.

To begin ignoring specific directories (submodules), simply open the following file:

```
/var/www/audio-analyzer/bash/git/local_ignore_rules
```

and, add respective directories to the `haystack` array.

**Note:** each repository (or submodule) has it's own `.git/info/exclude` file.

####Submodule Installation

We need to install our *Sphinx* related submodules:

```
cd /var/www/audio-analyzer/pocketsphinx/sphinxbase
./autogen.sh
sudo make install

cd ../pocketsphinx
./autogen.sh
sudo make install
```

####Automation

#####Grunt

We will automate Grunt's task management, which will encompass tools such as [Sass](https://github.com/gruntjs/grunt-contrib-sass), [Uglify](https://github.com/gruntjs/grunt-contrib-uglify), and [Modernizr](https://github.com/Modernizr/grunt-modernizr).  *Grunt* requires setup only once within a system. or web-server (depending on setup).  Each *web-application* within this system may define its own `gruntfile.js` to leverage this automation process.

More information regarding setting-up *Grunt* can be found within the [README.md](https://github.com/jeff1evesque/grunt/blob/master/README.md) file from our [*Grunt repository*](http://github.com/jeff1evesque/grunt).

## Testing / Execution

###Test Scripts:

Before we attempt to translate our own audio files, it is worthwhile to test if our application is in operation.  For this reason, we've created a test-script which can be run as follows:

```
cd /var/www/audio-analyzer/bash/tests
./test_pocketsphinx_continuous
```

The above script uses `sample.wav` file from our *PocketSphinx* submodule as input to the *PocketSphinx* translation engine.  So, be sure to initialize all submodules (as outlined in the GIT subsection).  Now, executing the script will produce a text-file containing the text translation of our `sample.wav` file:

```
/var/www/audio-analyzer/audio/recording_text/test_sample.txt
```

It will also produce a log-file:

```
/var/www/audio-analyzer/bash/logs/log_test_pocketsphinx_continuous
```
