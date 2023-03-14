---
title: "Stat.Ink Splatoon Data"
tags: datasci splatoon gaming dataviz
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/statink/chill2.png
cover: /media/statink/PolarHalf%20-%20Chill%20Season%202022.png
---

<br>

Integrating [stat.ink](https://stat.ink/) Splatoon weapons' frequency data into [SplatStats](https://github.com/Chipdelmal/SplatStats).

<!--more-->

# Intro

As mentioned in some of my previous [posts](./2022-12-28-SplatStats2.html), I've been using [stat.ink](https://stat.ink/) to upload and keep track of my [personal Splatoon 3 matches' data](https://stat.ink/@chipdelmal/spl3); what I hadn't realized, however, is that battles' data uploaded by different players is also [available for download](https://stat.ink/downloads) in CSV files. These large data files provide with a new interesting way to analyze the status of the community at a higher level, so I started coding some scripts to check for trends and interesting facts.

# Data Exploration

Once again, the first thing to do was to jump into the CSVs structures to figure out what's the available information for us to work with:


```
season, period, game-ver, lobby, mode, stage, time, win, knockout, rank, power, 
alpha-inked, alpha-ink-percent, alpha-count, alpha-color, alpha-theme, 
bravo-inked, bravo-ink-percent, bravo-count, bravo-color, bravo-theme, 
A1-weapon, A1-kill-assist, A1-kill, A1-assist, A1-death, A1-special, A1-inked, A1-abilities, 
A2-weapon, A2-kill-assist, A2-kill, A2-assist, A2-death, A2-special, A2-inked, A2-abilities, 
A3-weapon, A3-kill-assist, A3-kill, A3-assist, A3-death, A3-special, A3-inked, A3-abilities, 
A4-weapon, A4-kill-assist, A4-kill, A4-assist, A4-death, A4-special, A4-inked, A4-abilities, 
B1-weapon, B1-kill-assist, B1-kill, B1-assist, B1-death, B1-special, B1-inked, B1-abilities, 
B2-weapon, B2-kill-assist, B2-kill, B2-assist, B2-death, B2-special, B2-inked, B2-abilities, 
B3-weapon, B3-kill-assist, B3-kill, B3-assist, B3-death, B3-special, B3-inked, B3-abilities, 
B4-weapon, B4-kill-assist, B4-kill, B4-assist, B4-death, B4-special, B4-inked, B4-abilities, 
medal1-grade, medal1-name, medal2-grade, medal2-name, medal3-grade, medal3-name
```

Now, the information in the files is encoded with keys that need some translation into human-readable form, so I had to have a look at the [patterns and schema](https://github.com/fetus-hina/stat.ink/wiki/Spl3-%EF%BC%8D-CSV-Schema-%EF%BC%8D-Battle) used to store the data but, all in all, it was pretty much smooth sailing in terms of putting together the pre-processing end of the codebase.


# Code Description

I wanted these scripts to be part of the larger [SplatStats package](https://github.com/Chipdelmal/SplatStats), so I created an independent [StatInk class](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/StatInk.py) and auxiliary files (for [constants](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/statInkConstants.py) and [auxiliary functions](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/statInkStats.py)), which are wrapped together in the same [pypi package](https://pypi.org/project/SplatStats/).

## Reading Files

The first thing to do in its initializer is to take the list of CSV files and read them into a large dataframe. As we read the data, it's a generally a good idea to parse it in their corresponding data types, so we define a dictionary to be called when loading the files:

```python
STATINK_DTYPES = {
  # Match information -------------------------------------------------------
  'season': 'string', 'period': 'datetime64', 'game-ver': 'string', 
  'lobby': 'string', 'mode': 'string', 'stage': 'string',
  'time': 'uint16', 'win': 'string', 'knockout': 'string', 
  'rank': 'string', 'power': 'float',
  # Team results ------------------------------------------------------------
  'alpha-inked': 'Int64',         'bravo-inked': 'Int64',
  'alpha-ink-percent': 'float',   'bravo-ink-percent': 'float',
  'alpha-count': 'Int64',         'bravo-count': 'Int64',
  'alpha-color': 'string',        'bravo-color': 'string',
  'alpha-theme': 'string',        'bravo-theme': 'string',
  # Team A stats ------------------------------------------------------------
  'A1-weapon': 'string',   'A1-kill-assist': 'Int8',
  'A1-kill': 'Int8',       'A1-assist': 'Int8', 
  'A1-death': 'Int8',      'A1-special': 'Int8',
  'A1-inked': 'Int16',     'A1-abilities': 'object',
  ...
  'B4-weapon': 'string',   'B4-kill-assist': 'Int8',
  'B4-kill': 'Int8',       'B4-assist': 'Int8', 
  'B4-death': 'Int8',      'B4-special': 'Int8',
  'B4-inked': 'Int16',     'A4-abilities': 'object',
  # Medals ------------------------------------------------------------------
  'medal1-name': 'string', 'medal1-grade': 'string',
  'medal2-name': 'string', 'medal2-grade': 'string',
  'medal3-name': 'string', 'medal3-grade': 'string'
}
```

With the dictionary in place, we can read the CSV files from a directory with the following class initializer:

```python
def __init__(self, resultsPaths, fNamePattern='*-*-*.csv'):
    self.fPaths = glob(path.join(resultsPaths, fNamePattern))
    # Read csv's into a dataframe ----------------------------------------
    rawDFList = [
        pd.read_csv(
            f, parse_dates=['period'], dtype=ink.STATINK_DTYPES, 
        ) for f in self.fPaths
    ]
    self.rawResults = pd.concat(rawDFList)
    # Clean dataframe to standard names/times ----------------------------
    self.battlesResults = self.cleanBattlesDataframe(self.rawResults)
```

Where the last line will be explained in the following section of this post.

## Cleaning Data

Now, the second part after reading the files is to clean our dataframe a bit so that the entries are stored in human-readable form. The [lobby and game mode](https://github.com/fetus-hina/stat.ink/wiki/Spl3-%EF%BC%8D-CSV-Schema-%EF%BC%8D-Battle) are somewhat simple, as they  involve just a few values, so these were copied by hand from the [schema](https://github.com/fetus-hina/stat.ink/wiki/Spl3-%EF%BC%8D-CSV-Schema-%EF%BC%8D-Battle):

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
```

The [stages](https://stat.ink/api-info/stage3) dictionary is also somewhat small, so it could have been also copied by hand:

```python
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

The [weapons schema](https://stat.ink/api-info/weapon3), on the other hand, is quite large, so it was easier to download the available [json file](https://stat.ink/api/v3/weapon?full=1), get the pairwise combinations of names, and paste them into the script:


```python
WPNS_DICT = {
  '52gal': '.52 Gal',
  '96gal': '.96 Gal',
  '96gal_deco': '.96 Gal Deco',
  'bold': 'Sploosh-o-matic',
  'bold_neo': 'Neo Sploosh-o-matic',
  'bottlegeyser': 'Squeezer',
  'heroshooter_replica': 'Hero Shot Replica',
  'jetsweeper': 'Jet Squelcher',
  ...
  'nautilus47': 'Nautilus 47',
  'splatspinner': 'Mini Splatling',
  'splatspinner_collabo': 'Zink Mini Splatling',
  'campingshelter': 'Tenta Brella',
  'parashelter': 'Splat Brella',
  'spygadget': 'Undercover Brella',
  'lact450': 'REEF-LUX 450',
  'tristringer': 'Tri-Stringer'
}
```

And this is pretty much it. We just fix and change some values for `NaN`, and we have our full dataframe ready for analysis! I won't post the code for this section as it is just a bunch of replacements of dataframe columns, but it can be accessed [here](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/StatInk.py#L32) for reference.

# Use Example

One quick and simple thing to do with the new dataframe is to get a count of how many times a weapon was used in a match, along with the win rate. This can be achieved in a number of different ways and I chose to go about it in a slightly more complicated way so that it could be more flexible for future uses (such as weapon dominance over other weapons). For this reason, I'll just be showing a brief snippet of the [full code](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/dev/statInkMatrix.py) where we read the files, filter them on a Splatoon season, and calculate the times a weapon has won, lost, and participated in matches:

```python
SEASON = 'Chill Season 2022'
DATA_PATH = '/home/chipdelmal/statInk/'
# Reading data ----------------------------------------------------------------
statInk = splat.StatInk(path.join(DATA_PATH, 'battle-results-csv'))
btls = statInk.battlesResults
# Filtering by season ---------------------------------------------------------
btlsFiltered = btls[btls['season']==SEASON]
# Calculate frequency matrix --------------------------------------------------
(names, matrix) = splat.calculateDominanceMatrixWins(btlsFiltered)
# Calculate Win/Lose/Matches --------------------------------------------------
(totalW, totalL) = (np.sum(matrix, axis=1), np.sum(matrix, axis=0))
totalM = totalW + totalL
```

Now, with some re-shapping, styling and our original [polar barcharts](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/plots.py#L890) function, we can compare the use frequency of the weapons in matches, along with their win/lose rates on the two current Splatoon 3 seasons:

<center>
  <img width="49%" src="/media/statink/PolarHalf%20-%20Drizzle%20Season%202022.png"><img width="49%" src="/media/statink/PolarHalf%20-%20Chill%20Season%202022.png">
</center>

Again, I'll go into more detail in a future post where the way the dominance matrix is calculated will be detailed, along with some other filtering and analysis.

# Facts and Future Work

I'm currently working in some "Seasonal Summary Panels", where we'll be able to get a snapshot of what the community was playing the most! For example, in these panels we can see the matches counts spikes in "Turf War" when Splatfests took place, along with the Splash-o-Matic getting a lot of use throught all match modes, whereas other weapons (such as the Aerospray RG) getting some niche uses:

<div class="swiper my-3 swiper-demo swiper-demo--0">
  <div class="swiper__wrapper">
    <div class="swiper__slide"><img src="/media/statink/ChillSeason2022.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/statink/DrizzleSeason2022.png" style="width:100%;"></div>
  </div>
  <!-- <div class="swiper__pagination"></div> -->
  <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
  <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
  <!-- <div class="swiper-scrollbar"></div> -->
</div>

<script>
  {%- include scripts/lib/swiper.js -%}
  var SOURCES = window.TEXT_VARIABLES.sources;
  window.Lazyload.js(SOURCES.jquery, function() {
    $('.swiper-demo--0').swiper(); $('.swiper-demo--1').swiper();
    $('.swiper-demo--2').swiper(); $('.swiper-demo--3').swiper();
    $('.swiper-demo--4').swiper({ animation: false });
  });
</script>

Some interesting facts I've been able to see so far are:

*  The total number of matches rose significantly from the first to the second season (Drizzle to Chill).
*  Splash-o-Matic has been dominating across game-modes (which is unsurprising due to its use in competitive play).
*  Splattershots (my main) are also fairly well represented in the community. With the Tentatek getting a lot of use ever since it's addition to the game in Chill Season.
*  There are a lot of players doing Series matches. This was a bit surprising to me at first but makes sense with the context that the dataset is comprised of uploads of a very specific sub-segment of the Splatoon playerbase: "players who play enough Splatoon to be compelled to setup the scripts to upload their battles to online servers", which is likely to be on the competitive end of the gaming spectrum.

# Code Repo

* Repository: [Github Repo](https://github.com/Chipdelmal/SplatStats)
* Dependencies: [matplotlib](https://matplotlib.org/), [pandas](https://pandas.pydata.org/), [numpy](https://numpy.org/), [dill](https://pypi.org/project/dill/), [termcolor](https://pypi.org/project/termcolor/), [colorutils](https://pypi.org/project/colorutils/), [tqdm](https://pypi.org/project/tqdm/), [scipy](https://pypi.org/project/scipy/), [DateTimeRange](https://pypi.org/project/DateTimeRange/), [pywaffle](https://pypi.org/project/pywaffle/), [squarify](https://pypi.org/project/squarify/)