# [TigerPorts](http://tigerports.com) -> Contributions Quick Start Guide

Covered here includes:

* An understanding of the repos involved in this project.
* Common portfile additions/conditionals.
* Common Macros to use for source files involving adding tiger support.
* MacPorts basics.
* How to make proper patches.

## Repos

This is the 'flow' of the upstreams so to say:

* [https://github.com/alex-free/powerpc-ports](https://github.com/alex-free/powerpc-ports) (main branch) tracks [https://github.com/macos-powerpc/powerpc-ports](https://github.com/macos-powerpc/powerpc-ports) (main branch) and is used as a branching point for submitting pull requests to PowerPC Ports.

* [https://github.com/alex-free/powerpc-ports/tree/tigerports](https://github.com/alex-free/powerpc-ports/tree/tigerports) (tigerports branch) is what all tiger fixes get submitted to. It has priorty over [https://github.com/macports/macports-ports](https://github.com/macports/macports-ports), which it is combined with frequently to create one complete ports tree tigerports.com server can fetch, at: [https://github.com/alex-free/tigerports-ports](https://github.com/alex-free/tigerports-ports).

Pull requests from end users may either be:

* Based on the [https://github.com/alex-free/powerpc-ports/tree/tigerports](https://github.com/alex-free/powerpc-ports/tree/tigerports) (tigerports branch) directly and submitted to said branch. The can be upstreamed to powerpc ports by myself if applicable from there.

* Based on the [https://github.com/macos-powerpc/powerpc-ports](https://github.com/macos-powerpc/powerpc-ports) (main branch) and directly submitted to said branch. These will trickle down into the tigerports branch and therefore still end up in [https://github.com/alex-free/tigerports-ports](https://github.com/alex-free/tigerports-ports).  **Please make sure that what your submitting specifically applies to powerpc ports if you do this. TigerPorts is a downstream of PowerPC Ports, but may have it's own differences at any given time.**

## Common Portfile Additions

If Intel:
```
if {${configure.build_arch} eq "i386"} {

}
```

If Power:
```
if {${configure.build_arch} eq "ppc"} {

}
```

If we need a newer compiler then these (next priority is gcc16 when both are blacklisted):

```
compiler.blacklist *gcc-4.2 *gcc-4.0
```

If you run into incompatible-pointer-types within the software, and or it requires C23 (default -std= value set by gcc16):
```
if {[string match macports-gcc-* ${configure.compiler}]} {
    configure.cflags-append    -Wno-error=incompatible-pointer-types
}
```

If you run into incompatible-pointer-types within the software caused by the 10.4u.sdk (can also be -std=gnu17, or anything that isn't c23 or gnu23):
```
if {[string match macports-gcc-* ${configure.compiler}]} {
    configure.cflags-append    -std=c17
}
```

If this is tiger specific:
```
platform darwin 8 {

}
```

If this would technically apply for i.e. panther as well (like no @rpath support):
```
if {${os.platform} eq "darwin" && ${os.major} < 9} {

}
```

If Xcode 2.5's ancient gmake fails you:
```
depends_build-append port:gmake
build.cmd ${prefix}/bin/gmake
```

If we need legacy support in the Portfile:
```
PortGroup           legacysupport 1.1
legacysupport.newest_darwin_requires_legacy 8
```

## Macros

For source files to detect a code path for newer then 10.4 (can easily be reversed as well):
```
#if defined(__APPLE__)
#include "AvailabilityMacros.h"
#if MAC_OS_X_VERSION_MIN_REQUIRED > 1040

#endif
#endif
```

Only apply something tiger specific (should have included `AvailabilityMacros.h` previously though):
```
#if defined(__APPLE__) && MAC_OS_X_VERSION_MIN_REQUIRED != 1040

#endif
```

## MacPorts Basics

To test a port after it's been built, you need to uninstall it. 

If this is a 'minor' thing that doesn't affect linked/built code (i.e. header/include fix, config fix, etc.):

```
sudo port -d uninstall <port name>
```

```
sudo port -d -s install <port name>
```

If this affects things built/link and that immedietly depend on it, you should do this cleanly and rebuild everything from source:

```
sudo port -d uninstall --follow-dependents <port name>
```

```
sudo port -d -s install <port name>
```

## Making Patches

Note: these are variant specific if your doing variants.

Clean the port.

```
sudo port clean <port name>
```

Extract the port.

```
sudo port extract <port name>
```

Change into the extracted port dir.

```
cd $(port work <port name>)
```

Find the extraction root dir that macports uses:

```
ls
```

```
cd <extraction root dir>
```

Find the target file to patch and make a .orig:

```
cp <relative filepath>/<source file to patch> <relative filepath>/<source file to patch>.orig
```

Edit/patch the target file. Any text editor will do, and vim is available in tigerports (I recommend setting :set mouse=i while in vim interactive mode to make copy/paste work like default vi from mac os):

```
sudo vi <relative filepath>/<source file to patch>
```

Create the unified diff. Make sure it looks clean:

```
diff -u <relative filepath>/<source file to patch>.orig <relative filepath>/<source file to patch>
```

Then when it looks good, write it to somewhere that doesn't need sudo privs like `~/Desktop`:

```
diff -u <relative filepath>/<source file to patch>.orig <relative filepath>/<source file to patch> > ~/Desktop/<relative filepath all lowercase, any /'s for subdirs changed to an underscore>_<source file to patch all lowercase>_tiger.diff
```

Move it to the port dir `files` directory:

```
sudo mkdir -p $(port dir <portname>)/files
```

```
sudo cp -v ~/Desktop/<relative filepath all lowercase>_<source file to patch all lowercase>_tiger.diff $(port dir <port name>)/files
```

If you have lots of patches, perhaps combine them into one:

```
cd $(port dir <port name>)/files
```

```
cat *.diff > ~/Desktop/<whatever the patches do collectively>_tiger.diff
```

```
sudo cp ~/Desktop/<whatever the patches do collectively>_tiger.diff $(port dir <port name>)/files
```

Add the patch(es) to the `Portfile`:

```
sudo vi $(port dir <portname>)/Portfile
```

If no other patches exist before this, you can:

```
patchfiles    <your patch>
```

However it might be simpler to always use:

```
patchfiles-append    <your patch>
```

For multiple patches (espically with longer names as we want it to be no more wider then 80 characters), you could do:

```
patchfiles-append \
       <patch 1> \
       <patch 2> \
       <patch 3>
```