---
title: "Birds Timelapse"
tags: ffmpeg timelapse bash bird
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/birdTlapse/back.png
cover: /media/birdTlapse/thumb.png
---

<!--more-->

# Intro

I setup a small bird feeder on my balcony and wanted to record the birds that visited throughout the day. Setting up my GoPro Session 5 was the obvious choice but revisiting my [ffmpeg timelapse script](./2023-09-21-TLapse.html) I decided to make the process a bit more straigthforward.

## GoPro Settings

For this application I used similar setting as I did for my previous timelapse applications:

```
Interval: 1 photo / 1 sec
Megapixels: 10MP Linear
Spot Meter: Off
Protune: On
  Color: Flat
  White Balance: Auto
  ISO Max: 800
  EV Comp: 0.0
  Sharpness: Medium
```

## File Structure

GoPros nest file exports into different subfolders as follows:

```
- DCIM
    - 144GOPRO
        - G0014667.JPG
        - G0014668.JPG
        ...
        - G0014987.JPG
    - 145GOPRO
    - 146GOPRO
    ...
    - 165GOPRO
```

so we will have to go through them in generating our video.

## Code

This time, instead of moving files around as I did [last time](./2023-09-21-TLapse.html), I wanted to do everything in place to make the processing easier and faster.

### Getting a full filelist

The simplest way to build the timelapse video from files named sequentially placed in different folders is to [compile a sorted list of their paths into a text file](https://gist.github.com/jkalucki/c81f8fe17599a8c9cd51b565d7dc27eb):

```bash
find . \
  -type f \( -name "*.jpg" -o -name "*.png" \) \  # Get all files with JPG or PNG extensions in subpaths
  | sort -V | \                                   # Sort naturally
  awk '{print "file \x27" $0 "\x27"}' \           # Prepends the word file and surround by quotation marks
  > list.txt                                      # Export to a txt file
```

### Generating Video

To get the filelist compiled into a video we can use the [following command](https://gist.github.com/jkalucki/c81f8fe17599a8c9cd51b565d7dc27eb):

```bash
ffmpeg \
  -f concat \         # Invokes the concatenator demuxer
  -safe 0 \           # Allows ffmpeg to read absolute and paths with special characters
  -r 50 \             # Sets the framerate to a specific value
  -i list.txt \       # Specifies the input file 
  -crf 28 \           # Sets the constant rate factor (higher means more compression but less quality)
  -preset slow \      # Sets the compression preset (slower means more compression but more processing time)
  -c:v libx265 \      # Sets the video encoder to libx265
  -pix_fmt yuv420p \  # Converts the video color format to yuv420p
  tlapse.mp4 
```

### Metadata Overlay

One final thing that I wanted to add was an overlay with the date and timestamp so that I could track the times at which birds would more often visit. This was a bit trickier but after some searching I came across the [following commands](https://stackoverflow.com/questions/47543426/ffmpeg-embed-current-time-in-milliseconds-into-video):

```bash
-vf "drawtext=fontfile='/Library/Fonts/Arial.ttf':text='%{metadata\:DateTime\:def_value}':fontcolor=white:fontsize=75:box=1:boxcolor=black@0.5:x=w-tw-100:y=h-th-100,scale=1920:-2"
```

### Full Set of Instructions

With these parts in place, all we have to do is to navigate to our root folder from the terminal and run the following commands:

```bash
# Generate filelist
find . \
  -type f \( -name "*.JPG" -o -name "*.png" \) \
  | sort -V | awk '{print "file \x27" $0 "\x27"}'\
  > list.txt
  
# Create video
ffmpeg -f concat \
  -safe 0 \
  -r 50 \
  -i list.txt \
  -vf "drawtext=fontfile='/Library/Fonts/Arial.ttf':text='%{metadata\:DateTime\:def_value}':fontcolor=white:fontsize=75:box=1:boxcolor=black@0.5:x=w-tw-100:y=h-th-100,scale=1920:-2" \
  -crf 28 \
  -preset slow \
  -c:v libx265 \
  -pix_fmt yuv420p tlapse.mp4
```


# Final Thoughts

These commands have saved quite a bit of processing time and the overlay helps quite a bit. Will put them together into a proper bash script in the future but for now this will have to do.

<center>
  <iframe width="560" height="315" src="https://www.youtube.com/embed/t3Kxexh-Kn0?si=M1IREbbzj1gmsmtu" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</center>