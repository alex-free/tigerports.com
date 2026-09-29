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

## Merging

This is more so for me when combining PowerPC Ports upstream:

```
git remote add upstream https://github.com/macos-powerpc/powerpc-ports.git
```
```
git fetch upstream
```
```
git merge upstream/main
```

To resolve all conflicts with upstream winning:
```
git checkout --theirs .
```
```
git add .
```
```
git commit -m "merge with upstream"
```
```
git push origin tigerports
```

Specifically for one file:

```
git checkout --theirs path/to/conflicted-file
```
```
git add path/to/conflicted-file
```
```
git commit
```

## Notes About Tiger Arches

* You can build CLI ports targeting any of the 4 arches on Tiger (retail or server they both support all of them): i386, x86_64, ppc, and ppc64. However, Tiger does not contain any framework support for any 64 bit arch. CLI ports can be built targeting 64 bit (change your build_arch), but they can't link any Apple frameworks in Tiger. Leopard and above is where frameworks have 64 bit slices that can be linked too. Note that Snow Leopard PPC does NOT have ppc64.

* ONLY i386/ppc is really tested at the moment. Ideally, if this is to be focused on, some ports can 'drop' from x86_64 to i386 and the like but this is not currently something implemented anywhere.

* Rosetta can build some ports (like libgcc16/gcc16) but fails on others (apple-gcc-42). One way to work around this is to use ppc built binaries to get to where you can build libgcc/gcc16 on Rosetta. Keep in mind with Rosetta you may need to lower makebuildjobs (I have my 2xdual core xeons with 4 cores set to 2 makebuildjobs) and that it will probably be slower then i.e. a G5. 

* You must uninstall **every** installed port if your changing a current MacPorts install `build_arch`. So if you changing build_arch from say i386 to ppc, you need to `sudo port uninstall installed` before installing anything or it will try to force universal builds that are broken and unsupported by ppcports and tigerports.

## Common Portfile Additions

Intel Tiger target is SSE3 maximum. If we are disabling i.e. SSE4 and AVX/AVX2/AVX512, gate it at Snow Leopard (as these technologies are only introduced in 2011 Intel CPUs):

```
if {${os.platform} eq "darwin" && ${os.major} < 11 && \
${configure.build_arch} in [list i386 x86_64]} {

}
```

i386:
```
if {${configure.build_arch} eq "i386"} {

}
```

x86_64/i386
```
if {${configure.build_arch} in [list i386 x86_64]} {

}
```

PPC:
```
if {${configure.build_arch} eq "ppc"} {

}
```

PPC/PPC64:
```
if {${configure.build_arch} in [list ppc ppc64]} {

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

If we need legacy support in the Portfile:
```
PortGroup           legacysupport 1.1
legacysupport.newest_darwin_requires_legacy 8
```

## Macros

For source files to detect a code path for newer then 10.4:
```
#if defined(__APPLE__)
#include <AvailabilityMacros.h>
#if MAC_OS_X_VERSION_MIN_REQUIRED > 1040

#endif
#endif
```

For source files to detect a code path for older then 10.5:
```
#if defined(__APPLE__)
#include <AvailabilityMacros.h>
#if MAC_OS_X_VERSION_MIN_REQUIRED < 1050

#endif
#endif
```

## Handling XGETBV (Intel Optimization Detection)

**Many** developers call xgetbv if gcc is sufficently new enough to find out what optimizations (like AVX) are available. The issue is, most don't _detect_ if the call they use for _detection itself_ is available. This causes issues on targeted intel CPUs (Core Solo/Core Duo is the minimum, but this affects Xeons, and others too).

So before xgetbv is called:

```
#include <cpuid.h>

// https://www.intel.com/content/dam/doc/manual/64-ia-32-architectures-optimization-manual.pdf
unsigned eax, ebx, ecx, edx;
// Get basic capability information. Only care about ecx bit 27 (OSXSAVE),
// but need all 4 registers for call.
 
 __get_cpuid(1, &eax, &ebx, &ecx, &edx);

if (ecx & (1 << 27))
{
   printf("XGETBV DETECTED\n");
} else {
    printf("XGETBV NOT AVAILABLE\n");
}
```

This depends on cpuid.h, which requires GCC 4.3 from what I can tell. Most software is gaurding xgetbv to GCC 4.9 so this isn't an issue, but if for some reason GCC 4.2 needed to be supported this could be rewritten without cpuid.h using 32 bit and 64 bit specific inline asm.

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

Get CMake build options:
```
sudo port -d extract <portname>
```

```
cd $(port work <port name>)
```

```
cmake -S . -B build -LAH
```

Get automake options:
```
sudo port -d extract <portname>
```

```
cd $(port work <port name>/<port extract root dir>)
```

```
./configure --help
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