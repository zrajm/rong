# Rong

> “Write in Rong” *or* “It’s Rong to be headless.”

**This is in no way a functional editor (yet).**


## What’s Rong?

Rong is an daemonic headless text editor.

It’s *Daemonic* because, when you first invoke Rong, a server process (also
known as a *daemon* in Unix parlance) is started up in the background. This
server then responsible for keeping your texts in memory, and is used to load,
edit, and save them.

It’s *Headless* because Rong doesn’t show you your text in a graphical (or even
terminal) window. Instead subcommands are used to load, manipulate, and save
your texts. When you invoke Rong, it talks to the server, which changes the
text for you.


## Development

To check out this project and install the recommended Git hooks, use:

    git clone git@github.com:zrajm/rong.git
    cd rong
    git config --local core.hooksPath .githooks/

This will add a `pre-commit` hook that tries to make sure you have updated the
`$VERSION` and `$VERSION_DATE` variables in Rong before committing.

<!--[eof]-->
