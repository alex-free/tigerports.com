# Setting Up Mac OS X 10.4 Tiger For Development

This document will explain configuring the following:

* Building tigerports from source.
* Replacing Tiger SSH with modern OpenSSH.
* Installing/configuring git with modern OpenSSL for github.


## Step 1: Install TigerPorts

Download the [TigerPorts tarball](http://tigerports.com#downloads). Extract it, then `cd` into the extracted directory. To build:

```sudo ./bootstrap; make; sudo make install```

```export PATH=/opt/local/bin:/opt/local/sbin:$PATH```

```sudo port selfupdate```

## Step 2: Replace Tiger SSH With MacPorts SSH

The built-in Tiger SSH does not support modern algorithms and will fail to connect to modern remote servers. It is also insecure and modern clients fail to ssh in by default. We can replace the built-in SSH with an up to date one from TigerPorts to resolve all of these issues.

Make sure the built-in SSH of Tiger is disabled. Open `System Preferences.app`, go to `Sharing`, and uncheck `Remote Login`.

Install OpenSSH, and activate the server:

```sudo port install openssh```

```sudo port load openssh```

Backup your SSH server config (optional):

```sudo cp /opt/local/etc/ssh/sshd_config /opt/local/etc/ssh/sshd_config.orig```


By default, as to not conflict with the system SSH, MacPorts/TigerPorts SSH server runs on port 2222. Everytime you would ssh into your Tiger mac you would need to do ssh <ip address> -p 2222. To edit the SSH sever config:

```sudo vi /opt/local/etc/ssh/sshd_config```

Change `Port 2222` to `Port 22`.

While your here, enable PAM for account/password based auth to work. Change `#UsePam no` to `usePam yes`.

Reload the server for changes to take effect:

```sudo port unload openssh```

```sudo port load openssh```

## Step 3: Setup Git With SSH Access

We will be using my github-ssh-setup script for this next part. To make this as easy as possible:

* Make sure PowerFox is set as your default browser and auto-logs in to your github account (optional, but allows github ssh setup to open the right page in a browser github supports to paste the ssh key it copies automatically to your clipboard).

* Make sure you have completed step 1, and especially that you have done ```export PATH=/opt/local/bin:/opt/local/sbin:$PATH``` for your terminal session so it doesn't use the built in mac os x versions of the software.

Install git:

```sudo port install git```

Run [gsshs](https://github.com/alex-free/github-ssh-enabler):

```./gsshs <github account email> <github user name>```

In the browser click create a new key, and paste it into the box.



 