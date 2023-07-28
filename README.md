# utl-embedding-dosubl-in-a-macro-and-returning-an-updated-environment-variable-contents
Embedding dosubl in a macro and returning an updated environment variable contents    
    %let pgm=utl-embedding-dosubl-in-a-macro-and-returning-an-updated-environment-variable-contents;

    Embedding dosubl in a macro and returning an updated environment variable contents
    I don't think this was previouls possible?

    github
    https://tinyurl.com/37wfav9p
    https://github.com/rogerjdeangelis/utl-embedding-dosubl-in-a-macro-and-returning-an-updated-environment-variable-contents

    Basicall you are using an PRSET system enviroment space as defined by a system
    enviroment variable as a input/output scratch area for dosubl..

    If this works, it provides a way to run dosubl inside a macro and return an updated
    environment variable.

    Before I ran the macro I created a permanent system environment variable SCRATCH
    (type env in search box to set it)

     set SCRATCH with the value PARENT  (There are limitations to how long a system variable can be)

    I then rebooted because I was not sure environment variables would be reset using 'path=C' would work.
    Set PATH=C may work but I am  sure you will also have to leave interactive SAS.

    Problem :
        Create a macro to list of numeric or character variable.
        This macro has to execute at macro time.

    Related

    https://github.com/rogerjdeangelis/utl_sharing_a_block_of_memory_with_dosubl
    https://github.com/rogerjdeangelis/utl-twelve-interfaces-for-dosubl
    https://github.com/rogerjdeangelis/utl_dosubl_subroutine_interfaces
    https://github.com/rogerjdeangelis/utl_passing-in-memory-sas-objects-to-and-from-dosubl

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    sashelp.class

    /**************************************************************************************************************************/
    /*                                       |                                                                                */
    /* SASHELP.CLASS                         |  RULES %wps_varlist(sashelp.class,type=num);                                   */
    /*                                       |                                                                                */
    /*                                       |                                                                                */
    /*   Variables in Creation Order         |   1. proc contents and output _tem_ file with viable name and type             */
    /*                                       |   2. use sql to create a single macro variable with the list of variable names */
    /*  #    Variable    Type    Len         |   3. copy the list to the aformentioned SCRATCH system environment variable    */
    /*                                       |   4. after macro execution                                                     */
    /*  1    NAME        Char      8         |      %let lst =  %sysfunc(sysget(SCRATCH));                                    */
    /*  2    SEX         Char      1         |      %put &=lst;                                                               */
    /*  3    AGE         Num       8         |      LST=AGE HEIGHT WEIGHT                                                     */
    /*  4    HEIGHT      Num       8         |                                                                                */
    /*  5    WEIGHT      Num       8         | Note you could return the address of a block of storage and length in          */
    /*                                       | a system environment variable, see my common and equivalence macros            */
    /*                                       |                                                                                */
    /**************************************************************************************************************************/

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */


    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* How to get the result from the macro                                                                                   */
    /*                                                                                                                        */
    /* %let lst =  %sysfunc(sysget(SCRATCH));                                                                                 */
    /* %put &=lst;                                                                                                            */
    /* LST=AGE HEIGHT WEIGHT                                                                                                  */
    /*                                                                                                                        */
    /**************************************************************************************************************************/


    %symdel _vrs / nowarn;

    options set=SCRATCH "MAIN";

    data _null_;
      length env $200;
      env=sysget('SCRATCH');
      put env=;
    run;quit;

    /*----  ENV=MAIN                                                         ----*/

    %macro wps_varlist(data,type=)/des="Create a list of numeric or character variables";

      %symdel _vrs /nowarn;

      %let rc=%sysfunc(dosubl('

       ods output variables=_temp_;
       proc contents data=&data;
       run;quit;

       proc sql;
          select variable into :_vrs separated by " " from _temp_ where upcase(type)="%upcase(&type)"
       ;quit;

        %put xxxx &=vrs;
        options set=SCRATCH "&_vrs";

      run;quit;
       '));

    %mend wps_varlist;

    %wps_varlist(sashelp.class,type=num);

    %let lst =  %sysfunc(sysget(SCRATCH));
    %put &=lst;

    /*---- or  (see macros below)                                            ----*/


    %let lst =  %sysget(SCRATCH));
    %put &=lst;


    /*                                                             _      ___         _                 _     _
     _ __ ___   __ _  ___ _ __ ___  ___   ___ _   _ ___  __ _  ___| |_   ( _ )     __| | ___  ___ _   _| |__ | |
    | `_ ` _ \ / _` |/ __| `__/ _ \/ __| / __| | | / __|/ _` |/ _ \ __|  / _ \/\  / _` |/ _ \/ __| | | | `_ \| |
    | | | | | | (_| | (__| | | (_) \__ \ \__ \ |_| \__ \ (_| |  __/ |_  | (_>  < | (_| | (_) \__ \ |_| | |_) | |
    |_| |_| |_|\__,_|\___|_|  \___/|___/ |___/\__, |___/\__, |\___|\__|  \___/\/  \__,_|\___/|___/\__,_|_.__/|_|
                                              |___/     |___/
    */

    %macro dosubl(arg)/des="simplify macro calls to dosubl";
      %let rc=%qsysfunc(dosubl(&arg));
    %mend dosubl;

    %macro dosubl(arg)/des="simplify macro calls to sysget";
      %let rc=%qsysfunc(sysget(&arg));
    %mend dosubl;

    /*   _                 _     _
      __| | ___  ___ _   _| |__ | |  _ __ ___ _ __   ___  ___
     / _` |/ _ \/ __| | | | `_ \| | | `__/ _ \ `_ \ / _ \/ __|
    | (_| | (_) \__ \ |_| | |_) | | | | |  __/ |_) | (_) \__ \
     \__,_|\___/|___/\__,_|_.__/|_| |_|  \___| .__/ \___/|___/
                                             |_|
    */
    https://github.com/rogerjdeangelis/Dynamic_variable_in_a_DOSUBL_execute_macro_in_SAS
    https://github.com/rogerjdeangelis/utl-DOSUBL-running-sql-inside-a-datastep-to-check-if-variables-exist-in-another-table
    https://github.com/rogerjdeangelis/utl-No-need-to-convert-your-datastep-code-to-macro-code-use-DOSUBL
    https://github.com/rogerjdeangelis/utl-a-better-call-execute-using-dosubl
    https://github.com/rogerjdeangelis/utl-academic-pipes-dosubl-open-defer-and-dropping-dowm-to-multiple-languages-in-one-datastep
    https://github.com/rogerjdeangelis/utl-adding-female-students-to-an-all-male-math-class-using-sql-insert_and_dosubl
    https://github.com/rogerjdeangelis/utl-append-and-split-tables-into-two-tables-one-with-common-variables-and-one-without-dosubl-hash
    https://github.com/rogerjdeangelis/utl-applying-meta-data-and-dosubl-to-create-mutiple-subset-tables
    https://github.com/rogerjdeangelis/utl-cleaner-macro-code-using-dosubl
    https://github.com/rogerjdeangelis/utl-dosubl-a-more-powerfull-macro-sysfunc-command
    https://github.com/rogerjdeangelis/utl-dosubl-and-do-over-as-alternatives-to-explicit-macros
    https://github.com/rogerjdeangelis/utl-dosubl-more-precise-eight-byte-float-computations-at-macro-excecution-time
    https://github.com/rogerjdeangelis/utl-dosubl-persistent-hash-across-datasteps-and-procedures
    https://github.com/rogerjdeangelis/utl-dosubl-remove-text-within-parentheses-of-macro-variable-using-regex
    https://github.com/rogerjdeangelis/utl-dosubl-using-meta-data-with-column-names-and-labels-to-create-mutiple-proc-reports
    https://github.com/rogerjdeangelis/utl-drop-down-using-dosubl-from-sas-datastep-to-wps-r-perl-powershell-python-msr-vb
    https://github.com/rogerjdeangelis/utl-embed-sql-code-inside-proc-report-using-dosubl
    https://github.com/rogerjdeangelis/utl-error-checking-sql-and-executing-a-datastep-inside-sql-dosubl
    https://github.com/rogerjdeangelis/utl-extracting-sas-meta-data-using-sas-macro-fcmp-and-dosubl
    https://github.com/rogerjdeangelis/utl-in-memory-hash-output-shared-with-dosubl-hash-subprocess
    https://github.com/rogerjdeangelis/utl-let-dosubl-and-the-sas-interpreter-work-for-you
    https://github.com/rogerjdeangelis/utl-load-configuation-variable-assignments-into-an-sas-array-macro-and-dosubl
    https://github.com/rogerjdeangelis/utl-loop-through-one-table-and-find-data-in-next-table--hash-dosubl-arts-transpose
    https://github.com/rogerjdeangelis/utl-macro-klingon-solution-or-simple-dosubl-you-decide
    https://github.com/rogerjdeangelis/utl-macro-with-dosubl-to-compute-last-day-of-month
    https://github.com/rogerjdeangelis/utl-maitainable-macro-function-code-using-dosubl
    https://github.com/rogerjdeangelis/utl-passing-a-datastep-array-to-dosubl-squaring-the-elements-passing-array-back-to-parent
    https://github.com/rogerjdeangelis/utl-potentially-useful-dosubl-interface
    https://github.com/rogerjdeangelis/utl-re-ordering-variables-into-alphabetic-order-in-the-pdv-macros-dosubl
    https://github.com/rogerjdeangelis/utl-rename-variables-with-the-same-prefix-dosubl-varlist
    https://github.com/rogerjdeangelis/utl-sas-array-macro-fcmp-or-dosubl-take-your-choice
    https://github.com/rogerjdeangelis/utl-select-high-payment-periods-and-generating-code-with-do_over-and-dosubl
    https://github.com/rogerjdeangelis/utl-some-interesting-applications-of-dosubl
    https://github.com/rogerjdeangelis/utl-transpose-multiple-rows-into-one-row-do_over-dosubl-and-varlist-macros
    https://github.com/rogerjdeangelis/utl-twelve-interfaces-for-dosubl
    https://github.com/rogerjdeangelis/utl-use-dosubl-to-save-your-format-code-inside-proc-report
    https://github.com/rogerjdeangelis/utl-using-dosubl-and-a-dynamic-arrays-to-add-new-variables
    https://github.com/rogerjdeangelis/utl-using-dosubl-to-avoid-klingon-obsucated-macro-coding
    https://github.com/rogerjdeangelis/utl-using-dosubl-to-avoid-macros-and-add-an-error-checking-log
    https://github.com/rogerjdeangelis/utl-using-dosubl-with-data-driven-business-rules-to-split-a-table
    https://github.com/rogerjdeangelis/utl-using-dynamic-tables-to-interface-with-dosubl
    https://github.com/rogerjdeangelis/utl_avoiding_macros_and_call_execute_by_using_dosubl_with_log
    https://github.com/rogerjdeangelis/utl_dosubl_do_regressions_when_data_is_between_dates
    https://github.com/rogerjdeangelis/utl_dosubl_macros_to_select_max_value_of_a_column_at_datastep_execution_time
    https://github.com/rogerjdeangelis/utl_dosubl_subroutine_interfaces
    https://github.com/rogerjdeangelis/utl_dynamic_subroutines_dosubl_with_error_checking
    https://github.com/rogerjdeangelis/utl_overcoming_serious_deficiencies_in_call_execute_with_dosubl
    https://github.com/rogerjdeangelis/utl_pass_character_and_numeric_arrays_to_dosubl
    https://github.com/rogerjdeangelis/utl_passing-in-memory-sas-objects-to-and-from-dosubl
    https://github.com/rogerjdeangelis/utl_read_all_datasets_in_a_library_and_conditionally_split_them_with_error_checking_dosubl
    https://github.com/rogerjdeangelis/utl_sharing_a_block_of_memory_with_dosubl
    https://github.com/rogerjdeangelis/utl_using_dosubl_instead_of_a_macro_to_avoid_numeric_truncation
    https://github.com/rogerjdeangelis/utl_using_dosubl_to_avoid_klingon_macro_quoting_functions
    https://github.com/rogerjdeangelis/utl_why_proc_import_export_needs_to_be_deprecated_and_dosubl_acknowledged


    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
