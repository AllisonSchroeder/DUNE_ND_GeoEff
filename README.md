# Instructions to Run EtrimAnalysis Interactively on Fermi Grid

## 1. open an SL7 Container 
```
/cvmfs/oasis.opensciencegrid.org/mis/apptainer/current/bin/apptainer shell --shell=/bin/bash -B /cvmfs,/exp,/nashome,/pnfs/dune,/opt,/run/user,/etc/hostname,/etc/hosts,/etc/krb5.conf --ipc --pid /cvmfs/singularity.opensciencegrid.org/fermilab/fnal-dev-sl7:latest
```

## 2. (First time only) Setup directory:

  2.1 Move to your directory cd ```/exp/dune/app/users/$USER/NDEff```

  2.2 Make directory for muon code: ```mkdir EtrimAnalysis```

  2.3 Move to muon code directory: ```cd EtrimAnalysis```

  2.4 Clone muon code branch: ```git clone -b EtrimAnalysis https://github.com/AllisonSchroeder/DUNE_ND_GeoEff.git```

## 3. Move to EtrimAnalysis Directory
```
cd PATHToMuonEffCode: cd  /exp/dune/app/users/$USER/NDEff/EtrimAnalysis/DUNE_ND_GeoEff/Grid
```
## 4. Setup
```
source setupsForOsc.sh
```
(re)compile code
```
make
```
## Run for with or without oscillations
for no oscillations:
```
./NtupleOutVetoAndTrimE_AssumeEqualEffAtAllOA_NOOscSpectrumNOCAFAna_FDRateAtND HadronOutputFile  MuonOutputFile
```
for oscillations:
```
./NtupleOutVetoAndTrimE_AssumeEqualEffAtAllOA_WithOscSpectrumNOCAFAna_FDRateAtND HadronOutputFile  MuonOutputFile
```
where HadronOutputFile is the output file from the Hadron Geometric Efficiency step (see readme in master branch) and MuonOutputFile is the output file from the Muon Geometric Efficiency step (see readme in MuonCode branch)
