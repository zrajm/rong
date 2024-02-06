# Rong

Motto:

> “Write in Rong” *or* “It’s Rong to be headless.”

**This is in no way a functional editor (yet).**


## What’s Rong?

Rong is an daemonic headless text editor.

It’s *daemonic* because, when you first invoke Rong, a server process (also
known as a *daemon* in Unix parlance) is started up in the background. This
server then responsible for keeping your texts in memory and perform all
operations on them.

It’s *headless* because Rong doesn’t show the text in a graphical (or even
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


## Protocol

Socket: The Rong client talks to the server process using a Unix Domain socket,
usually called `/tmp/rong-1000/default` (where `/tmp` is your machine's
tempdir, `rong` is the name of the program binary, and `1000` is your real user
id number). The protocol is built to facilitate exploring and the easiest way
to figure things out is probably `rong repl` (or, if you want to go hardcore,
using `netcat -U /tmp/rong-1000/default` will also work).

Client request: Each command sent from the client to the server consists of a
single line, starting with a command, and followed by zero or more arguments.
Command and arguments are separated by spaces, but may themselves contain space
if quoted (using single or double quotes). The following, commonly used,
escapes are also supported: `\a` (bell), `\b` (backspace), `\e` (escape), `\f`
(formfeed), `\n` (linefeed, or newline), `\r` (carriage return), and `\t`
(tab), as well as `\x##` (were `##` are two hexadecimal digits) for characters
0–255 in Unicode. Finally, the literal backslash, and quote characters must be
escaped, using `\\` (backslash), `\'` (single quote) and `\"` (double quote).

Server response: May (optionally) start with one or two periods (for multi-line
responses), after that comes a status keyword, `OK` or `ERR` indicating on the
success or failure of the given command. For single line responses, the status
keyword may be followed by message terminating at the end of the line.

The simplest possible server response (meaning, “everything went okay, nothing
further to add”) will be returned, for example, after a successful `load`
command. It looks like this:

    OK

For commands which return a single line of output, such as the `cd` command,
the server response might look like this:

    OK /home/arthur

Or, in the case of an error:

    ERR Command 'cd': Directory does not exist

Multi-line responses (where status keyword is preceded by `.` or `..`) always
end with a single period on a line of its own (`<LF>.<LF>`). Lines in the
response that begin with a period are escaped by the server by prefixing them
with an additional period (meaning that the client should remove the first
character from each line that begins with a period, before processing the
response any further).

Single-period multi-line responses are used for multi-line output where the
existence of a final linefeed is irrelevant (such as, when listing the files
Rong currently has loaded). The output of `list` may look like this (note the
added dot before `.gitignore`):

    .OK
    ..gitignore
    README.md
    .

In the case of commands that give multiple errors, you might also see something
along the lines of (exactly what this case look like is likely to change in the
future):

    .ERR No such file loaded: "./testfile1"
    ERR No such file loaded: "./testfile2"
    ERR No such file loaded: "./testfile3"
    .

Double-period multi-line responses are used by commands that output file
content, and which need to preserve the existence (or non-existence) of any
trailing linefeed. This works by adding a blank line at the end of the response
(before the `<LF>.<LF>`), which the client is expected to remove. So, given you
have loaded a file containing `Hello, world!` (without a trailing newline), and
you output it using `cat`:

    ..OK
    Hello, world!
    .

If there is linefeed at the end of the line you will instead get:

    ..OK
    Hello, world!
    
    .

<!--[eof]-->
