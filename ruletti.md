# C64 "Ruletti"-game source code analysis

## Main program

Main program spans from linenumbers 90 to 160. This contains only subroutine calls with GOSUB. Initialization of some data is done in the 4500-subroutine, Intro-screen on the 4000-subroutine and the main game-loop is subroutines 1000 (choose betting), 2000 (rotating the roulette) and 3000 (payment of winnings). If we still have money (RA > 0), we loop back to line 130.

    90 gosub 4500:rem alkuparametrit
    100 gosub 4000:rem kuva
    120 :
    130 gosub 1000:rem valinnat
    140 gosub 2000:rem pyoritys
    150 gosub 3000:rem voitot
    160 if ra>0 then 130

## Static data initialization

The subroutine at line 4500 is called only once during program execution and it defines values of some static data variables. First one is string variable tx$ (string-variable names have $-postfix) that contains text that will be scrolled on the screen in the game intro.

 	4500  
 	4510 rem **** parametrit ****
 	4520 tx$="                     ruletin on tehnyt robert brotherus 17.4 - 1987."
 	4530 tx$=tx$+"..........paina 'f1' aloittaaksesi pelin.                       "

Lines 4540 defines four array variables with DIM-statement. Variables n, x and y define for each roulette location its number, x- and y-coordinate (text row and column). The si-variable keeps track of money has been betted on each number or red/black or small/large.

 	4540 dim n(23),x(23),y(23),si(27)

Lines 4550-4590 read values for n,x,y triplets from data-statements. Since the data in DATA-statements is read in the loop three at a time (N, X,Y) the groups of three consequent numbers belong logically together. But due to the limitation of the language, this grouping cannot be expressed with the language syntax which makes the data confusing to read.

 	4550 for t=0 to 22:read n(t),x(t),y(t):next t
 	4560 data 0,17,5,8,23,5,1,27,5,16,31,5
 	4565 data 5,31,8,10,31,11,7,31,14
 	4570 data 20,31,17,3,31,20,12,27,20
 	4575 data 17,23,20,14,19,20,19,15,20
 	4580 data 6,11,20,9,7,20,2,3,20
 	4585 data 21,3,17,22,3,14,11,3,11
 	4590 data 18,3,8,15,3,5,4,7,5,13,11,5
 	4640 return

## Game startup sequence

Subroutine starting at linenumber 4000 is for the initial intro screen, drawing the main board, intro scroller and asking for starting money. It first clears the screen and sets screen to black:

    4000 rem **** kuva ****
    4010 print"{clr}":poke 53281,0:poke 53280,0

Then it draws the big "Ruletti" title text using block-graphics characters:

 	4020 print"{down}...{down}{orng}"
 	4030 print"        {CBM-C}{rvon}{CBM-D}{CBM-F}{rvof}{CBM-F}{rvon}{CBM-F}{rvof}{CBM-V}{rvon}{CBM-F}{rvof}{CBM-V}{rvon}{CBM-F}{rvof}{CBM-V} {rvon}{CBM-F}{CBM-I}{rvof}{CBM-K}{rvon}{CBM-D}{CBM-D}{rvof}{CBM-K}{rvon}{CBM-D}{CBM-D}{rvof}{CBM-K}{rvon}{CBM-F}{rvof}{CBM-V}"
 	...
 	4070 print"        {CBM-C}{rvon}{CBM-I}{rvof}{CBM-C}{CBM-V} {rvon}{CBM-I}{rvof}{CBM-V} {rvon}{CBM-I}{CBM-I}{rvof}{CBM-V}{rvon}{CBM-I}{CBM-I}{rvof}{CBM-V}{CBM-C}{rvon}{CBM-I}{rvof} {CBM-C}{rvon}{CBM-I}{rvof} {rvon}{CBM-I}{rvof}{CBM-V}"

This title text is first printed to lower part of the screen. Then a series of empty print statement cause the the content of the screen to scroll upward

	4080 for t=1 to 19:print:for tt=1 to 20:next tt:next t

Line 4082 calls subroutine at 4700 to draw white frame to the center of the board.

 	4082 gosub 4700
	...
	4700 rem ****** tyhjennys *****
	4710 print"{gry3}{home}{down}{down}{down}{down}{down}{down}{down}{down}";
	4720 print tab(7)"UCCCCCCCCCCCCCCCCCCCCCCI"
	4730 print tab(7)"B                      B"
	4740 print tab(7)"{CBM-Q}CCCCCCCCCCCCCCCCCCCCCC{CBM-W}"
	4750 for t=1 to 8:print tab(7)"B                      B":next t
	4760 print tab(7)"JCCCCCCCCCCCCCCCCCCCCCCK{grn}"
	4770 return


### Drawing the board locations

Then loop at lines 4085-4170 draw the roulette locations 0 to 22. This uses the arrays n, x and y that have been earlier initialized with location number and x/y-coordinates. The poke-sys trick on line 4085 moves cursor to the beginning of line y(t).

 	4085 for t=0 to 22:poke 781,y(t):poke 780,0:poke 782,0:sys 65520
	 ...
	4170 next t

