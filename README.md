# utl-proc-report-group-by-three-columns-with-three-levels-across-and-create-an-output-sas-dataset
proc report group by three columns with three levels across and create a output sas dataset 
    %let pgm=utl-proc-report-group-by-three-columns-with-three-levels-across-and-create-an-output-sas-dataset;

    %stop_submission;

    proc report group by three columns with three levels across and create a output sas dataset

    Note we get a sas crosstab sas dataset that looks like the text.
    Just wrap your proc tabulate with the utl)odsrpt macro.


    original algorithm by

    Bartosz Jablonski
    yabwon@gmail.com


    github
    https://tinyurl.com/ypaezbf6
    https://github.com/rogerjdeangelis/utl-proc-report-group-by-three-columns-with-three-levels-across-and-create-an-output-sas-dataset

    communities.sas.com
    https://tinyurl.com/yc57wm3y
    https://communities.sas.com/t5/SAS-Programming/Transpose-data-for-two-columns-based-on-two-sectors/m-p/846356#M334598
               4 related repos
    macros
    https://tinyurl.com/y9nfugth
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories


       CONTENTS

          1 tabulate output dataset that looks like the ascii text output
          2 original static ascii text report
          3 macro utl_odsrpt (provide datasets when the proc output does not conform to the listing)
            also
              utl_odsfrq  (proc freq crosstab dataset that looks like ascii static report)
              utl_odstab  (proc tabulate crosstab dataset that looks like ascii static report)
              proc corresp (can create do a wide range of crosstab tables)
              ods excel and read the excel sheet

              Run your proc report without the macro first
              Surround your proc report with utl_odsrpt
              Create pipe delimited list of output column names
          4   related repos


    /*               _     _
     _ __  _ __ ___ | |__ | | ___ _ __ ___
    | `_ \| `__/ _ \| `_ \| |/ _ \ `_ ` _ \
    | |_) | | | (_) | |_) | |  __/ | | | | |
    | .__/|_|  \___/|_.__/|_|\___|_| |_| |_|
    |_|
    */

    /*---- CREATE A DATASET THAT LLOKS LIKE THE TEXT OUTPUT FROM PROC REPORT ----*/

    /*********************************************************************************************************************************/
    /*            INPUT                     |         PROCESS                     |        OUTPUT WORK.WANT_ODSRPT                   */
    /*            =====                     |  RUN REPORT WITHOUT MACRO FIRST     |        =======================                   */
    /*                                      |  ==============================     |                                                  */
    /*                                      |                                     |                                                  */
    /* PID AGE SEX RACE EVENT2 SECTOR SCORE | 1 TABULATE OUTPUT DATASET           | SAS DATASET NOT STAIC TEXT REPORT                */
    /*                                      | =========================           |                                                  */
    /* 101  21  1    1  fever   EMEA    25  |                                     |                   SEC    SEC   SEC   SEC         */
    /* 102  21  2    2  pain    FDA     45  | proc datasets lib=sd1 nolist        |                   EMEA   EMEA  FDA   FDA         */
    /* 103  31  1    3  headac  FDA     54  |  nodetails;                         | PID AGE SEX RACE  EV2    SCOR  EV2   SCOR        */
    /* 104  43  2    4  joint   EMEA    66  |  delete want_odsrpt;                |                                                  */
    /* 105  43  2    5  cold    EMEA   100  | run;quit;                           | 101 21   1   1   fever    25           .         */
    /* 106  75  2    6  anemia  EMEA   232  |                                     | 102 21   2   2             .  pain    45         */
    /*                                      | options validvarname=v7;            | 103 31   1   3             .  headac  54         */
    /* options validvarname=upcase;         | %utl_odsrpt(setup);                 | 104 43   2   4   joint    66           .         */
    /* libname sd1 "d:/sd1";                | options ps=5000 ls=255 missing="."; | 105 43   2   5   cold    100           .         */
    /* data sd1.have;                       | proc report data=sd1.have           | 106 75   2   6   anemia  232           .         */
    /*  input pid age sex                   |   nowd                              |                                                  */
    /*  race event2 $ sector $ score;       |   out=want_rpt                      | INTERMEDIATE FILE FOR PROC IMPORT                */
    /* cards4;                              |   formchar="|"                      | =================================                */
    /* 101 21 1 1 fever EMEA 25             |   noheader                          |                                                  */
    /* 102 21 2 2 pain FDA 45               |   box;                              | |PID|AGE|sex|RACE|SEC_EMEA_EV2|SEC_EMEA_SCOR|.   */
    /* 103 31 1 3 headac FDA 54             |  format _numeric_ 14.;              | |101|21|1|1|fever|25||.|                         */
    /* 104 43 2 4 joint EMEA 66             | title %sysfunc(compress("|PID|AGE   | |102|21|2|2||.|pain|45|                          */
    /* 105 43 2 5 cold EMEA 100             |    |sex|RACE|SEC_EMEA_EV2           | |103|31|1|3||.|headac|54|                        */
    /* 106 75 2 6 anemia EMEA 232           |    |SEC_EMEA_SCOR|SEC_FDA__EV2      | |104|43|2|4|joint|66||.|                         */
    /* ;;;;                                 |    |SEC_FDA__SCOR|"));              | |105|43|2|5|cold|100||.|                         */
    /* run;quit;                            | column pid age gender race          | |106|75|2|6|anemia|232||.|                       */
    /*                                      |   sector,(event2 score);            |                                                  */
    /*                                      | define pid / group;                 |                                                  */
    /*                                      | define age/group;                   |                                                  */
    /*                                      | define gender/group;                |                                                  */
    /*                                      | define race/group;                  |                                                  */
    /*                                      | define sector/across;               |                                                  */
    /*                                      | run;quit;                           |                                                  */
    /*                                      | options ps=65;                      |                                                  */
    /*                                      | %utl_odsrpt(outdsn=want_odsrpt);    |                                                  */
    /*                                      |                                     |                                                  */
    /*                                      | proc print data=want_odsrpt         |                                                  */
    /*                                      |    split='_' width=minimum;         |                                                  */
    /*                                      | run;quit;                           |                                                  */
    /*                                      |                                     |                                                  */
    /*                                      | options formchar=                   |                                                  */
    /*                                      |  ='|----|+|---+=|-/\<>*' ;          |                                                  */
    /*                                      |                                     |                                                  */
    /*                                      |----------------------------------------------------------------------------------------*/
    /*                                      |                                     |                                                  */
    /*                                      | 2 STATIC REPORT                     |  LESS USEFULL STATIC ASCII TEXT OUTPUT           */
    /*                                      | ===============                     | ---------------------------------------------    */
    /*                                      |                                     | |                              SECTOR       |    */
    /*                                      | proc report                         | |                       EMEA          FDA   |    */
    /*                                      |   data=sd1.have box;                | | PID AGE SEX RACE EVENT2 SCORE EVENT2 SCORE|    */
    /*                                      | column pid age gender race          | |-------------------------------------------|    */
    /*                                      | sector,(event2 score);              | | 101| 21|  1|   1|fever |   25|      |    .|    */
    /*                                      | define pid / group;                 | |----+---+---+----+------+-----+------+-----|    */
    /*                                      | define age/group;                   | | 102| 21|  2|   2|      |    .|pain  |   45|    */
    /*                                      | define gender/group;                | |----+---+---+----+------+-----+------+-----|    */
    /*                                      | define race/group;                  | | 103| 31|  1|   3|      |    .|headac|   54|    */
    /*                                      | define sector/across;               | |----+---+---+----+------+-----+------+-----|    */
    /*                                      | run;                                | | 104| 43|  2|   4|joint |   66|      |    .|    */
    /*                                      |                                     | |----+---+---+----+------+-----+------+-----|    */
    /*                                      |                                     | | 105| 43|  2|   5|cold  |  100|      |    .|    */
    /*                                      |                                     | |----+---+---+----+------+-----+------+-----|    */
    /*                                      |                                     | | 106| 75|  2|   6|anemia|  232|      |    .|    */
    /*                                      |                                     | --------------------------------------------     */
    /*********************************************************************************************************************************/

    /*____                                        _   _            _               _
    |___ /  _ __ ___   __ _  ___ _ __ ___   _   _| |_| |  ___   __| |___ _ ___ __ | |_
      |_ \ | `_ ` _ \ / _` |/ __| `__/ _ \ | | | | __| | / _ \ / _` / __| `__| `_ \| __|
     ___) || | | | | | (_| | (__| | | (_) || |_| | |_| || (_) | (_| \__ \ |  | |_) | |_
    |____/ |_| |_| |_|\__,_|\___|_|  \___/  \__,_|\__|_|_\___/ \__,_|___/_|  | .__/ \__|
                                               |___|                  |_|
    */

    filename ft15f001 "c:/oto/utl_odsrpt.sas";
    parmcards4;
    %macro utl_odsrpt(outdsn);
       %local _ps_ _fc_ _tmp2_ _tmp1_;
       %if %qupcase(&outdsn)=SETUP %then %do;
            %put @@@@ &=sysindex.;
            %let _tmp1_=a&sysindex.;
            %put xxxx &=_tmp1_;
            filename &_tmp1_ clear;  * just in case;
            %utlfkil(%sysfunc(pathname(work))/&_tmp1_..txt);
            filename &_tmp1_ "%sysfunc(pathname(work))/&_tmp1_..txt";
            %let _ps_= %sysfunc(getoption(ps));
            %let _fc_= %sysfunc(getoption(formchar));
            OPTIONS ls=max ps=32756  FORMCHAR='|'  nodate nocenter;
            title; footnote;
            proc printto print=&_tmp1_;
            run;quit;
       %end;
       %else %do;
            /* %let outdsn=tst;  */
            %put @@@ &=sysindex.;
            %let _tmp2_=b&sysindex.;
            %let _tmp1_=a%eval(&sysindex - 2);
            %put xxxx  &=_tmp1_;
            %put xxxx  &=_tmp2_;
            proc printto;
            run;quit;
            filename &_tmp2_ clear;
            %utlfkil(%sysfunc(pathname(work))/&_tmp2_.txt);
            filename &_tmp2_ "%sysfunc(pathname(work))/&_tmp2_.txt";
            data _null_;
              infile &_tmp1_ length=l;
              input lyn $varying32756. l;
              if countc(lyn,'|')>1;
              lyn=compress(lyn);
              putlog lyn;
              file &_tmp2_;
              put lyn;
            run;quit;
            proc import
               datafile=&_tmp2_
               dbms=dlm
               out=&outdsn(drop=VAR:)
               replace;
               delimiter='|';
               getnames=yes;
            run;quit;
            filename &_tmp1_ clear;
            filename &_tmp2_ clear;
            %utlfkil(%sysfunc(pathname(work))/&_tmp1_.txt);
            %utlfkil(%sysfunc(pathname(work))/&_tmp2_.txt);
            options FORMCHAR="&_fc_" ;
       %end;
    %mend utl_odsrpt;
    ;;;;
    run;quit;

    /*  _              _       _           _
    | || |    _ __ ___| | __ _| |_ ___  __| |  _ __ ___ _ __   ___  ___
    | || |_  | `__/ _ \ |/ _` | __/ _ \/ _` | | `__/ _ \ `_ \ / _ \/ __|
    |__   _| | | |  __/ | (_| | ||  __/ (_| | | | |  __/ |_) | (_) \__ \
       |_|   |_|  \___|_|\__,_|\__\___|\__,_| |_|  \___| .__/ \___/|___/
                                                       |_|
    */

    https://github.com/rogerjdeangelis/utl-are-procedures-that-create-tables-better-than-proc-tabulate-output
    https://github.com/rogerjdeangelis/utl-complex-proc-tabulate-in-proc-report-with-output-table-that-is-an-image-of-tabulate
    https://github.com/rogerjdeangelis/utl-conformal-ods-table-output-for-proc-tabulate-and-proc-freq
    https://github.com/rogerjdeangelis/utl-create-a-crosstab-dataset-instead-of-a-listing-from-proc-tabulate
    https://github.com/rogerjdeangelis/utl-create-useful-column-names-from-tabulate-output
    https://github.com/rogerjdeangelis/utl-creating-a-clinical-n-mean-stddev-median-min-max-sas-dataset-from-proc-tabulate
    https://github.com/rogerjdeangelis/utl-creating-a-new-variable-as-a-function-of-other-variables-using-just-proc-tabulate
    https://github.com/rogerjdeangelis/utl-creating-a-table-that-looks-like-the-proc-tabulate-listing-
    https://github.com/rogerjdeangelis/utl-creating-an-output-dataset-from-proc-tabulate-that-mimics-the-printed-data
    https://github.com/rogerjdeangelis/utl-crosstab-output-tables-from-corresp-report-not-static-tabulate
    https://github.com/rogerjdeangelis/utl-fixing-bugs-in-proc-report-tabulate-and-freq-output-crosstab-datasets
    https://github.com/rogerjdeangelis/utl-formatting-aggregated-across-cells-as-character-in-proc-report-not-possible-in-tabulate
    https://github.com/rogerjdeangelis/utl-is-proc-report-more-flexible-than-proc-summary-or-tabulate-when-an-output-dataset-is-needed
    https://github.com/rogerjdeangelis/utl-make-proc-tabulate-dataset-output-more-usable
    https://github.com/rogerjdeangelis/utl-proc-tabulate-listing-converted-to-an-output-sas-dataset
    https://github.com/rogerjdeangelis/utl-reshape-table-use-utl_odsrpt-macro-and-proc-tabulate
    https://github.com/rogerjdeangelis/utl-save-crosstab-proc-tabulate-feq-report-printed-output-in-sas-tables
    https://github.com/rogerjdeangelis/utl-simple-proc-report-and-tabulate-n-and-percent-crosstab-ourput-datasets-
    https://github.com/rogerjdeangelis/utl-three-dimensional-crosstab-proc-freq-tabulate-corresp-and-report
    https://github.com/rogerjdeangelis/utl-transposing-sorting-and-summarizing-with-a-single-proc-corresp-freq-tabulate-and-report
    https://github.com/rogerjdeangelis/utl-use-freq-and-corresp-table-output-to-avoid-tabulate-static-printouts
    https://github.com/rogerjdeangelis/utl-use-report-instead-of-tabulate-and-get-the-bonus-of-an-output-table
    https://github.com/rogerjdeangelis/utl-using-excel-to-get-a-usefull-proc-tabulate-output-table
    https://github.com/rogerjdeangelis/utl_more_flexible_and_maintainable_crosstab_than_proc_tabulate
    https://github.com/rogerjdeangelis/utl_ods-output-datasets-for-a-very-complex-proc-tabulate

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
