# This folder contains all the necessary documentation for the geometric efficiency of FD Events at the ND 

The code as it is right now is taking events from the FD (information from the FD events is saved in FD ntuples - 
more info than the standard FD CAFs, they have info regarding simulated hadronic energy deposits at a given x, y, z position,
simulated muon momentum and vertex coordinates at the start and end position, type of neutrino interaction (CC or NC) etc

These FD ntuples are the starting point in the Geo Efficiency of FD events at the ND.

## Analysis steps 

In principle the working steps are the following:
1. Get the hadron geometric efficiency of FD events at the ND
2. Get the muon geometric efficiency (NN) of FD events that pass the hadron geo eff -> get the combined efficiecny (passing both hadron cuts and muon cuts)
3. Analyse the FD events selected at the ND (access Etrim histograms of each passing throw from hadron geo Eff, weight them by the probability the event passes the muon cuts, Pmu(Emu,throws)
4. Compare All selected FD events at the ND with PRISM linearly combined ND data

More details about each individual steps: 
1. Hadron Geometric efficiency of FD events at the ND 
