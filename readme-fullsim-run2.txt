 setenv VO_CMS_SW_DIR /cvmfs/cms.cern.ch/
 setenv SCRAM_ARCH slc6_amd64_gcc481
 source $VO_CMS_SW_DIR/cmsset_default.csh

 cmsrel CMSSW_7_0_6_patch1



1. prepare the proper hadronization python script

    a) go to src directory
       
    b) copy  Hadronizer_Tune4C_13TeV_generic_LHE_pythia8_cff.py into 
             Configuration/GenProduction/python/ThirteenTeV/
       the attached Hadronizer_Tune4C_13TeV_generic_LHE_pythia8_cff.py file is modified 
       from the original one to include slha file.

    c) scram b

    d) put your lhe and slha files in src directory,

2.  GEN-Hadronization


   cmsDriver.py Configuration/GenProduction/python/ThirteenTeV/Hadronizer_Tune4C_13TeV_generic_LHE_pythia8_cff.py -s GEN \
        --eventcontent RAWSIM --datatier GEN-SIM \
        --conditions POSTLS170_V7::All \
        --pileup=NoPileUp \
        --datamix NODATAMIXER  \
        --filein=file:uwDM14events.lhe \
        --filetype=LHE \
        --fileout monojetGEN.root \
        --python_filename mnjGEN.py \
        -n 2 \
        --no_exec \
        --customise SLHCUpgradeSimulations/Configuration/postLS1Customs.customisePostLS1


   cmsRun mnjGEN.py


3.  SIMulation

   cmsDriver.py Simulation_step -s SIM \
        --eventcontent RAWSIM --datatier GEN-SIM \
        --conditions POSTLS170_V7::All \
        --pileup=NoPileUp \
        --datamix NODATAMIXER  \
        --filein=file:monojetGEN.root \
        --fileout monojetSIM.root \
        --python_filename mnjSIM.py \
        -n 2 \
        --no_exec \
        --customise SLHCUpgradeSimulations/Configuration/postLS1Customs.customisePostLS1

   cmsRun mnjSIM.py


4.  DIGItization 

   cmsDriver.py REDIGI -s DIGI,L1,DIGI2RAW,HLT:GRun \
        --eventcontent RAWSIM --datatier GEN-SIM-RAW \
        --conditions POSTLS170_V7::All \
        --pileup=NoPileUp \
        --datamix NODATAMIXER  \
        --filein=file:monojetSIM.root \
        --fileout monojetDIGI.root \
        --python_filename mnjDIGI.py \
        -n 2 \
        --no_exec \
        --customise SLHCUpgradeSimulations/Configuration/postLS1Customs.customisePostLS1

  cmsRun mnjDIGI.py


5. RECOnstruction

  cmsDriver.py STEP2 -s RAW2DIGI,L1Reco,RECO \
        --eventcontent AODSIM --datatier AODSIM \
        --conditions POSTLS170_V7::All \
        --pileup=NoPileUp \
        --datamix NODATAMIXER  \
        --filein=file:monojetDIGI.root \
        --fileout monojetRECO.root \
        --python_filename mnjRECO.py \
        -n 2 \
        --no_exec \
        --customise SLHCUpgradeSimulations/Configuration/postLS1Customs.customisePostLS1

  cmsRun mnjRECO.py


6. MINIAOD 

  cmsDriver.py miniAOD-prod -s PAT --eventcontent  MINIAODSIM  --runUnscheduled  --mc   \
            --filein  file:monojetRECO.root --conditions PLS170_V7AN1::All  --no_exec -n 2 \
            --fileout monojetMINIAOD.root --python_filename mnjMINIAOD.py

  cmsRun mnjMINIAOD.py


 




 