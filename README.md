# utl_passing-in-memory-sas-objects-to-and-from-dosubl
Passing in memory sas objects to and from dosubl  
    %let pgm=utl_passing-in-memory-sas-objects-to-and-from-dosubl;

      Passing in memory sas objects to and from dosubl

      This is an academic exercise.
      I wish I could figure out how to pass SAS in memory objects to R, Perl and Python

      github
      https://tinyurl.com/6ay5663w
      https://github.com/rogerjdeangelis/utl_passing-in-memory-sas-objects-to-and-from-dosubl

      Problem Pass an in memory array to dusubl and return a updated in memory array.

        1.  Pass an in memory numeric and a character array  from the
            parent process to the subprocess. I am using a dosubl child subprocess (proc, sql ...)
        2.  Multiply in memory numeric array elements by 2 and return the updated
            in memory array back to the parent process.
        3.  Reverse the strings in the in memory character array and pass the updated
            in memory array back to the parent process.

       WHY ADDR, PEEK AND POKE ARE SO POWERFUL.

       FUNDAMENTAL COMPUTER INSTRUCTION (THINK IBM/INTEL ASSEMBLY LANGUAGES)

         MIIMAL USEFULL INSTRUCTION SET (LOAD AND STORE(PEEK/POKE) ARE THE MOST IMPORTANT?)
         MINUS IO (Just read the blinking lights on the computer console for I/O Haha)

            LOAD                   PEEK, ADDR
            STORE                  POKE        (STORE IS ONLY AVAILABLE IN THE CLASSIC 1980'S EDITOR
                                                1980's editor is more functional because of this)
            ADD
            SUB
            BRANCH on CONDITION

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    data have;
      rec=1;
      nums= 11111111;
      chrs='aaabbbbb';
      output;
      rec=2;
      nums= 33333333;
      chrs='ccccdddd';
      output;
    run;quit;

    /**************************************************************************************************************************/
    /*                                              |                                                                         */
    /* INPUT: HAVE total obs=2 07JUN2023:10:41:51   |  RULES                                                                  */
    /*                                              |                                                                         */
    /*  REC      NUMS        CHRS                   | LOAD DATA INTO TWO ARRAYS                                               */
    /*                                              |                                                                         */
    /*   1     11111111    aaabbbbb                 | CHRS[2]                                                                 */
    /*   2     33333333    ccccdddd                 |  CHR[1]=aaabbbbb                                                        */
    /*                                              |  CHR[2]=ccccdddd                                                        */
    /*                                              |                                                                         */
    /* OUTPUT: WANT total obs=2 07JUN2023:10:56:21  | NUMS[2}                                                                 */
    /*                                              |  NUM[1]=11111111                                                        */
    /*  WANT total obs=2 07JUN2023:10:56:21         |  NUM[2]=33333333                                                        */
    /*                                              |                                                                         */
    /*                                              |                                                                         */
    /*         TIMES 2     REVERSED                 |                                                                         */
    /*  REC      NUMS        CHRS                   | PASS ARRAYS to SUBPROCESS(DOSUBL)                                       */
    /*                                              |                                                                         */
    /*   1     22222222    bbbbbaaa                 |   MULTIPLE NUMS ARRAY ELEMENTS BY 2                                     */
    /*   2     66666666    ddddcccc                 |   REVERSE THE ODDER OF THE CHRS ARRAY                                   */
    /*                                              |                                                                         */
    /*                                              | RETURN THE UPDATED ARRAYS TO PARENT PROCESS                             */
    /*                                              |                                                                         */
    /*                                              |  AFTER SUBPROCESS BACK IN PARENT PROCESS WE HAVE                        */
    /*                                              |                                                                         */
    /*                                              |  CHR[1]=bbbbbaaa  ** reversed in subprocess                             */
    /*                                              |  CHR[2]=ddddcccc                                                        */
    /*                                              |                                                                         */
    /*                                              |  NUM[1]=22222222  ** times two in subprocess                            */
    /*                                              |  NUM[2]=66666666                                                        */
    /*                                              |                                                                         */
    /**************************************************************************************************************************/

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */
    
    proc datasets lib=work mt=cat;          
      delete sasmac1 sasmac2 sasmac3;       
    run;quit;                               
                                            
    proc datasets lib=work nolist nodetails;
     delete want;                           
    run;quit;                               
                                            
    %symdel adrNum adrChr / nowarn;         


    data want;

       /*---- BEST TO PUT THESE FIRST?    ----*/
       array num[2] 8 _temporary_;
       array chr[2] $8 _temporary_;

       /*----       LOAD ARRAYS           ----*/

       do until (dne);
          set have end=dne;
          num[rec] = nums;
          chr[rec] = chrs;
       end;

    put "before " /
    chr[1] = /
    chr[2] = //
    num[1] = /
    num[2] = ;

       /*---- LOAD ADDRESS                ----*/

    call symputx('adrChr',put(addrlong(chr[1]),$hex16.));
    call symputx('adrNum',put(addrlong(num[1]),$hex16.));

       /*---- CALL SUBPROCESS DOSUBL      ----*/

      rc=dosubl('
         data _null_;

         length chr $8;

         do i=0 to 1;

          /*---- LOAD ADDRESS CONTENTS    ----*/

           chr  =peekclong (ptrlongadd ("&adrChr"x,i*8),8);
           chr=reverse(chr);
           put chr=;


          /*---- STORE CONTENTS IN ARRAY  ----*/
           call  pokelong(chr,ptrlongadd ("&adrChr"x,i*8),8);

          /*---- DO THE SANE FOR NUMS     ----*/

           num  =input(peekclong (ptrlongadd ("&adrNum"x,i*8),8),rb8.);
           num=2*num;
           put num=;
           call  pokelong(num,ptrlongadd ("&adrNum"x,i*8),8);

         end;
         run;quit;
      ');
    /*----  BACK TO PARENT PROCESS L      ----*/

    put "AFTER " /
    chr[1] = /
    chr[2] = //
    num[1] = /
    num[2] = ;

    /*---- OUTPUTUPDATED VALUES    L      ----*/

    do rec=1 to 2;
      nums=num[rec];
      chrs=chr[rec];
      output;
    end;

    stop;
    run;quit;

    /*
    | | ___   __ _
    | |/ _ \ / _` |
    | | (_) | (_| |
    |_|\___/ \__, |
             |___/
    */


    39330   data want;
    39331      /*---- BEST TO PUT THESE FIRST?    ----*/
    39332      array num[2] 8 _temporary_;
    39333      array chr[2] $8 _temporary_;
    39334      /*----       LOAD ARRAYS           ----*/
    39335      do until (dne);
    39336         set have end=dne;
    39337         num[rec] = nums;
    39338         chr[rec] = chrs;
    39339      end;
    39340   put "before " /
    39341   chr[1] = /
    39342   chr[2] = //
    39343   num[1] = /
    39344   num[2] = ;
    39345      /*---- LOAD ADDRESS                ----*/
    39346   call symputx('adrChr',put(addrlong(chr[1]),$hex16.));
    39347   call symputx('adrNum',put(addrlong(num[1]),$hex16.));
    39348      /*---- CALL SUBPROCESS DOSUBL      ----*/
    39349     rc=dosubl('
    39350        data _null_;
    39351        length chr $8;
    39352        do i=0 to 1;
    39353         /*---- LOAD ADDRESS CONTENTS    ----*/
    39354          chr  =peekclong (ptrlongadd ("&adrChr"x,i*8),8);
    39355          chr=reverse(chr);
    39356          put chr=;
    39357         /*---- STORE CONTENTS IN ARRAY  ----*/
    39358          call  pokelong(chr,ptrlongadd ("&adrChr"x,i*8),8);
    39359         /*---- DO THE SANE FOR NUMS     ----*/
    39360          num  =input(peekclong (ptrlongadd ("&adrNum"x,i*8),8),rb8.);
    39361          num=2*num;
    39362          put num=;
    39363          call  pokelong(num,ptrlongadd ("&adrNum"x,i*8),8);
    39364        end;
    39365        run;quit;
    39366     ');
    39367   /*----  BACK TO PARENT PROCESS L      ----*/
    39368   put "AFTER " /
    39369   chr[1] = /
    39370   chr[2] = //
    39371   num[1] = /
    39372   num[2] = ;
    39373   /*---- OUTPUTUPDATED VALUES    L      ----*/
    39374   do rec=1 to 2;
    39375     nums=num[rec];
    39376     chrs=chr[rec];
    39377     output;
    39378   end;
    39379   stop;
    39380   run;

    before
    CHR[1]=aaabbbbb
    CHR[2]=ccccdddd

    NUM[1]=11111111
    NUM[2]=33333333

    SYMBOLGEN:  Macro variable ADRCHR resolves to 2058B43052020000
    SYMBOLGEN:  Macro variable ADRNUM resolves to E055B43052020000
    CHR=bbbbbaaa
    NUM=22222222
    CHR=ddddcccc
    NUM=66666666
    NOTE: DATA statement used (Total process time):
          real time           0.00 seconds
          user cpu time       0.00 seconds
          system cpu time     0.00 seconds
          memory              1704.34k
          OS Memory           50704.00k
          Timestamp           06/07/2023 11:40:06 AM
          Step Count                        6969  Switch Count  0


    AFTER
    CHR[1]=bbbbbaaa
    CHR[2]=ddddcccc

    NUM[1]=22222222
    NUM[2]=66666666

    NOTE: There were 2 observations read from the data set WORK.HAVE.
    NOTE: The data set WORK.WANT has 2 observations and 4 variables.
    NOTE: DATA statement used (Total process time):
          real time           0.27 seconds
          user cpu time       0.07 seconds
          system cpu time     0.15 seconds
          memory              1704.34k
          OS Memory           50704.00k
          Timestamp           06/07/2023 11:40:06 AM
          Step Count                        6969  Switch Count  1

    39380 !     quit;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
