---
title: "Stat.Ink Weapon Matrix"
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



# Dominance Matrix


First we get the total number of battles, who won the match (defaults to `True` if team `alpha` won), and the [weapons used by each team](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/statInkStats.py#L82):

```python
btlsNum = btls.shape[0]
# Get weapons used by each team
tmsWpns = getTeamsWeapons(btls)
(alpha, bravo) = (tmsWpns['alpha'], tmsWpns['bravo'])
# Get the winner of the match
winAlpha = list(btls['win'])
```

Next, we the [list of weapons that appear in the matches](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/statInkStats.py#L110):

```python
wNames = getWeaponsSet(btls)
wpnsNumbr = len(wNames)
```

Now comes the fun part. The goal is to generate a squared matrix that will contain the information of how many times a given weapon (row) has beaten another weapon in a match (column). The order of the appearance of the weapons in the matrix corresponds to the sorting in the `wNames` list, which is alphabetical by default (although a list can be provided to [the function](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/statInkStats.py#L10), if so desired). With this in mind, we iterate over each match (controlled by the battle index `bix`) where we get the positions of the weapons on both teams as they appear in our `wNames` list, and who won the match. If `alpha` won, we iterate over their weapons' rows incrementing by one all of the columns that correspond to a weapon that exists in team `bravo`; conversely, if `bravo` won, we go through all of the rows of the teams' weapons and increment all the columns corresponding to weapons in `alpha` by one:

```python
# Initialize empty matrix
domMtx = np.zeros((wpnsNumbr, wpnsNumbr))
for bix in range(btlsNum):
    # Get names for weapons in both teams
    (wpnsNmA, wpnsNmB) = (list(alpha.iloc[bix]), list(bravo.iloc[bix]))
    # Get indices for weapons in both teams
    (wpnsIxA, wpnsIxB) = (
        [wNames.index(w) for w in wpnsNmA], 
        [wNames.index(w) for w in wpnsNmB]
    )
    # Fill the corresponding row/column combination
    if winAlpha[bix]:
        # Team Alpha won
        for ixA in wpnsIxA:
            for ixB in wpnsIxB:
                domMtx[ixA, ixB] = domMtx[ixA, ixB]+1
    else:
        # Team Bravo won
        for ixB in wpnsIxB:
            for ixA in wpnsIxA:
                domMtx[ixB, ixA] = domMtx[ixB, ixA]+1
```

As an example, let's inspect a single match and how it gets converted to the frequency matrix. We get the dataframe entry, inspect the match's weapons, and who won:


```python
match = btls.iloc[0:1]
(weapons, alphaWon) = (splat.getTeamsWeapons(match), match['win'])
(weapons['alpha'], weapons['bravo'], alphaWon)


out: 
  (
    A1-weapon       "Z+F Splat Charger"
    A2-weapon     "Neo Sploosh-o-matic"
    A3-weapon          "Splash-o-matic"
    A4-weapon          "Splash-o-matic"
    Name: 0, dtype: object,

    B1-weapon          "Splash-o-matic"
    B2-weapon  "Forge Splattershot Pro"
    B3-weapon                "N-ZAP 89"
    B4-weapon     "Tri-Slosher Nouveau"
    Name: 0, dtype: object,

    False
  )
```

Ok, so we have the weapons used by both teams, and we can see that team `bravo` won. Now, let's calculate dominance matrix of the match:

```python
splat.calculateDominanceMatrix(btls.iloc[0:1])

out: 
  (
    [
      "Forge Splattershot Pro",
      "N-ZAP '89",
      "Neo Sploosh-o-matic",
      "Splash-o-matic",
      "Tri-Slosher Nouveau",
      "Z+F Splat Charger"
    ],
    array([
      [0, 0, 1, 2, 0, 1],
      [0, 0, 1, 2, 0, 1],
      [0, 0, 0, 0, 0, 0],
      [0, 0, 1, 2, 0, 1],
      [0, 0, 1, 2, 0, 1],
      [0, 0, 0, 0, 0, 0]
    ])
  )
```

Let's have a look at the "Forge Splattershot Pro". This weapon, which has an index `0` in the matrix, was part of team `bravo`, so its row should have positive entries in the columns that match the weapons used by team `alpha` (as presented in the `wNames` list). We can see that its row has a one in the second column, which corresponds to the "Neo Sploosh-o-matic", a 2 in the third column for the two "Splash-o-matics" present in the opossing team, and a 1 in the last column for the "Z+F Splat Charger". In contrast, let's have a look at a weapon the "New Sploosh-o-matic". This weapon's row (index 2) is empty but its column (same index 2) has a value of 2 in the rows of the weapons used by team `alpha` (0, 1, 3, and 4).

When the dominance matrix function is fed a whole dataframe of matches, it repeats the same process for all the battles and weapons; which results in a large squared and assymetric matrix with the winning frequencies for the combinations that have appeared over the dataset.

# Normalized Matrix

Having our frequency matrix already calculated, we can [normalize the results](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/statInkStats.py#L43) so that the entries are fractions of wins/losses between the weapons. Doing this is relatively easy, as we can take a weapon's row vector, and divide it by its column vector (as they are sorted in the same order). In an extra step, we subtract 1 to each entry so that it centers around 0, meaning that 0 would represent neither weapon "dominates" the other one, while positive numbers mean the weapon in the row dominates the one in the column, and negative numbers represent the inverse case:


```python
# Initialize matrix
mSize = len(domMtx)
tauW = np.zeros((mSize, mSize))
# Iterate through rows
for (ix, _) in enumerate(wNames):
    winsDiff = domMtx[ix]/domMtx[:,ix]
    tauW[ix] = winsDiff
tauX = np.nan_to_num(tauW)-1
```

Finally, in an optional step, we can sort the matrix by the rank frequency of weapons that each entry dominates (the reason for doing this will become clearer when we plot the results):

```python
sorting = list(np.argsort([np.sum(r>0) for r in tauX]))[::-1]
(tauS, namS) = (tauX[sorting][:,sorting], [wNames[i] for i in sorting])
```

# Dominance Plot


We are finally ready to generate our [plot](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/statInkPlots.py#L102). Thankfully, we took enough steps in our previous processes as to make this as streamlined as possible. In essence, we are simply doing a matrix plot, capping the minimum/maximum allowed values and applying a [custom colorscale](https://github.com/Chipdelmal/SplatStats/blob/main/SplatStats/colors.py#L226) in the form of:

```python
(fig, ax) = plt.subplots(figsize=(20, 20))
ax.matshow(sMatrix, vmin=vRange[0], vmax=vRange[1], cmap=cmap)
```

A heuristically-good value for the values range is `(-0.75, 0.75)` for season-span datasets, with duotone colormaps that go through white at value 0 being good for contrasting the win/loss ratios.

```python
counts = [np.sum(r>0) for r in sMatrix]
(mWpnWins, mWpnLoss) = (
    np.sum(mMatrix, axis=1)/4, 
    np.sum(mMatrix, axis=0)/4
)
tVect = (mWpnWins+mWpnLoss)[sSort]
lLabs = [
    '{} ({})'.format(n, c) 
    for (n, c) in zip(sNames, counts)
]
tLabs = [
    '({}{}) {}'.format(c, scaler[0], n)
    for (n, c) in zip(sNames, [int(i) for i in tVect/scaler[1]])
]

ax.set_xticks(np.arange(0, len(sNames)))
ax.set_yticks(np.arange(0, len(sNames)))
ax.set_xticklabels(tLabs, rotation=90, fontsize=12.5)
ax.set_yticklabels(lLabs, fontsize=12.5)
```