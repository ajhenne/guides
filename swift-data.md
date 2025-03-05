
# Table of Contents

1.  [WHEN RUNNING ON HPC](#orgfcb9eb4)
2.  [XRT Data](#org7902fa1)
    1.  [Temporal Data](#org6894c9a)
    2.  [Spectral Data](#org7efb098)
3.  [BAT Data](#org9422972)
    1.  [Generating a light curve](#orgc359c01)
4.  [Optical Data](#org6006b06)
    1.  [AB](#org73e59c2)
    2.  [Vega](#orgb1d1676)
    3.  [Extinction](#orgd2be3d4)
5.  [UVOT Data](#org3ae582b)
    1.  [Extinction](#org27d4990)
6.  [Other optical data](#org1a5b71a)



<a id="orgfcb9eb4"></a>

# WHEN RUNNING ON HPC

You must add these two lines to your script. You can run the commands interactively from HPC, i.e. by just typing them into command lines. But if you write them into a batch script and attempt to submit, you&rsquo;ll be met with some very unhelpful errors that will cause you much stress trying to troubleshoot - there&rsquo;s not much other discussion out there on HEASOFT.

Even though you may provide the command with all parameters, such that in the command line nothing is done interactively, the program seems to still expect some sort of nteractive session - so this always fails in a batch environment.

    export HEADASNOQUERY=
    export HEADASPROMPT=/dev/null

Should also explicitly redirect /dev/null into sandard input to stop any potential rogue cases of the command expecting an interactive input:
`batgrbproduct input output < /dev/null`

<https://heasarc.gsfc.nasa.gov/lheasoft/scripting.html>


<a id="org7902fa1"></a>

# XRT Data


<a id="org6894c9a"></a>

## Temporal Data

Can be obtained directly from the light curve pages.


<a id="org7efb098"></a>

## Spectral Data

Swift Archive -> Spectrum -> Add time sliced spectral
Will provide the appropriate files, with desired slicing, to put straight into Xspec


<a id="org9422972"></a>

# BAT Data

Obtain data through burst > archive, then select observation IDs. BAT data is usually in the first obs (maybe second&#x2026;?).

-   bat, trdss and auxilare required

The data is processed using software as part of the HEASOFT packages. On Leicester HPC this can be called using:
`module load heasoft`

Then there are several steps we run to process the data. On the downloaded and unzipped directory:
`batgrbproduct <indir> <outdir>`

In report.txt that is generated, we can obtain <battime> in seconds, simply add the desired length to tstop (or use the given stop time). You must calculate and paste the actual number. You cannot just write the sum with the + !!

    batbinevt detmask=auxil/*_qmap.fits tstart=<battime> tstop=<battime>+<stoptime>
    input -> events/*_all.evt
    output -> filename.pha
    PHA1
    0
    u
    CALDB:80

For spectal fitting. To create lightcurves &rsquo;0&rsquo; and &rsquo;u&rsquo; should be changed accordingly.

    batupdatephakw
    input -> filename.pha
    auxil/*_all.evaux

    batphasyserr
    input -> filename.pha
    CALDB

    batdrmgen
    input -> filename.pha
    output -> filename.rsp
    NONE

From here, we can load the data into Xspec for spectral fitting. Ignore the data outside the BAT range:
`ignore **-15.0 150.0-**`


<a id="orgc359c01"></a>

## Generating a light curve

All the commands are the same with a modification to &rsquo;batbinevt&rsquo;. &rsquo;batgrbproduct&rsquo; will also generate an lc directory which comes with some light curves. By default, the time column uses BAT mission time. To modify this to be with respect to BAT trigger time, we need to subtract the mission time.

    fcalc
    input -> file.lc
    output -> file_time.lc
    resultant name -> time
    arithmetic -> time - <battime>

To quickly plot this:

    fplot
    time
    rate[error]
    \xw
    wd filename.qdp


<a id="org6006b06"></a>

# Optical Data

Two main magnitude systems: Vega and AB.

To convert from magnitude to flux we use:
$\textrm{flux} = 10^{-0.4 * mag} \times F_{0}$

Full table of conversions and F<sub>0</sub> values can be found in my notebook (NAM2022) or in Python scripts GRB200925B plotting scripts.

fukugita1995 and fukugita1996 for AB and Vega conversions respectively - for SDSS/Sloan filters. Sloan usually comes in AB mag
bessel1998 for Johnson-Cousins filters. Usually comes in Vega mag.


<a id="org73e59c2"></a>

## AB

Usually flux zero point is 3631 Jy


<a id="orgb1d1676"></a>

## Vega

Differing flux zero points.


<a id="orgd2be3d4"></a>

## Extinction

Use this website to calculate extinction for some given sky position:
<https://ned.ipac.caltech.edu/extinction_calculator>
I use Landolt for Johnson-Cousin filters, this seems pretty close.


<a id="org3ae582b"></a>

# UVOT Data

Mag values should usually be given in Vega(?). The filter system is similar but not exactyly the same as the Johnson-Cousin filter system.
Can&rsquo;t find flux zero point values, only magnitude. But since the AB system always has the same 3631 Jy zero point flux value, we can convert Vega -> AB mag using conversions available for Swift - Breeveld indirectly gives these (table 2 - subtract difference between systems), also available at <https://swift.gsfc.nasa.gov/analysis/uvot_digest/zeropts.html>. Then we can convert to flux.
Yi2023 gives extinction value


<a id="org27d4990"></a>

## Extinction

Since these filters aren&rsquo;t included in the ned extinction calulator, I manually calculate the extinction. The E(B-V) value for a GRB is usually given in the GCN circulars for a burst in the UVOT section. Then I use values from Yi2023 for the extinction coefficients and can finally calculate the values using:
$A_{\lambda} = R_{\lambda} \times E(B-V)$
Where A is the extinction at given wavelength, and R is coefficinet at given wavelength.
Schlafly 2011


<a id="org1a5b71a"></a>

# Other optical data

