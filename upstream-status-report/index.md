# [TigerPorts](http://tigerports.com) -> Upstream Status Report

This page reports the following:

* Status of changes submitted to [PowerPC Ports](https://github.com/macos-powerpc/powerpc-ports) (At any given time [TigerPorts Ports](http://github.com/alex-free/tigerports-ports) may differ, but the goal is to NOT have too many differences or _any_ if possible).

* Status of changes submitted to [MacPorts Ports](http://github.com/macports-ports/ports) (if applicable to more then just tiger).

* Status of change submitted to the actual software.

* Until these are in some way 'complete' I will maintain them to keep the support in tiger going.

* Note that this isn't a list of 'everything that works on tiger'. Much more works then just the software in this list. This is a list of everything that is either currently problematic with tiger or has been fixed to work on tiger and it's current status.

* This list is also tiger/tiger ports specific, and does not reflect the status of these ports on 10.5, 10.6, or that of any other macports config.

## Dependency Chains

These are the big software goals to make tiger compatible (and make binaries available for). They result in maintaining support for many dependencies if they don't work with tiger OTB (see the ports section below for ports needing active maintence).

| Port | PowerPC Tiger Status | Intel Tiger Status | Notes |
|------|----------------------|--------------------|-------|
| curl | yes | yes | none |
| ffmpeg | yes | WIP | Intel still has dependencies that need to be fixed before building can be attempted |
| gcc16 | almost (one last fix to submit but binaries available) | yes | none |
| git | yes | yes | none. |
| libsdl | yes | ?? | none |
| libsdl2 +opengl +pulseaudio | yes | yes | this is now the default when doing `sudo port install libsdl2` for tiger! |
| openssh | yes | yes | none |
| openssl | yes | yes | none |
| openvpn2 | almost (needs cleanup before being made public but privately works). | almost (same deal) | Needs some cleanup before being made public (specifically dnsupdown2 script needs to point to newer bash in shabang, and perhaps become more intentionally tiger compatible) |
| python314 | yes | yes | none |
| xorg | WIP | WIP | We have a legacy implementation already setup by kencu, but we need to fix some dependencies first before it will build. |

## Ports

| Port | PPC Ports | MacPorts Ports | Software | Notes |
|------|-----------|----------------|----------|-------|
| m4 | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/182) | N/A | todo | M4 has tiger fixes upstreamed only for power and not intel |
| python27 +bootstrap | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/207) | N/A | N/A | Complete. |
| gcc16 | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/193) | N/A| todo | Needs expansion to everything pre-10.6 regardless of arch to fix rosetta and G5 |
| gawk | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/196) | N/A | N/A | Complete. |
| libarchive | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/206) | N/A | todo | 10.4 server has ACL but it doesn't support symlinks, 10.4 retail has no ACL implementation |
| icu | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/200) | ?? | N/A | Perhaps test leopard on macports official to see if it also applies. |
| brotli | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/208) | todo | N/A | Should test leopard on macports official, probably same issue. |
| p5-xml-parser | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/209) | todo | N/A | Can remove from overlay once submitted to macports |
| python314 | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/211) | ?? | todo?? | Find out a way to upstream to macports official if it applies, or the upstream should detect this. Or try newer linker? |
| orc | Intel fixes [merged](https://github.com/macos-powerpc/powerpc-ports/pull/213), PPC fixes not [yet](https://github.com/macos-powerpc/powerpc-ports/pull/264) | todo | todo | Tiger lacks certain expected headers and intel expects a 2011 x86_64 CPU that understands xgetbv. Perhaps detect this better in a different way? Power might be able to default to altivec perhaps on tiger in-source build system? And macports could use the xgetbv patch to stop blacklisting clang? |
| libjpeg-turbo | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/214) | todo | N/A | This should apply to macports-ports for leopard/snow leopard |
| brotli | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/208) | todo | N/A | Should also apply to macports-ports for leopard/snow leopard |
| libdeflate | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/215) | ?? | ?? |
| bash | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/216) | N/A | N/A | This can actually be fixed by either specifying pre-C23 with -std, or updating tiger headers to be C23 compliant. REDO. |
| utl-linux | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/217) | N/A | N/A | This can probably be fixed like bash above |
| libsndfile | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/219) | todo | todo | This is actually an issue with newer C standards in general |
| atk | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/220) | N/A | ?? | This might be able to be fixed in upstream so that docs work |
| pango | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/222) | N/A | ?? | Might be another bash situation that can be redone |
| libxkbcommon | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/224) | N/A | todo | This should be upstreamed to the software |
| py-gobject3 | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/226) | N/A | ?? | Might be another bash situation that can be redone |
| libsdl2+pulseaudio | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/227) | ?? | ?? | Certainly not everything can ever be upstreamed as it is, but perhaps pulseaudio could become 'allowed' for mac os somewhere? |
| libdiscid | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/228) | N/A | N/A | Complete. |
| dbus | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/229) | N/A | todo | Need to figure out what is going wrong with this build system linking the wrong paths (most likely an rpath-less issue). |
| mesa | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/239) | ?? | todo | The build system should be more accomadting to legacy intel cpus. |
| uavs3d | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/243) | ?? | todo | build system should be able to address legacy intel cpus. |
| vvenc | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/244) | todo | todo | This needs to go to upstream |
| davs2 | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/245) | N/A | N/A | Complete |
| zimg | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/246) | ?? | ?? | Should support legacy intel |
| gnutls | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/248) | N/A | N/A | Same issue as bash probably |
| tuntaposx | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/252) | N/A | N/A | Complete. Maybe try to get a newer version but all newer versions target much newer Xcode, and apple-gcc42 isn't happy to build extensions for tiger on tiger yet |
| ghostscript | [not yet](https://github.com/macos-powerpc/powerpc-ports/pull/253) | ?? | ?? | MacPorts might want this for old gcc. | 
| py-cairo+x11 | [N/A](https://github.com/macos-powerpc/powerpc-ports/pull/266) | N/A | This isn't a 'supported' variant in official macports (they force +x11+quartz), and perhaps it shouldn't be here either but we don't have a working xquartz yet. Not breaking other stuff to be pedantic so I allow it for now |
| libfmt12 | [not yet](https://github.com/macos-powerpc/powerpc-ports/pull/269) | N/A | todo | This needs to be upstreamed as it's a bug in the software when built on tiger |
| doxygen | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/270) | N/A | N/A | Complete... Perhaps get latest doxygen working? |
| minizip-ng| [not yet](https://github.com/macos-powerpc/powerpc-ports/pull/271) | N/A | todo | Need to get this into upstream as they mandate O_NOFOLLOW |
| expat | [merged](https://github.com/macos-powerpc/powerpc-ports/pull/272) | N/A| TODO | Perhaps their build system should not use rpaths when detecting 10.4 |