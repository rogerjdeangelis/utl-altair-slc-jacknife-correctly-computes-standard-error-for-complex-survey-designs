# utl-altair-slc-jacknife-correctly-computes-standard-error-for-complex-survey-designs
Altair slc jacknife correctly computes standard error for complex survey designs
    %let pgm=utl-altair-slc-jacknife-correctly-computes-standard-error-for-complex-survey-designs;

    %stop_submission;

    Altair slc jacknife correctly computes standard error for complex survey designs

    Too long to post on a list, see github
    https://github.com/rogerjdeangelis/utl-altair-slc-jacknife-correctly-computes-standard-error-for-complex-survey-designs

    Altair post
    https://community.altair.com/discussion/49468/jackknife-tests/p1?tab=all

    PROBLEM

      Provide better estimate of standard error(SE) then weighted standard error

      The jackknife standard error (7.28) is 76% larger than the naive
      weighted standard error (4.12) because:

      Within-cluster correlation reduces effective sample size
      Unequal weights increase variance
      Stratification was accounted for correctly
      Design effect = (Jackknife SE / WEIGHTED SE)² = (7.28/4.12)² = 3.12
      The correct Jacjnife SE is 2.12 times larger

      The jackknife correctly accounts for the complex survey design,
      providing valid inference for the population parameter.

      Jacknnife recognizes the true reduced indepenent degrees of freedom,
        because of clustering. This inflates the SE.

    There are many other uses for the jacknife.

    CONTENTS

      1 simple weighted SE wrong
      2 jacknife SE wider SE

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    proc datasets lib=workx kill;
    run;quit;

    libname workx sas7bdat "d:/wpswrkx";

    data workx.survey_data;
        input cluster stratum weight value;
    cards4;
    1 1 2.5 45
    1 1 2.5 52
    1 1 2.5 48
    2 1 3.0 67
    2 1 3.0 71
    3 2 1.8 33
    3 2 1.8 38
    4 2 2.2 56
    4 2 2.2 59
    ;;;;
    run;

    /**************************************************************************************************************************/
    /*  WORKX.SURVEY_DATA total obs=9                                                                                         */
    /*                                                                                                                        */
    /*   Obs    CLUSTER    STRATUM    WEIGHT    VALUE                                                                         */
    /*                                                                                                                        */
    /*    1        1          1         2.5       45                                                                          */
    /*    2        1          1         2.5       52                                                                          */
    /*    3        1          1         2.5       48                                                                          */
    /*    4        2          1         3.0       67                                                                          */
    /*    5        2          1         3.0       71                                                                          */
    /*    6        3          2         1.8       33                                                                          */
    /*    7        3          2         1.8       38                                                                          */
    /*    8        4          2         2.2       56                                                                          */
    /*    9        4          2         2.2       59                                                                          */
    /**************************************************************************************************************************/

    /*       _                 _                     _       _     _           _  ____  _____
    / |  ___(_)_ __ ___  _ __ | | ___  __      _____(_) __ _| |__ | |_ ___  __| |/ ___|| ____|__      ___ __ ___  _ __   __ _
    | | / __| | `_ ` _ \| `_ \| |/ _ \ \ \ /\ / / _ \ |/ _` | `_ \| __/ _ \/ _` |\___ \|  _|  \ \ /\ / / `__/ _ \| `_ \ / _` |
    | | \__ \ | | | | | | |_) | |  __/  \ V  V /  __/ | (_| | | | | ||  __/ (_| | ___) | |___  \ V  V /| | | (_) | | | | (_| |
    |_| |___/_|_| |_| |_| .__/|_|\___|   \_/\_/ \___|_|\__, |_| |_|\__\___|\__,_||____/|_____|  \_/\_/ |_|  \___/|_| |_|\__, |
                        |_|                            |___/                                                            |___/
    */

    proc means data=workx.survey_data mean std stderr clm;
        weight weight;
        var value;
        title "METHOD 1: Simple Analysis (WRONG for survey data)";
        title2 "Ignores stratification, clustering, and design effects";
    run;

    /**************************************************************************************************************************/
    /*  METHOD 1: Simple Analysis (WRONG for survey data)                                                                     */
    /* Ignores stratification, clustering, and design effects                                                                 */
    /*                                                                                                                        */
    /* The MEANS Procedure                                                                                                    */
    /*                                                                                                                        */
    /*                           Analysis Variable: VALUE                                                                     */
    /*                                                                                                                        */
    /*                                                     Lower 2-        Upper 2-                                           */
    /*                                                 sided CL for    sided CL for                                           */
    /*         Mean         Std Dev          stderr            Mean            Mean                                           */
    /*                                                                                                                        */
    /* 53.827906977    19.346392609     4.172350333    44.206449855    63.449364098                                           */
    /**************************************************************************************************************************/


    /* METHOD 2: PROPER SURVEY ANALYSIS with Jackknife */
    proc surveymeans data=workx.survey_data
                     varmethod=jackknife
                     mean clm;
        stratum stratum;
        cluster cluster;
        weight weight;
        var value;
        title "METHOD 2: Proper Survey Analysis with Jackknife";
        title2 "Accounts for complex design";
    run;

    /*___      _            _          _  __                           _     _             ____  _____
    |___ \    (_) __ _  ___| | ___ __ (_)/ _| ___   ___  ___ __      _(_) __| | ___ _ __  / ___|| ____|
      __) |   | |/ _` |/ __| |/ / `_ \| | |_ / _ \ / __|/ _ \\ \ /\ / / |/ _` |/ _ \ `__| \___ \|  _|
     / __/    | | (_| | (__|   <| | | | |  _|  __/ \__ \  __/ \ V  V /| | (_| |  __/ |     ___) | |___
    |_____|  _/ |\__,_|\___|_|\_\_| |_|_|_|  \___| |___/\___|  \_/\_/ |_|\__,_|\___|_|    |____/|_____|
            |__/
    */


     /**************************************************************************************************************************/
     /*   The SURVEYMEANS Procedure                                                                                            */
     /*                                                                                                                        */
     /*             Data Summary                                                                                               */
     /*                                                                                                                        */
     /*                                                                                                                        */
     /*  Number of Strata                 2                                                                                    */
     /*  Number of Clusters               4                                                                                    */
     /*  Number of Observations           9                                                                                    */
     /*  Sum of Weights            21.50000                                                                                    */
     /*                                                                                                                        */
     /*         Variance Estimation                                                                                            */
     /*                                                                                                                        */
     /*  Method                   Jackknife                                                                                    */
     /*  Number of Replicates             4                                                                                    */
     /*                                                                                                                        */
     /*                               Statistics                                                                               */
     /*                                                                                                                        */
     /*                                 Std Error                                                                              */
     /*  Variable            Mean         of Mean        95% CL for Mean                                                       */
     /*                                                                                                                        */
     /*  VALUE          53.827907        7.283199    22.4908297    85.1649843                                                  */
     /**************************************************************************************************************************/


    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
