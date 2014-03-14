---
layout: post
title: "Debugging a Homebrew Failed Install"
date: 2014-03-14 13:10:30 -0700
comments: true
categories: [OSX, Homebrew, Bazaar, Python]
---

This morning I wanted to install a Go environment on my Mac Air, only to be 
greeted with this ominous error from homebrew: 

``` bash 
$ brew install bzr
==> Downloading https://launchpad.net/bzr/2.6/2.6.0/+download/bzr-2.6.0.tar.gz
Already downloaded: /Library/Caches/Homebrew/bazaar-2.6.0.tar.gz
==> make man1/bzr.1
==> make
Cannot build extension "bzrlib._annotator_pyx".
Use "build_ext --allow-python-fallback" to use slower python implementations instead.

error: command 'clang' failed with exit status 1
make: *** [extensions] Error 1

READ THIS: https://github.com/Homebrew/homebrew/wiki/troubleshooting
``` 
<!-- more -->
Now I had just built out several different Golang development environments earlier 
in the week.  Several searches on Google weren't providing any relief or 
guidance.

My first step was to update homebrew.

``` bash
$ brew update  && brew upgrade
``` 

No relief there either....  At least not yet.  I then turned up the logging and 
sent it to a gist.

### Send debugging data to a gist

``` bash
brew gist-logs --config --doctor bzr
``` 

Here is the output I found:

```
building extension modules.
python setup.py build_ext -i 
No Cython, trying Pyrex...
 
The python package 'Pyrex' is not available. If the .c files are available,
they will be built, but modifying the .pyx files will not rebuild them.
 
running build_ext
building 'bzrlib._annotator_pyx' extension
creating build
creating build/temp.macosx-10.9-intel-2.7
creating build/temp.macosx-10.9-intel-2.7/bzrlib
clang -fno-strict-aliasing -fno-common -dynamic -g -Os -pipe -fno-common -fno-strict-aliasing -fwrapv -mno-fused-madd -DENABLE_DTRACE -DMACOSX -DNDEBUG -Wall -Wstrict-prototypes -Wshorten-64-to-32 -DNDEBUG -g -fwrapv -Os -Wall -Wstrict-prototypes -DENABLE_DTRACE -pipe -arch x86_64 -I/System/Library/Frameworks/Python.framework/Versions/2.7/include/python2.7 -c bzrlib/_annotator_pyx.c -o build/temp.macosx-10.9-intel-2.7/bzrlib/_annotator_pyx.o
clang: error: unknown argument: '-mno-fused-madd' [-Wunused-command-line-argument-hard-error-in-future]
clang: note: this will be a hard error (cannot be downgraded to a warning) in the future
 
  Cannot build extension "bzrlib._annotator_pyx".
  Use "build_ext --allow-python-fallback" to use slower python implementations instead.
 
error: command 'clang' failed with exit status 1
make: *** [extensions] Error 1
 
HOMEBREW_VERSION: 0.9.5
HEAD: f1a2a667ebc1c630f03298ac8be5882426e7d454
CPU: quad-core 64-bit sandybridge
OS X: 10.9.2-x86_64
Xcode: 5.1
CLT: 5.1.0.0.1.1393561416
X11: 2.7.5 => /opt/X11
``` 
This led me to belive I needed to install either Cython or Pyrex.  I decided 
upon attempting to install Pyrex first.  I quickly learned at the Bazaar 
website they force Mac users to use Python 2.6 with their install scripts.  
This means I had to use sudo to get Pyrex to install...

``` bash 
$ sudo /usr/bin/easy_install-2.6 pyrex
``` 

One more attempt at installing Bazaar with Homebrew left me with this error
during make:

``` 
$ brew gist-logs --config --doctor bzr

building extension modules.
python setup.py build_ext -i 
running build_ext
skipping 'bzrlib/_annotator_pyx.c' Cython extension (up-to-date)
building 'bzrlib._annotator_pyx' extension
creating build
creating build/temp.macosx-10.9-intel-2.7
creating build/temp.macosx-10.9-intel-2.7/bzrlib
clang -fno-strict-aliasing -fno-common -dynamic -g -Os -pipe -fno-common -fno-strict-aliasing -fwrapv -mno-fused-madd -DENABLE_DTRACE -DMACOSX -DNDEBUG -Wall -Wstrict-prototypes -Wshorten-64-to-32 -DNDEBUG -g -fwrapv -Os -Wall -Wstrict-prototypes -DENABLE_DTRACE -pipe -arch x86_64 -I/System/Library/Frameworks/Python.framework/Versions/2.7/include/python2.7 -c bzrlib/_annotator_pyx.c -o build/temp.macosx-10.9-intel-2.7/bzrlib/_annotator_pyx.o
clang: error: unknown argument: '-mno-fused-madd' [-Wunused-command-line-argument-hard-error-in-future]
clang: note: this will be a hard error (cannot be downgraded to a warning) in the future
 
  Cannot build extension "bzrlib._annotator_pyx".
  Use "build_ext --allow-python-fallback" to use slower python implementations instead.
 
error: command 'clang' failed with exit status 1
make: *** [extensions] Error 1
 
HOMEBREW_VERSION: 0.9.5
HEAD: f1a2a667ebc1c630f03298ac8be5882426e7d454
CPU: quad-core 64-bit sandybridge
OS X: 10.9.2-x86_64
Xcode: 5.1
CLT: 5.1.0.0.1.1393561416
X11: 2.7.5 => /opt/X11
```
### Resolution

Now I was really scratching my head, when I finally realized Apple had just 
updated Xcode this week.  Earlier in the day, so had the Homebrew team.  They were 
already on the case and had [pushed a fix](https://github.com/Homebrew/homebrew/issues/27534#issuecomment-37691498) 
whilst I was fighting this issue. 

Once more I ran 
``` bash
$ brew update

$ brew install bzr
```

And I was met with a successful install.  If you find your way here, I hope 
this helps you along your path when debugging a failed Homebrew install.  A 
special thanks to the [Homebrew team](https://github.com/Homebrew) for working 
hard to provide a great product and even better service!
