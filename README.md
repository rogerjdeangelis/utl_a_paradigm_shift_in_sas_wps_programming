# utl_a_paradigm_shift_in_sas_wps_programming
Loop over variables with name ending character.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.


    Loop over variables with name ending character.

       Two Solutions

            1. %do_over
            2. Without do_over

    I think we may be entering a new paradigm on SAS-L with SAS/WPS programming
    but not applicable to SAS Forum.

    Separating meta data from mainline programming is the shift?

    I have found myself thinking more and more about the relationship of classic editor, common storage, address spaces,
    compile directives, early execution, meta data, macro language, python, R, dosubl, hashes and of course
    all of base SAS/WPS.

    I think the SAS/WPS language are uniquely positioned to substantually improve the programming landscape.

    I few pieces of meta data like the dimensions of an output array and source code
    generation ala '%do_over' greatly simplify the OPs question?

    However I hope I don't reget putting to much faith on this particulr example.

    My solution requires 'V1*' variables be before V2* variables, which makes sense to me.
    There is no hardcoding. and it allows arbirary sets of 'V' variables

    inspired by
    https://communities.sas.com/t5/Base-SAS-Programming/Loop-over-variables-with-name-ending-character/m-p/465300

    I do realize you can simplify this particular example with two lines of code.
    But non of these handle the general case, you need a piece of compile time data.
    This is a over simplified example of compile time data extraction.

    data x;
      v1a=1;
      v1b=2;
      out1=sum(of v1:);
      out2=sum(of v2:);
    run;quit;

    I also realize you can use a macro wrapper, I prefer open code.
    In addtion you can load the entire PDV into an array and parse it.


    INPUT
    =====
                                                                       RULES
                                                                       =====
     WORK.HAVE total obs=4                            | OUT1 = V1A + V1B    OUT2 = V2A + V2B
                                                      |
        ID    V1A    V1B    X1    V2A    V2B    X2    |     OUT1                OUT2
                                                      |
         1     0      0      9     0      0      9    |       0                   0
         2     1      1      9     2      2      9    |       2  1+1              4  2+2
         3     3      3      9     4      4      9    |       6                   8
         4     5      5      9     6      6      9    |      10  5+5             12  6+6



     Keep in mind this structure grows by an oder of magnitude.
     If I add 'C' variables we have 9 'V' variables. ( 5 and 25 variables)

      ID    V1A    V1B  V1C   X1   V2A  V2B  V2C  X2    V3A  V3B  V3C  X2

       1     0      0    0    9     0    0    0    9     0    0    0    9
       2     1      1    1    9     2    2    1    9     2    2    1    9
       3     3      3    3    9     4    4    3    9     4    4    3    9
       4     5      5    5    9     6    6    5    9     6    6    5    9


    PROCESS
    =======

    1. DO_OVER
    ----------
       %symdel dim / nowarn;

       data want ;

         if _n_=0 then do;
           %let rc = %sysfunc(dosubl('
               data _null_;
                  set have(obs=1);
                  array vs[*] v:;
                  call symputx("dim",sqrt(dim(vs)));
               run;quit;
               %put **** &=dim ****;
            '));
         end;

         * three lines of code;
         set have;

         %array(Vs,values=1-&dim.);
         %do_over(Vs,phrase=%str(out? =sum(of v?%str(:));));

       run;quit;

    2. Without do_over
    ------------------
       %symdel dim / nowarn;
       data want ;

         if _n_=0 then do;
           %let rc = %sysfunc(dosubl('
               data _null_;
                  set have(obs=1);
                  array vs[*] v:;
                  call symputx("dim",sqrt(dim(vs)));
               run;quit;
               %put **** &=dim ****;
            '));
         end;

         * &dim is the number in each set v1a-v1b = 2  v1a-v1c = 3 ;
         * note the repeated use of dim;

         set have;

         array vbin[&dim,&dim.] v:;    * if v1a-v1b then 4 Vs. if v1a-v1c then 9 Vs;
         array outVars[&dim.] out1-out&dim.;  *  number of sets of size dim;;

         do i=1 to &dim.;
           do j=1 to &dim;
             outVars[i]=sum(outVars[i] , vbin[i,j]);
           end;
         end;

         drop i j;

       run;quit;


    OUTPUT
    ======


     WORK.WANT total obs=4

      ID    V1A    V1B    X1    V2A    V2B    X2    OUT1    OUT2

       1     0      0      9     0      0      9      0       0
       2     1      1      9     2      2      9      2       4    1+1 and 2+2
       3     3      3      9     4      4      9      6       8
       4     5      5      9     6      6      9     10      12

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|
    ;

    data have ;
     input id v1a v1b x1 v2a v2b x2 ;
     cards4 ;
    1 0 0 9 0 0 9
    2 1 1 9 2 2 9
    3 3 3 9 4 4 9
    4 5 5 9 6 6 9
    ;;;;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|
    ;

    see process;

