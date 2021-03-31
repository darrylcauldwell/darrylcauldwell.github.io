+++
title = "Customizing Zsh Prompt"
date = "2021-01-27"
description = "Customizing my Zsh prompt with colour and git info"
tags = [
    "macos"
]
categories = [
    "handy hacks"
]
thumbnail = "clarity-icons/code-144.svg"
+++
macOS 10.15 Catalina changed default Terminal from bash to zsh. When I first moved I tried Powerline extension but it was always a bit slow and really I all I need is a custom prompt. So today I revisited this and created a custom prompt for Zsh and learned a little along the way.

First I create an empty ~/.hushlogin to suppress the status message at the start of each Terminal session.

```
touch ~/.hushlogin
```

[Prompt expansion](http://zsh.sourceforge.net/Doc/Release/Prompt-Expansion.html) can be used to customize the experience. The default prompt in zsh is %n%m%#.
* %n username
* %m hostname
* %# shell state, # when privileges, otherwise % 

So my default login prompt would show
```
dcauldwell@dcauldwell-a01 ~ %
```

PROMPT is variable so can be be set interactively give you immediate feedback and when you're happy add to a startup script like ~/.zshrc so it loads automagically every time.

The options I choose to set:
* %(?.%F{green}√.%F{red}?%?) will show  previous Exit Code
* %F{blue} defines foreground colour
* %/ current working directory
* %F{cyan} defines foreground colour
* %# shell state

Zsh ships with a framework for getting information from version control systems, called vcs_info. When working within Git repositories in Terminal its useful to know which is active branch your editing.

We make the vcs_info framework by loading it:

```
autoload -Uz vcs_info
```

Run vcs_info just before a prompt is displayed (precmd)

```
precmd_vcs_info() { vcs_info }
precmd_functions+=( precmd_vcs_info )
```

Enable option for PROMPT_SUBST which turns on command substitution in the prompt:

```
setopt prompt_subst
```


We can then call its methods and consume within prompt, my new ~/.zshrc file looks like:

```
cat ~/.zshrc
autoload -Uz vcs_info
precmd_vcs_info() { vcs_info }
precmd_functions+=( precmd_vcs_info )
setopt prompt_subst
PROMPT='%(?.%F{green}√.%F{red}?%?)%F{blue} %/ %F{yellow}${vcs_info_msg_0_}%F{magenta} %# '
```

![This is the end state](/images/zsh-profile.jpg)
