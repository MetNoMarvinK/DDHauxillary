# DDH in HARMONIE-AROME

Diagnostics on a horizontal domain (DDH) was originally developed in Météo-France as a tool to provide, on user-defined domains, 
the budget of prognostic variables of NWP models (momentum, temperature, water vapour, etc). The DDH tool is used by researchers 
and model developers to understand the model’s dynamical and physical interactions and tendencies, thus contributing to the 
parametrization development process. Each model point is described within the DDH software by its geographical position, 
a scale factor, the orientation of the geographical North vector, a mean value of each variable as well as some horizontal 
derivatives.

# Running an experiment with DDH (on ATOS)

## 1) You can use the main CY46 repo as the changes are there now.

```bash
mkdir -p $SCRATCH/harmonie_releases/git/HCY46_DDH
cd $SCRATCH/harmonie_releases/git/HCY46_DDH
git clone git@github.com:Hirlam/Harmonie.git
cd Harmonie
```
## 2) Set-up a HARMONIE experiment as normal. Update your config_exp.h

- To use DDH, set USEDDH="yes" in ecf/config_exp.h
- To archive DDH data to ec or ectmp include ddh in the archiving strategy in ecf/config_exp.h

```bash
ARSTRATEGY="climate:fg:verif:odb_stuff: \
              [an|fc]_fa:pp_gr: \
              _nc:_json:ddh"                  # Files to archive on ECFS, see above for syntax
```

- Set TFLAG to minute (min) if you wish to have minute output (hourly output is OK too). Note that the conversion of FA to grib format does not automatically work with this setting so it is best to set the conversion to "no".

If TFLAG="min", you need to change the file output times to something like
```bash
  HWRITUPTIMES="00-4320:60"               # History file output times
  FULLFAFTIMES=$HWRITUPTIMES              # History FA file IO server gather times
  PWRITUPTIMES="00-4320:15"               # Postprocessing times
  VERITIMES="00-4320:60"                  # Verification output times, may change PWRITUPTIMES
  SFXSELTIMES=$HWRITUPTIMES               # Surfex select file output times
  SFXSWFTIMES=-1                          # SURFEX select FA file IO server gathering times
  SWRITUPTIMES="00-180:60"                      # Surfex model state output times
  SFXWFTIMES=$SWRITUPTIMES                # SURFEX history FA file IO server gathering times
```
## 3) Update your namelist (first copy nam/harmonie_namelists.pm your own experiment and place in nam folder which you'll have to create).

Setting up the namelist (nam/harmonie_namelists.pm).

A. Below is an example where point data for Dublin is extracted.
```bash
%ddh=(
NAMDDH => {
'LFLEXDIA' => '.TRUE.,', # Must be TRUE
'BDEDDH(1,01)' => '4.,', # 4 means a point
'BDEDDH(2,01)' => '1.,',
'BDEDDH(3,01)' => '-006.000000,', # Dublin lon
'BDEDDH(4,01)' => '0053.000000,', # Dublin lat
'LHDGLB' => '.FALSE.,',
'LHDZON' => '.FALSE.,',
'LHDDOP' => '.TRUE.,', # Must be TRUE
'LHDPRG' => '.FALSE.,',
'LHDPRZ' => '.FALSE.,',
'LHDPRD' => '.FALSE.,',
'LHDEFG' => '.FALSE.,',
'LHDEFZ' => '.FALSE.,',
'LHDEFD' => '.TRUE.,', # Must be TRUE
'LHDHKS' => '.TRUE.,', # Must be TRUE
'LHDMCI' => '.FALSE.,',
'LHDENT' => '.FALSE.,',
},
);
```
B. Below is an example where the point with index (100,100) is chosen.
```bash
%ddh=(
NAMDDH => {
'LFLEXDIA' => '.TRUE.,', # Must be TRUE
'BDEDDH(1,01)' => '1.,', # 4 means a point
'BDEDDH(2,01)' => '1.,',
'BDEDDH(3,01)' => '100.,', 
'BDEDDH(4,01)' => '100.,',
'LHDGLB' => '.FALSE.,',
'LHDZON' => '.FALSE.,',
'LHDDOP' => '.TRUE.,', # Must be TRUE
'LHDPRG' => '.FALSE.,',
'LHDPRZ' => '.FALSE.,',
'LHDPRD' => '.FALSE.,',
'LHDEFG' => '.FALSE.,',
'LHDEFZ' => '.FALSE.,',
'LHDEFD' => '.TRUE.,', # Must be TRUE
'LHDHKS' => '.TRUE.,', # Must be TRUE
'LHDMCI' => '.FALSE.,',
'LHDENT' => '.FALSE.,',
},
);
```
# Examining the output

As a result of your experiment the system produces DDH files with names of the form DHFDLHARM+XXXX, located in ectmp (on Atos) and in the forecast folder in the work directory e.g. $SCRATCH/$USER/hm_home/46h1_ddh/2024061200_00/forecast/.
As they work directories are frequently cleaned by the scripting system, you will most likely find your files in ectmp.

If you do not have any DDH files of your own, you can find some at https://opensource.umr-cnrm.fr/attachments/5978

The content of these files can be viewed using the lfa tools in the ddhtoolbox. To download the toolbox, you can download it from github:

https://github.com/UMR-CNRM/ddhtoolbox

After having the folder,

```bash
cd /ddhtoolbox/tools/
export PATH=.:$PATH
```

and run two scripts to install it:

```bash
./install clean
./install
```

Now the various "tools" are compiled. To enable the fa tools, you need to set up some paths. For example on ATOS, you need to set the following paths (which may differ a bit for you depending on where you downloaded the ddhtoolbox to).

