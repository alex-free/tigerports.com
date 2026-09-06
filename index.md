# TigerPorts

This is not just a fork of MacPorts. It is a recreation of the entire MacPorts infrastructure, tailored to our favorite OS. TigerPorts supports Intel and PowerPC Macs.

* [Tiger orientated ports tree](https://github.com/alex-free/tigerports-ports) focused on software that works on Tiger. This is a managed snapshot of my fork of powerpc-ports tigerports branch (which tracks upstream powerpc-ports, and changes are submitted to them as well when possible) merged with the MacPorts Ports tree, creating one unified ports tree for tiger.

* [Tiger orientated base](https://github.com/alex-free/tigerports-base), configured in source to use tigerports.com. This tracks upstream macports-base as close as possible, includes tiger-specific fixes/functionally, and additional features supported by PPCPorts-base as well. In addition, TigerPorts-Base is TLSv1.2 capable with modern certs so that it can connect to modern distfile sites/mirrors.

* [Tiger orientated infrastructure](https://github.com/alex-free/tigerports-infrastructure), allows hosting tigerports.com on a Debian VPS rather then a Mac like MacPorts has it. This includes numerous improvements to the sync scripts, adds binary package signing management, and includes a setup script to recreate my server.

* [Tarball releases](#downloads) to install TigerPorts from source on your Mac, same as official MacPorts (PKG installer is WIP).

* The [tigerports.com](http://tigerports.com/) rsync server, which syncs with the [Tiger orientated ports tree](https://github.com/alex-free/tigerports-ports) every 15 minutes, exactly like real MacPorts. Any pull requests merged there will be available in no later then a quarter hour to all TigerPorts users via `sudo port selfupdate`. 

* The tigerports.com http server, which serves distfiles and compiled port binaries ([list of binaries](http://tigerports.com/macports/packages)).

* Security is kept the same, just not managed by MacPorts. The ports tree, portindex, and binary packages served directly by tigerports.com are all signed.

## Forum Threads

* [MacRumors Early Intel Macs Forum Thread](https://forums.macrumors.com/threads/tigerports-com-entire-macports-infrastructure-revived-for-mac-os-x-10-4.2485572)

* [MacRumors PowerPC Macs Forum Thread](https://forums.macrumors.com/threads/tigerports-com-entire-macports-infrastructure-revived-for-mac-os-x-10-4.2485567/)


## Table Of Contents

* [Downloads](#downloads)

* [Usage](#usage)

* [TODO](#todo)

## Downloads

### v2.12.04.002 (8/26/2026)

Changes:

* Fixed bootstrap ppc detection.

* Improved bootstrap rebuild detection so that it will not rebuild only if curl successfully was built (and -f wasn't given, and its not set to false for version bump via selfupdate).

* Bootstrap now extracts tarballs instead of copying extracted source directories. This not only makes diffing this against other macports-base projects easier, but fixes issues related to copying extracted sources after they have been uploaded to git.

* [TigerPorts-2.12.04.002.tar.bz2](http://tigerports.com/macports/distfiles/MacPorts/TigerPorts-2.12.04.002.tar.bz2) _bzip2 release tarball ([verification signature](http://tigerports.com/macports/distfiles/MacPorts/TigerPorts-2.12.04.002.tar.bz2.sig))_

* [TigerPorts-2.12.04.002.tar.gz](http://tigerports.com/macports/distfiles/MacPorts/TigerPorts-2.12.04.002.tar.gz) _gzip release tarball ([verification signature](http://tigerports.com/macports/distfiles/MacPorts/TigerPorts-2.12.04.002.tar.gz.sig))_

* [TigerPorts-2.12.04.002.chk.txt](http://tigerports.com/macports/distfiles/MacPorts/TigerPorts-2.12.04.002.chk.txt) _cryptographic checksum manifest to verify the integrity of TigerPorts downloads_

[Previous versions](http://tigerports.com/macports/distfiles/MacPorts)

## Requirements

TigerPorts requires Mac OS X 10.4.11 and Xcode v2.5. For your convenience I host an archive for these Apple downloads here on tigerports.com for direct download on legacy Macs. 

* [MacOSXUpdCombo10.4.11Intel.dmg](http://tigerports.com/apple/MacOSXUpdCombo10.4.11Intel.dmg) (MD5: cc6e64bfe6b00910cdcf60ba2028840d)

* [MacOSXUpdCombo10.4.11PPC.dmg](http://tigerports.com/apple/MacOSXUpdCombo10.4.11PPC.dmg) (MD5: 378b21fbd51471b0fa86129ead81541b)

* [xcode25_8m2558_developerdvd.dmg](http://tigerports.com/apple/xcode25_8m2558_developerdvd.dmg) (MD5: 3bd6c24d8fbbdf9007e15861d173764d)

## Usage

It's just like MacPorts. Right now, there are only source releases of base that you need to build yourself. See the page [Getting Started With Tiger Development](getting-started-with-tiger-development) for build instructions. Also documented there is how to replace the ancient ssh built-in to tiger with a modern secure one from ports, as well as setting up/configuring git over ssh that works for push/pull with github.

