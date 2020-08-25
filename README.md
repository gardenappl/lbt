# lbt

**lbt** is a collection of command-line tools for interacting with the LBRY network, written in POSIX shell.

Currently, there are four tools available:

* **lbt open** will open LBRY content in the user's preferred application. (think *xdg-open but for LBRY*)
* **lbt get** will simply fetch LBRY content and output it, either into a file or into standard output. (think *wget but for LBRY*)
* **lbt ls** will list all LBRY content that's saved on this system.
* **lbt rm** will delete saved LBRY content.

All these tools support lbry:// protocol URLs, as well as [lbry.tv](https://lbry.tv) and [open.lbry.com](https://open.lbry.com) links.

## Why?

**lbt** might be useful if:

* You don't want to use the official Electron app or the lbry.tv web interface (which some might call "bloated").
* You want to work with LBRY content in your own shell scripts but don't want to mess with JSON output.

### lbt open

**lbt open** uses its own configuration file to determine how to open LBRY URLs: which program to use, whether or not the program supports streaming, etc. By default, it uses more minimalist software for specific file formats (video, audio, etc) or just saves them into your downloads folder if it doesn't recognise the file type.

**Example of usage (with the default configuration):**

`lbt open "lbry://@BrodieRobertson#5/easy-motion-how-did-i-use-vim-until-now#9"`

opens [this video](https://open.lbry.com/@BrodieRobertson:5/easy-motion-how-did-i-use-vim-until-now:9) in MPV, as a stream.

`lbt open "https://open.lbry.com/@lbry:3f/Jeremy-LBRY-Inc-AMA:4"`

downloads [this GIF](https://open.lbry.com/@lbry:3f/Jeremy-LBRY-Inc-AMA:4) and opens it using [imv](https://github.com/eXeC64/imv).

This behaviour can be changed using a simple config file, namely `~/.config/lbt/mimetypes`. It is based on the file's [MIME type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types).

```
#Uncomment this if you want to open files using XDG default applications:
#video/*	stream	gtk-launch $(xdg-mime query default $mimetype) "$1"
#audio/*	stream	gtk-launch $(xdg-mime query default $mimetype) "$1"
#text/html	stream	gtk-launch $(xdg-mime query default $mimetype) "$1"
#*		save	gtk-launch $(xdg-mime query default $mimetype) "$1"

video/*		stream	mpv "$1"
audio/*		stream	mpv "$1"
image/*		save	imv "$1"
text/html	stream	$BROWSER "$1"
application/pdf	save	zathura "$1"
*		save 		# empty third column means "save to downloads directory"
```

The config file is automatically generated on the first run. The simplest way to find out the MIME type is with the program's `--get-mime` parameter, for example:

```
> lbt open -m "https://lbry.tv/@grin:4/keep-it-simple:4"
Resolving lbry://@grin#4/keep-it-simple#4...
MIME type is text/markdown
```

### lbt get

**lbt get** is a lower-level utility: instead of opening content with an appropriate program, it simply grabs the needed file from the network and pipes it out. It can also just print out a URL that other programs can use.

**Examples:**

`lbt get "https://lbry.tv/@lbry:3f/julian-chandra-joins-lbry:8" | less`

downloads [this](https://lbry.tv/@lbry:3f/julian-chandra-joins-lbry:8) document and pipes it into `less`.

`lbt get --resolve --stream "https://open.lbry.com/@Jreg:2/chonky-cat:1"`

opens an HTTP stream and prints out the URL. This URL can be opened with a browser, a video player, etc.

`lbt get -rf "lbry://@TheLinuxGamer#f/Will-we-SURVIVE-the-Oregon-Trail#d"`

downloads the video into your local download directory, and prints out the file name.

### lbt ls

Lists out all LBRY content which is saved on your system. Accepts lots of options for showing/hiding columns of information, sorting, filtering, etc.

**Examples:**

`lbt get --files`

prints out all LBRY content which is saved in your downloads directory, as opposed to only being stored in blob format.

`lbt get --channel --mime --sort=size --reverse`

prints out additional columns for the channel name and MIME type, and sorts files from largest to smallest.

### lbt rm

**Examples:**

`lbt rm "#aec4347ca0eaefea5eb92d4a51b25451d0581996" --file`

will delete [this](https://open.lbry.com/@davidpakman:7/how-the-internet-destroyed-your-brain:a) video from your LBRY library, as well as from your downloads folder.

`lbt rm "https://lbry.tv/@lbry:3f/fullscreenrelease:7" --no-blobs -f`

will delete the video from your downloads folder, but not from your LBRY library.

## Installation

### Arch Linux

Users of Arch and its derivatives can use the [AUR package](https://aur.archlinux.org/packages/lbt/), maintained by me.

### Other systems

Just download the `lbt` file and put it into your PATH.

**Dependencies:**

* LBRY (**you must make sure that `lbrynet` [is in your PATH](https://lbry.com/faq/how-to-cli)**)
* curl
* [jq](https://stedolan.github.io/jq/)
* GNU gettext (for localizations, support is incomplete)
* sed, awk
* GNU coreutils (cut, ...), util-linux (column, getopt, ...)


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
