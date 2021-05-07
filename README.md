# lbt

**lbt** is a collection of command-line tools for interacting with the LBRY network, written in POSIX shell.

Currently, there are four tools available:

* **lbt open** will open LBRY content in the user's preferred application. (think *xdg-open but for LBRY*)
* **lbt feed** is a very simple way to see the latest content from LBRY channels. (think [sfeed](https://codemadness.org/sfeed-simple-feed-parser.html) *but for LBRY*)
* **lbt ls** will list all LBRY content that's saved on this system.
* **lbt get** will simply fetch LBRY content and output it, either into a file or into standard output. (think *wget but for LBRY*)
* **lbt rm** will delete saved LBRY content.

All these tools support lbry:// protocol URLs, as well as [Odysee](https://odysee.com) and [open.lbry.com](https://open.lbry.com) links.

## Why?

**lbt** might be useful if:

* You don't want to use the official Electron app or the lbry.tv web interface (which some might call "bloated").
* You want to work with LBRY content in your own shell scripts but don't want to mess with JSON output.

### lbt open

**lbt open** uses its own configuration file to determine how to open LBRY URLs: which program to use, whether or not the program supports streaming, etc. By default, it uses more minimalist software for specific file formats (video, audio, etc) or just saves them into your downloads folder if it doesn't recognise the file type.

**Example of usage (with the default configuration):**

`lbt open "lbry://@BrodieRobertson#5/easy-motion-how-did-i-use-vim-until-now#9"`

opens [this video](https://open.lbry.com/@BrodieRobertson:5/easy-motion-how-did-i-use-vim-until-now:9) in MPV, as a stream.

`lbt open "https://open.lbry.com/@AlexandreMarcotte777:0/brodie-robertson:1"`

downloads [this GIF](https://open.lbry.com/@AlexandreMarcotte777:0/brodie-robertson:1) and opens it using [imv](https://github.com/eXeC64/imv).

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

### lbt feed

Output the latest content from LBRY channels.

**Examples:**

`lbt feed 'https://odysee.com/@BrodieRobertson:5' 'https://odysee.com/@christitustech:5'`

prints this:

```
  2021-05-07 13:15  @christitustech   This is the future...                                 lbry://this-is-the-future...#2
  2021-05-06 21:00  @BrodieRobertson  Is Rambox An Even Better Messaging Browser?           lbry://is-rambox-an-even-better-messaging#1
  2021-05-05 21:00  @BrodieRobertson  Wayland Is The Future Of Linux, What About Now?       lbry://wayland-is-the-future-of-linux,-what#5
  2021-05-04 21:00  @BrodieRobertson  Trackma Is The Best Way To Track My Anime             lbry://trackma-is-the-best-way-to-track-my#6
  2021-05-03 21:00  @BrodieRobertson  Xinitrc, Xprofile And More, What Do They All Do       lbry://xinitrc,-xprofile-and-more,-what-do-they#6
  2021-05-03 15:33  @christitustech   First Install of Rocky Linux LIVE!                    lbry://rocky-linux-live-install#1
  2021-05-02 21:00  @BrodieRobertson  YouTube Is Still Being DESTROYED By Spam Bots         lbry://youtube-is-still-being-destroyed-by-spam#7

...
```

Just for fun, I made it so that output is compatible with [sfeed_plain](https://codemadness.org/sfeed-simple-feed-parser.html).

You can also get raw "sfeed"-style output with the `--sfeed` option, which allows you to do things like:

`lbt feed 'lbry://@ashesashescast#f' --sfeed | sfeed_curses`

![lbt + sfeed_curses](lbt-sfeed.png lbt + sfeed curses)


### lbt ls

Lists all LBRY content which is saved on your system. Accepts lots of options for showing/hiding columns of information, sorting, filtering, etc.

**Examples:**

`lbt ls --files`

prints out all LBRY content which is saved in your downloads directory, as opposed to only being stored in blob format.

`lbt ls --channel --mime --sort=size --reverse`

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

Grab the latest version of `lbt` on the [Releases page](https://gitlab.com/gardenappl/lbt/-/releases). Extract the `lbt` file and put it into your PATH.

**Dependencies:**

* LBRY (**you must make sure that `lbrynet` [is in your PATH](https://lbry.com/faq/how-to-cli)**)
* curl
* [jq](https://stedolan.github.io/jq/)
* GNU gettext (for localizations, support is incomplete)
* sed, awk
* GNU coreutils (cut, ...), util-linux (column, getopt, ...)


If you want to use `lbt open` as the default handler for lbry:// links:

1. Get the `lbt-open.desktop` file.
2. Put it into `~/.local/share/applications`
3. Run `xdg-mime default lbt-open.desktop x-scheme-handler/lbry`

## What's next?

Unimplemented features:

1. Interacting with paid content.
2. Working with remote `lbrynet` daemons.
