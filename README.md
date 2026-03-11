# This folder contains all the necessary documentation for the geometric efficiency of FD Events at the ND 

The code as it is right now is taking events from the FD (information from the FD events is saved in FD ntuples - 
more info than the standard FD CAFs, they have info regarding simulated hadronic energy deposits at a given x, y, z position,
simulated muon momentum and vertex coordinates at the start and end position, type of neutrino interaction (CC or NC) etc

These FD ntuples are the starting point in the Geo Efficiency of FD events at the ND.

The FD events are then translated to the ND where random throws are generated and the hadron and muon geometric efficiency of a given FD event at the ND is then calculated on a data-driven approach. The last part of the code is regarding the analysis of the FD events translated to the ND, more explicitly how to translate the obtained results in terms of a PRISM-dedicated analysis. The main goal (so far) is to be able to obtain a distribution of the selected FD events in the ND that would match the linearly combined selected ND events obtained from the ND-CAFs with a model-dependent MC efficiency. To fulfill this goal a very important variable is the **energy**, as we want to make sure the same variable is comparred. Currently we use the deposited energy (obtained from GEANT 4) for the hadrons in the ND + true muon energy. For the ND-CAFs this variable is already defined within the CAFAna framework and is called ```ERec```. For the FD events in the ND this variable is further referred to as ```VisEtrim``` and is the sum between the hadronic energy deposited inside ND-LAr active volume (```TrimEnergy```) and true muon energy. 

The information below tries to give an overview of the several steps used in the analysis to obtain the end-goal comparison between the FD selected events at the ND and the linearly combined selected ND events as obtained from MC NDCAFs.

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
  
  - header with various dimensions and variables / constants definition is at ```app/Helpers.h```

  - file with most of the geometric efficiency coded functions is at  ```src/geoEff.cpp```

The code itself is heavily based on Flynn's previous work.
  ## Big picture 

We work with neutrino events simulated inside the FD. We take the (only CC interactions)  events from the FD ntuples located at ```/pnfs/dune/persistent/users/flynnguo/myFDntuples/myntuple_53993115```: 1 neutrino event has inofrmation about the hadron energy depoits and muon (coordinates x, y, z and amount of energy and momentum) in the FD. We then translate the FD event (with all the hadron and muon info) to the ND (accounting for Earth curvature).

We then put the ND at different positions with respect to the neutrino beam axis: variable to set the ND position is ```ND_LAr_dtctr_pos_vec```. In the current version of the code we only set the ND on-axis, i.e ```ND_LAr_dtctr_pos_vec.emplace_back(0);``` but you can easily change this with different off-axis positions. For example if interested in having the detector at 30m off axis use ```ND_LAr_dtctr_pos_vec.emplace_back(-3000)```. detector positions are given in cm. If you want to run with different detector positions at once just add all the desired detector positions in this variable.

The loop of events is the following: loop over neutrino events in FD (CC interactions only) -> loop over detector positions -> loop over vtx_x inside the ND -> loop over throws. Therefore, each neutrino event simulated in the FD which has a CC interaction, will be put in the ND. Then: 
  
  - for a fixed detector position (let's say on axis for now) we place the event at different vtx_x position inside the ND. This is done using the ```generatePoints``` function inside the code, and picking from an array of ```non_dead_regions``` in the ND. Once the vtx_x position has been fixed :
      - for a fixed vtx_x position inside the ND: we generate random throws of the event (particularly of the hadron energy deposits) at vtx_y, vtx_z and with a given rotation. basically once the event is posistioned at a fixed vtx_x position inside the ND we start randomly `throwing` (or moving) the event at various positions inside the ND in y and z coordinates, as well as rotating the event under different configurations.
          - for each throw we can access the **passing result** ```throw_result```, i.e if the given event under the given configuration (vtx_x, vtx_y, vtx_z and rotation) has passed the hadronic veto cut (i.e less than 30 MeV deposited in the outer 30cm of the ND) , the amount of hadronic energy deposited inside ND-LAr active volume (further referred to as **Trim Energy, Etrim** ```TrimEnergy``` in the code)  the amount of energy deposited outside the ND-LAr active volume, the amount of energy deposited in the veto region. Since this is inside the throws loop, every variable we save here (for example trim energy) will have a given distribution (many throws) at fixed vtx_x and detector position 
      - Once the throw loop is exited, using the information above, we can get the ```hadron geometric efficency``` = PassingThrows/ValidThrows at a fixed vtx_x position and fixed detector position  

After having run this code we have information regarding the hadron geometric efficiency of FD events as seen by the ND accounting for the different volumes between the FD and the ND: since FD has a much bigger volume, events with high hadronic energy deposits would never be seen entirely by the ND, much of this energy would be deposited outside of the ND-LAr active volume, which is why we in fact want to get the ND sensitivity to FD events with respect to the energy deposited inside the ND-LAr active volume, the trim energy Etrim -> we can get information regarding the efficiency and hadronic deposited energy of a given FD event inside the ND-LAr active volume (Etrim)

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
      
  - 6. Generate random throws
       At a fixed ```vtx_x``` position the event is thrown randomly at different ```vtx_y, vtx_z ``` with different rotations. The values for these throws are generated using the ```eff->throwTransforms()``` function and they are further saved in the output file using:
    ```bash       
      throwVtxY.clear();
      throwVtxZ.clear();
      throwRot.clear();
      throwVtxY = eff->getCurrentThrowTranslationsY();
      throwVtxZ = eff->getCurrentThrowTranslationsZ();
      throwRot  = eff->getCurrentThrowRotations();
      ThrowsFD->Fill();
    ```
      the result of each throw (passing the hadron veto cut or not) is saved in the variable ```hadron_throw_result``` , calculated using the function ```eff->getHadronContainmentThrows_FD_GEC(false)``` defined in ```src/geoEff.cpp```
  
      -- functions that return the veto energy, ```eff->getVetoEPerThrow_FD_GEC()``` and the trimmed energy (```TrimE = totEnergyFDatND_float - outEnergyFDatND_float```) , ``` eff->getTrimEPerThrow_FD_GEC()``` on a throw by throw basis are defined in ```src/geoEff.cpp```

      -- while looping over each throw (each detector position, each vtxX position, each throw) we only save information about passing throws (passing throws has a non zeo hadron result: ```throw_result != 0 ``` . information we need later in the analysis is the trim energy of each passing throw, ```TrimEnergyEventsPass.emplace_back(trimEThrow[0][0][(counter5-1)*64 + ithrow])```

      -- also save the number of times an event passed the throws, ```hadronpass```, and then divide to the total number of throws, ```validThrows```, to get the hadron geometric efficiency of the particular FD Event, at a given detector position in a given vtx_x position inside the detector:
    ```bash
    if ( validthrows > 0 ) hadron_contain_eff = hadronpass*1.0/validthrows;
    ND_GeoEff = hadron_contain_eff;
    ```

    ### 1.3 Write important variables to the output file:

    - variables further used for the muon efficiency part:
            
       muon info: ```ND_OffAxis_Sim_mu_start_v_xyz_LAr, ND_OffAxis_Sim_mu_start_p_xyz_LAr", ```

       throw info: ```throwVtxY, throwVtxZ, throwRot,  hadron_throw_result_LAr```
      

    - variables used in the end analysis:
   
        vertex_x and detector postions: ```ND_LAr_vtx_pos, ND_LAr_dtctr_pos ```

        GeoEff and throws info: ```ND_GeoEff, NPassedThrows, TrimEnergyEventsPass```

        Event info: ```ND_Gen_numu_E, ND_E_vis_true, totEnergyFDatND_f, muonEdep_f, muonTrackLength_f, ND_OffAxis_Sim_mu_start_E_xyz_LAr```
      
        
More variables (than the ones enumerated here) are written in the output file, but I think these are the most important ones so far. 


## 2. Muon Geometric efficiency of FD events at the ND

The code for this step can be found on the ```MuonCode``` branch of this repository: https://github.com/icaracas/DUNE_ND_GeoEff/
  
  - the main code of interest is in ```code/FD_maketree.py``` 

The code itself is based on Flynn's previous work but has several additions / changes. However the NN part has not been touched - don't have more details regarding to how this one works..
 ## Big picture 

 This part of the code is using information obtained from the previous, hadron geometric efficiency step. The input file needed for this step is the output file obtained after having run the hadron geometric efficiency. Since in the end we are interested in events that pass both the hadron and muon selection cuts, the muon geometric efficiency is only calculated for events that have already passed the hadron geometric efficiency cut (i.e an event that doesn't pass the hadron veto cut would not be selected at the ND so it is useless to calculate the muon efficiency for such an event).  The muon geometric efficency is however following the same approach. A FD event is translated to the ND, the ND is positioned at different off-axis positions with respect to the neutrino beam, variable in this code that sets the ND position is ```LAr_position```. It is important to set the same value here as you set in the hadron geometric efficiency part. For each of the throws that pass the hadron geometric selection, for a fixed detector position we put the event at different vtx_x positions (same values for vtx_x as used in the hadron geo efficiency step). For each of these configurations of the event we use a NN that tells us whether the muon is either contained in ND-LAr or further matched int the downsream tracker, TMS. Once the NN ran we can acces the information related to : proability muon is contained in NDLAr, ```nnContained``` , probability the muon is matched in TMS, ```nnTracker``` or the combined muon selection efficiency = ```nnContained + `nnTracker``` per throw. Again since we run this code on events that have already passed the hadron selection, the results obtained from ```nnContained``` resp ```nnTracker``` are already in terms of **combined efficiecny = muon(contained or tracker) and hadron efficency** (hadron efficiency is already applied). However since in the end we care about muon probability to be either contained in ND-LAr *OR* matched in tracker, the variable of interested is `weightPmuon = nnContProb*hadresult + nnTrackerProb*hadresult` which gives us exaclty this probability. This is saved on a throw by throw basis as well and can further be used in the end analysis ( see point 3 ) 

### 2.1 Read information from the root files obtained from the hadron geo eff part

If for example your output file from the hadron geo eff part is called OutputHadronGeoEff.root than run the muon efficiency code as: ```python FD_maketree.py OutputHadronGeoEff.root``` . As a result you will obtain a file with information regarding the combined (hadron and muon efficiency).

  - variables needed as an input:

    muon info: position and momentum of the muon startign point (x, y, z) `ND_OffAxis_Sim_mu_start_v_xyz_LAr`, `ND_OffAxis_Sim_mu_start_p_xyz_LAr`

    throws info: result of the hadron throws (only saved for throws that pass the hadron selection cuts), `hadron_throw_result_LAr`, throws coordinates: `throwVtxY`, `throwVtxZ`, `throwRot`
 
### 2.2 Get Muon momentum direction vector

Need to get the corresponding coordinates of the muon direction at a givent detector position and vtx_x position inside the detector. For each neutrino events in the FD -> for each detector position -> for each vtx_x position -> for each passing throws from the hadron geo eff part:
    #### Get variables needed to evaluate muon neural network for each throw
   - Prepare vertex positions for each random throw:

        x coordinate is fixed `this_vtx_x` and repeated for all throws (same values as in hadron geo eff). -> `throw_x`

        y, z, and phi are taken from the random throws (throwsFD) and corrected by detector offsets -> `throw_y`, `throw_z`, `throw_phi`

  - Determine the neutrino production point

       Use the vertex x-position to estimate the neutrino decay position along the beamline.

       Convert this decay position from beamline coordinates → detector coordinates.

  - Build direction vectors from production point

       Compute the vector from the decay point → original vertex.

       Compute vectors from the decay point → each randomly thrown vertex.

  - Compute rotation needed due to vertex translation

       Calculate the angle between the original and translated vectors.

       Compute the rotation axis using the cross product.

       Convert axis + angle into a rotation vector.

  - Compute additional random rotation

       Apply a random φ rotation around the neutrino direction for each throw.

  - Build rotation matrices

       One rotation accounts for the change in neutrino direction due to vertex translation.

       Another rotation accounts for the random φ rotation around that direction.

  - Rotate the muon momentum

       Apply the translation-induced rotation.

       Apply the φ rotation.

       The resulting momentum vector is used for the neural network evaluation -> `this_p`
 
### 2.3 Run NN to get the probability of muon passing the muon selection cuts

Give the input variables (muon momentum and throws coordinates) to the network: 
```bash
features=np.column_stack((this_p[:,0], this_p[:,1], this_p[:,2], throw_x, throw_y, throw_z))
```

Evaluate the network:
```bash
with torch.no_grad(): # Evaluate neural network #neural network output is 2D array of probability a set of events being contained-detected, tracker-detected,
                        netOut=net(features) # or not detected #I don't use the 3rd column (not-detected probability)
                        netOut=torch.nn.functional.softmax(netOut).detach().numpy
```
Access the result (this is on a throw by throws basis): 

  -- variables: `nnContained`, `nnTracker`
  
  -- define probability that muon is either contained ND-LAr, or futhrer matched in downstream tracker, Pmu: 
  ```bash
  weightPmuon.push_back([nnContProb*hadresult + nnTrackerProb*hadresult])
  ```
  


### 2.4 Save important variables in an output file

Variables saved and further used in the analysis file:
  - ```hadron_selected_eff```
  
  - ``` muon_tracker_eff```
  
  - ```muon_contained_eff```
  
  - ```muon_selected_eff```
  
  - ```combined_eff``` - this is the combined (hadron and muon selection cuts) efficiency

  - ```vtxX"```
  
  - ```weightPmuon```


As a result of running the muon efficiency part an output file, i.e `OutputMuonFDGeoEff.root` is created and contains the above mentioned variables.


## 3.  Analyse the FD events selected at the ND 

The code for this step can be found on the ```EtrimAnalysis``` branch of this repository: https://github.com/icaracas/DUNE_ND_GeoEff/ . 

The 2 main files of interest are: ```Grid/NtupleOutVetoAndTrimE_AssumeEqualEffAtAllOA_NOOscSpectrumNOCAFAna_FDRateAtND.C``` and ```Grid/NtupleOutVetoAndTrimE_AssumeEqualEffAtAllOA_WithOscSpectrumNOCAFAna_FDRateAtND.C```
The difference between those 2 files are whether one wants to work with the oscillated spectrum or with the non-oscillated one -- codes are very similar but will try to highlight the main differences below.

This part is using information obtained from the hadron geoemtric efficiency and muon geometric efficiency parts and puts it together towards the final desired distribution of selected FD events in the ND. 

## Big picture
As already mentioned, the end goal is a direct **comparison between the FD events slected at the ND (GeoEFFcorrected) and the linearly combined ND events**. The main variable of interest here is **ETrim**, which is the total amount of hadronic energy deposited by a FD event inside the active volume of the ND. Since the ND volume is smaller than the FD one, the amount of energy deposited inside the ND will be smaller. This is why we need to trim out any amount of hadronic energy that is deposited outside the active volume of the ND. This is already done in step 1 - Hadronic geometric efficiency section. Here we want to properly access this information and get our end results in terms of **selected events FD events when put in the ND as a function of the total amount of energy deposited in the ND and at a given off-axis position**. 

The main idea here is that every FD event is translated to the ND, the ND is placed at different positions (in the framework so far the ND is only put on-axis and then here in the analysis we assume same efficiency at all off-axis positions), and then for each ND position we put the event in different vtx_x position inside the detector. For each of this vtx_x position we randomly translate and rotate the event and we evaluate both it's hadron efficiency (does the event pass the hadron veto cut? -- done in step 1 Hadron Geometric efficiency ) as well as the muon efficiency (for events that pass the hadron cut, what is the probability the muon would pass the selection criteria -- either contained in ND-Lar or further match in the downstream TMS -- done in step 2 Muon Geometric efficiency). 

For each event that passes the hadron geometric efficiency selection (we will have many passing throws at a fixed detector position and fixes vtx_x) we have a corresponding **trim energy: `Etrim`**, a corresponding **probability that the muon passes the selection criteria: `weightPmuon`** and the corresponding **muon energy `Emu`**. (NOTE: muon energy `Emu` is the true muon energy of the event so therefore this will be the same for 1 FD event regardless where it is put in the ND. However it is still used on a throw by throw basis for consistency.) That means that taking 1 FD event, for each fixed detector position and fixed vtx_x value we have a **distribution (histogram) of Etrim values**. For this 1 FD event (fixed detector position, fixed vtx_x) the amount of entries in this Etrim histogram will be equal to the amount of passing throws (from hadron geo efficiency). Since each of the passing throws in the hadron geo efficiency has a corresponding muon probability (for muon selection) and since we want to have the distribution of events passing ALL selection criteria, each event in this histogram will further be weighted by this muon probability, `weightPmuon` . 

At this point we have the `vis Etrim` (vis Etrim = Etrim + Emu is what we call "the visible trimmed energy" and is nothing else than the sum between the hadron energy deposited inside ND LAr and the muon energy) distribution of this particular FD event when translated to the ND, at a fixed vtx_x position inside the ND and fixed detector postion with respect to the neutrino beam axis. We can get such **distributions for each vtx_x and detPos** of interest. So we can then **represent this FD event in a 2D distribution, -- 2D histogram -- as a function of off-axis position ```OAPos = vtx_x + detPos``` and visible trimmed energy**  (vis Etrim = Etrim + Emu). -- i.e remember that at different vtx_x positions inside the detector we had different efficiencies and also different amount of energy deposited inside the ND-LAr active volume: more likely to deposit more energy outside the detector (so lower Etrim values) when the event is close to the ND edges. 

In short, **each FD event can be mapped into the ND as a 2D distribution of off-axis position versus visible trimmed energy, capturing the geometric effects of both hadron and muon selection efficiencies**, allowing a **direct comparison between ND-observed and FD-derived events**. The last steps in the analysis is looping over all FD events (all ntuples) and summing them together in such a 2D histogram. This is the `SelectedEventsVsOAPosVsTotalETrim` histogram in the code, which is created per each event and is further summed for all events in the final (step 5) step of comparing the selected FD events with the ND linearly combined data.  

Below there is a more detailed explanation of the code itself and the additional cuts (i.e CAF-like cut for low energy muons + FD EventRate at ND ) that are applied to this `SelectedEventsVsOAPosVsTotalETrim` histogram.


### 3.1 Read information from the root files obtained from the hadron geo eff and muon geo eff part

The code is run using both the hadron geo eff and muon geo eff output files. i.e assuming you have ` OutputHadronGeoEff.root` from hadron and `OutputMuonFDGeoEff.root` from muon, you would run:

-- for no oscillations
```bash
./NtupleOutVetoAndTrimE_AssumeEqualEffAtAllOA_NOOscSpectrumNOCAFAna_FDRateAtND OutputHadronGeoEff.root OutputMuonFDGeoEff.root
```
-- for oscillations
```bash
./NtupleOutVetoAndTrimE_AssumeEqualEffAtAllOA_WithOscSpectrumNOCAFAna_FDRateAtND OutputHadronGeoEff.root OutputMuonFDGeoEff.root
```

We read in more variables (than we actually use) but I think it's always useful to have access to more information. 

 - variables needed as an input (from the hadron geo eff ):
   - `iwritten` -- event counter (goes from 0 to nr of FD events in the original FD ntuple used as input for hadron and muon eff)
   - `ND_LAr_dtctr_pos` -- detector off-axis position with respect to neutrino beam (so far this is only equal to 0 in the current analysis , but would be useful for the future when using different detector poistions)
   - `ND_LAr_vtx_pos` -- vtx_x choices inside ND-LAr
   - `ND_GeoEff` -- this is the hadron geometric efficiency -> 1 value / vtxX / dePos/ FDEvent
   - `validThrows` -- this is the total number of throws of a given FD event -> 1 value/ vtxX / detPos / FD Event
   - `NPassedThrows` -- number of throws that passed the hadron selection cut (for each vtxX position this number of passing throws is different) -> 1 value/ vtxX / detPos / FD Event
   - `TrimEnergyEventsPass` -- trimmed energy (i.e hadronic energy deposited inside ND-LAr active volume) of events that pass the hadron selection -- here we have one value for each passing throw -> 1 value/ vtxX /PassingThrowAtVtx /  detPos / FD Even

   - `totEnergyFDatND_f` -- total hadronig energy deposited inside FD Active volume (this is what we trim from) -> 1 value / FD Event
   - `muonEdep_f` -- deposited energy of muon in FD -> 1 value / FD Event
   - `muonTrackLength_f` -- muon track length -> 1 value / FD Event
   - `ND_Gen_numu_E` -- true neutrino energy at generator level -> 1 value / FD Event
   - `ND_E_vis_true` -- true visible neutrino energy -> 1 value / FD Event
   - `ND_OffAxis_Sim_mu_start_E_xyz_LAr` -> muon energy momentum vector: used to get muon energy -> 1 value / FD Event

- variables needed as input (from muon geo eff):
   - `combined_eff` -- combined (hadron and muon) efficiency -> 1 value/ vtxX / detPos / FD Event
   - `muon_selected_eff` -- muon efficiency -> 1 value/ vtxX / detPos / FD Event
   - `muon_tracker_eff` -- muon tracker efficiency (i.e only passing the tracker cuts) -> 1 value/ vtxX / detPos / FD Event
   - `weightPmuon` -- muon probability to be either contained in ND-LAr *OR* matched in tracker -> 1 value/ vtxX/ PassingThrowAtVtx / detPos / FD Event (PassingThrowAtVtx is taken from the hadron eff part)

### 3.2 Analysis specifics variables and functions

- variables we need and use / define within the code:
  - `a_ND_off_axis_pos_vec` -- these are the detector positions we need to use, same as those for the ND CAFs. Since currently we assume the same efficiency at all detector positions, we only run the hadron and muon geometric efficiencies for a detector situated on-axis and we then use different off-axis positions within this analysis part. The idea here is the detector is moved at each inidivudal position inside this vector, and then the efficiency vs vtxX is 'copied' as being the same at every one of these positions. The detector positions used are: `0, -1.75, -2, -4, -5.75, -8, -9.75, -12, -13.75, -16, -17.75, -20, -21.75, -24, -25.75, -26.25, -28, -28.25, -28.5` in meters.
NOTE: if in the future one chooses to not assume equal efficiency at all detector positions then this would be changed within the muon and hadron geometric efficiency part and this vector in the code would not be used, instead you shoud use all the detector possitions used in the geo eff and muon code, saved in the `ND_LAr_dtctr_pos` variable
   - `a_ND_vtx_vx_vec` -- vtx_x positions, this variable is recreating the same vtx_x positions used in the hadron and muon geo eff code.
   - `OAPos` -- this variable is calculating the off-axis position at each detector and vtx_x position: `OAPos = vtx_x+det_pos` -> this is the Off-axis position variable we also use in the PRISM analysis and represents the off-axis position of a given event
   - `CoefficientsAtOAPos` -- these are the off-axis coefficients; in the current version of the analysis we don't use them anymore, I am fairly sure one should use either the coefficients or the FD Event Rate at ND (more details about this below) but in any case they are still in the code and can be accessed at any time
   - `WeightEventsAtOaPos` -- this is a weight that accounts for different (non-flat) distribution of events at different detector positions: our detector covers 4m (from -2 m to 2m) width in x direction; since we move the detector at several locations, more often than every 4 m, we are covering some off-axis regions more often than others and have to correct for this. If you are interested in how this weight looks like check the `HistOAPos` histogram.
   - `EnuTrue` -- variable where we save the true neutrino energy / FD Event
   - `TotalLeptonMom` -- variable with muon total energy / FD Event

### FD Event Rate at ND Functions (Try to use Ehad - Elep to avoid model dependence):

As mentioned above, we have to account for the FD event rate at the ND. Since we take the FD events and put them in the ND, at different off-axis positions, we have to account for this rate. This is related to the fact that neutrinos with a given true energy have different probabilities of appearing at a different off-axis position in the ND. Take for example a very low energy neutrino: this would have a much higher chance to appear from somewhere far off-axis than from the on-axis (left figure below). Similarly an event with a high neutrino energy will have a much higher probability to appear from an on-axis ND then from very far off-axis (right side figure below) -- remember the further we go off-axis with the ND the sampled neutrino spectra peaks at lower and lower energies and gets narrower and narrower. Therefore each event in the FD will have a corresponding probability, `FDEventRateAtND(E, OAPos)` to appear at a given off-axis position depending on its energy. This probability is normalized to 1 over the entire off-axis range. If we don't account for this **FD Event Rate at the ND**, we wrongly give an equal probability for any FD Event, regardless of its energy, to appear in the ND, at any off-axis position, which is wrong!
<p align="center">
  <img src="https://github.com/user-attachments/assets/48e29025-2acc-4e70-9bc5-9b18ead9ac97" width="45%">
  <img src="https://github.com/user-attachments/assets/08e91540-a3f8-4f75-8c59-19c3f56d821b" width="44%">
</p>

Depending on the energy variable used -- in this example I mentioned true neutrino energy, but one can use different energies, for example hadron energy and muon energy of a given event,  hadron +muon energy etc. The true neutrino energy should in principle be the most correct variable. However by using true neutrino energy we add model-dependence. So it would be ideal to be able to use energy variable that we can directly see in the detector. 

There are 2 possibilities of using such a function in the current code:
  - true energy - dependent: `FDEventRateAtND_ETrue(ETrue, OAPos)`
    ```bash
    double FDEventRateAtND_ETrue(FunctionCache& cache, double Etrue, double OAPos)
    ```
    This function is reading the values of the FDEventRateAtND function for different true neutrino energy bins (or ranges) from an output file called `Splines_FDEventRateAtND_ETrue.root`. This root file can be found on the `EtrimAnalysis` branch of this reposiotory, where the file `EffFitFunctionCorrectEbinningSelectedEventsSplineETrue.C` to create this function is also located. The `Splines_FDEventRateAtND_ETrue.root` gives the value of this probability over the entire off-axis range of interest for fixed true neutrino energy bins. The bin edges corresponding to these bins, `kTrueEBinEdges` are also saved in the file (i.e `f_norm_etrue1` is the proability of FD event rate at the ND for neutrino energy ranging from bin edge0 to bin edge 1 etc)

    The function can further be used and called within the code as
    ```bash
    FDEventRateAtND_ETrue(cacheEtrue, EnuTrue[i_iwritten], OAPos)`
    ```
     where
    ```bash
    FunctionCache cacheEtrue;
    cacheEtrue.loadFunctions("Splines_FDEventRateAtND_ETrue.root", FunctionType::ETrue);
    ```
    and `EnuTrue[i_iwritten]` is the true neutrino energy of the event and `OAPos` is the corresponding off-axis position within the ND.
    
  - hadron energy and lepton energy `FDEventRateAtND(Ehad, Elep, OAPos)` : 1 given neutrino event has a specific hadron energy deposited inside the detector and a given muon energy; this can be further used in order to define this event rate function for a fixed hadron energy and muon energy
    ```bash
    double FDEventRateAtND(FunctionCache& cache, double Ehad, double Emu, double OAPos)
    ```
    Since we use the ND selected events to obtain this FD Event Rate at the ND probability, and since we are interesed in how a given FD event would be seen by the ND, the `Ehad` used here should be the hadronic energy deposited inside the ND-LAr active volume -> so `Ehad` here is in fact our trimmed energy `Etrim`.

    This function is reading the value of the FDEventRateAtND function for different - fixed `Ehad` and `Elep` bins over the entire off-axis range, from an ouput file called `Splines_FDEventRateAtND.root`. This root file can be found on the `EtrimAnalysis` branch of this reposiotory, where the file `EffFitFunctionCorrectEbinningSelectedEventsSplineEhadElep.C` to create this function is also located. The `Splines_FDEventRateAtND.root` gives the value of this probability over the entire off-axis range of interest for a fixed hadron energy and lepton energy bin. The bin edges corresponding to these bins, `kHadBinEdges` and `kLepBinEdges` are also saved in the file (i.e `f_norm_lep1_had1` is the proability of FD event rate at the ND for an event with hadronic energy ranging from bin HadBinEdge0 to bin HadBinEdge1 and a muon energy ranging from LepBinEdge0 to LepBinEdge1.. etc)

    The function can further be used and called within the code as
    ```bash
    FDEventRateAtND(cacheLepHad, info.Etrim *1E-3 , info.Emu*1E-3, OAPos));
    ```
    where
    ```bash
    FunctionCache cacheLepHad;
    cacheLepHad.loadFunctions("Splines_FDEventRateAtND.root", FunctionType::LepHad);
    ```
    and ```info.Etrim *1E-3``` is the trimmed energy of a given throw in GeV, `info.Emu*1E-3` is the muon energy in GeV, and `OAPos` is the corresponding off-axis position in the ND.
    
    **This is the function that should be used - if possible all the time in the analysis, rather than the true energy one**. By using Ehad and Elep , which are the deposited energies in the detector (at this point we use true muon energy in the entire analysis simply because we have no muon reconstruction but this would eventually change) we avoid any kind of neutrino interaction model dependency. The probability as a function of true energy should in principle the most correct one and can be used to cross check there is nothing wrong in the analysis regarding this particular step but should in principle be avoided.

- Histograms in the code (most important ones)

- Information (mainly histogram) needed in the output root file:
  
TO Be Continued 

Since we also have access to the total number of throws (```validThrows```) per event / per vtx_x we can easily scale the histogram to the number of ```validThrows``` in order to have the integral of this histogram equal to the corresponding efficiency of this particular FD event when translated to the ND, when the detector is positioned at a given detPos, at a given vtx_x value inside the ND.

The idea would be that once we put the FD event at different configurations in the ND and once we have


In order to ensure the correct **same conditions** for our FD events when put in the ND, several steps are used in this analysis section. 

(access Etrim histograms of each passing throw from hadron geo Eff, weight them by the probability the event passes the muon cuts, Pmu(Emu,throws)

