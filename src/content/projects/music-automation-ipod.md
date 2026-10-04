---
title: Music Automation <2026>
summary: Local pipeline to automate my music organisation. Also iPod 
date: 2026-01-01
images: []
---

## Overview

A project to build a local, automated music library, also an iPod. Hardware and software project with quality check, efficient organisation and device syncing. 


- [`music-auto`](https://github.com/y2kr/music-auto) manages downloads and the local library.

<blockquote class="chopin-goat">
  <p>Simplicity is the final achievement.</p>
  <footer>— Chopin</footer>
</blockquote>

## The music pipeline

### `music-download`

I wanted a seamless way to download my music and have confidence in file quality, for this I built this music pipeline. 

Typical usage looks like the following: 

```bash
music-download "Kevin MacLeod - Oddities" "Koi-discovery - Prolégomènes"
```
Notice the **FREE** and **LEGAL** to distribute music I am downloading.

<video controls autoplay muted loop playsinline preload="metadata" style="display:block;width:100%;max-width:760px;height:auto;margin-inline:auto">
  <source src="/music-download-cli-demo.mp4" type="video/mp4" />
  Your browser does not support video. <a href="/music-download-cli-demo.mp4">Download the demo</a>.
</video>

Under the hood this triggers a bash wrapper that parses the request and options, then queries the local beets database. The wrapper normalises the request and beets library entries and does prefix matching to allow `Kevin MacLeod - Oddities` to match an owned edition named `Kevin MacLeod - Oddities (20th anniversary edition)` for example. For an owned match it prints:

```
already in clean, skipping: Kevin MacLeod - Oddities (matched "Kevin MacLeod - Oddities")
```

If the requests remain after the duplication check, the wrapper invokes Sockseek. The cmd looks roughly like this: 

```bash
sockseek <request-or-list> \
    [--input-type=list] \
    --output-dir <inbox-or-hidden-staging> \
    --no-write-index \
    --interactive \
    --concurrent-jobs 3 \
    --concurrent-searches 1 \
    --pref-format flac,wav,aiff,ape \
    --pref-min-bitrate 320 \
    --pref-max-bitrate 99999 \
    --pref-max-samplerate 192000 \
    --pref-max-bitdepth 32 \
    <user-supplied-options>
```


The `--pref` options are quality preferences, not requirements. They prioritise lossless format, but this doesn't guarantee the audio is genuinely lossless. These checks are later done by the importer. 


With `--interactive` sockseek outputs matching folders so the user can choose what source to download. Searches are run one at a time, while 3 download jobs can be run in parallel so an earlier download can continue while browsing what source to download for the next. 

The wrapper handles the configuration and workflow but thankfully Sockseek handles auth, request interpretation, search ranking, peer connections, queueing, transfer process and retries. Sockseek is great. Sockseek's output passes through unbuffered `sed` for some cleanup before appearing in the terminal. The selected music is downloaded to `~/Music/inbox` ready to be imported.

### `music-import`

Downloading a lossless file doesn't mean the audio is genuinely lossless. You can convert a mp3 to FLAC, making the file larger without recovering the missing audio. I wanted a way to verify this before importing to my library. 

Typical usage: 

```bash
music-import
```
<video controls autoplay muted loop playsinline preload="metadata" style="display:block;width:100%;max-width:760px;height:auto;margin-inline:auto">
  <source src="/music-import-cli-demo.mp4" type="video/mp4" />
  Your browser does not support video. <a href="/music-import-cli-demo.mp4">Download the demo</a>.
</video>


This triggers another bash wrapper that checks folders `~/Music/inbox/*`. It takes a lock so 2 importers can't run at the same time, skips folders containing incomplete downloads or files modified within the last 2mins. These stay in the inbox for a later run. 

Folders containing FLAC, WAV, AIFF, APE files the wrapper invokes flac-detective:

```bash
   flac-detective "$dir" --format json --output "$report"
 ```

flac-detective checks the audio for signs of lossy compression, such as sharp frequency cutoffs left by a MP3 encoder as converting to FLAC doesn't restore these missing freqs. The wrapper uses `jq` (cli tool to process json) to check the lossless report. **AUTHENTIC** and **WARNING** pass. Anything else rejects the whole folder to avoid importing an album with a missing track. Rejected folders are moved to `~/Music/quarantine`, flac-detective generates a HTML report for the user to review. 

Once a folder passes, the wrapper invokes `beets`: 

 ```bash
   beet import -q -I "$dir"
 ```

Beets handles release identification, metadata, artwork, duplicates and organises the files. It moves the music into the following structure:  

 ```text
   ~/Music/clean/
   └── Artist/
       └── Album/
           ├── 01 First Song.flac
           ├── 02 Second Song.flac
           └── cover.jpg
 ```

 If beets can't identify a release with confidence it stays in the inbox. Then the user can choose a release interactively:

  ```bash
   music-import --readmit "$HOME/Music/inbox/Album"
 ```

 The final step in the wrapper compares the beets album IDs from before and after the import to find new added albums. For each one it runs: 
 ```bash
   rsgain easy -q -S -m 4 "$album"
 ```

This writes ReplayGain loudness metadata so a player can adjust the playback volume between tracks and albums. This is so my ears aren't blown off for a random loud song. It doesn't re-encode or change the audio. 



## iPod build

To bring my organised music library with me I dug out my old iPod Classic 6.5 gen 2008, refurbished and modernised it with a few parts and installed Rockbox. Here's the full part list:

| Parts |
| --- |
| Flexible pry tool |
| iFlash-uDUAL |
| Click wheel |
| Click wheel and button (for the button) |
| 3000mAh battery |
| Clear light blue front faceplate |
| Custom black 1TB thin back cover |
| 2 × Samsung EVO Select 256GB microSDXC cards |
| Plastic dock bezel |
| Clear case |

<div class="gallery">
  <button type="button" data-gallery-image data-full="/files/imgs/music/pod1.jpg" aria-label="Open image 1: iPod front and back before the build">
    <img src="/files/imgs/music/pod1.jpg" alt="iPod front and back before the build" loading="lazy" />
  </button>
  <button type="button" data-gallery-image data-full="/files/imgs/music/pod2.jpg" aria-label="Open image 2: Open iPod showing its battery and circuit board">
    <img src="/files/imgs/music/pod2.jpg" alt="Open iPod showing its battery and circuit board" loading="lazy" />
  </button>
  <button type="button" data-gallery-image data-full="/files/imgs/music/pod3.jpg" aria-label="Open image 3: Finished iPod with a blue transparent front">
    <img src="/files/imgs/music/pod3.jpg" alt="Finished iPod with a blue transparent front" loading="lazy" />
  </button>
  <button type="button" data-gallery-image data-full="/files/imgs/music/pod4.jpg" aria-label="Open image 4: Finished iPod back with stickers">
    <img src="/files/imgs/music/pod4.jpg" alt="Finished iPod back with stickers" loading="lazy" />
  </button>
</div>

## My fooyin config:

I also have a custom fooyin config: 

<button class="demo-image" type="button" data-gallery-image data-full="/files/imgs/music/fooyin.png" aria-label="Open image: My fooyin music player layout">
  <img src="/files/imgs/music/fooyin.png" alt="My fooyin music player layout" loading="lazy" />
</button>

## References / links

- [music-auto](https://github.com/y2kr/music-auto)
- [Bash](https://www.gnu.org/software/bash/)
- [Sockseek](https://github.com/fiso64/sockseek)
- [sed](https://www.gnu.org/software/sed/)
- [flac-detective](https://github.com/Guillain-RDCDE/FLAC_Detective)
- [jq](https://jqlang.org/)
- [beets](https://beets.io/)
- [rsgain](https://github.com/complexlogic/rsgain)
- [Rockbox](https://www.rockbox.org/)
- [fooyin](https://fooyin.org/)