Lines 4090-4110 set the color of the location: green (5), red (2) or gray (11). Only location t=0 is green, whereas other locations are red (even) or gray (odd). Cursor color is set then with POKE 646:

 	4090 v=11:if t=0 then v=5:goto 4110
 	4100 if int(t/2)=t/2 then v=2:goto 4110
 	4110 poke 646,v: ...

The location square is then drawn to column x(t) with similar block-graphics and reverse as the title. 

 	4120 print tab(x(t))"{rvon}   {CBM-M}{rvof}"
 	4130 print tab(x(t))"{rvon}   {CBM-M}{rvof}"
 	4140 print tab(x(t))"{rvon}{CBM-P}{CBM-P}{CBM-P}{SHIFT-@}{rvof}"

Finally the number of the location square is printed in its center

	4110 ... a$=str$(n(t)):a$="{rvon}"+a$
	...
 	4150 print"{up}{up}";tab(x(t))a$
 	
 After drawing the location squares the code *optionally* returns from the gosub call:  

 	4195 if r=1 then r=0:return

This is because the location square drawing piece of the subroutine is used both in the context of the wider initialization subroutine starting at 4000 *and* from `3195 r=1:gosub 4085` for only re-initializing the locations. Because subroutines are in this simple Basic-language not defined with static syntax but by runtime execution of GOSUB and RETURN, we can do this kind of confusing hacks of reusing part of bigger subroutine as a smaller subroutine. It would be of course more clear to extact the commonly used part as a separate subroutine, but it's kind of difficult to extract code with hard-coded line-numbers and without proper editor, so this kind of hacks can be understandable.

### Intro scroller

Lines 4200-4250 implement scroller where the intro-text of the game is scrolled horizontally in the middle of the board until F1 is pressed or until the whole text has been scrolled in which case *the whole program is restarted* with `RUN` to show the intro again:

	4200 m=1:poke 198,0
	4210 m$=mid$(tx$,m,22)
 	4220 print"{grn}{home}{down}...{down}{rght}...{rght}";m$
 	4225 for t=1 to 50:next t
 	4230 get a$:if a$="{f1}"then 4300
 	4240 m=m+1:if m=115 then run
 	4250 goto 4210

T  he scroller uses variable `m` to denote current scroll location and takes with `mid$(tx$,m,22)` part of the text to be scrolled from the total intro text `tx$`.

The speed for scrolling is controlled by delay-loop on line 4225: `for t=1 to 50:next t`. There is no command for pausing in C64 Basic, so delays are implemented by busy loops doing dummy work. In a single-tasking operating system on a processor without sleep-state this does not matter of course.

### Initial money input

Next lines 4400-4480 input from player the amount of inital money from 1 to 100 mk to start with to variable `ra` (from "raha", finnish for "money"). When ready, F1 finally returns from the startup subroutine to proceed to the main game loop:

 	4400 gosub 4700:poke 650,128
 	4410 print"{home}{down}{down}...{down}{rght}...{rght}paljonko rahaa ?"
 	4415 print"{down}{down}{down}{down}{rght}...{rght}'f1' = valmis"
 	4420 ra=10
 	4425 print"{home}{down}...{down}{rght}...{rght}                     "
 	4430 print"{home}{down}...{down}{rght}...{rght}rahaa :";ra;"mk."
 	4440 get a$:if a$="" then 4440
 	4450 if a$="{f1}" then gosub 4700:return
 	4460 if a$="+" then ra=ra+1
 	4470 if a$="-" then ra=ra-1
 	4475 if ra=0 then ra=1
 	4477 if ra=101 then ra=100
 	4480 goto 4425
 	4650  

Lines 4425-4480 form input-loop, where current amount of money is first printed. The print-commands at 4425 and 4430 both start with `{home}` which places cursor at (1,1) and then follow by number of `{down}` and `{right}` to start printing from desired absolute position.

Command GET A$ at line 4440 gets character-press from keyboard. GET does not wait for input - if there are no keypresses in the keyboard buffer memory, it simply returns empty string. In this case test in line 4400 loops immediately back to itself.

## Placing the bets

Main loop next calls the first subroutine of the main game loop, "valinnat" ("choices") for choosing how to place bets. `POKE 650,128` on line 1010 disables keyboard repeat behavior so that if user presses keys bit longer time they don't accidentally produce multiple keypresses. Then 1010 and 1015 print instructions:

	1000 rem ***** valinnat *****
	1010 poke 650,128:print"{home}{down}...{down}{rght}...{rght}aseta panoksesi."
	1015 print"{home}{down}...{down}{rght}...{rght}'f1' = valmis"

