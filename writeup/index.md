[TigerPorts](http://tigerports.com) -> Writeup

## LAST UPDATED 10/9/2026

This document aims to describe how I run a replacement for macports.org. This could be cleaned up (alot), but the important pieces (that you won't find documented anywhere else on the internet if you for some reason also wanted to do this) are here.

## MacPorts-Base

The goal of forking macports base is to point it to tigerports.com, and restore tiger compatibility.

### Branch Setup

* Master branch is where development occurs. The only people that should use this are the actual base developers as it has expiremental features on by default, and the ports tree identifies this as `<current version>.99`, i.e. the `macports-base/config/macports_version` file right now as of writing this has `12.6.99` so that the ports tree is considerate of the current master version's expiermental features/setup. This can still break any time though and you can't realistically use this without that exact version number (which I can not do). It is a MUCH better expierence to instead focus the release branch below:

* release-x.xx branch. This is where they have the actual release code for the versions. Right now we are at v2.12.6, so the current branch is release-2.12. There is a release-2.11, release-2.10, etc. But how they do this is at a certain point in the master branch (usually months) they move over commits into the release-2.12 branch to freeze it, then occasionally selectively cherry pick commits from current master (just importanting single commit bug fixes) into release-2.12. The import thing to know is release-2.12 at a certain commit IS what a release tag like v2.12.6 is based on, NOT master.

So what I did was with base is make a tigerports-release-2.12 branch that tracks release-2.12. Then at the very commit that 2.12.6 was created, added the tiger specific patches and tigerports.com rsync/http server edits, and then create my own tag on that branch called v2.12.6.001. This desitinctly describes that is a tigerports release (they have never had x.xx.x.xxx version scheme, I'm adding another decimal here to show it based on 2.12.6 but at a certain release for my setup).

See my fork's [tags](https://github.com/alex-free/macports-base/tags).

### Merges With Upstream

```
git clone -b tigerports-release-2.12 git@github.com:alex-free/macports-base

cd macports-base

git remote add upstream https://github.com/macports/macports-base

git fetch upstream

git merge --no-gpg-sign v2.12.6

# take upstream ChangeLog, tigerports has it's own at tigerports-changelog.md)
git checkout --theirs ChangeLog
git add ChangeLog

# take upstream manual pages, will be regenerated)
git checkout --theirs doc
git add doc

# Take upstream configure, it's a generated file. only worry about conflicts in configure.ac
git checkout --theirs configure
git add configure

# Resolve other conflicts

git push -u origin tigerports-release-2.12
```

### Restoring Tiger Compatibility

Tiger incompatibility has mostly been due to (even more so) outdated userland then Leopard, and upstream removing either workarounds for these versions and or conditional code for 10.4.

The current [diff](https://github.com/macports/macports-base/compare/release-2.12...alex-free:macports-base:tigerports-release-2.12).

[@barracuda156](https://github.com/barracuda156) and contributors have credit for identifying many things upstream dropped, and adding new additions to [macos/ppcports-base tiger_upd branch](https://github.com/macos-powerpc/ppcports-base/tree/tiger_upd).

I have updated some additional things, dropped some obsolete ones with the new setup, etc but it currently is _way_ too much to go in depth specifically for each one. One big thing though that I want to explain is the new bootstrap system.

### bootstrap_configure.sh

Since base can't even link with the libcurl tiger shipped anymore, we can solve 2 issues at once with a special [bootstrap_configure.sh](https://github.com/alex-free/macports-base/blob/6d7c92ee805ff11196aceb59ce62d06e99720a5c/bootstrap_configure.sh) script I have created:

* Builds newer openssl and curl (all mac os versions, though tested up to 10.12). Base is then set to use this as the 'system curl', which is configured with modern certificates and TLS v1.2 capable so that distfiles (source tarballs for ports, binary archives for tiger are handled with http via tigerports.com) can be downloaded.

* On Mac OS 10.4 Tiger only, it builds the same GNUMake 3.81 with Apple patches from 10.5.8 open source apple releases and configures it as the system make. This makes Tiger for the first time the most capable 

* On Mac OS 10.4 Tiger only, a newer gzip, tar, and libarchive are built and configured as the system versions. These are the same versions shipped on Leopard, so that I don't have to accomadate bugs/limitations in the the tiger system archival versions.

The method to build with bootstrap_configure.sh is as follows:

```
./bootstrap_configure.sh && make && sudo make install
```

As bootstrap_configure.sh _runs_ ./configure with args to configure replacement system utilities. You can even specify a custom prefix with --prefix= argument to ./bootstrap_configure.sh. On 10.5 and up however, you can still run:

```
./configure && make && sudo make install
```

To not use the bootstrap (though this would severly reduce what very old Mac OS X versions can download).

### Changes To Make MacPorts-Base Work With Tigerports.com

In Makefile:

`DISTNAME = MacPorts-${DISTVER}` -> `DISTNAME = TigerPorts-${DISTVER}`

`$(INSTALL) -o "${DSTUSR}" -g "${DSTGRP}" -m 444 $(srcdir)/macports-pubkey.pem "$(DESTDIR)${datadir}/macports/"` ->  `$(INSTALL) -o "${DSTUSR}" -g "${DSTGRP}" -m 444 $(srcdir)/tigerports-pubkey.pem "$(DESTDIR)${datadir}/macports/"`

In Mk/macports.autoconf.mk.in:

```
# Need Linux binaries to load after install from install location rather then build dir.
ifeq (@OS_PLATFORM@,darwin)
SQLITE3_LIBS		= @TCL_BIN_DIR@/pkgs/@SQLITE3_LIBNAME@/lib@SQLITE3_LIBNAME@$(SHLIB_SUFFIX)
else
SQLITE3_LIBS		= -L@abs_top_builddir@/@VENDOR_DESTROOT@@TCL_PREFIX@/lib/@SQLITE3_LIBNAME@ -Wl,-rpath,@TCL_PREFIX@/lib/@SQLITE3_LIBNAME@ -l@SQLITE3_LIBNAME@
endif
```

change to make sqlite runpath correct when building base on Linux (which is the OS the server uses for tigerports.com).

In doc/macports.conf.in:


Changed master rsync_server:

```
#rsync_server        	rsync.macports.org
rsync_server        	tigerports.com
```

In doc/sources.conf:

Changes ports rsync configuration:

```
#rsync://rsync.macports.org/macports/release/tarballs/ports.tar.gz [default]
#rsync://tigerports.com/macports/release/tarballs/ports.tar.gz [default]
# Faster after initial port sync for updates after IMHO to use rsync directory.
rsync://tigerports.com/macports/release/ports/ [default]
```

In portmgr/rsync.repos-tigerports:

Change rsync modules path, this file is actually to be used by the tigerports.com server.

```
--- portmgr/rsync.repos	2026-10-09 16:50:40.000000000 -0600
+++ portmgr/rsync.repos-tigerports	2026-10-09 17:39:01.000000000 -0600
@@ -61,18 +61,20 @@
 #######
 
 
+[macports]
+    comment = TigerPorts Infrastructure Base Namespace
+    path = /var/www/html/macports
+    read only = yes
+    list = yes
+
 [release]
     comment = MacPorts rsync release repository, providing the current MacPorts "released" sources and the ports tree.
-    path = /Volumes/data/rsync/macports/release
+    path = /var/www/html/macports/release
 
 [trunk]
     comment = MacPorts rsync "trunk" repository, providing experimental MacPorts sources in svn's trunk/base.
-    path = /Volumes/data/rsync/macports/trunk
-
-[dpupdate]
-    comment = Legacy MacPorts rsync repository for backwards compatibility with 1.4.x releases, providing svn's trunk/base and the ports tree.
-    path = /Volumes/data/rsync/macports/dpupdate
+    path = /var/www/html/macports/trunk
 
 [dpupdate1]
     comment = Legacy MacPorts rsync repository for backwards compatibility with 1.4.x releases, providing the current MacPorts "released" sources.
-    path = /Volumes/data/rsync/macports/dpupdate1
+    path = /var/www/html/macports/dpupdate1
MacBook-Air:macports-base alex$ 
```

In src/macports1.0/selfupdate.tcl:

Change release_version_urls to "https://raw.githubusercontent.com/alex-free/tigerports-base/master/config/RELEASE_URL" and "http://tigerports.com/macports/distfiles/MacPorts/RELEASE_URL" (this is the urls used to download RELEASE_URL, which is used by selfupdate to discover what the newest version of base is).

Changed release_urls to "https://github.com/alex-free/tigerports-base/releases/download/v{version}/TigerPorts-{version}.tar.bz2" and "http://tigerports.com/macports/distfiles/MacPorts/TigerPorts-{version}.tar.bz2" (this is the urls used to download a new base tarball).

Selfupdate has been changed to include bootstrap configure args depending on version.

### Creating Keys

MacPorts has moved to mandating signify for port binary archives, and has 3 different keys for ports tree, port archives, and base (not mandated by base yet, it can fallback to ripemd160).

```
cd keys/base && signify -G -p tigerports-base-2027.pub -s tigerports-base-2027.sec

cd keys/ports && signify -G -p tigerports-ports-2027.pub -s tigerports-ports-2027.sec

cd keys/archives && signify -G -p tigerports-archives-2027.pub -s tigerports-archives-2027.sec
```

These are alternatives to RIPEMD160: https://trac.macports.org/ticket/63885

For RIPEMD160 (being phased out, but I still use it for tigerports-pubkey.pem for portindex/ports tree):

```
	# The macports-base/doc/pubkey.in file says this.
	openssl genrsa -des3 -out tigerports-privkey.pem 2048
	# No -des3 to avoid entering passphrase.
	#openssl genrsa -out tigerports-privkey.pem 2048
	openssl rsa -in tigerports-privkey.pem -pubout -out tigerports-pubkey.pem
```

NO macports keys are used for tigerports.com, because they don't have any darwin 8 portindexes or binary archives so there is no point.

### Generating Release Tarballs

I currently do not implement binary packages, but I do release source tarballs signed with my keys. This is done by the [uver.sh](https://github.com/alex-free/macports-base/blob/tigerports-release-2.12/uver.sh) script, based on documentation in macports-base as well as figuring out stuff by trial and error.

## Macports Infrastucture - How To Run Your Own Macports.org

MacPorts Repo: https://github.com/macports/macports-infrastructure 
TigerPorts Repo: https://github.com/alex-free/macports-infrastructure/tree/tigerports

Purpose: [Scripts and Tools](https://github.com/alex-free/macports-infrastructure/tree/tigerports/jobs) to manage hosting the macports.org script. This is where [build bots live](https://github.com/alex-free/macports-infrastructure/tree/tigerports/buildbot) (though I don't use those for anything fun yet, more on that later). Key in operation however is the [mprsyncup](https://github.com/alex-free/macports-infrastructure/blob/tigerports/jobs/mprsyncup) script.

### MPRSYNCUP: How MacPorts Designed It 

1) Set some variables for use when compiling macports-base. This expects some ports to already be installed (like git, rsync, and some more), so _technically_ you'd want a working macports system before ever running mprsyncup for the first time. This is also where important locations for rsync, and an http server are specified, as the whole point of this script is to generate what is hosted by i.e. macports.org.

2) If it has never been ran before, it does an intial clone of `https://github.com/macports/macports-base` and `https://github.com/macports/macports-ports`.

3) It then compiles base, and installs into the specified place from previously set variables.

4) Immedietly after installing base, it will install a `RELEASE_URL` file available from base sources saying what the current version number/tag is for selfupdate (more on this later). However at this point of successful build by mprsyncup, as long as the rsync server is serving the right things (essentially a clean copy of base that is installed elsewhere) it can be used by an existing base on an end users machine for selfupdate by rsync.

5) Now, it will start portindexing that ports tree it cloned earlier for **every** supported system. This is decided in a var and can be changed (foreshadowing).

6) Once the portindexes are finished, they are installed to be available in rsync of the ports tree. That's right, the server hosts the ports tree and base via rsync in this way.

### Setting Up An Enviorment For MPRSYNCUP The MacPorts Way On Linux Is Confusing (But Makes Sense On Mac OS To The Surprise Of No One)

So immedietly I faced a problem. Those vars specifiying 'port versions' of i.e. git, gmv (gnu coreutils mv), etc. didn't make any sense on a Debian Linux VPS, which is what I use to currently host tigerports.com. Now you may be confused, why would I do this/how could it even work? Well MacPorts is so awesome that Linux is actually one of the many OSes it is designed to technically run on. Even the ports tree currently has some things in it specific to non-darwin operating systems. The issue is Mac OS is _much, much more tested_. So for example, I had to do that base [change on shared library paths](https://github.com/alex-free/macports-base/blob/e34855f23855c0697f800a7786b1ea3c80716c42/Mk/macports.autoconf.mk.in#L31) for my debian linux. But besides that, there was another problem. The ports tree isn't all that tested, so it was a gamble if all the ports mprsyncup wanted would actually build and be available. This is how macports _seems_ to want you to build base:

1) You build macports-base (`./configure && make && sudo make install`). Though on Linux you would want to _at least_ install some tools from your distro package manager to just get the same deps a Mac OS shipps with (`sudo apt install --yes mtree-netbsd zlib1g-dev libcurl4-openssl-dev curl`) because base itself depends on these. Then, you need to pass some envars to configure to get it to work at all (Note I did not trim this enough as most is stolen from mprsyncup as I gave up doing it this way, see more soon on why):

