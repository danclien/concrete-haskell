# Linking libraries from Homebrew (on OS X)

Here's a quick writeup on how to get Haskell libraries that link to system libraries working on OS X. I'll be using the `text-icu` library from Hackage as an example.

I'll clean up this post over the weekend for better clarity.

## Problem

```
$ mkdir icu-test
$ cd icu-test
$ cabal sandbox init
$ cabal install text-icu
```

This fails with the following error message which may not be very helpful:

```
$ cabal install text-icu
...
Configuring text-icu-0.7.0.0...
Failed to install text-icu-0.7.0.0
Build log ( /Users/dan/temp/icu/.cabal-sandbox/logs/text-icu-0.7.0.0.log ):
Configuring text-icu-0.7.0.0...
setup-Simple-Cabal-1.18.1.3-x86_64-osx-ghc-7.8.3: Missing dependencies on
foreign libraries:
* Missing C libraries: icuuc, icui18n, icudata
This problem can usually be solved by installing the system packages that
provide these libraries (you may need the "-dev" versions). If the libraries
are already installed but in a non-standard location then you can use the
flags --extra-include-dirs= and --extra-lib-dirs= to specify where they are.
cabal: Error: some packages failed to install:
text-icu-0.7.0.0 failed during the configure step. The exception was:
ExitFailure 1
```

## Solution

### Finding the missing library

I don't have a good solution here. Hopefully the library author tells you were you can get the required library or there's a StackOverflow answer that points you in the right direction. :(

Most of the required libraries are probably on Homebrew or MacPorts though.


### Installing the missing library

For `text-icu`, the missing C libraries can be installed from Homebrew using:

```
brew install icu4c
```


### Finding where the library is located

You will need to find where the library is located by using:

```
brew info icu4c
```

Output

```
$ brew info icu4c
icu4c: stable 54.1 (bottled), HEAD
http://site.icu-project.org/

This formula is keg-only.
Mac OS X already provides this software and installing another version in
parallel can cause all kinds of trouble.

OS X provides libicucore.dylib (but nothing else).

/opt/boxen/homebrew/Cellar/icu4c/54.1 (242 files, 65M)
  Built from source
From: https://github.com/Homebrew/homebrew/blob/master/Library/Formula/icu4c.rb
==> Options
--c++11
  Build using C++11 mode
--universal
  Build a universal binary
--HEAD
  Install HEAD version
```

The directory we'll need is `/opt/boxen/homebrew/Cellar/icu4c/54.1`.

### Updating `~/.cabal/config`

Cabal needs to know where those libraries are located before it can link them correctly. Update your `~/.cabal/config` file to add the `icu4c/54.1/include` and `icu4c/54.1/lib` directories.

```
extra-include-dirs:   /opt/boxen/homebrew/Cellar/icu4c/54.1/include
extra-lib-dirs:       /opt/boxen/homebrew/Cellar/icu4c/54.1/lib
```

## Conclusion

That should be everything you need to get it working.

```
$ cabal install text-icu
Resolving dependencies...
Notice: installing into a sandbox located at
/Users/dan/temp/icu-test/.cabal-sandbox
Configuring text-icu-0.7.0.0...
Building text-icu-0.7.0.0...
Installed text-icu-0.7.0.0
$
```