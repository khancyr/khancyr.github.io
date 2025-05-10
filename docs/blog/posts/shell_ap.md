---
title: "Shell completion for Tools"
date:
  created: 2019-11-20
  updated: 2019-11-20
draft: false
categories: 
    - ArduPilot
tags:
    - linux

authors:
  - khancyr
---

https://youtu.be/UpY7SZ6FXi8
Hello friends,

What has always bother me on ArduPilot development was the lack of completion on the tools we used.
Our command-line commands can be quite long and memorizing them when we aren't using them every day could be a pain.

To reduce this pain and allow newcomers to have some more help, I have made some shell completion for `waf` and `sim_vehicle.py`.
For those that don't know what it is, the completion system allows you type the start of a word or command and let the system complete it, by double press `TAB` key ! When you use a terminal that is a very convenient feature, plenty of software have some completion.

The completion scripts are now in master branch, but they should appear on 4.x series on each vehicle soon. Otherwise, I will make some documentation to put the scripts on the system to allow completion to work on each branch.

I made completion for two shell : Bash and Zsh. Most users should be on Bash, as it is the default shell on Debian like system (like Ubuntu).

## on Bash
To install the completion, you should either, source the main script on your current terminal or source the main script on your bash configuration.

### Direct use
From ArduPilot root directory, simply use :
```` bash
source ./Tools/completion/completion.bash
````
And now completion works from your terminal instance. If you close the terminal, the completion feature is removed.

### Permanent use
Edit you `.bashrc` file, it is on your Home directory but it is a hidden file (CTRL+H on Ubuntu to reveal them). Then put at the end of the file :
```` bash
source PATH_TO_ARDUPILOT_DIRECTORY/Tools/completion/completion.bash
````
where PATH_TO_ARDUPILOT_DIRECTORY is the path to ArduPilot directory (simple :crazy_face:)

### Usage
You can now abuse of your TAB key on `waf` and `sim_vehicle.py` call ! See the video at the end.

## On ZSH
Zsh don't allow live loading of completion. So you have to source the completion script in your `.zshrc` file. Like for Bash, you will find it hiding on your home !
Put at the end of the file :
```` bash
source PATH_TO_ARDUPILOT_DIRECTORY/Tools/completion/completion.zsh
````
where PATH_TO_ARDUPILOT_DIRECTORY is the path to ArduPilot directory (simple :crazy_face:). Notice the difference, the extension is `.zsh` !

## Video
I have made a simple video to show usage of completion :
[https://youtu.be/UpY7SZ6FXi8](https://youtu.be/UpY7SZ6FXi8)

## Future improvements
I am not a command-line wizard so I tried my best to get a maintainable completion feature. Not all features are available yet, but most used one's are implemented. If some magician knows how to make the completion faster/better, feel free to submit a PR !