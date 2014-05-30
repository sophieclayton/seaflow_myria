SeaFlow Calibration
==================

Question: Can we look at all SeaFlow data as a single dataset, rather than Cruise by Cruise.

## Calibration overview

Several steps are required in this meta-analysis

0. Check the log transformation. See <https://github.com/fribalet/flowPhyto/blob/master/R/Load.R#L46>

    Version 1 that should work, but didn't. (The bug has since been fixed; see <https://github.com/uwescience/datalogcompiler#150>)

        DEF transform(x): pow(10, x/pow(2,16)*3.5);
        AllData = SCAN(armbrustlab:seaflow:all_data);
        AllDataLinear = SELECT Cruise, Day, File_Id
                             , transform(fsc_small) as fsc_small
                             -- fsc_perp is measured differently, defer for later
                             , transform(chl_small) as chl_small
                             , transform(pe) as pe
                        FROM AllData;
        STORE(AllDataLinear, armbrustlab:seaflow:all_data_linear);

    Version 2 that did work:
        
        AllData = SCAN(armbrustlab:seaflow:all_data);
        AllDataLinear = SELECT Cruise, Day, File_Id
                             , pow(10, fsc_small/pow(2,16)*3.5) as fsc_small
                             -- fsc_perp is measured differently, defer for later
                             , pow(10, chl_small/pow(2,16)*3.5) as chl_small
                             , pow(10, pe/pow(2,16)*3.5) as pe
                        FROM AllData;
        STORE(AllDataLinear, armbrustlab:seaflow:all_data_linear);
    
    Runtime: 150s.
    
1. Check dynamic range (and possibly correct?)
   a. We can only use `fsc_small` and/or `chl_small` because neither `pe` nor `fsc_perp` will saturate.
   b. This query gets the bounds for each cruise:
   
          AllDataLinear = SCAN(armbrustlab:seaflow:all_data_linear);
          AllDataBounds = SELECT Cruise, Day, File_Id, MAX(fsc_small) as max_fsc_small, MAX(chl_small) as max_chl_small
                          FROM AllDataLinear;
          STORE(AllDataBounds, armbrustlab:seaflow:all_data_linear_bounds);

      Runtime: 95s.
    
2. "PMT" (an acronym related to the gain) - e.g., up the gain so we can see _pro_ which is small