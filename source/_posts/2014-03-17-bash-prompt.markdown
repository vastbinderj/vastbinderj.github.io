---
layout: post
title: "My Personal Bash Prompt"
date: 2014-03-17 12:10:00 -0700
comments: true
categories: [Linux, OSX, Bash, How-To]
---

I like to have my prompt tell me several things: current directory, who I am 
logged in as, current git branch and language specific context.  Recently, I've been 
jumping between ruby, python, node and golang as we decide upon the primary engine 
for Ottemo.  It has been an interesting process as we want [Ottemo](http://www.ottemo.io)
 to be rock-solid and secure as well as super easy to setup and of course, blindingly fast.  

<!-- more -->
This vetting process is better served in a future post as I just want to provide 
a sample light-weight prompt for others to follow, today.  

I enjoy a color prompt, so the first thing I do is enable 256 colors in my terminal
emulator:   

``` bash .bash_profile
# uncomment for a colored prompt
force_color_prompt=yes
export GREP_OPTIONS='--color=auto'
export CLICOLOR=1

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color) color_prompt=yes;;
esac

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
	color_prompt=yes
    else
	color_prompt=
    fi
fi
``` 

Next, I want to distinguish between when I'm logged in as root on computers 
I control, so I change the colors in my prompt for my account and root's account.  You
can find a great list of colors [available here](http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO/x329.html).

``` bash showme_colors.bash
#!/bin/bash
#
#   This file echoes a bunch of color codes to the 
#   terminal to demonstrate what's available.  Each 
#   line is the color code of one foreground color,
#   out of 17 (default + 16 escapes), followed by a 
#   test use of that color on all nine background 
#   colors (default + 8 escapes).
#

T='gYw'   # The test text

echo -e "\n                 40m     41m     42m     43m\
     44m     45m     46m     47m";

for FGs in '    m' '   1m' '  30m' '1;30m' '  31m' '1;31m' '  32m' \
           '1;32m' '  33m' '1;33m' '  34m' '1;34m' '  35m' '1;35m' \
           '  36m' '1;36m' '  37m' '1;37m';
  do FG=${FGs// /}
  echo -en " $FGs \033[$FG  $T  "
  for BG in 40m 41m 42m 43m 44m 45m 46m 47m;
    do echo -en "$EINS \033[$FG\033[$BG  $T  \033[0m";
  done
  echo;
done
echo
```

Here is the excerpt from my dotfiles where I set my prompt.  Its not pretty but
it is functional.  


``` bash .bash_profile
# give root a different colored prompt so I have a visual cue
if [ "$(whoami)" = 'root' ]; then
  RED='\e[0;31m'
  GREEN='\e[0;32m'
  NC='\e[0m'
  GIT_BRANCH='$(__git_ps1 "(%s)")'
  GVM='$(gvm-prompt "(%s)")'
  PS1="[${RED}\u@\h:\W \t.\d${GREEN}${GIT_BRANCH}${NC} ${GVM}] \n > "
else
  RED='\e[0;31m'
  GREEN='\e[0;32m'
  NC='\e[0m'
  GIT_BRANCH='$(__git_ps1 "(%s)")'
  GVM='$(gvm-prompt "(%s)")'
  PS1="[${GREEN}\u@\h:\W ${RED}${GIT_BRANCH}${NC} ${GVM}] \n > "
fi
unset color_prompt force_color_prompt
```

And finally an example of my current bash prompt:

{% img /images/bash-prompt/shell_prompt.png %}
