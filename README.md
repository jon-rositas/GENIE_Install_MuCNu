# GENIE_Install_MuCNu
The installation of GENIE that is used for some neutrino interaction studies in a future Muon Collider.

According to the GENIE website, "Due to removal of the ROOT/Pythia6 interface starting with ROOT v6.32, [version 3.6.2] is expected to be the last to use Pythia6 hadronization by default." Therefore, this installation includes steps to facilitate use with Pythia8. This installation assumes that you have ROOT installed.

This installation is assumed to be carried out in the directory `genie-install`

```
mkdir genie-install
cd genie-install
```

# DOWNLOAD
1. Go to the Genie Release github page https://github.com/GENIE-MC/Generator/releases/tag/R-3_06_02
2. Download the tar.gz file to `genie-install` (or wherever you are performing this installation).
3.  `tar -xvzf Generator-R-3_06_02.tar.gz`
4.  `rm Generator-R-3_06_02.tar.gz`

# FOUNDTIONS
Before you build, you will need to verify that you have the necessary dependencies.
```
cd /genie_install/
mkdir -p externals
cd externals
```
Run these commands. If they return nothing, then follow their blocks for installation:
```
echo $LHAPDF_INC
echo $GENIE
```

## No $LHAPDF_INC
Recall that we are using Pythia8 here. First install it.
```
wget https://www.pythia.org/download/pythia83/pythia8311.tgz
tar -xf pythia8311.tgz
cd pythia8311
./configure --prefix=$(pwd)
make
make install

wget https://lhapdf.hepforge.org/downloads/?f=LHAPDF-6.5.6.tar.gz -O LHAPDF-6.5.6.tar.gz
tar -xf LHAPDF-6.5.6.tar.gz
cd LHAPDF-6.5.6

./configure --prefix=$(pwd)/install PYTHON=python3
make
make install

rm LHAPDF-6.5.6.tar.gz
rm pythia8311.tgz
```
Then update your environment variables in your `~/.bashrc` file. Open `~/.bashrc` in your favorite text editor and append this to the end:
```
# =====================================================================
# GENIE & PYTHIA8 Environment Setup for Muon Collider Neutrino Study
# =====================================================================
export GENIE=/path/to/genie_install/Generator-R-3_06_02
export GENIE_TARGET=/path/to/genie_install/genie-v3.6.2
export PYTHIA8_DIR=/path/to/genie_install/externals/pythia8311
export LHAPDF6_DIR=/path/to/genie_install/externals/LHAPDF-6.5.6/install

# Targets for GENIE compilation
export GENIE_BIN=$GENIE_TARGET/bin
export GENIE_LIB=$GENIE_TARGET/lib

# Pythia8 specific flags GENIE looks for
export PYTHIA8_INC=$PYTHIA8_DIR/include
export PYTHIA8_LIB=$PYTHIA8_DIR/lib

export LHAPDF6_INC=$LHAPDF6_DIR/include
export LHAPDF6_LIB=$LHAPDF6_DIR/lib
# Update system paths
export PATH=$GENIE_BIN:$LHAPDF6_DIR/bin:$PATH
export LD_LIBRARY_PATH=$GENIE_LIB:$PYTHIA8_LIB:$LHAPDF6_LIB:$LD_LIBRARY_PATH
export GXMLPATH=$GENIE/config
```

NB: During my installation, I also needed to install to following to build properly:
```
sudo apt update
sudo apt install liblog4cpp5-dev
```

# BUILDING
```
source ~/.bashrc
cd $GENIE

./configure \
  --prefix=$GENIE_TARGET \
  --enable-fnal \
  --enable-baryon-residence \
  --enable-validation-tools \
  --disable-pythia6 \
  --enable-pythia8 \
  --disable-lhapdf5 \
  --enable-lhapdf6 \
  --with-pythia8-inc=$PYTHIA8_INC \
  --with-pythia8-lib=$PYTHIA8_LIB \
  --with-lhapdf6-inc=$LHAPDF6_INC \
  --with-lhapdf6-lib=$LHAPDF6_LIB \
  --with-optiz-level=O2

make
```
This should print a summary block of settings that you can glance at to verify they are to your liking.

# REMOVE PYTHIA6
Update the GENIE existing architecture that uses Pythia6 be defualt to use Pythia8 now

remove line 128 in `PythiaDecayer.xml`
```
cd $GENIE/config
find . -name "*.xml" -exec sed -i 's/Pythia6Decayer2023/Pythia8Decayer2023/g' {} +
find . -name "*.xml" -exec sed -i 's/AGCharmPythia6Hadro2023/AGCharmPythia8Hadro2023/g' {} +
find . -name "*.xml" -exec sed -i 's/Pythia6/Pythia8/g' {} +
```
verify that there are no remaining instances of Pythia6 with `grep -rn "Pythia6" .`

# GENIE WORKSPACE
I find it is best to work in a dedicated directory for GENIE generation that stores the relevant information close at hand without cluttering your install. So, we will create that here. Replace `$SPACE` with the preferred name of your workspace.
```
mkdir $SPACE && cd $SPACE
mkdir geometry
mkdir splines
```

