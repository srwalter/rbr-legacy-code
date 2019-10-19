# Rust in Legacy Embedded Systems

## Who
- Senior Firmware Engineer at Lexmark
    - Kernel team
    - Network team
    - Build system team
- Retrocomputing

- Aviation

## What
- Laser printers
- Linux ARM computer (with motors and lasers)
- Lots of FOSS
    - apache
    - android
    - systemd
    - python
- Technical Specs
    - 2-core or 4-core ARM A53's
    - 256M RAM up to 4GB RAM
    - 128M code store up to 4GB eMMC

## When
- Proprietary bits
    - PostScript / PCL interpreters
    - Graphics engine
    - Laser and Mech control

- Some code is very old
- Scary bad C code
    - "void\*" all the things
    - Still some K&R style prototypes

- Memory Leaks
- Security issues
- etc, etc

## When
- Why does a laser printer need Linux?

- Printers can do:
    - Active Directory
    - CIFS
    - JVM for third-party applications
    - RFID Badge Readers
    - CAC authentication for DoD
    - DHCP, Ethernet, Wi-Fi, (Fiber!)
    - Embedded Web Server
    - Airprint
    - Google Cloud Print
    - Scan to Email
    - OCR
    - Audit Logging
- Though newer, it's not much better...

## Where
- Started with minimal components
    - Allowed for easy retreat if problems arose
    - Kept rust in its own process(es)

- Then more complex components
    - Prefer minimal dependencies
    - Want to avoid FFI complexities

- Now anything is fair game, primarily:
    - Security sensitive
    - Parsers (serde rocks)
    - Client/server (tokio/futures)
    - Unsafe code strongly discouraged

## bitbake primer
- Build your own Linux distro

- Analogous to buildroot

- Part of Yocto Project

- Based on gentoo's emerge

- Recipes to generate rpm (or deb) packages

- Images create a full root filesystem from images

## Example recipe
```sh
SUMMARY = "Lisp REPL"
HOMEPAGE = "http://google.com"
LICENSE = "CLOSED"

SRC_URI += "git://github.com/swalter/ferrisp"
SRCREV = "e51f5a1992f7c9f3d4c5aed2c2ea6fe1b807157c"
S = "${WORKDIR}/git"

DEPENDS = "rust-native cargo-native"

do_fetch () {
}

do_compile () {
}
```

## How
- Yocto / bitbake

- meta-rust[1] (co-maintainer)

- cargo-bitbake
    - generates a recipe for your crate
    - handles crate dependencies

[1] https://github.com/meta-rust/meta-rust

## The Good
- If it builds, it works

- Corollary: unsafe always fails

- Low runtime memory use and predictable

## Code Size (The Bad)
- Started introducing rust on the high-end
    - Now it needs to squeeze into 128M

- Normally, shared libraries to the rescue

- meta-rust supports shared object for std
    - doesn't help for other common dependencies

- rust shared lib support is immature
    - no standardized ABI
    - still lots of generated code in the binary
    - not intended to reduce code size (cdylib)

## What doesn't work
- Bypass cargo entirely
    - Make a recipe for every rust crate
    - invoke rustc directly
    - Not as bad as in other languages...
- Doesn't scale well

- build.rs completely breaks this

## Single Binary? (The Ugly)
- fuse all the rust into a single large object

- uranium: super-binary a la busybox
    - with a single binary, no need for shared libstd
    - with static libstd, unused parts aren't included
    - code for generics only present once

## uranium
- individual rust programs converted to rlib's

- use git submodules to get all the sources

- uranium dispatches to "lib::main()"

- build with a single pass of cargo

- unlike with C, no implicit dependency creep

- actually liburanium.so plus executable

## Con's
- Slow build time
- Many places to touch for updates:
    - Actual code repo
    - uranium
    - bitbake meta-layer
- Difficult to manage subsets of ``applets''

## Future Work
- Use an internal registry
    - Avoids some of the downside of git submodules
- Enhance cargo to support intermediate builds
    - Helps build time by only rebuilding what's necessary
    - Bitbake sstate handles ABI

## Zephyr Rust
- Run a Rust app on the Zephyr kernel

- Can call all Zephyr syscalls

- Safe wrappers for many syscalls

- Initial support for async/await

- Doesn't require no_std
    - Anything without OS-specific calls will build
    - But may not run :-)
- Available on github[1]

[1] https://github.com/tylerwhall/zephyr-rust

# Thank You!

