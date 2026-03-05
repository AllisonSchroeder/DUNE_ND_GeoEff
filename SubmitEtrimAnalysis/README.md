# 🟢 How to Submit Jobs for Etrim Analysis
This guide explains how to submit **Etrim Analysis** jobs on the Fermi computing grid.

One need to already have Hadron Geo Eff and Muon Geo Eff files in order to run this analysis step

## 1.0 Create working directory
```bash
mkdir SubmitEtrimAnalysisJobs
cd SubmitEtrimAnalysisJobs
```

## 1.1 First get the work env setup

you find it either at ```/exp/dune/app/users/icaracas/SubmitEtrimAnalysisOnGrid``` or on git at https://github.com/icaracas/DUNE_ND_GeoEff/blob/EtrimAnalysis/Grid/

you need ```setupNDEff-grid.sh```, ```run_NDEff_autogrid.sh``` and / or ```run_NDEff_autogridOscSpectrum.sh```

use ```run_NDEff_autogrid.sh```  for the "No Oscillations" scenario and ```run_NDEff_autogridOscSpectrum.sh``` for the "with Oscillations" scenario

Things you might want to change:
- in ```setupNDEff-grid.sh```: the currently working and cloned git repository is my git repo: https://github.com/icaracas/DUNE_ND_GeoEff.git
	 -- for the future you might want to clone everything from my repo and make your own git repository where you would make any modifications, so the replace this repo with your own repository
- in ```run_NDEff_autogrid.sh``` : output directory where your files are saved; now it is OUTDIR=/pnfs/dune/scratch/users/${GRID_USER}/EtrimAnalysis_FromFDGeoEffinND/CombinedEffWithCAFLikeMuCuts/AllOAPosEtrueProb/NoOsc -> you might want to make your own named directory etc
- in ```run_NDEff_autogridOscSpectrum.sh``` : output directory where your files are saved; now it is ```OUTDIR=/pnfs/dune/scratch/users/${GRID_USER}/EtrimAnalysis_FromFDGeoEffinND/CombinedEffWithCAFLikeMuCuts/AllOAPosEtrueProb/OscSpectrumNonNegativeEvRateVals``` -> you might want to make your own named directory etc

## 1.2 Write the list of input root files (from hadron and from muon geo eff) to a txt file each

```bash
ls -d "/pnfs/dune/persistent/users/icaracas/FDGeoEffFiles/FromFlynnsNtuples/NDFV400m/HadronEff/WithTrackLenghtInfo/"/* | sed "s\/pnfs\root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr\g" > myOutputFDGeoEffHadron.txt
ls -d "/pnfs/dune/persistent/users/icaracas/FDGeoEffFiles/FromFlynnsNtuples/NDFV400m/MuonEff/WithTrackLenghtInfo/"/* | sed "s\/pnfs\root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr\g" > myOutputFDGeoEffMu.txt
```

-- this also changes pnfs to xrootd so that worker node can access

Things you need to change:

```/pnfs/dune/persistent/users/icaracas/FDGeoEffFiles/FromFlynnsNtuples/NDFV400m/HadronEff/WithTrackLenghtInfo/``` with the path of your output files from Hadron Geo Eff
```/pnfs/dune/persistent/users/icaracas/FDGeoEffFiles/FromFlynnsNtuples/NDFV400m/MuonEff/WithTrackLenghtInfo/``` with the path of your output files from Muon Geo Eff


## 1.3 Now make the tarball,

```bash
tar -czvf TarEtrimAnalysis.tar.gz setupNDEff-grid.sh myOutputFDGeoEffHadron.txt myOutputFDGeoEffMu.txt
```


## 1.4. submit jobs

this submits N jobs (N = number of input files, so each job runs 1 file)

-- the nr of N jobs should be the same as the number of lines in either myOutputFDGeoEffHadron.txt or myOutputFDGeoEffMu.txt (those 2 files should have the same nr of lines / files )
```bash
jobsub_submit -G dune -N 9083 --memory=8GB --disk=1GB --expected-lifetime=3h --cpu=1 --resource-provides=usage_model=DEDICATED,OPPORTUNISTIC,OFFSITE --tar_file_name=dropbox:///exp/dune/app/users/icaracas/SubmitEtrimAnalysisOnGrid/TarEtrimAnalysis.tar.gz --use-cvmfs-dropbox -l '+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest\"' --append_condor_requirements='(TARGET.HAS_Singularity==true&&TARGET.HAS_CVMFS_dune_opensciencegrid_org==true&&TARGET.HAS_CVMFS_larsoft_opensciencegrid_org==true&&TARGET.CVMFS_dune_opensciencegrid_org_REVISION>=1105&&TARGET.HAS_CVMFS_fifeuser1_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser2_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser3_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser4_opensciencegrid_org==true)' file:///exp/dune/app/users/icaracas/SubmitEtrimAnalysisOnGrid/run_NDEff_autogrid.sh
```

Things you should change:

```/exp/dune/app/users/icaracas/SubmitEtrimAnalysisOnGrid/``` with the path to your working directory SubmitEtrimAnalysisJobs
                                   
