
# So you want to use gnu screen???

### by Bailey Harrington

### Day after St Patrick's Day, 2020 — Day 3 of coronavirus-induced isolation

`gnu screen` is a terminal window manager. It allows you to do an **awful lot**. This tutorial is not going to cover that. It **is** going to teach you how to use `screen` to:

- have multiple shell sessions active *without having to have multiple shells open*;
- name your shell sessions based on what they are doing;
- have a session on a remote server that persists if you lose your connection or logout; and
  - this extends to interactive sessions, such as qlogins, which you will now be able to reaccess. WOW.

It will also show how you can make using `screen` more closely resemble your normal shell set-up. 
  
So. First things first. Open a console/terminal/shell window. However you use the command line.

(There is an implicit `Enter` after all of these commands. They won't all produce characters you see on the display, but remember to hit `Enter` anyway.)

### To create a new, named, screen instance type:


```bash
screen -R <name>
```

This will take you to a `screen` window. Depending on your system settings, this may or may not look the same as your regular shell. (If it doesn't, and you want it to, see my example `.screenrc` file at the end.)

From here, you can use the window as you normally would (mostly; I'll discuss why this is not entirely true shortly).

### Introducing the screen 'control key'
#### *Pro tip: this is not **exactly** the same as the control key on your keyboard.

You may or may not be aware that some 'full page applications' that run inside the shell (`vim`, `less`, `history`, to name a few) have their own set of keyboard commands/shortcuts that help to make these tools *extremely* powerful. Well, so does `screen`.

**However,** you can't just type these when the screen is open. You first have to *tell screen you want to use a screen command* (rather than a command line one) by hitting the control sequence.

By default with `screen`, this is `Ctrl + a`, that is, you hit the `control` and 'a' keys **simultaneously**. Then release before typing whatever command you want.

(Now, you may be aware that `Ctrl + a` is, in many shells, the sequence used to navigate to the beginning of the command line prompt. In my `.screenrc file`, I hve included a line to remap the 'control key' so this behaviour is not lost.

For now...

### Detaching a(n attached) screen

In order to detach the current screen hit `Ctrl + a` (or whatever you've set the control key to), release, and type:


```bash
:detach
```

This should return you to the shell where you invoked the screen, almost like you'd never left!

The screen has not been terminated; we've simply detached it (the equivalent of setting it aside to pick back up again later).

Now that we have at least one existing screen, there are a few more things we can do.

First:

### Listing the screens that currently exist

To do this, simply type:


```bash
screen -ls
```

This shows a list of all of the screens we have now. What if we want to go back to the one we just made (or to another one)?

### Reattaching a screen

To do this, type:


```bash
screen -d -r <name>
```

NB: The name here can just be the name you assigned. You don't need to include the number appended to its front end.

Now, to actually terminate a screen (end it forever, rather than just detach) we...

### Kill screen

But really. In order to kill the current screen you can just type:


```bash
exit
```

 Or you can go for a the control key option and hit `Ctrl + a` (or whatever you've set the control key to), release, and type:


```bash
k
```

(It stands for kill.)

That's all there is to it. Now if you try listing screens, you won't see the one you just murdered. Which is how it should be.

So those are the super-basic things about using `screen`. Now, if you actually try using it, you may notice sometimes, `screen` *behaves oddly*.

### How to fix `screen`'s, frankly bizarre, behaviour

#### First, scrolling.
You will very quickly notice that `screen` will not let you scroll up within the window to see things you've run previously, or output. And when the output from a command is long, this can be quite maddening.

There are a few ways to work around this. I'll go through them from worst to best (based on my very strong opinions).

1. Type `Ctrl + a` (or your modified control key), release, then hit `Escape`. Now you can scroll.
2. Hold down `Shift` and scroll as normal.
3. Have this line in your `.screenrc` file (it will look like gibberish):


```bash
termcapinfo xterm* ti@:te@
```

The next two things I will describe will be noticeable if you use a full-screen application, like `vim`, or others (I assume). When you exit said application, rather than disappearing (as is normal, and what should be the way it always works) the file you were just editing will remain visible, but with the command-line prompt at the bottom.

There is no good reason I know of for this.

Luckily it is easy to fix.

### Closing full-screen applications when you are done with them

There are two ways to do this. (Really they're the same, but one you can do interactively, and the other takes forethought.)

#### In an open screen window

Type `Ctrl + a` (or whatever your control sequence is), release, and type:


```bash
:altscreen on
```

If you use `vim`, this sort of syntax may look familiar to you. (If not, you should use `vim`!)

This will cause full-page things like `vim` to disappear when they close.

#### Preemptively

Place this line in your `.screenrc` file (and note it does not have the colon this time):


```bash
altscreen on
```

Sometimes, when you try to scroll, or type, or anything inside a screen (or shell) window, weird characters start to appear; they aren't the ones you're typing, they appear at an alarming rate, and there's an awful lot of `[ [ [` s.

This means something didn't quite terminate correctly.

#### Stopping the seemingly spontaneous sequence of punctuation

There are two ways you can stop this. In `screen`, you can type `Ctrl + a` (say it with me now: *or whatever you're control sequence is*), release, and then type:


```bash
Z
```

Outside of screen, (and possibly also inside it?), you can just type:


```bash
reset
```

Et voilà!

Now, for my amazing(ly short, but super handy):

# `.screenrc` file

Save the following lines inside a file named .screenrc inside your home directory.


```bash
#make sure full-page applications close on exit
altscreen on
#change the control key to z
escape ^za
#run .bash_profile on startup
shell -$SHELL
#allow scrolling
termcapinfo xterm* ti@:te@
```

You may notice two lines in here that I have not explained thus far.

`escape ^za` is the line that changes the control key. It makes the control sequence `Ctrl + z`.

`shell -$SHELL` tells screen to run my `.bash_profile` as well as `.screenrc` upon startup. Lines like this may be added for any of a number of language or other applications' start-up files. If you have edited things like your `.bash_profile` to get things 'just so', you will definitely want to add this.

And that's it. If you feel so inclined, there are MANY more things you can do with `screen`. Go crazy! (Between the man page length and coronavirus isolation, it's practically guaranteed!)

## Acknowledgements

This tutorial was made possible, in part, Carmen Amador, who provided my own introduction to `gnu screen` via a post-it note.
