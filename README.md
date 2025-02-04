# rgbloom

This Python script retrieves RGB magnitudes computed from low resolution
spectra published in *Gaia* DR3, following the work described in 
[Carrasco et al. (2023)](#3).
These magnitudes are given in the standard system defined by
[Cardiel et al. (2021a)](#1).

This code is an updated version of [rgblues](https://github.com/guaix-ucm/rgblues),
which provides RGB magnitudes from *Gaia* EDR3 photometric data, as
explained in [Cardiel et al. (2021b)](#2).

The RGB magnitudes provided by [Carrasco et al. (2023)](#3) 
are more reliable because 
they have been directly computed from the source spectrum without
the need to employ any approximate calibration, nor introducing
constraints on the source colour or extinction. In addition, the number
of sources with RGB estimates has increased from ~15 million to
~200 million objects (the 200M sample). Anyway, since the sky 
coverage of the 200M sample is still not very good at some 
high Galactic latitudes, `rgbloom` also provides RGB estimates
for sources that do not belong to the 200M sample making use of the
polynomial calibrations of [Cardiel et al. (2021b)](#2), which
may still be useful for those users requiring calibrated RGB 
sources at those sky regions.

The code `rgbloom` performs a cone search defined by coordinates 
right ascension and declination on the sky and a search radius.
The cone search is performed making use of the Astroquery coordinated 
package of astropy. 

You need to have a live connection to the Internet for 
the script to work!

## Installing the code

In order to keep your current Python installation clean, it is highly 
recommended to install a python 3 *virtual environment* first.

### Creating and activating the python virtual environment

```bash
$ python3 -m venv venv_rgb
$ . venv_rgb/bin/activate
(venv_rgb) $
```

### Installing the package

```bash
(venv_rgb) $ pip install git+https://github.com/guaix-ucm/rgbloom.git@main#egg=rgbloom
```



## Executing the program

Just execute it from the command line:

```bash
(venv_rgb) $ rgbloom 56.66 24.10 1.0 12
```

The last instruction executes the program providing the 
four positional arguments: right ascension, declination, search radius and 
limiting *Gaia* G magnitude. *Note that the coordinates and search radius 
must be given in decimal degrees*.

Each time the code is executed, some auxiliary files are
downloaded to your computer (if they have not been downloaded
in a previous execution). These files are kept in a cache directory
that is displayed in the terminal output (you do not have to worry
about its location unless you need to delete them in order to
recover disk space).

The execution of this example should led to the following output in the
terminal (except for the absolute path where the auxiliary downloaded files 
are stored):

```
        Welcome to rgbloom version 1.2
        ==============================

Downloading data from 'http://nartex.fis.ucm.es/~ncl/rgbphot/gaiaDR3/reference_healpix8.csv' to file '/Users/cardiel/Library/Caches/pooch/635cd722cf61b23bd8eee20635e4d580-reference_healpix8.csv'.
<STEP1> Starting cone search in Gaia DR3... (please wait)
  INFO: Query finished. [astroquery.utils.tap.core]
        --> 310 objects found
        --> 23 objects classified as VARIABLE
<STEP2> Estimating RGB magnitudes in DR3 query using C21 polynomials OK!
<STEP3> Retrieving objects from the 200M sample in the enclosing HEALPIx level-8 tables
Downloading data from 'http://nartex.fis.ucm.es/~ncl/rgbphot/gaiaDR3/RGBsynthetic_NOVARIABLES/sortida_XpContinuousMeanSpectrum_006602-007952_RGB_NOVARIABLES_final.csv.gz' to file '/Users/cardiel/Library/Caches/pooch/2d94d5acfcb380d6dff1eaa207caa086-sortida_XpContinuousMeanSpectrum_006602-007952_RGB_NOVARIABLES_final.csv.gz'.
        * Required file: /Users/cardiel/Library/Caches/pooch/2d94d5acfcb380d6dff1eaa207caa086-sortida_XpContinuousMeanSpectrum_006602-007952_RGB_NOVARIABLES_final.csv.gz
          md5:f9cf7ed0f84eecda13ef6a408d291b96
        --> Number of objects: 100553
        --> Total number of objects: 100553
<STEP4> Cross-matching DR3 with 200M sample
        --> Number of objects in the 200M subsample............: 100553
        --> Number of objects in DR3 query.....................: 310
        --> Number of DR3 objects within the 200M sample.......: 248
        --> Number of DR3 objects no present in the 200M sample: 62
<STEP5> Saving output CSV files
        --> file rgbloom_200m.csv saved
        --> file rgbloom_no200m.csv saved
<STEP6> Generating PDF plot
End of program
```

The `rgbloom` script executes the following steps:

- Step 1: cone search in *Gaia* DR3, gathering the following parameters: 
  `source_id`, `ra`, `dec`, `phot_g_mean_mag`, `phot_bp_mean_mag`,
  `phot_rp_mean_mag` and `phot_variable_flag`

- Step 2: initial RGB magnitude estimation using the polynomial 
  transformations given in Eqs. (2)-(4) of [Cardiel et al. (2021b)](#2).
  These values are only provided for objects in the field of view 
  that do not belong to the 200M sample.
  
- Step 3: downloading of the RGB magnitude estimates corresponding to 
  the 200M sample objects within the HEALPIx level-8 tables enclosing 
  the region of the sky defined in the initial cone search.
  
- Step 4: cross-matching between the DR3 and 200M subsamples to identify objects
  with RGB estimates derived from the low resolution *Gaia* DR3 spectra.
  
- Step 5: generation of the output files. Two files (in CSV format) are 
  generated: 

    - `rgbloom_200m.csv`: objects belonging to the 200M sample 
      with RGB magnitudes computed as described in [Carrasco et al. (2023)](#3).
      This CSV file provides the following columns:
      - `number`: consecutive number of the object in the CSV file (used in the final plot)
      - `source_id`: identification in *Gaia* DR3
      - `ra`: right ascension (from *Gaia* DR3)
      - `dec`: declination (from *Gaia* DR3)
      - `RGB_B`: blue RGB magnitude estimate
      - `RGB_G`: green RGB magnitude estimate
      - `RGB_R`: red RGB magnitude estimate
      - `errRGB_B`: uncertainty in the blue RGB magnitude estimate
      - `errRGB_G`: uncertainty in the green RGB magnitude estimate
      - `errRGB_R`: uncertainty in the red RGB magnitude estimate
      - `objtype`: type of source, according to the classification provided by
        *Gaia* DR3 (see [description of
        `GAIA_SOURCE`](https://gea.esac.esa.int/archive/documentation/GDR3/Gaia_archive/chap_datamodel/sec_dm_main_source_catalogue/ssec_dm_gaia_source.html) table for details):
        - `1`: object flagged as `NON_SINGLE_STAR`
        - `2`: object flagged as `IN_QSO_CANDIDATES`
        - `3`: object flagged as `IN_GALAXY_CANDIDATES`
        - `0`: none of the above
      - `qlflag`: global quality flag:
        - `0`: reliable source
        - `1`: suspicious source (blending, contamination, non-stellar
          identification)

    - `rgbloom_no200m.csv`: objects not included in the 200M sample, which
      RGB magnitudes are estimated using the approximate polynomial
      calibrations of [Cardiel et al. (2021b)](#2).
      This CSV file contains the following columns:
      - `number`: consecutive number of the object in the CSV file (used in the final plot)
      - `source_id`: identification in *Gaia* DR3
      - `ra`: right ascension (from *Gaia* DR3)
      - `dec`: declination (from *Gaia* DR3)
      - `phot_variable_flag`: photometric variability flag (from *Gaia* DR3)
      - `bp_rp`: G_BP-G_RP colour (from *Gaia* DR3)
      - `RGB_B`: blue RGB magnitude estimate
      - `RGB_G`: green RGB magnitude estimate
      - `RGB_R`: red RGB magnitude estimate

  The list of objects in these two files is sorted by right ascension.

- Step 6: generation of a finding chart plot (in PDF format): `rgbloom.pdf`. 
  The execution of the previous example generates a cone search around 
  the [Pleiades](https://en.wikipedia.org/wiki/Pleiades) star cluster:
  ![Pleiades plot](http://nartex.hst.ucm.es/~ncl/rgbphot/gaiaDR3/pleiades_v4.png)
  The objects in this plot are color coded based on the *Gaia* G_BP - G_RP 
  colour. Stars brighter than a pre-defined threshold are displayed 
  with big star symbols. To facilitate the identification of each object, the
  consecutive identification numbers in the two files `rgbloom_200m.csv` and
  `rgbloom_no200m.csv` are also displayed, in red
  and black, respectively. The identification numbers corresponding to the less
  reliable sources in `rgbloom_20m.csv` (`qlflag=1`) appear inside a rectangle
  with a light-gray border. Note that the identification numbers are not
  displayed when using the parameter `--nonumbers` in the command line. 

  In the case of objects that do not belong to the 200M sample (i.e., those in
  `rgbloom_no200m.csv`), a blue square
  has been overplotted on the sources flagged as variable in *Gaia* DR3, and a
  grey diamond on objects outside the *Gaia* -0.5 < G_BP - G_RP < 2.0 colour
  interval.

Note that the three output archives (1 PDF and 2 CSV files) share the same root
name `rgbloom`. This can be easily modified using the optional argument
`--basename <newbasename>` in the command line.

### Additional help

Some auxiliary optional arguments are also available. See description 
invoking the script help:

```bash
$ rgbloom --help

...
...

positional arguments:
  ra_center             right Ascension (decimal degrees)
  dec_center            declination (decimal degrees)
  search_radius         search radius (decimal degrees)
  g_limit               limiting Gaia G magnitude

optional arguments:
  -h, --help            show this help message and exit
  --basename BASENAME   file basename for output files
  --brightlimit BRIGHTLIMIT
                        objects brighter than this Gaia G limit are displayed with star symbols (default=8.0)
  --symbsize SYMBSIZE   multiplying factor for symbol size (default=1.0)
  --nonumbers           do not display object identification number in PDF chart
  --noplot              skip PDF chart generation
  --nocolor             do not use colors in PDF chart
  --verbose             increase program verbosity
```

## Citation

If you find this Python package useful, 
please cite [Cardiel et al. (2021a)](#3)
(to quote the use of the standard RGB system)
and [Carrasco et al. (2023)](#3) (where the computation of the RGB magnitudes
from the low resolution spectra published in *Gaia* DR3 is explained).

## Related information

You can visit the [RGB Photometry](https://guaix.ucm.es/rgbphot) web page at
the Universidad Complutense de Madrid.

## Bibliography

<a id="1">Cardiel et al. (2021a)</a>, 
MNRAS, https://ui.adsabs.harvard.edu/abs/2021MNRAS.504.3730C/abstract

<a id="2">Cardiel et al. (2021b)</a>, 
MNRAS, https://ui.adsabs.harvard.edu/abs/2021MNRAS.507..318C/abstract

<a id="3">Carrasco et al. (2023)</a>, Remote Sensing, in press
