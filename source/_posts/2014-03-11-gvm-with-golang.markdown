---
layout: post
title: "using gvm with golang"
date: 2014-03-11 00:32:40 -0700
comments: true
categories: [Golang, How-To, GVM, Linux, OSX]
---
# Using GVM in your Golang development workflow

Recently I began work on a project using [golang](http://golang.org/), and it 
quickly became apparent I needed a way to map a Go environment to each project 
to manage dependencies.  I wasn't able to find everything I needed in a single 
location which became the genesis for this post. I try to follow this 
[pattern](http://golang.org/doc/code.html) set by the golang team at Google.


For both OSX and Ubuntu you'll need the following dependencies:

#### OSX

``` bash  
$ brew install mercurial bzr
``` 

#### Ubuntu 

```
$ sudo apt-get update
$ sudo apt-get install curl git mercurial make binutils gcc bzr bison -y
```

### Download and Install GVM 

``` bash 
$ bash < <(curl -s https://raw.github.com/moovweb/gvm/master/binscripts/gvm-installer) 
```
This adds the following to the bottom of your .bashrc, which you may want to move 
around according to your liking.

``` bash 
# gvm config
[[-s "$HOME/.gvm/scripts/gvm"]] && source "$HOME/.gvm/scripts/gvm"
```

### Install Go Environments 

We are going to install version golang version 1.2, but you may install any of the
available versions listed in this command:

#### List all Go Versions installed
``` bash 
$ gvm listall
```

#### Install Go1.2 and set as default
``` bash 
$ gvm install go1.2
$ gvm use go1.2 --default
```
#### Create a Project specific package set 
``` bash 
$ gvm pkgset create ottemo
$ gvm pkgset use ottemo
```

### Configure Your Golang Workspace 

Here we are going to download a small project I'm working on and assign
a specific gvm package set to be used while developing.  Using gvm like
this helps maintain a clean seperation of concerns by project.  Let's use
the gvm package set we created above.

#### List all package sets 
``` bash 
$ gvm pkgset list

    global
=>  ottemo
```

Now lets create a workspace relative to our $HOME directory and set up 
the necessary project tree.  We'll use a single command to create our 
skeleton directory tree.

#### Create Directory Tree 
``` bash 
$ mkdir -p $HOME/go/{pkg,bin,src}
$ git clone https://github.com/ottemo/ottemo-go.git $HOME/go/src/github.com/ottemo/ottemo-go
```

The very last step is to add workspace's GOPATH to your environment.  We need 
to edit our gvm package environment for ottemo.  This will open your favorite 
editor specified by the environment variable: $EDITOR

#### Edit GOPATH 
``` bash 
$ gvm pkgenv ottemo
```

You want to edit lines 12 and 16, to add your personal workspace tree.

#### Edit GOPATH and PATH Environment Variables 
``` bash 
# original line
export GOPATH; GOPATH="/Users/james/.gvm/pkgsets/go1.2/ottemo:$GOPATH"
# new edited line
export GOPATH; GOPATH="/Users/james/.gvm/pkgsets/go1.2/ottemo:$HOME/go:$GOPATH"
# original line
export PATH; PATH="/Users/james/.gvm/pkgsets/go1.2/ottemo/bin:${GVM_OVERLAY_PREFIX}/bin:${PATH}"
# new edited line
export PATH; PATH="/Users/james/.gvm/pkgsets/go1.2/ottemo/bin:${GVM_OVERLAY_PREFIX}/bin:$HOME/go/bin:${PATH}"
```

### Notes

I use gvm-prompt which comes with GVM to tell me which Golang enviroment I'm currently using.  
I've included my prompt for bash as a sample.

{% codeblock lang:bash Sample Prompt Definition %}
if["$(whoami)"='root']; then
  RED='\e[0;31m'
  GREEN='\e[0;32m'
  NC='\e[0m'
  GIT_BRANCH='$(__git_ps1 "(%s)")'
  GVM='$(gvm-prompt "(%s)")'
  PS1="[${RED}\u@\h:\W \t.\d${GREEN}${GIT_BRANCH}$NC ${GVM}] \n >"
else
  RED='\e[0;31m'
  GREEN='\e[0;32m'
  NC='\e[0m'
  GIT_BRANCH='$(__git_ps1 "(%s)")'
  GVM='$(gvm-prompt "(%s)")'
  PS1="[${GREEN}\u@\h:\W ${RED}${GIT_BRANCH}$NC ${GVM}] \n >"
fi
{% endcodeblock %}