## TEST RUN
We don't have any splines or geometry loaded yet, but we should be able to run a validation. It will take about an hour to finish because it has to do spline calculation itself on the fly. This wil generate one event and use tune `G18_10a_02_11a`, which is arbitrary in this step. However, this should validate the installation as it presently stands:

```
gevgen -e 100 -n 1 -p 14 -t 1000260560 --tune G18_10a_02_11a
```

### ROOT OUTPUT
Since GENIE output files aren't flat root files but have some extra C++ code in them, we have to prepare root to be able to open the GENIE output.
From within whatever directory you pefromed the simulation in:
```
root
gSystem->Load("libGFwUtl");
gSystem->Load("libGFwMsg");
gSystem->Load("libGFwGHEP");
TFile *f = TFile::Open("FILENAME.root");
```
You can then open the file however you'd like. I like TBrowser from within root with `new TBrowser`.

# GEOMETRY
GENIE needs to know what geometry you are trying to simulate neutrino interactions in for proper event generation. This is done simply by passing it the proper `gdml` file. It is recommended that you place the correct `gdml` files inside of your `$SPACE/geometry` directory.

# SUB 1 TeV SPLINES
While we do not yet have splines for the elements and isotopes in muon collider detectors above 1 TeV, we do have spline files under 1 TeV that we can use for validation studies in the meantime. These can be acquired from the [Fermilab SciSoft Server](https://scisoft.fnal.gov/scisoft/packages/genie_xsec/). Described here is the process for downloading the preliminary splines for validation. The reasoning for this tune and these splines is that they were simply the most recent set of splines I could find that included the elements we need. I adjusted my tune to work with them.

1. From the [SciSoft](https://scisoft.fnal.gov/scisoft/packages/genie_xsec/) page for GENIE splines, navigate to the [v3_06_00]([https://scisoft.fnal.gov/scisoft/packages/genie_xsec/v3_06_02_sbn2](https://scisoft.fnal.gov/scisoft/packages/genie_xsec/v3_06_00)) directory.
2. Download the `genie_xsec-3.06.00-noarch-G1802a00000-k250-e1000.tar.bz2` tarball
3. Navigate to your spline directory in your workspace `cd $SPACE/splines`
4. Move the tarball to `splines`
5. Untar the tarball `tar -xvzf genie_xsec-3.06.00-noarch-G1802a00000-k250-e1000.tar.bz2`

You will now have the splines accessible at `$SPACE/splines/genie_xsec/v3_06_00/NULL/G1802a00000-k250-e1000/data/`

We need a a large set of elements in our spline files to be able to work with our detector, so we use `gxspl-NUbig.xml.gz`, which is the "big" spline file including elements typically extraneous to neutrino detectors.

# GENERATING EVENTS
In order to generate events, you will need a `gsimple` flux file for GENIE to read. These can be generated by using the [Interactive Muon Collider Neutrino Flux Generator](https://github.com/jon-rositas/GEANT4_MuC_Nu_Flux_Gen). Assuming that you now have a valid `gsimple` file to pass to GENIE, events can be generated by navigating to your workspace with `cd $SPACE` and running the following command, replacing `$GEOM` and `$GSIMPLE` with your corresponding detector `gdml` file and `gsimple` flux `root` file, respectively:
```
nohup gevgen_fnal\
  -n 30\
  -f gsimple:$GSIMPLE,DET,-12,12,-14,14\
  -r 2\
  -g geometry/$GEOM\
  -t world_volume\
  --tune G18_02a_00_000\
  --cross-sections /SPACE/splines/genie_xsec/v3_06_00/NULL/G1802a00000-k250-e1000/data/gxspl-NUbig.xml.gz\
  > GENIE_event.log 2>&1 &
```
Where the flags and commands are explained as follows
```
nohup gevgen_fnal\  # nohup tells your terminal to run GENIE in the background and not stop running if your computer locks. gevgen_fnal runs GENIE
  -n {INTEGER}\     # Number of events to generate
  -f gsimple:{GSIMPLE_PATH},DET,-12,12,-14,14\  # File path to your (specifically) gsimple input. DET tells GENIE to consider the following pdg codes
  -r {INTEGER}\     # Run number, this is the only identifying info GENIE will let you set for your output file, so make sure you don't accidentally overwrite!
  -g {GDML_PATH}\   # Geometry path
  -t {WORLD_VOLUME_NAME}\  # Top-level volume name from gdml file so GENIE knows what geometry to actually try to simulate interactions inside of
  --tune {TUNE}\  # Tune information, must match your splines
  --cross-sections {SPLINE_PATH}\  # Cross-section information path, aka, path to your spline files
  > {LOG_NAME}.log 2>&1 & # ends the nohup command and defines a place to write the output since it won't show up in your terminal. Stores stdout and stderr
```

# FLATTENING OUTPUT
GENIE's output files don't play nicely with other scripts as they are saved to a specialized, proprietary output. To continue the investigation with these files, it is best to convert them to a flattened root file that can be more easily read by other platforms. This is accomplished with the script below where you should take care to use the run number that you specified in the previous step. This will give you the chance to name the output unqiuely as well.
```
gntpc -i gntp.{RUN_NUMBER}.ghep.root -f gst -o {OUTPUT_NAME}.root
```

This file may then be handed to GEANT4 to propagate and simulate all of the daughter particles that GENIE generated for the neutrino interaction vertex.






