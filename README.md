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

# TEST RUN
We don't have any splines loaded yet, but we should be able to run a validation. It will take about an hour to finish because it has to do spline calculation itself on the fly. However, this should validate the installation as it presently stands:
Change directory to wherever you'd like to perform the event generation, then run.
```
gevgen -e 100 -n 1 -p 14 -t 1000260560 --tune G18_10a_02_11a
```

# ROOT OUTPUT
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








