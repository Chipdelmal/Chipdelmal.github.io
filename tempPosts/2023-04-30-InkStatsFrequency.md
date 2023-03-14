---
title: "Stat.Ink Frequency Analysis"
tags: datasci splatoon gaming
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/statink/thumb.png
cover: /media/statink/thumb.png
---

<br>

Analyzing [stat.ink](https://stat.ink/) Splatoon weapons' data.

<!--more-->

# Intro

As mentioned in some of my previous [posts](./2022-12-28-SplatStats2.html), I've been using [stat.ink](https://stat.ink/) to upload and keep track of my [personal Splatoon 3 matches' data](https://stat.ink/@chipdelmal/spl3), what I hadn't realized is that battles' data uploaded by different players is also [available for download](https://stat.ink/downloads) in CSV files.

# Data Exploration

Once again, the first thing to do was to jump into the CSVs structures to figure out what's the available information for us to work with.


```

```

Now, the information in the files is encoded with keys that need some translation into human-readable form, so I had to have a look at the [patterns and schema](https://github.com/fetus-hina/stat.ink/wiki/Spl3-%EF%BC%8D-CSV-Schema-%EF%BC%8D-Battle) used to store the data.


# Code Description

I wanted these scripts to be part of the larger [SplatStats package](https://github.com/Chipdelmal/SplatStats), so I created an independent [StatInk class](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/StatInk.py) and auxiliary files. 

```python
GAME_MODE = {
  'nawabari': 'Turf War', 'area': 'Splat Zones', 
  'yagura': 'Tower Control', 'hoko': 'Rainmaker', 
  'asari': 'Clam Blitz'
}
LOBBY_MODE = {
  'regular': 'Regular',
  'bankara_challenge': 'Anarchy (Series)',
  'bankara_open': 'Anarchy (Open)',
  'xmatch': 'X Battle',
  'splatfest_challenge': 'Splatfest (Pro)',
  'splatfest_open': 'Splatfest (Open)'
}
STGS_DICT = {
  'yunohana': 'Scorch Gorge', 'gonzui': 'Eeltail Alley',
  'kinmedai': "Museum d'Alfonsino", 'mategai': 'Undertow Spillway',
  'namero': 'Mincemeat Metalworks', 'yagara': 'Hagglefish Market',
  'masaba': 'Hammerhead Bridge', 'mahimahi': 'Mahi-Mahi Resort',
  'zatou': 'MakoMart', 'chozame': 'Sturgeon Shipyard',
  'amabi': 'Inkblot Art Academy', 'sumeshi': 'Wahoo World',
  'hirame': 'Flounder Heights', 'kusaya': 'Brinewater Springs',
  'manta': 'Manta Maria', 'nampla': "Um'ami Ruins"
}
```


# Use Example


# Future Work


# Code Repo

* Repository: [Github Repo](https://github.com/Chipdelmal/SplatStats)
* Dependencies: [matplotlib](https://matplotlib.org/), [pandas](https://pandas.pydata.org/), [numpy](https://numpy.org/), [dill](https://pypi.org/project/dill/), [termcolor](https://pypi.org/project/termcolor/), [colorutils](https://pypi.org/project/colorutils/), [tqdm](https://pypi.org/project/tqdm/), [scipy](https://pypi.org/project/scipy/), [DateTimeRange](https://pypi.org/project/DateTimeRange/), [pywaffle](https://pypi.org/project/pywaffle/), [squarify](https://pypi.org/project/squarify/)