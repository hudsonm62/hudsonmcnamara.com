---
title: Moving to Plex Music Library from Spotify
date: 2024-04-08
draft: false
description: "Setup a Plex Music Library and download all your music from Spotify via spotDL - A Python powered Spotify downloader."
summary: "Setup a Plex Music Library and download all your music from Spotify."
tags:
  - HomeLab
  - Plex
  - Spotify
---

A while back, I spent a couple days moving off Spotify.

Main reason being, over time, they remove songs for various reasons (whether its licensing, copyright claims, artist issues, etc.), regardless of the reasons, it's gone.

This marked the final straw of my music streaming days. I'd much rather purchase albums, merch and concert tickets directly from the artists, instead of relying on Spotify for it all anyway.

---

If you already have a [Plex Media Server](https://www.plex.tv/personal-media-server/) configured, you can add a Music section to your library, and then listen with the specialized [PlexAmp](https://www.plex.tv/en-au/plexamp/) app on either your PC or Mobile, or directly through [app.plex.tv](https://app.plex.tv)

PlexAmp also supports Chromecast, Android Auto & Apple CarPlay which were mostly the features I was after when looking into open-sourced alternatives for this project.

## Setting up Plex

First create a Folder on your server (if you haven't already) to store your music, then Simply add a "New Library" in your Plex Server settings, and select "Music". Select the new folder for your library.

{{< details summary="Docker" >}}
If you're using Docker, remember to add a mapping to your music library:

```yaml
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    volumes:
      - ${MEDIA_DIR}/music:/music:ro
```

{{< /details >}}

{{< details summary="Add new Library" >}}
Settings 🔧 > under "Manage" > "Libraries"
{{< /details >}}

### Folder File Structure

You must follow a file structure similar to the below - There will be odd songs/albums/artists out depending on the state of the database, but generally everything will follow it:

{{< alert icon="link" >}}
From [Plex Support | Adding Music Media From Folders](https://support.plex.tv/articles/200265296-adding-music-media-from-folders/)
{{< /alert >}}

```txt title="Example Structure for /music"
/music
   /Pink Floyd
      /The Wall
         101 - In the Flesh.mp3
         102 - The Thin Ice.mp3
         201 - Hey You.mp3
         202 - Is There Anybody Out There.mp3
      /Wish You Were Here
         01 - Shine On You Crazy Diamond (Parts I-V).m4a
         02 - Welcome to the Machine.mp3
         03 - Have a Cigar.mp3
   /Foo Fighters
      /One By One
      /There is Nothing Left to Lose
   /U2
      /Joshua Tree
   /Various Artists
      /Guardians Of The Galaxy - Awesome Mix Vol. 1
         01 - Hooked On A Feeling.mp3
         02 - Go All The Way.mp3
         03 - Spirit In The Sky.mp3
      /The Crow - Original Motion Picture Soundtrack
         01 - Burn.mp3
         02 - Golgotha Tenement Blues.mp3
         03 - Big Empty.mp3
```

## Moving from Spotify

**I moved away from Spotify by using [SpotDL]** - A [Python](https://www.python.org/) powered CLI tool for downloading music based off of Spotify.

It uses metadata from the [Spotify WebAPI](https://developer.spotify.com/documentation/web-api) and matches it up to [YouTube Music](https://music.youtube.com/) for downloading.

```bash
pip install spotdl # --upgrade
```

There were a few things I needed to keep in mind:

1. Some Albums have multiple "CDs", which need to be accounted for in the [folder structure](#folder-file-structure), **as [SpotDL] has no way to automate this**.
2. Albums or Singles that have songs with more than 1 Artist, _typically_ need to go under a `Various Artists` folder in the root, instead of the artist folder itself.
3. Soundtracks for specific media (games, movies, etc.), also _usually_ go into `Various Artists`
4. [SpotDL] never downloads above 128kbps ([except w/ YT Premium](https://spotdl.github.io/spotify-downloader/usage/#youtube-music-premium))

Because of this, I couldn't just download entire Playlists at a time, because they _usually_ wouldn't sort properly.

The best combination I managed to pull off, was to download single Albums, by Artist `--output "{album-artist}/{album}/` one at a time. If that was too slow, I could also do entire Artists THEN manually move music as I find they need to be moved or renamed in my library.

You can still download by playlist or Spotify profile, but it's going to be **very messy** depending on how the song's album is structured, especially if the playlists consist of heaps of different artists and albums.

{{< alert icon="fire" cardColor="black" iconColor="red" textColor="white" >}}
YouTube Premium users can use their account to download songs at a higher bitrate (256kbps) with `--bitrate disable` - [See how here](https://spotdl.github.io/spotify-downloader/usage/#youtube-music-premium)
{{< /alert >}}

### Commands

- [SpotDL | Downloading](https://spotdl.github.io/spotify-downloader/usage/#downloading)

#### Album (Single Artist)

```bash title="Album (Single Artist)"
spotdl download "https://open.spotify.com/album/..." --output "{album-artist}/{album}/{track-number} - {title}.{output-ext}" --save-errors error.log
```

#### Album (Various Artists)

```bash title="Album (Various Artists)"
spotdl download "https://open.spotify.com/album/..."  --output "Various Artists/{album}/{track-number} - {title}.{output-ext}" --save-errors error.log
```

#### Album (256kbps Bitrate)

```bash title="High Quality"
spotdl download "https://open.spotify.com/album/..." --bitrate disable --output "{album-artist}/{album}/{track-number} - {title}.{output-ext}" --save-errors error.log
```

[SpotDL]: https://spotdl.github.io/spotify-downloader/

## References

- [SpotDL Documentation](https://spotdl.github.io/spotify-downloader/)
- [SpotDL CLI Options](https://spotdl.github.io/spotify-downloader/usage/#command-line-options)
- [Plex Support | Getting Started with Plex Music](https://support.plex.tv/articles/205744257-getting-started-with-plex-music/)

> Article Photo by <a href="https://unsplash.com/@blocks?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">blocks</a> on <a href="https://unsplash.com/photos/wireless-headphones-leaning-on-books-T3mKJXfdims?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>

{{< alert iconColor="white" textColor="white" cardColor="#ca0000" >}}
**Important**: Users are responsible for their actions and potential legal consequences. We do not support unauthorized downloading of copyrighted material and take no responsibility for user actions.
{{< /alert >}}