```
# Paths we'll work on:
ROOT=/var/tmp/macports
PREFIX=${ROOT}/opt/local
GITROOT=/var/tmp/macports
TBASE=${GITROOT}/trunk/base
RBASE=${GITROOT}/release/base
PORTS=${GITROOT}/release/ports
CONTRIB=${GITROOT}/release/contrib
# RSYNCROOT=/rsync/macports
RSYNCROOT=/var/www/html/macports
#MPTOOLSPREFIX=/opt/local
#MPBIN=${MPTOOLSPREFIX}/bin

# Commands we need. For options to be substituted correctly, these must
# not be substituted within double quotes. Thus, there must not be any
# globbing characters, and the command itself must not contain spaces.

# GIT="${MPBIN}/git"
GIT="/usr/bin/git"

# RSYNC="${MPBIN}/rsync -q"
RSYNC="/usr/bin/rsync -q"
RM="/bin/rm"
MKDIR="/bin/mkdir"
#MV="${MPBIN}/gmv"
MV="/bin/mv"
LN="/bin/ln"
TAR="/bin/tar"
OPENSSL="/usr/bin/openssl"
AWK="/usr/bin/awk"
#STAT="${MPBIN}/gstat"
STAT="/usr/bin/stat"
#GZIP="${MPBIN}/gzip"
GZIP="/bin/gzip"
BASENAME="/usr/bin/basename"
TCLSH="${PREFIX}/bin/port-tclsh"
PORTINDEX="${PREFIX}/bin/portindex"
PORTINDEX2JSON="${CONTRIB}/portindex2json/portindex2json.tcl"

make distclean
./autogen.sh
./configure \
            --prefix="${PREFIX}" \
            --with-install-group="$(id -gn)" \
            --with-install-user="$(id -un)"
make 
make install
```

