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

The Rong client talks to the server process using a Unix Domain socket, usually
called `/tmp/rong-1000/default` (where `/tmp` is your machine's tempdir, `rong`
is the name of the program binary, and `1000` is your real user id number). The
protocol is built to facilitate exploring and the easiest way to figure things
out is probably `rong repl` (or, if you want to go hardcore, using `netcat -U
/tmp/rong-1000/default` will also work).

Client commands consist of a single line, with one or more words. Words are
separated by whitespace. If you want to include a whitespace in a word, quote
it using either single or double quotes. The following, commonly used, escapes
are supported: `\a` (bell), `\b` (backspace), `\e` (escape), `\f` (formfeed),
`\n` (linefeed, or newline), `\r` (carriage return), and `\t` (tab), as well as
`\x##` (were `##` are two hexadecimal digits) for characters 0–255 in Unicode.
Finally, the literal backslash, and quote characters must be escaped, using
`\\` (backslash), `\'` (single quote) and `\"` (double quote).

Each server response start with one of the keywords `OK` or `ERR` (optionally
followed by a message on the same line).

Server responses may be single or multi-line.

For a multi-line response, the first line starts with a period (turning the
leading keyword into `.OK` or `.ERR`), and the response ends with a period on a
line of its own. Lines in the response that begin with period are escaped by
prefixing them with an additional period (this to allow lines with a single
period inside the response). An extra linefeed is also added at the end of the
response (before the terminating single period) this to make it possible to
distinguish responses that ends in a linefeed from those that do not.

A successful command (for example `load`) might thus return the following
(meaning, “everything went okay, nothing further to add”):

    OK

Or, for a command that return a single line of output (like, for example, `cd`)
the output might look like this:

    OK /home/arthur

A multi-line response that is exactly equivalent to the above is:

    .OK
    /home/arthur
    .

Multi-line output from a successful command (for example `cat`) might look like
the following (note the extra period tacked onto the beginning of the line that
starts in a period, and the blank line at the end indicating that the previous line ended in a linefeed):

    .OK
    hello <- Single word.
    .. <- Single period.
    
    .

Each error messages always consist of only one line, and might look like this:

    ERR Command 'cat': No such file loaded: "./testfile1"

A command might, however, return multiple error messages in which case a
multi-line response will be sent, with one error message per line. (For
example, loading a number of non-existent files might result in the
following.):

    .ERR Command 'cat': No such file loaded: "./testfile1"
    ERR Command 'cat': No such file loaded: "./testfile2"
    ERR Command 'cat': No such file loaded: "./testfile3"
    .

<!--[eof]-->
