---
title: "Lego Portrait Optimizer (Coming Soon)"
tags: artsci image-processing pixel color lego optimization knapsack run-length
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/lego/sami_Lego.png
cover: /media/lego/sami_Lego.png
---

<br>

<!--more-->

Some time ago I was talking to a friend who had visited a Lego store where he got the kit to make his own portrait (as depicted [here](https://www.lego.com/en-us/product/mosaic-maker-40179)). The idea looked pretty cool and we started wondering if there'd be a better way to do it. A couple of things immediately popped-out to us: it's only five shades of color, and it uses 1-by-1 size blocks. This started the idea of coding an algorithm that could generate these kind of images with more colors and that included longer blocks. 

<center><img src="/media/lego/2022-04-02_10-43.png" style="width:25%;"><img src="/media/lego/2022-04-02_10-43_1.png" style="width:25%;"></center>

Initially, we thought it'd be a somewhat run-of-the-mill optimization problem, but it took some image processing, run-length encoding, solving a multiple-knapsack problem, and some image decoding to get the whole task done!




# General Questions and Overview of the Solution

Given a pixel-based image and a pool of colored blocks:

1. Can I recreate the image with my available blocks?
2. How should I arrange them so that I don't run out without the image being completed?

Some things to note when simplifying the problem-space are:

* The blocks are, in escence, one dimensional (the length of each block is the goal and constraint).
* Each color set is completely independent from the others (solving each color is a different problem).
* Each color can be mapped to a 1D run-length encoded vector (where each entry is the length of the gap to fill).
* Each entry in these 1D vectors is, in escence, either a subset-sum or a knapsack problem (in which we want to get as close as possible to maximum length without overshooting it, with the blocks we have).
* When trying to solve a full color vector, we are solving a multiple-knapsack problem, where the length is the "weight" and the "value" is how much preference we give to a certain type of block in the pool (given its length).


# Code Dev

## Image Preprocessing

* __Goal:__ *To start with a large image with many different colors and process it to a pixel one with a quantized set of colors*
* __Input:__ *PNG image.*
* __Output:__ *Quantized and downscaled image.*

As in our previous post on [PixArt-Beads](./2022-03-10-PixelArt.html), we need to rescale and quantize our source images so that we can work with them in a reasonable color space and in manageable computing times. The original [LEGO Mosaic Maker](https://www.lego.com/en-us/product/mosaic-maker-40179) uses a 50x50 layout, so our default will be the same.

We will quantize our image using [PIL](https://pillow.readthedocs.io/en/stable/) (for more details have a look at the [PixArt-Beads](./2022-03-10-PixelArt.html) post):

```python
cpal = fun.paletteReshape(PALETTE)
imgQnt = fun.quantizeImage(
  img, colorsNumber=cpal[0], colorPalette=cpal[1], method=0
)
```

Now, we downscale and save our image to disk:

```python
imgDwn = imgQnt.resize(SIZE, resample=Image.BILINEAR)
imgDwn.save(pthDWN)
```

<center><img src="/media/lego/sami.png" style="width:20%;"><img src="/media/lego/sami_DWN.png" style="width:20%;"></center>

## [Data Reshaping](https://github.com/Chipdelmal/LegoOptimizer/blob/main/preprocess.py)

* __Goal:__ *To transform the image data from an image format into some shape that we can feed into the optimization algorithm (run-length encoding).*

With our image resized and in quantized colors, we now need to reshape it so that we end up with 1D color vectors representing the gaps to fill with our pool of blocks.

The first thing we do is to take our image and replace the color values in each pixel with a dictionary color-key. This is merely for convenience and is not strictly necessary (we could work with the hex strings just fine), but it does make it a bit easier to handle the data.

```python
# Process image colors
pixs = tuple([tuple([tuple(i) for i in row]) for row in img])
colPal = list(set(list(itertools.chain(*pixs))))
# Change colors into keys
colDict = {col: ix for (ix, col) in enumerate(colPal)}
colDeDict = {ix: col for (ix, col) in enumerate(colPal)}
pixDict = tuple([tuple([colDict[i] for i in row]) for row in pixs])
```

We now have an 2D array that stores the color indices of our pixels. The next step we have to take is to count how many adjascent pixels are the same color and store that number in a list (repeating this process for every color). This is pretty much a variation on [Run-Length Encoding](https://en.wikipedia.org/wiki/Run-length_encoding) where we just separate each encoded color to a different list.

```python
dictVals = list(colDict.values())
runL = [fun.runLength(i) for i in pixDict]
# Get flattened vectors counts
pVectors = {}
for dix in dictVals:
  pValPerRow = [[c if i == dix else 0 for (c, i) in row] for row in runL]
  pValPerRow = [[c for (c, i) in row if i==dix] for row in runL]
  pValPerRow = fun.flatten(pValPerRow)
  pVectors[dix] = pValPerRow
```

The way I did it is not the most efficient by any stretch (as I go through the arrays several times), but it is fast enough for the application at hand. The `pVectors` dictionary contains the `{colorIndex: runLengths}` for each color in the image. Finally, we serialize the following dictionary to disk:

```python
pDict = {
  'imageMapped': pixDict,
  'colorMapper': colDict, 'colorDeMapper': colDeDict,
  'runLengthVectors': pVectors, 'runLengthLengths': pLengths
}
```

## [Optimization](https://github.com/Chipdelmal/LegoOptimizer/blob/main/optimizer.py)

* __Goal:__ *To solve the block-assignment problem (multiple-knapsack)*

We are now ready for the most complicated section of the application, the NP Knapsack Problem. Specifically, problem we are facing here is the [Multiple-Knapsack Problem](https://en.wikipedia.org/wiki/Knapsack_problem#Multiple_knapsack_problem). The reason why I decided to use the Knapsack version over the [Multiple-Subset Sum Problem](https://en.wikipedia.org/wiki/Multiple_subset_sum) is that it allows us to set a "value" parameter for each one of the blocks, which will be handy if we want the algorithm to give preference to larger blocks, for example.

While searching online for approaches to solve this optimization problem, I came across a solution using [Google's OR-Tools](https://developers.google.com/optimization/bin/multiple_knapsack), so I gave it a try. I'm not gonna go through the whole code because I'm by no means an expert in the tool, but in general, we to the following:

```python
# Load an instance of the solver
solver = pywraplp.Solver.CreateSolver('SCIP')
# Generate a vector of possible assignments
(x, solver) = genSolverXVector(data, solver)
# Setup problem's constraints
(x, solver) = setSolverConstraints(data, x, solver)
# Setup problem's objective
(x, solver, objective) = setSolverObjective(data, x, solver)
# Solve the problem
status = solver.Solve()
```

For the full breakdown of the code, please have a look at the [github repo](https://github.com/Chipdelmal/LegoOptimizer), with the breakdown of each one of these functions being available in this [code file](https://github.com/Chipdelmal/LegoOptimizer/blob/main/functions.py#L37). To iterate through each color we run the following loop:

```python
for colorIx in range(colorsNum):
  gaps = pDict['runLengthVectors'][colorIx]
  solution[colorIx] = fun.solveColor(
      gaps, blocks, values=values, verbose=VERBOSE
  )
```

Where each `solution` dictionary entry contains the solution for every color in the form: 

```python
solution[colorIndex]: [[blocks_for_gap_1], [blocks_for_gap_2], ..., [blocks_for_gap_N]]
```

Once we get the solution for all the colors (which might take a while), we serialize it to disk to pass it along in our pipeline.

## [Data Decoding](https://github.com/Chipdelmal/LegoOptimizer/blob/main/decoder.py)

* __Goal:__ *To re-shape the data from the optimization results into structures that can be handled in a visualization algorithm.*

Solving the problem is good and all, but it still leaves us with the question: where should we place our blocks?

```python
mapDict = {}
for rix in range(len(lDict)):
  (ld, od) = (lDict[rix], oDict[rix])
  mapping = [[ld[bix], od[bix]] for bix in range(len(lDict[rix]))]
  mapDict[rix] = mapping
```

Where each entry of the `mapDict` contains a list of tuples where each tuple is `(gapLength, [lengthsOfBlocks])` in which, ideally, the sum of the `lengthsOfBlocks` should be equal to the `gapLength` if the optimization algorithm found a solution.

With this at hand, we are going to iterate through the elements of the image and replace the index of the color with its RGB value, along with the lengths of the blocks to be placed in each row.

For example, an image row of:

```python
imgRow = (12, 12, 12, 12, 8, 8, 8, 12, 12)
```

could be fransformed to:

```python
decodedRow = [
  ((255, 127,   0), [2, 2]),
  ((  0,   0,   0), [1, 2]),
  ((255, 127,   0), [2   ])
]
```

So that, after we repeat this for the whole image, we could go through each row and draw the rectangles with their respective colors to reconstruct the original image from the pieces. A special case was added to this process to handle the case where the algorithm didn't find an optimal solution to the gaps. In this situation, the list ends with a negative value representing the number of gaps that remain unfilled.

## [Image Reconstruction](https://github.com/Chipdelmal/LegoOptimizer/blob/main/reconstruct.py)

* __Goal:__ *To plot the blocks in their respective locations in the image.*

The final step in the pipeline is to re-create the original image with the blocks outlined on top of it. 

# Use Example

# Code Repo

* **Repository:** [Github repo](https://github.com/Chipdelmal/LegoOptimizer)
* **Dependencies:** [opencv-python](https://pypi.org/project/opencv-python/), [Pillow](https://pillow.readthedocs.io/en/stable/), [numpy](https://numpy.org/), [OR-Tools](https://developers.google.com/optimization/install), [compress-pickle](https://pypi.org/project/compress-pickle/), [termcolor](https://pypi.org/project/termcolor2/)