Lines 1020 initializes variables before the main input loop. The `SI` array stores amount betted for each number (indexes 0-22) and for gray (index 23), red (index 24), small 1-11 (index 25) and large 12-22 (index 26). Variable `SN` keeps the current number to be betted on and `PA` keeps the total amount that has been betted. PA could be calculated as sum of SI-array items, but I deemed it simpler at the time to keep it separate. 

	1020 pa=0:sn=0:for t=0 to 26:si(t)=0:next t
	1025 print"{home}{down}...{down}{rght}...{rght}                "
	1030 print"{home}{down}...{down}{rght}...{rght}rahaa :";ra;"mk."
	1035 print"{rght}...{rght}                     "
	1040 print"{up}{rght}...{rght}sijoitettu :";pa;"mk."
	1045 print"{rght}...{rght}----------------------"
	1047 print"{rght}...{rght}                     "
	1050 print"{up}{rght}...{rght}'";
	1055 if sn<23 then print sn;:goto 1100
	1060 if sn=23 then print"{rvon}{gry1} harmaa {rvof}{grn}";
	1070 if sn=24 then print"{rvon}{red} punainen {rvof}{grn}";
	1075 if sn=25 then print"1-11";
	1080 if sn=26 then print"12-22";
	1100 print"' -";si(sn);"mk."
	1120 get a$:if a$=""then 1120
	1130 if a$<>"{f1}"then 1150
	1140 if pa>0 then 1200
	1145 goto 1120
	1150 if a$="@"and sn>0 then sn=sn-1:goto 1025
	1160 if a$="/"and sn<26 then sn=sn+1:goto 1025
	1170 if a$="+"and ra>0 then si(sn)=si(sn)+1:ra=ra-1:pa=pa+1:goto 1025
	1180 if a$="-"and si(sn)>0 then ra=ra+1:si(sn)=si(sn)-1:pa=pa-1:goto 1025
	1190 goto 1120
	1200 gosub 4700:poke 650,0
	1240 for t=0 to 22
	1250 if si(n(t))>0 then poke 1024+x(t)+y(t)*40,170
	1260 next t
	1265 print"{home}{down}{down}{down}{down}{down}{down}{down}{down}{down}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}odota..."
	1270 return

## Rolling the roulette wheel

 	2000 rem ***** pyoritys *****
 	2015 r=int(rnd(0)*23)+40
 	2030 t=0:k=0:vi=1
 	2050 poke 781,y(t):poke 780,0:poke 782,0:sys 65520
 	2055 if int(t/2)=t/2 then v1=2:v2=10:goto 2060
 	2057 v1=11:v2=12
 	2060 if t=0 then v1=5:v2=13
 	2065 if k>r then vi=vi*1.5
 	2070 poke 646,v2
 	2080 print tab(x(t))"{rght}{rvon}  {CBM-M}{rvof}"
 	2085 print tab(x(t))"{rvon} {rght}{rght}{CBM-M}{rvof}"
 	2090 print tab(x(t))"{rvon}{CBM-P}{CBM-P}{CBM-P}{SHIFT-@}{rvof}{up}{up}{up}"
 	2095 poke 646,v1:for tt=1 to vi:next tt
 	2097 if vi>800 then 2160
 	2100 print tab(x(t))"{rght}{rvon}  {CBM-M}{rvof}"
 	2105 print tab(x(t))"{rvon} {rght}{rght}{CBM-M}{rvof}"
 	2110 print tab(x(t))"{rvon}{CBM-P}{CBM-P}{CBM-P}{SHIFT-@}{rvof}"
 	2120 t=t+1:if t=23 then t=0
 	2130 k=k+1
 	2150 goto 2050
 	2160 t=n(t)
 	2170 vv=t
 	2190 return

## Paying winnings

	3000 rem ***** voitot *****
	3030 gosub 4700
	3040 print"{grn}{home}{down}{down}{down}{down}{down}{down}{down}{down}{down}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}voitot:"
	3050 vo=0:print:for t=0 to 22
	3060 if t<>vv then 3070
	3062 if si(t)=0 then 3070
	3065 vo=vo+si(t)*22:print"{grn}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}no.";vv;" voit."si(t)*22"mk."
	3070 next t
	3075 if vv>12 or si(25)<1 then 3082
	3080 print"{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}'1-11' voit. "si(25)*2"mk.":vo=vo+si(25)*2:goto 3090
	3082 if si(26)<1 then 3090
	3085 if vv>11 then print"{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}'12-22' voit."si(26)*2"mk.":vo=vo+si(26)*2
	3090 if vv=0 then 3120
	3092 if int(vv/2)<>vv/2 then 3100
	3093 if si(23)=0 then 3120
	3095 print"{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rvon}{gry1}harmaa{rvof}{grn} voit.";si(23)*2"mk.":vo=vo+si(23)*2
	3097 goto 3120
	3100 if si(24)=0 then 3120
	3110 print"{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rvon}{red}punainen{rvof}{grn} voit.";si(24)*2"mk.":vo=vo+si(24)*2
	3120 print"{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}voitot yht. :";vo;"mk."
	3130 print"{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}sijoitukset :";pa
	3150 print"{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}tulos       :";vo-pa"mk."
	3160 ra=ra+vo
	3170 print"{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}rahaa       :";ra
	3180 print"{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}{rght}    paina 'f1'"
	3185 poke 198,0
	3190 get a$:if a$<>"{f1}"then 3190
	3195 r=1:gosub 4085:gosub 4700:print"{grn}"
	3200 return
