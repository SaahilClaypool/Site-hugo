---
date: 2017-03-17T13:22:14-04:00
draft: false
title: Windows Subsystem
type: post
description: Linux on Windows!
tags: 
- programming
- 2017
---

*jump to [configuration](#configuration)*

# What is it? 

The Windows Subsystem for Linux lets linux programs *pretend* they are running on top of a linux kernel. It maps the Linux system calls through a compatibility layer and to native windows system calls. This makes allows user-space linux programs to as if they were running natively on linux. 


# Why? 

This allows windows users to gain almost all of the power of the linux command line on windows. Command line utilities such as grep, awk, and ag are all available just as normal linux. The WSL comes with Ubuntu 14.04 (unless you are running the developer preview builds) and as a result has access to all the Ubuntu packages.

For example, installing [ag the silver searcher](https://github.com/ggreer/the_silver_searcher), a super fast grep replacement, is super simple: 

``` sh
$ apt-get install silversearcher-ag
```

### So what? 

Many programming utilities are just better on linux. Python, for example, is much harder to configure on windows. Linux and Mac are the first targets for such languages and tools. 

### Why not a virtual machine? 

Virtual machines are a pain. They are slow to startup, resource intensive, and crush the battery life of your laptop. Moreover, the real power of linux comes from the *command line*. Much of the time I spend in the command line is in the terminal. It is overkill to load up an entire virtual machine just to use a simple bash prompt.


# Okay, sign me up!

Its easy to get up and running, but a few tweaks run will make it run flawlessly. 

## Installation 

This is a high level overview of getting the WSL up and running, more information can be found at the [microsoft installation guide](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide)

1. Turn on developer mode

  Hit the 'Windows' key or icon to search. Search for 'Use developer features'
  
  Select the 'Developer Mode' option

2. Install the WSL

  Open Powershell as Administrator and enter 

  ``` sh 
  > Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
  ``` 

3. Restart your computer

4. Run Bash 

  Open the command prompt (hit the windows key / icon and search for 'CMD')
  
  run 
  ``` sh 
  > bash 
  ``` 

  and hit y when prompted.

5. Create a new username and password for linux

Thats it! you should see an icon called 'bash' which will start a command prompt running the linux terminal. The other option is to run the bash command from the normal CMD prompt. 


# Configuration

<TODO>