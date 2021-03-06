Using FlopR
================
Alex J H Fedorec
30/07/2020

  - [Prerequisite Knowledge](#prerequisite-knowledge)
  - [Installation](#installation)
  - [A Quick Note on .csv Files](#a-quick-note-on-.csv-files)
  - [Plate Reader Calibration](#plate-reader-calibration)
  - [Processing Plate Reader Data](#processing-plate-reader-data)
      - [Autofluorescence normalisation
        details](#autofluorescence-normalisation-details)
  - [Flow Cytometry Processing](#flow-cytometry-processing)
      - [Process a single .fcs file](#process-a-single-.fcs-file)
      - [Process a folder of .fcs
        files](#process-a-folder-of-.fcs-files)

## Prerequisite Knowledge

We have attempted to make our software usable with minimal prior
knowledge of the R programming language or programming in general. You
will need to be familiar with the idea of running commands from a
console or writing basic scripts. For R beginners,
[this](https://moderndive.netlify.app/1-getting-started.html) is a great
starting point, there are some good resources
[here](https://education.rstudio.com/learn/beginner/) and we suggest
using the [RStudio application](https://rstudio.com/products/rstudio/).
It provides an environment for writing and running R code.

## Installation

FlopR relies on several other R packages. Most of them are available
through the “Comprehensive R Archive Network (CRAN)” which just means
that they can be automatically installed. There are, however, a couple
that need to be manually installed. To do this use the following
commands:

``` r
install.packages("devtools", repos = "https://cloud.r-project.org/")

if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install(c("flowCore", "flowClust", "flowStats"))
```

Once those packages have been installed we are ready to install FlopR.

``` r
devtools::install_github("ucl-cssb/flopr")
```

This grabs the latest version of FlopR from GitHub and builds it on your
computer. If it installed successfully we should now be able to load the
FlopR package in R.

``` r
library(flopr)
```

The examples in this document can be run using the data in the “example”
folder. Download the whole folder and make sure that you have set your
current working directory (you can use the `setwd()` command in R) to
the location where you saved it on your computer

## A Quick Note on .csv Files

This package makes extensive use of .csv (comma separated variables)
files for data and meta-data. Unfortunately, not all .csv are created
equal.

  - When using a Windows OS computer we recommend that you save your
    .csv files explicitly with a UTF-8 file encoding (as shown below).
    This means that your files can be used by someone running a
    different operating system without error.
  - Some parts of the world use decimal commas instead of decimal
    points. In these places, .csv files use “;” as a separator instead
    of “,”. Unfortunately, we are unable to automatically detect which
    format your .csv files are in. As such, you will have to ensure that
    your files are saved using “,” as the separator and decimal points
    for the decimal mark. [This
    article](https://support.collaborativedrug.com/hc/en-us/articles/115004985746-CSV-files-comma-separator-and-decimal-number-format)
    has some help for doing so.

![From Microsoft Excel, save as UTF-8 encoded .csv
file](examples/csv_save.png)

## Plate Reader Calibration

The experimental protocols for producing absorbance and fluorescence
calibration data can be found
[here](https://www.protocols.io/workspaces/igem-measurement).

The data produced by plate readers from different manufacturers comes in
different formats. The first step we need to do is to “parse” the data
from the plate reader into a standard format. We have written an example
parser for use with data from Tecan plate readers. If you need help
writing a parser for your plate reader, please contact us and we’ll see
what we can do.

For a Tecan plate reader, the data is saved as an Excel .xls file. It
first needs to be saved as a .csv file (open in Excel and “Save As”
.csv) that can be read by R. We also need a .csv file telling us what is
in each well of our microtitre plate. An example can be found in the
“examples/plate\_reader/DATA” folder, but the first few rows looks
like this:

<div class="kable-table">

| calibrant   | fluorophore | media | concentration | replicate | well |
| :---------- | :---------- | :---- | ------------: | --------: | :--- |
| fluorescein | GFP         | PBS   |       10.0000 |         1 | A1   |
| fluorescein | GFP         | PBS   |        5.0000 |         1 | A2   |
| fluorescein | GFP         | PBS   |        2.5000 |         1 | A3   |
| fluorescein | GFP         | PBS   |        1.2500 |         1 | A4   |
| fluorescein | GFP         | PBS   |        0.6250 |         1 | A5   |
| fluorescein | GFP         | PBS   |        0.3125 |         1 | A6   |

</div>

Once we have the calibration data and layout .csv files we can parse the
data.

``` r
flopr::spark_parse(data_csv = "examples/plate_reader/DATA/191219_calibration_membrane.csv",
                   layout_csv = "examples/plate_reader/DATA/calibration_plate_layout.csv",
                   timeseries = FALSE)
```

The `data_csv` argument takes the path to the calibration data.
`layout_csv` is the path to the plate layout .csv file. Finally, the
Tecan plate readers save timeseries data differently from single
timepoint data, so we have a Boolean flag, `timeseries`, that lets the
parser know that this is not a timeseries.

The `spark_parse()` function saves the parsed calibration data in a new
.csv file, in the same location as the calibration data, with "\_parsed"
appended to the filename. The first few rows look like this:

<div class="kable-table">

| calibrant   | fluorophore | media | concentration | replicate | well |  OD600 |  OD700 | GFP 40 | GFP 50 | GFP 60 | GFP 70 | GFP 80 | GFP 90 | GFP 100 | GFP 110 | GFP 120 | row | column |
| :---------- | :---------- | :---- | ------------: | --------: | :--- | -----: | -----: | -----: | -----: | -----: | -----: | -----: | -----: | ------: | ------: | ------: | :-- | -----: |
| fluorescein | GFP         | PBS   |       10.0000 |         1 | A1   | 0.0921 | 0.0831 |   1830 |   9811 |  36911 |     NA |     NA |     NA |      NA |      NA |      NA | A   |      1 |
| fluorescein | GFP         | PBS   |        5.0000 |         1 | A2   | 0.0942 | 0.0852 |    932 |   4993 |  19221 |  54510 |     NA |     NA |      NA |      NA |      NA | A   |      2 |
| fluorescein | GFP         | PBS   |        2.5000 |         1 | A3   | 0.1015 | 0.0928 |    453 |   2434 |   9448 |  27864 |     NA |     NA |      NA |      NA |      NA | A   |      3 |
| fluorescein | GFP         | PBS   |        1.2500 |         1 | A4   | 0.0985 | 0.0899 |    232 |   1253 |   4853 |  14513 |  36349 |     NA |      NA |      NA |      NA | A   |      4 |
| fluorescein | GFP         | PBS   |        0.6250 |         1 | A5   | 0.0957 | 0.0905 |    114 |    617 |   2404 |   7128 |  18282 |  41812 |      NA |      NA |      NA | A   |      5 |
| fluorescein | GFP         | PBS   |        0.3125 |         1 | A6   | 0.1027 | 0.0965 |     54 |    294 |   1137 |   3388 |   8672 |  20095 |   42244 |      NA |      NA | A   |      6 |

</div>

The parser has extracted the information we need from the calibration
data and merged it with the plate layout information so that we now have
columns containing each of the measurements for each well.

Now we can actually calculate our calibration coefficients. To do this
we just need to use one function and give it our parsed data.

``` r
flopr::generate_cfs(calibration_csv = "examples/plate_reader/DATA/191219_calibration_membrane_parsed.csv")
```

For details about how this process works, you can read our paper
[here](). At the end, there should be two .pdf images showing the
calibration curves for absorbance and fluorescence, along with a new
.csv file, appended with "\_cfs", containing the parameters for use in
the future, the first few rows of which look like this:

<div class="kable-table">

|         cf |        beta | calibrant   | fluorophore | measure |
| ---------: | ----------: | :---------- | :---------- | :------ |
|   184.7274 |   0.0057476 | fluorescein | GFP         | GFP 40  |
|   996.4472 | \-0.0035424 | fluorescein | GFP         | GFP 50  |
|  3821.0497 | \-0.0004737 | fluorescein | GFP         | GFP 60  |
| 11298.9531 |   0.0002015 | fluorescein | GFP         | GFP 70  |
| 29128.3783 | \-0.0007481 | fluorescein | GFP         | GFP 80  |
| 65834.9342 |   0.0012751 | fluorescein | GFP         | GFP 90  |

</div>

The “cf” column contains the calibration coefficients that will be used
to calibrate our data in later experiments.

Before we get too excited we need to check the images to make sure that
the calibration curves look sensible. The software attempts to remove
data points which it deems are invalid, but this process isn’t perfect
and occasionally may need you to remove data points from the
"\*\_parsed.csv" file. Using the example data, you can see
([here](examples/plate_reader/DATA/191219_calibration_membrane_parsed_absorbance_cfs.pdf))
that some of the fluorescein wells are considered valid absorbance
measurements. In this case, it isn’t the end of the world since we would
never use the parameters produced from those two curves.

n.b. Currently the software is setup to work with “microspheres” for
calibrating cell number and “fluorescein” for calibrating GFP
fluorescence. We hope to extend it in the near future to work with other
calibrants.

## Processing Plate Reader Data

As mentioned above, because plate readers from different manufacturers
save the data in different formats, the first step we need to do is
parse our raw data. The parser that we provide takes Tecan plate reader
data in the form of a .csv file. We also need a .csv file telling us
what is in each well of you microtitre plate. This can include any
information that you wish; we include as much meta-data as possible as
it makes our data analysis later much smoother. The only requirement is
that **the last column must be named “well”** and include an identifier
(usually the well id i.e. B2) that can be matched to the same identifier
in the plate reader data. Here’s an example where we include information
about the strains, plasmids, media, inducers, etc.:

<div class="kable-table">

| strain | host | plasmid | plasmid\_2 | strain\_2 | media | sugar | amino\_acids | inducer | concentration | init\_ratio | init\_dilution | well |
| :----- | :--- | :------ | :--------- | :-------- | :---- | :---- | :----------- | :------ | ------------: | ----------: | -------------: | :--- |
|        |      |         |            |           |       |       |              |         |            NA |          NA |             NA | A1   |
|        |      |         |            |           |       |       |              |         |            NA |          NA |             NA | A2   |

</div>

Now we can use our parsing function.

``` r
flopr::spark_parse(data_csv = "examples/plate_reader/DATA/200228_example_data.csv",
                   layout_csv = "examples/plate_reader/DATA/200228_example_layout.csv",
                   timeseries = TRUE)
```

The data is extracted and the meta-data from the layout is attached.
Note that this time the data a timeseries so we set the `timeseries`
flag to `TRUE`. A new .csv file is produced containing the parsed data
with "\_parsed" appended to the filename.

Now we can start processing our data. There is one function that does
all the work for: `process_plate()`. There are a few arguments that we
need to give the function which will control what happens.

``` r
flopr::process_plate(data_csv = "examples/plate_reader/DATA/200228_example_data_parsed.csv",
                     blank_well = c("C12", "D12"),
                     neg_well = c("C6", "D6", "E6"),
                     od_name = "OD700",
                     flu_names = c("GFP", "mCherry"),
                     af_model = "spline",
                     to_MEFL = TRUE,
                     flu_gains = 135,
                     conversion_factors_csv = "examples/plate_reader/DATA/191219_calibration_membrane_parsed_cfs.csv")
```

Let’s walk through what each of the arguments do:

  - `data_csv` is the path to our parsed data.
  - `blank_well` are the well identifiers of wells containing media
    blanks. These are used for normalising absorbance. If you only have
    one blank well you can specify it using `blank_well = "C12"` for
    example.
  - `neg_well` are the well identifiers of wells containing negative
    controls. These are used for normalising fluorescence. As above, if
    you only have one negative control well you can specify it using
    `neg_well = "C6"` for example.
  - `od_name` is the name of the column containing our absorbance values
    in the parsed data .csv file. Currently we can only use one
    absorbance column, so if you record absorbance at multiple
    wavelengths (like I do), you will have to pick one.
  - `flu_names` are the names of the columns containing our fluorescence
    values. You can include as many or as few of you fluorescence
    columns as you like. Whichever columns are named in here, we will
    attempt to normalise.
  - `af_model` allows you to choose the type of model that we are going
    to use for fluorescence normalisation. We’ll discuss the available
    choices below.
  - `to_MEFL` is a Boolean flag that lets you tell the function if you
    want to convert the fluorescence data into calibrated units. You can
    only do this if you have carried out the calibration as detailed
    above.
  - `flu_gains` is where you specify the gain at which your fluorescence
    data was recorded for each fluorescence channel. Here we only have
    calibration parameters for “GFP” so we only specify one gain value.
  - `conversion_factors_csv` is the path to the calibration parameters
    that you generated using the protocol detailed above.

When we run this function the absorbance is normalised, then the
fluorescence and finally (if desired) the absorbance and fluorescence
values are calibrated. Finally, the processed data is saved in a new
.csv file with "\_processed" appended to the filename and with
additional columns for each of the processed values.

We also save some .pdf images comparing the raw and normalised data, and
images showing the fluorescence normalisation curves. Using these plots,
there are a few checks that we should make before celebrating.

  - Check that none of the blank wells you used showed any contamination
    (it happens to the best of us). If there is growth in a blank well
    it will throw off the absorbance normalisation.
  - Check that the fluorescence normalisation curves fit the data. Read
    below for more detail on possible issues.

### Autofluorescence normalisation details

Autofluorescence is the fluorescence produced by anything other than the
fluorophores that we are interested in measuring. A small amount usually
comes from growth media and can be minimised by choosing certain medias.
A large contribution comes from molecules produced and secreted by our
cells. Some of these molecules show particularly strong emission at
similar wavelengths to GFP. We observe that the level of
autofluorescence is not simply proportional to the number of cells or
optical density of our culture. As cells enter stationary phase,
autofluorescence increases, perhaps due increased production of the
autofluorescent molecules and changes in cell size.

In order to remove this autofluorescence from our sample data we fit a
curve to our negative control data. We provide four different models
that the user can choose to fit their data (and if desired more models
can be added). There are two smoothing models: “loess” and “spline”. The
primary difference to the user between these two models is that the
“spline” is able to extrapolate beyond the negative control data
provided. This means that if the range of absorbance values at which you
have measurements of your negative control is smaller than the range of
your samples, we can still make an attempt at normalisation. However,
this extrapolation is very crude (linear from the last data point) and
can produce poor normalisation in the extrapolated range. Fortunately,
negative controls tend to grow better than fluorescent samples, so
extrapolation is often not an issue. We also provide a second-order
polynomial and an exponential model, specified by “polynomial” and
“exponential” repsectively. These are inherently able to make
predictions beyond the range of normalised data and therefore may be
good starting points.

Prolonged periods in stationary phase can cause autofluorescence to
increase while absorbance remains stable and in some cases absorbance
can start decreasing while autofluorescence does not. In these
circumstances, none of the models perform particularly well. The
performance of each of them should be checked and if none of them
perform satisfactorily it may be necessary to trim the data to remove
confounding timepoints.

## Flow Cytometry Processing

We have two functions for processing flow cytometry data:

  - `process_fcs` takes a single .fcs file, removes debris and doublets
    and saves the trimmed data in a new .fcs file.
  - `process_fcs_dir` takes a folder of .fcs files and performs the same
    trimming on each. It can also perform fluorescence normalisation if
    you have a negative control and fluorescence calibration if you have
    measured a calibrant.

### Process a single .fcs file

To process a single .fcs file we can run the following command:

``` r
flopr::process_fcs(fcs_file = "examples/flow_cytometry/DATA/20191121/pWeak_None_0_1.fcs",
                   flu_channels = "BL1-H",
                   pre_cleaned = TRUE,
                   do_plot = TRUE)
```

We need to give the function four bits of information

  - `fcs_file` is the path to the .fcs file that we want to process.
  - `flu_channels` are the names of the fluorescence channels that we
    recorded. When processing a single .fcs file like this we don’t
    actually do anything with the fluorescence data. However, if you
    include the channel names you can see the data in a plot that is
    saved. If you have more than one fluorescence channel, the argument
    needs to be a vector, which will look something like this:
    `flu_channels = c("BL1-H", "BL2-H", YL2-H")`
  - `pre_cleaned` lets the function know if you have gated out debris on
    the flow cytometer. Most people do this when running their
    experiments by setting a threshold on forward-scatter and
    side-scatter. But some people like to record everything and process
    the data later.
  - `do_plot` lets the function know if you want a plot to be saved of
    the trimming process.

In the end we have a new .fcs file with "\_processed" appended to the
filename, and if you asked the function to save a plot we will have a
.pdf that looks something like
[this](examples/flow_cytometry/DATA/20191121/pWeak_None_0_1_processed.pdf).

### Process a folder of .fcs files

It’s much more likely that we have more than one sample that we want to
process. This is where the other function comes in handy.

``` r
flopr::process_fcs_dir(dir_path = "examples/flow_cytometry/DATA/20191121",
                       pattern = "*Med*.fcs",
                       flu_channels = "BL1-H",
                       pre_cleaned = TRUE,
                       do_plot = TRUE,
                       neg_fcs = "pNeg_None_0_1.fcs",
                       calibrate = TRUE,
                       mef_peaks = list(list(channel = "BL1-H",
                                             peaks = c(0, 822, 2114, 5911, 17013, 41837, 145365, 287558))))
```

There are a few more arguments to this function and some of them look a
bit complicated so let’s go through them.

  - `dir_path` is the path to the folder with your .fcs files in.
  - `pattern` allows us to just process a subset of the .fcs files in
    the folder. Here we are just going to process files with “Med” in
    the filename. Without going into too much detail, this uses a
    simplified version of “regular expressions” called [“globbing
    patterns”](https://en.wikipedia.org/wiki/Glob_%28programming%29). We
    can use “wildcard” symbols to represent any character (? is a place
    holder for an single character and \* is a place holder for 0 to any
    number of characters). In this example all we know is that “Med”
    appears somewhere in the filenames and they all end with “.fcs”. So
    we use the \* character to show that there are some unknown
    characters before “Med” and between “Med” and “.fcs”. If you want to
    process all .fcs files in your folder, the pattern would be
    "\*.fcs".
  - `flu_channels` is as above.
  - `pre_cleaned` is as above.
  - `do_plot` is as above.
  - `neg_fcs` is the filename of your negative control sample. It must
    be in the folder with the other .fcs files. This is used to
    normalise autofluorescence. If you don’t specify a filename here,
    the processing will be carried out without any normalisation steps.
  - `calibrate` tells the function whether you want to calibrate the
    fluorescence measurements. To be able to calibrate you must have an
    .fcs file with data from calibration beads and it must have “beads”
    somewhere in the filename. For discussion about calibration beads,
    read our paper or check out [TASBE](https://tasbe.github.io/) and
    [FlowCal](https://taborlab.github.io/FlowCal/).
  - `mef_peaks` is where we tell the function what the true fluorophore
    values are for our beads. These will be available from the bead
    manufacturer. We need to specify peaks for each of the channels that
    we want to calibrate and the channel name needs to correspond to one
    given in `flu_channels`. Here we are just calibrating the “BL1-H”
    channel which corresponds to GFP on our flow cytometer. If you
    wanted to calibrate two channels it would look something like this:

<!-- end list -->

``` r
mef_peaks = list(list(channel = "BL1-H", 
                      peaks = c(0, 822, 2114, 5911, 17013, 41837, 145365, 287558)),
                 list(channel = "YL2-H",
                      peaks = c(0, 218, 581, 1963, 6236, 15267, 68766, 181945)))
```

The function carries out the trimming as described above. Then, if a
negative control file is given, it performs fluorescence normalisation
by negating the geometric mean of the negative control’s fluorescence in
each channel from each of the other samples. Finally, if the files are
going to be calibrated, a calibration curve is fit to the bead peaks in
each of the fluorescence channels in `mef_peaks`. We use a model
developed for FlowCal for our calibration curve. We then calibrate both
the raw and normalised data since normalisation can produce fluorescence
values less than or equal to 0, which get removed during calibration due
to working with logged data (the output will contain a both sets of
calibration). The new data, any plots produced, and a data\_summary.csv
file (containing geometric statistics for each .fcs file) will be saved
to a new folder with the same name as the original but with
"\_processed" appended.

There are a few checks that should be made to reassure yourself that
everything has worked.

  - To confirm correct trimming, we recommend setting `do_plot = TRUE`
    so that you can see which events have been removed during
    processing. Check that the debris, if there is any, and doublets
    have been correctly identified and removed.
  - For each fluorescence calibration channel, a plot will be saved
    showing the bead peaks that have been identified and the calibration
    curve that we have fit to them. Sometime the bead peaks aren’t
    identified correctly. In this case, there is another argument that
    the function can take `beads_dens_bw`, which has a default value of
    `0.025`. To identify the beads we use something called a Gaussian
    kernel to smooth the fluorescence data and pick the highest points.
    Sometimes this smoothing isn’t quite right; we might not smooth
    enough and one peak is identified as two, or we smooth too much and
    lose peaks. In the former case we need to increase `beads_dens_bw`
    and in the latter we decrease it. If tweaking this value doesn’t
    work, we also provide a way to manually specify identify the peaks
    using the `manual_peaks` argument. For this data, a manually
    specified set of peaks would look like this:

<!-- end list -->

``` r
manual_peaks = list(list(channel = "BL1-H", 
                      peaks = c(1.9, 2.5, 2.9, 3.3, 3.7, 4.2, 4.6, 4.95)))
```
