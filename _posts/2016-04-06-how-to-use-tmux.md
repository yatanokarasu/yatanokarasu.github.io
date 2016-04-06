---
layout:         post
title:          How to use TMUX that is powerful terminal
description:    TMUX is very powerful terminal application.
keywords:       [Terminal, TMUX]
categories:     [Utilities]
tags:           [Terminal, TMUX]
---

TMUX is a very powerful terminal application and provides multiplicit terminal on a window, so we don't have to create terminal window.
This post introduce to use TMUX on Windows OS using Msys2.

<!-- more -->

# Install TMUX

In Msys2, the following command need to install `tmux`:

~~~
$ pacman -S tmux
~~~


# Overview TMUX

TMUX overview is as follows:

~~~
[Terminal Window(ex. putty, mintty)]
  `-- [tmux session 1]
        |-- [tmux window 1]
        |     |-- [tmux pane 1]
        |     |-- [tmux pane 2]
        |     |     :
        |     `-- [tmux pane 1]
        |-- [tmux window 2]
        |--    :
        `-- [tmux window N]

[Terminal Window(ex. putty, mintty)]
  `-- [tmux session 2]
        |-- [tmux window 1]
        |     |-- [tmux pane 1]
        |     |-- [tmux pane 2]
        |     |     :
        |     `-- [tmux pane 1]
        |-- [tmux window 2]
        |--    :
        `-- [tmux window N]
    :
~~~


# Usage

For details and tmux commands, you can find out `man tmux` or `tmux list-commands`.


## Create new session

The following command starts TMUX session (it it the same as `tmux new-session`):

~~~
$ tmux
~~~

{% include modal-image
    id="tmux-session-first"
    src="/images/posts/20160406/tmux-first-session.png"
    class="half center"
%}

You can see TMUX status bar appearing the bottom of terminal window.
At left-bottom corner, it represents `[Session No.] Window No:Window Nmae` (`*` is as __active window__).

Next, open new terminal window, run `tmux` and you can see that tmux status is as `[1] 0:bash*`.
Currently TMUX status is as follows:

~~~
[Terminal Window]
  `-- [tmux session 0]
        `-- [tmux window 0] *active

[Terminal Window]
  `-- [tmux session 1]
        `-- [tmux window 0] *active
~~~

The following command (it is same as `tmux list-sessions`) show you current session list:

~~~
$ tmux ls
0: 1 windows (created Wed Apr  6 22:20:40 2016) [81x20] (attached)
1: 1 windows (created Wed Apr  6 22:34:55 2016) [81x20] (attached)
~~~


## Create new window

You can create new TMUX window using by `Ctrl-b c` key bind.
For example, TMUX status will become `[0] 0:bash- 1:bash*` if use the key bind on `session 0`.
TMUX status is as follows in this time:

~~~
[Terminal Window]
  `-- [tmux session 0]
        |-- [tmux window 0] -inactive
        `-- [tmux window 1] *active

[Terminal Window]
  `-- [tmux session 1]
        `-- [tmux window 0] *active
~~~

You can see that he number of window of `session 0` is incremented:

~~~
$ tmux ls
0: 2 windows (created Wed Apr  6 22:20:40 2016) [81x20] (attached)
1: 1 windows (created Wed Apr  6 22:34:55 2016) [81x20] (attached)
~~~

To switch window, use key bind `Ctrl-b p`(Previous Window) and `Ctrl-b n`(Next Window) or `Ctrl-b 0 to 9`.
If you want to close active window, use key bind `Ctrl-b &`.


# Major commands and key binds

Also check them `man tmux`.

| key                                                   | means                                                 |
|:------------------------------------------------------|:------------------------------------------------------|
| `tmux attach -t <session no>`                         | Attach specified session                              |
| `Ctrl-b d`                                            | Detach current session                                |
| `Ctrl-b D`                                            | Choose a client to detach                             |
| `Ctrl-b (`                                            | Switch the attached client to the previous session    |
| `Ctrl-b )`                                            | Switch the attached client to the next session        |
| `tmux kill-session -t <session no>`                   | Kill specified session                                |
| `tmux kill-server`                                    | Kill all session                                      |
|---
| `Ctrl-b c`                                            | Create new window                                     |
| `Ctrl-b &`                                            | Kill the current window                               |
| `Ctrl-b f`                                            | Prompt to search in open windows                      |
| `Ctrl-b l`                                            | Move to the previously selected window                |
| `Ctrl-b p`                                            | Change to the previous window                         |
| `Ctrl-b n`                                            | Change to the next window                             |
| `Ctrl-b 0 to 9`                                       | Select window 0 to 9                                  |
|---
| `Ctrl-b %`                                            | Split the current pane into two, left and right       |
| `Ctrl-b "`                                            | Split the current pane into two, top and bottom       |
| `Ctrl-b x`                                            | Kill the current pane                                 |
| `Ctrl-b !`                                            | Kill the current pane out of the window               |
| `Ctrl-b o`                                            | Select the next pane in the current window            |
| `Ctrl-b UP, RIGHT, DOWN, LEFT`                        | Select the pane above, right, below, left             |
| `Ctrl-b Ctrl-UP, Ctrl-RIGHT, Ctrl-DOWN, Ctrl-LEFT`    | Resize the current pane                               |
{: rules="groups"}