```bash
export PATH=$HOME/ddhtoolbox/tools/lfa:$PATH
export PATH=$HOME/ddhtoolbox/tools:$PATH
```

Now you may use the lfa tools to view the contents of a DHFDLHARM+XXXX file by typing

```bash
lfaminm /path/to/DHFDLHARM+XXXX
```

Something like this should be seen:

```bash
l=     715, min=   0.000     max=  0.7583E-20 mea=  0.1061E-22 rms=  0.2836E-21|R4| TQRADJU
l=     715, min=   0.000     max=  0.3803E-05 mea=  0.3413E-07 rms=  0.2528E-06|R4| TQIADJU
l=     715, min=   0.000     max=  0.2221E-21 mea=  0.4995E-24 rms=  0.9278E-23|R4| TQSADJU
l=     715, min=   0.000     max=   0.000     mea=   0.000     rms=   0.000    |R4| TQGADJU
l=     715, min=   0.000     max=   164.8     mea=   6.129     rms=   27.53    |R4| VNT1
l=     715, min=   0.000     max=   12.39     mea=  0.3109E-01 rms=  0.4998    |R4| VNT0
l=     715, min= -0.1473E+06 max=  0.3615E+05 mea= -0.2768E+05 rms=  0.3256E+05|R4| TCTRAD
l=     726, min=   0.000     max=  0.1080E-10 mea=  0.1636E-12 rms=  0.1329E-11|R4| FCTRAYSO
l=     726, min= -0.2967E+07 max= -0.3987E+06 mea= -0.1693E+07 rms=  0.1804E+07|R4| FCTRAYTH
l=     715, min=   0.000     max=  0.1818E+05 mea=   155.8     rms=   1272.    |R4| TKESHEAR
l=     715, min=  -1.319     max=   783.6     mea=   2.660     rms=   38.88    |R4| TCTUP
l=     715, min= -0.3119E-03 max=  0.5250E-06 mea= -0.1059E-05 rms=  0.1547E-04|R4| TQVUP
l=     715, min=  -5541.     max=   5721.     mea=  0.4685E-02 rms=   297.9    |R4| TCTSCONV
l=     715, min= -0.1298E-02 max=  0.1349E-02 mea=  0.1021E-08 rms=  0.7002E-04|R4| TQVSCONV
l=     715, min=  -108.4     max=   146.7     mea= -0.1069     rms=   10.04    |R4| TUUVTUR
```

The names on the right can be interpreted as follows:

```bash
The first letter of the name is the info on the type of field:
V: variable 
T: tendency 
F: flux 
S: soil
```

The second and third letters of the name represented the physical variable:
```bash
PP: pressure,
QV: specific water vapour content,
QL: cloud liquid,
QI: cloud ice,
QR: rain,
QS: snow,
QG: graupel,
UU: zonal momentum,
VV: merional momentum,
WW: omega,
KK: kinetic energy,
CT: thermal energy,
EN: entropy,
M1: angular momentum,
EP: potential energy (Φ = g z).
```

The next 10 characters (suffix) represent the field-specific name.

```bash
For example FCTRAYSO stands for thermal energy flux caused by solar radiation (i.e. F=Flux, CT= thermal energy, RAYSO=Solar radiation)
```
```bash
**Exercise: **
Pick a physical variable, like thermal energy (CT), and try to understand which tendency components affect it. 
Find all variables of the form TCTxxxxxxxxxx and try to interpret what they mean.
Hint: very detailed ddh documentation about the variable names can be found at /path/to/ddhtoolbox/documentation/ddh.pdf. It should solve most of the problems you will ever face with ddh.
```
# Visualising the output

The next step is to visualize the file content. For that, we have prepared a set of ways to do it.

## 1.Use the ddhtoolbox

This toolbox includes ready-to-be-used scripts for ddh files. To enable these, export one more path and three variables:

```bash
export PATH=/path/to/ddhtoolbox/tools/.dd2gr/src:$PATH
export DDHB_BPS=/path/to/ddhtoolbox/ddh_budget_lists
export DDHI_LIST=/path/to/ddhtoolbox/ddh_budget_lists/conversion_list
export DDH_PLOT=dd2gr
```

Now one may visualize the content of a file in multiple different ways. Perhaps the most simple way is to illustrate some variable's budget at a particular time step. To do this, we use the script called ddhb (= ddh budget). NB ensure you are in the directory containing the DDH files and ensure files are for one domain only:

```bash
ddhb -v harome_46h1/CT -i DHFDLHARM+0003 -o DHFDLHARM+0003.CT.svg
```

This creates an .svg file of the temperature budget based on the input file DHFDLHARM+0003. The .svg file can be converted to e.g. .png with the command

```bash
convert filename.svg filename.png
```

A sample image called "CT_example.png" can be found in the attachments of this page.

```bash
The argument -v harome_46h1/CT points to the temperature budget list located in $DDHB_BPS/harome_46h1/CT.fbl (see above). Similarly, you can plot the budget of water vapour with harome_46/QV or budget of zonal momentum with harome_46h1/UU.
```

Note: in case you have run an experiment with multiple domains (="ddh points"), you may want to horizontally average all the points before running ddhb to produce a meaningful image. One may do this with ddht (=ddh transform):

```bash
ddht -cMOY_HORIZ -1DHFDLHARM+0003 -sDHFDLHARM+0003.mean
```

Note: there is no whitespace after -c/-1/-s when using ddht.

## 2. Daniel's scripts - See https://opensource.umr-cnrm.fr/attachments/5803. NB ensure you use latest version of Python on ATOS.

## 3. Marvin's scripts
