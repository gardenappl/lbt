# lbt

**lbt** is a collection of command-line tools for interacting with the LBRY network. At the moment, these are written in POSIX shell, however I'm considering rewriting them in Python at some point.

Currently, there are two tools available:

* **lbt open** will open LBRY content in the user's preferred application. (think *xdg-open but for LBRY*)
* **lbt get** will simply fetch LBRY content and output it, either into a file or into standard output. (think *wget but for LBRY*)

All these tools support lbry:// protocol URLs, as well as [lbry.tv](https://lbry.tv) and [open.lbry.com](https://open.lbry.com) links.

## Why?

**lbt** might be useful if:

* You don't want to use the official Electron app (which some might call "bloated") or the lbry.tv web interface.
* You want to work with LBRY content in your own shell scripts but don't want to mess with JSON output.

### lbt open

**lbt open** uses its own configuration file to determine how to open LBRY URLs: which program to use, whether or not the program supports streaming, etc. By default, it uses more minimalist software to open files, falling back on XDG default applications if it doesn't recognise the file type.

Example of usage (with the default configuration):

`lbt open "lbry://@lbrytech#19/ieee#e"`

opens [this video](https://open.lbry.com/@lbrytech:19/ieee:e) in MPV, as a stream.

`lbt open "https://lbry.tv/@grin:4/keep-it-simple:4"`

downloads [this blog post](https://lbry.tv/@grin:4/keep-it-simple:4) and opens it using [Glow](https://github.com/charmbracelet/glow).

This behaviour can be changed using a simple config file, namely `~/.config/lbt/mimetypes`. It is based on the file's [MIME type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types).

```
#Uncomment this if you want to open files using XDG default applications:
#video/*	stream	xdg-open "$1"
#audio/*	stream	xdg-open "$1"
#text/html	stream	xdg-open "$1"
#*		save	xdg-open "$1"

video/*		stream	mpv "$1"
audio/*		stream	mpv "$1"
image/*		save	imv "$1"
text/html	stream	$BROWSER "$1"
application/pdf	save	zathura "$1"
text/markdown	save	glow -s dark "$1"
*		save
```

The simplest way to find out the MIME type is with the program's `--get-mime` parameter, for example:

```
> lbt open -m "https://open.lbry.com/@ChrisWereDigital:2/youtube-vs-peertube-thoughts-on-peertube:8"
Resolving lbry://@ChrisWereDigital#2/youtube-vs-peertube-thoughts-on-peertube#8...
MIME type is video/mp4
```

### lbt get

**lbt get** is a lower-level utility: instead of opening content with an appropriate program, it simply grabs the needed file from the network and pipes it out. It can also just print out a URL that other programs can use.

Examples:

`lbt get "https://lbry.tv/@lbry:3f/julian-chandra-joins-lbry:8" | less`

downloads [this](https://lbry.tv/@lbry:3f/julian-chandra-joins-lbry:8) document and pipes it into `less`.

`lbt get --resolve --stream "https://open.lbry.com/@Jreg:2/chonky-cat:1"`

opens an HTTP stream and prints out the URL. This URL can be opened with a browser, a video player, etc.

`lbt get -rf "lbry://@TheLinuxGamer#f/Will-we-SURVIVE-the-Oregon-Trail#d"`

downloads the video into your local download directory, and prints out the file name.

## Installation

Just download the `lbt` file and put it into your PATH.

**Dependencies:**

* LBRY (**you must make sure that `lbrynet` [is in your PATH](https://lbry.com/faq/how-to-cli)**)
* curl
* [jq](https://stedolan.github.io/jq/)


If you want to use `lbt open` as the default handler for lbry:// links:

1. Download the `lbt-open.desktop` file.
2. Put it into `~/.local/share/applications`
3. Run `xdg-mime default lbt-open.desktop x-scheme-handler/lbry`

## What's next?

Unimplemented features:

1. Commands for interacting with channels
2. Interacting with paid content.
3. Working with remote `lbrynet` daemons.

At some point, I'd like to build a "companion app" for lbt, using the [ncurses library](https://en.wikipedia.org/wiki/Ncurses). It would use the same config files, but featuring a full-fledged TUI instead of just a command-line interface.
