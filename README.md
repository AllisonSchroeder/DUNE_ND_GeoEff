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
## 1. Hadron Geometric efficiency of FD events at the ND 
The code for this step can be found on the ```master``` branch of this repository: https://github.com/icaracas/DUNE_ND_GeoEff/

  - the main code of interest is in ```app/runGeoEffFDEvtSim.cpp``` 

The code itself is heavily based on Flynn's previous work.

### 1.1 Read information from the FD nutples
The FD ntuples used as an input can be found at ```/pnfs/dune/persistent/users/flynnguo/myFDntuples/myntuple_53993115```
Several variables like neutrino energy, hadron energy deposits and their coordinates in the FD (x,y,z) as well as muon momentum (px, py, pz) and muon energy at the start and end position, etc are read from these ntuples via ```t->SetBranchAddress()```  

One can add additional variables of interest from the ntuple to the analysis by simply reading-in a new branch

### 1.2 Translate the FD event in the ND
Take the FD event (from the ntuple) and put it in the ND:     
  - 1. account for Earth curvature rotation: function that does this is ```eff->getEarthCurvature()``` defined in ```src/geoEff.cpp```
  - 2. FD event to ND: after Earth curvature rotation: get hadron positions and muon start/end positions after Earth curvature rotation

        -- variables: ```ND_RandomVtx_Sim_mu_start_v, ND_RandomVtx_Sim_mu_end_v, ND_RandomVtx_Sim_mu_start_p, ND_RandomVtx_Sim_hadronic_hit_xyz```
  - 3. ND: move back to the beam center (once we properly accounted for the Earth curvature rotation we want to move the event in the beam center in the ND) :

       -- variables: ```ND_OnAxis_Sim_mu_start_v, ND_OnAxis_Sim_mu_end_v, ND_OnAxis_Sim_mu_start_p, ND_OnAxis_Sim_hadronic_hit, ND_OnAxis_Sim_hadronic_hit_xyz```

       -- function: ```eff->getTranslations```defined in ```src/geoEff.cpp```

  - 4. Translate from On-axis to Off-axis: by default, at step 3 we put the neutrino (so the event) on axis. this step is moving the event from on-axis to off-axis according to the off-axis position

       -- variables: ```ND_OffAxis_Unrotated_Sim_mu_start_v_xyz_LAr, ND_OffAxis_Unrotated_Sim_mu_end_v_xyz_LAr, ND_OffAxis_Unrotated_Sim_mu_start_p_xyz_LAr, ND_OffAxis_Unrotated_Sim_hadronic_hit_xyz_LAr```

    - only change the x component ( our detector is moved along the x direction) , y and z components remain unchanged for muon
       ```bash
       ND_OffAxis_Unrotated_Sim_mu_start_v[0] = ND_OnAxis_Sim_mu_start_v[0] + i_ND_off_axis_pos + i_vtx_vx;
       ```
    - for hadron part, use the already rotate muon (again just x direction is actually different) and translate the hadron deposits at a different off-axis position
      ```bash
      ND_OffAxis_Unrotated_Sim_hadronic_hit_xyz.emplace_back((float)eff->getTranslations(ND_OnAxis_Sim_hadronic_hit_array, ND_OnAxis_Sim_mu_start_v, ND_OffAxis_Unrotated_Sim_mu_start_v, i));
      ```
  - 5. ND Get eigen rotated vectors for step 4 ': here nothing is changed for the muons (we don't rotate the muons in this code), as the function ` eff->RemainUnchanged` is used, but this is wehere we define the variables further used for the off axis positions:

        -- variables for muons at different off-axis positions: ```ND_OffAxis_Sim_mu_start_v_xyz, ND_OffAxis_Sim_mu_end_v_xyz, ND_OffAxis_Sim_mu_start_p_xyz_vtx, ND_OffAxis_Sim_mu_start_E_xyz_vtx ```

        -- variables for hadrons: ```ND_OffAxis_Sim_hadronic_hit```

        -- function : ```Eff->getOffAxisHadronHitV``` defined in ```src/geoEff.cpp```

       At this point we have information about the position of the hadron hits inside the ND for each off axis position (at each detector positions chosend and at each vtx X positions generated),  ```ND_OffAxis_Sim_hadronic_hit``` and can access variables like total hadron energy deposited in the FD, as taken from ntuples, ```totEnergyFDatND_float```, energy deposited in the ND veto region (30 cm) , ```vetoEnergyFDatND_float```, or hadron energy deposited outside the ND active volume, ```outEnergyFDatND_float```. This energy deposited outside the ND active volume is energy that our ND would never see and that is why we trim it out.

       -- functions that return the veto energy, ```eff->getVetoEPerThrow_FD_GEC()``` and the trimmed energy (```TrimE = totEnergyFDatND_float - outEnergyFDatND_float```) , ``` eff->getTrimEPerThrow_FD_GEC()``` on a throw by throw basis (see point 6 for more details about the throws) are defined in ```src/geoEff.cpp```

  - 6. Generate random throws
       