2) You install a handful of ports (`sudo port -d -N install git rsync coreutils gzip` is my best guess from the script).

3) You can run mprsyncup at this point (Mac OS has some version of everything you need ).

4) Then you should be able to just run mprsyncup and mirror macports.

### Changing MPRSYNCUP To Make Sense On Linux

I decided to write my own mprsyncup ([mprsyncup-tigerports](https://github.com/alex-free/macports-infrastructure/blob/tigerports/jobs/mprsyncup-tigerports)) for the following reasons:

* I don't like this checken and egg thing with building base, installing ports (requiring the ports tree to work on linux well enough for a few ports at least, and needing linux portindexes) and only then being able to run mprsyncup successfully.

* Since mprsyncup already builds base, we can hijack that. We already need to install a few things on a barebones debian system to build base (`mtree-netbsd zlib1g-dev libcurl4-openssl-dev curl`), so lets just add `git` to that and we are linux so obviously coreutils are already available (just named without the 'g' prefix, i.e. 'gmv' is just 'mv' in linux land as that's a macports port thing). Same with gzip, we already have that in a base Debian install of Linux. So lets just _change those vars_ to the ones on our system, that either came with it or were installed by `apt`.

Great, now mprsyncup is sufficently happy. But I am not done. Something else, is I don't understand still why `RSYNCROOT=/rsync/macports` when the mprsyncup script wants to create a distfiles/MacPorts/RELEASE_URL file symlink there... That should be available via http server pre selfupdate.tcl in base and is also where you distfiles and software. It gets really confusing because base wants you to use the rsync.repos file for your rsync server. And my best guess is that is supposed to be 'included' (yes you can do that like a c header in a rsync conf). But that uses /Volumes/data/rsync/macports. That's a mac structure. And imho if your not hosting your http server there exactly as well it also gets confusing... So instead I change `RSYNCROOT=/rsync/macports` to the more sensible `RSYNCROOT=/var/www/html/macports`. Not only does this finally get us to a filesystem that makes sense to base, but it also makes sense when comparing to how macports.org hosts files (at least it is the only thing I can figure out, there is again no documentation on making your own master server for macports.org). So that entailed changing it in mprsyncup, and then base already has the changes:

`RSYNCROOT=/rsync/macports`

and then that rsync.repos-tigerports file is included in my rsync.conf on my server (more on that later).

### Changing MPRSYNCUP To Make Sense For A Custom Fork Of MacPorts Base

Because I am using my own branch with my own tags, I needed to tell mprsyncup about this:

```
 if [ -d "${TBASE}/.git" ]; then
-    ${GIT} -C "${TBASE}" pull -q
-    ${GIT} -C "${TBASE}" fetch --tags
+#    ${GIT} -C "${TBASE}" pull -q
+#    ${GIT} -C "${TBASE}" fetch --tags
+    ${GIT} -C "${TBASE}" fetch origin tigerports-release-2.12 --tags
+    ${GIT} -C "${TBASE}" checkout tigerports-release-2.12
+    ${GIT} -C "${TBASE}" pull -q origin tigerports-release-2.12
 else
-    ${GIT} clone -q "${BASEURL}" "${TBASE}"
+#    ${GIT} clone -q "${BASEURL}" "${TBASE}"
+    ${GIT} clone -q -b tigerports-release-2.12 "${BASEURL}" "${TBASE}"
 fi
```

This will need to be updated for 2.13, but works so that it is pulling my branch, with my repo, and finding my tags. I need to be careful that before importing any newer tag like v2.12.7 that I have out a 2.12.7.001, but that's ok and a problem for managing merges with base (which I can do as frequently as needed, but I won't just sync it constantly as that would break the intended setup).

Something else I'll mention here is this:

```
 
 ${MKDIR} -p "${RSYNCROOT}/trunk/base"
@@ -152,6 +163,7 @@
     PORTS_NEW_REV="$(${GIT} rev-parse HEAD)"
     [ "${PORTS_OLD_REV}" = "${PORTS_NEW_REV}" ] && PORTS_CHANGED=0
 else
+    cd "${PORTS}"
     ${GIT} clone -q --depth 1 "${PORTSURL}" "${PORTS}"
     PORTS_NEW_REV="$(${GIT} rev-parse HEAD)"
 fi
```

I had to add that cd "${PORTS}", seems like a bug to me at least on linux as it never was in the ports dir before that in this point of mprsyncup and that broke checking for new commits.

### Making A Setup Script For MPRSYNCUP

There's still some missing pieces with mprsyncup, not in the script itself but having it be setup with a cron job automatically, getting a sensible filesystem on the vps before running it, etc. That's why I made the setup script. So what this does is

(Prep):

```
login: tigerports (my regualar user)
cd ~`
git clone -b tigerports https://github.com/alex-free/macports-infrastructure`
cd macports-infrastructure
```

1) Sets sudo timeout to not bother us (personal thing, if your using this exact script verbatim you can comment it out but I don't have a need for the crazy sudo timeout default my vps provider setsup).

2) Figures out where the hell mprsyncup-tigerports actually is (it should be somewhere in your home directory). We need an absolute path for the cron job to use.

3) Creates priv/pubkey (equivlent to macports-pubkey-2024.pub) if it doesn't exist at `~/tigerports-privkey.pem`. Since I already did this, I have `tigerports-privkey.pem` in my home dir which setup looks at and wont regenerate it as long as I have it there.

4) Creates the rsync server config (notice the include of the base rsync.repos-tigerports file for it's configuration):

```
sudo tee /etc/rsyncd.conf > /dev/null << 'EOF'
# Global Linux Daemon Settings
uid = nobody
gid = nogroup
use chroot = yes
max connections = 10
pid file = /var/run/rsyncd.pid

&include /var/tmp/macports/release/base/portmgr/rsync.repos-tigerports
EOF
```

5) Creates my nginx config (nothing fancy, do have autoindex on, and it uses /var/www/html instead of what my vps provider wants which is some weird thing):

```
# My VPS needs to change the websrv root.
sudo tee /etc/nginx/sites-enabled/default > /dev/null << 'EOF'
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        autoindex on;

        index index.php index.html;

        server_name _;
	
	location ~ \.php$ {
        	include snippets/fastcgi-php.conf;
        	fastcgi_pass 127.0.0.1:9000;
        }
}
EOF
```

6) Reloads the webserver testing the configuration. 

7) Creates the rsync folder structure. 

8) Creates the http server structure.

7) Sets permissions on folder structure. Deletes tmp files generated by mprsyncup.

8) Adds an admin group (which base needs to build on linux, this is a macos leftover thing afaict).

9) Runs mprsyncup (initial run).

10) Creates the cron daemon to run mprsyncup every 15 minutes (as specified by macports-infrastructure docs, this is the interval macports.org uses but we of course don't have launchd on a linux vps so we have to make our own)

11) Restarts the rsync daemon

### What Happens In 15 Minutes?

Well, now every 15 minutes since we have done the initial clone of base and ports, this is how it handles it.

* Checks for a new tag of base. If no new tag exists from what it last cloned, then it **will not clone it**. This is good because your can arbitrarily commit to release-2.12 branch and not push that to thousands of users before it's ready.

* Checks for ANY commits to ports. Any commit to ports means some ports could be added, deleted, or updated so we need to run portindex (which it does for every new commit, for every specified platform).

## Ports Tree

This is the final piece in a complete macports.org replacement. We need to create a complete, coherenet ports tree in order to serve portindexes from tigerports.com. How this works is:

* I have a [fork of powerpc-ports](https://github.com/alex-free/powerpc-ports/tree/tigerports) where I merge tiger compatibility patches for ports. I then PR the [official powerpc-ports](https://github.com/macos-powerpc/powerpc-ports/pulls) to upstream them. I am also planning on trying to upstream to the original software authors, and keeping track of this at the [upstream status report](http://tigerports.com/upstream-status-report/).

* I then have another repo, [tigerports-ports](https://github.com/alex-free/tigerports-ports), which has a script called [merge](https://github.com/alex-free/tigerports-ports) that combines my fork of powerpc-ports with the current macports-ports port tree.

### Archives:

Key files, [_resources/port1.0/fetch/archive_sites.tcl](https://github.com/alex-free/powerpc-ports/blob/tigerports/_resources/port1.0/fetch/archive_sites.tcl) and [_resources/port1.0/fetch/mirror_sites.tcl](https://github.com/alex-free/powerpc-ports/blob/tigerports/_resources/port1.0/fetch/mirror_sites.tcl) are what point base towards tigerports.com for binary archives and distfiles respectively.

### Psuedo Build Bot

Since I am one person, I am focusing on a subset of keeping important key, fun, useful ports maintained and working on tiger. So what I am doing is (and these are in my fork of macports-infrastructure):

1) [build-archives](https://github.com/alex-free/macports-infrastructure/blob/tigerports/build-archives) has specific archives it is set to build. 

2) [xfer-archives](https://github.com/alex-free/macports-infrastructure/blob/tigerports/xfer-archives) signs them on the tiger machine with my private keys, and then transfers them to a temp directory on my server.

3) [push-archives](https://github.com/alex-free/macports-infrastructure/blob/tigerports/push-archives) ran on the tigerports.com vps, this moves archives and .sig files from the temp directory the tiger machine uploaded to into the ones that base actually looks for.