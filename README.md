# Instruction for running translation and rotations on FD n-tuples

Prerequisite: [Produce Ntuple from DUNE FD MC files](https://github.com/weishi10141993/myntuples#produce-ntuple-from-dune-fd-mc-files). The produced FD n-tuples will be used as input files for the following program to run. (Flynn Guo has previously made ntuple files to use, which can be located here: /pnfs/dune/persistent/users/flynnguo/myFDntuples)


Open SL7 container:
```
/cvmfs/oasis.opensciencegrid.org/mis/apptainer/current/bin/apptainer shell --shell=/bin/bash \
-B /cvmfs,/exp,/nashome,/pnfs/dune,/opt,/run/user,/etc/hostname,/etc/hosts,/etc/krb5.conf --ipc --pid \
/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-dev-sl7:latest
```

[First time only]
```
cd /exp/dune/app/users/$USER
mkdir NDEff (first time only)
cd NDEff
git clone -b master https://github.com/AllisonSchroeder/DUNE_ND_GeoEff.git      # Get geoEff library
cd DUNE_ND_GeoEff
source setup.sh                                                                                    # Necessary setups for build
cmake -DCMAKE_CXX_STANDARD=17 -DPYTHON_EXECUTABLE:FILEPATH=`which python` .                        # -DCMAKE_CXX_STANDARD=17 for running with root 6.28.10
make -j geoEff                                                                                     # Build geoEff (can also use: make -j pyGeoEff)
```

To (re)compile
```
cd /exp/dune/app/users/$USER/NDEff/DUNE_ND_GeoEff/
#
# In case you log out, need to source setup.sh to setup ROOT
#
source setup.sh         

# Compile program
cd app
make runGeoEffFDEvtSim                                                                       
```

To (re)run program,
```
cd ../bin
# Usage: ./runGeoEffFDEvtSim inputFDntuple
# for example: 
./runGeoEffFDEvtSim /exp/dune/app/users/alschroe/NDEff_New/DUNE_ND_GeoEff/myntuple_53993115_5466.root
```
this will produce a root file containing throws and the hadron throw result.

If the source files in src are changed, recompile:

```
source setup.sh
cmake -DPYTHON_EXECUTABLE:FILEPATH=`which python` .
make -j geoEff    
```

Tips on DUNE FNAL machines: if want to run interactively for longer time even when terminal connection is lost, use screen option:

```
screen
# do the enviroment setup, in this case: source setup.sh
nohup ./runGeoEffFDEvtSim >& out_throws_nohup.log &                                                # Check status: jobs -l
# To detach from the screen session, press Ctrl+a (release) and then d to detach the process/screen.
# To resume detached process, use: screen -r
# 10k evts: 6.20pm start, end second day 4:52am, 10hrs32mins
```

## Instruction for calculate FD event efficiency

The output root file from running ```runGeoEffFDEvtSim``` can be used to calculate FD event hadron containment efficiency by running:

```
cd /exp/dune/app/users/$USER/NDEff/DUNE_ND_GeoEff
source setup.sh
cd app
root -l -b -q FDEffCalc.C
# 5k evts: 10mins
```

The output root file from running ```runGeoEffFDEvtSim``` can also be used for lepton NN training.

## Event displays

The hadronic hits can be plotted in 2D event displays at FD via:
```
echo 'gROOT->ProcessLine(".L ReadHadronHitNtuple.cpp"); ReadHadronHitNtuple_FD()'| root -l -b
```

## Run on Grid

See instructions under the Documentation branch -> SubmitHadronGeoEff
