# Casio CZ-101, CZ-1000, and CZ-5000 MIDI specification

**This information from [youngmonkey](http://www.youngmonkey.ca/nose/audio_tech/synth/Casio-CZ.html).**

This document covers transmission of programming information ( both to and from the CZ ), setting of the
controllers you previously couldn't access like tone mix level, and other bits besides.

First off, let's recap on the simple stuff. The MIDI is a digital interface to musical instruments, and relies on serial transmission of data. These data are usually talked of in terms of bytes, and I shall be using hexadecimal numbers in this posting.

There are basically two types of bytes sent over MIDI - control bytes and data bytes. Control bytes are distinguished by having values over 0x80 ( 80 hex, 128 decimal ), and these have valious meanings:

## Note On

A note on message consists of sending a "NOTE ON" control byte, the note number you want to turn on, and the velocity at which you want to play the note. The note on control is 90 plus the channel number. So, for example, if you want to play note 32 ( = hex 20) at speed 64 (= hex 40), on the midi instrument receiving on channel two, then you would send:

| 92 | 20 | 40 |
| --- | --- | --- |
| NOTE ON, channel 2 | Note #32 | Velocity 64 |

If you wish to turn two or more notes on at the same time, the control byte need not be retransmitted.
For example, to turn note 35 on as well, you could send:

| 92 | 20  40 | 23  40 |
| --- | --- | --- |
|NOTE ON, ch. 2 | Note #32, velocity 64 | Note #35, velocity 64 |

These codes can be transmitted both ways on the CZ 101, 1000, 5000, but since they do not detect note velocity, it is always transmitted and recognized as 64 (= 40 hex).

## Note Off

Just send a note on message with velocity 0. For exmaple,  to turn note 35 off, send:

| 92 | 23 | 00 |
| --- | --- | --- |
| NOTE ON, ch. 2 | Note #35 | off |


## Control Change

There are several controls that can be set from MIDI. Just send a "CONTROL CH" byte , which is B0 plus the channel number, the number of the control that you wish to change, then the value you wish to set it
to. For example, for CZ101 portamento time, send

| B0 | 05 | 10 |
| --- | --- | --- |
| CONTROL, ch. 0 | Portamento time | 16 |

The controls are:

| Code | Function | Data |
| ---- | -------- | ---- |
| 01 (CZ-101/1000)  | Vibrato on/off | 0 = off, 7F = ON | 
| 01 (CZ-5000) | Modulation wheel | 00..7F |
| 05   | Portamento time | 00..63 (0..99) | 
| 06   | Master tune | 00..7F |
| 40 (CZ-5000) | Sustain pedal | 0 = off, 7F = ON |
| 41   | Portamento on/off | 0 = off, 7F = ON |

4) PROGRAM CHANGE
-----------------

This allows you to change between the preset sounds ( and your internal
sounds and cartridges ). Just send C0 plus the channel number, then the
program number. Eg to set CZ101 on channel 1 to Synth Bass:

        C1              07
PROGRAM ch 1            Program 7

Note that the preset tones are given numbers :

CZ101/1000

0..0F   Preset sounds 1..16
20..2F  Internal sounds 1..16
40..4F  Cartridge sounds 1..16

CZ5000

00..1F  Preset sounds A1,A2,A3....D6,D7,D8
20..3F  Internal sounds A1,A2.....D6,D7,D8


5) PITCH BEND CHANGE
--------------------

This is acheived by sending E0 plus the channel number, then two bytes
denoting the new value of pitch bend. The first byte is the most
significant, and the second the least significant. Note also that the
lower 6 bits of the lower byte are not used, and that the central
position of the wheel corres- ponds to the byte sequence 40 00.

        HIGHEST         7F 40

        HIGHER

        CENTER          40 00

        LOWER

        LOWEST          00 00


So, to bend the instrument on channel two UP by about half its maximum
amount, send

        E2              60      00
BEND channel two        ---1/2 up--


6) AFTER TOUCH
--------------
Is not supported on the CZ101/1000/5000. Sorry!


7) MODE CHANGE
--------------
This is very similar to the CONTROL CHANGE message, and can be regarded
as a special case.

OMNI ON         send E0 + channel, 7D, 00
OMNI OFF        send B0 + channel, 7C, 00
POLY ON         send B0 + channel, 7F, 00
POLY OFF        send B0 + channel, 7E, 00

OMNI mode plays any MIDI data received at the MIDI IN plug on the back
of the machine, regardless of channel. POLY mode is equivalent to the
SOLO button on the front panel. With the CZ101, for instance, POLY OFF
( =SOLO ) allows the synth to be used as four monophonic synthesizers
under remote control.

LOCAL ON        send B0 + channel, 7A, 7F
LOCAL OFF       send B0 + channel, 7A, 00

Local mode means that the keyboard is "connected" to the sound
producing part of the CZ within the machine itself. With LOCAL ON (the
default setting), playing the keyboard both sends note messages out of
the MIDI port, and also makes sounds at the same time. If you want to
do wierd things like keyboard splitting, LOCAL OFF will allow you to
see what the keyboard is doing without the CZ making any sound at all.
You could then act on that information and send the keyboard a command
depending on the keys that has nothing to do with them, eg program
change or pitch bend. The possibilities are endless !


SEQUENCER MESSAGES
------------------

The CZ5000 has its own internal sequencer, which can be controlled
by:

F8      Clock byte: transmitted 24 times per quarter note ( crotchet
)
FA      Start: same as pressing the PLAY button on the front panel
FB      Continue: continue song where last stopped
FC      Stop: stops song play at current position
FD      Active sense: basically, a cry of "Is there anybody out
        there". If no reply is received within about 1/3 second, it shuts
        the voice off.


SYSTEM EXCLUSIVE MESSAGES
-------------------------

At last, the really meaty stuff. :-)

These all have the basic form:

        F0      machine ID      some bytes      F7
SYS EX MESSG    YES, YOU        DO THIS         END OF SYS EX

Ok, so not very specific, bu that was deliberate to allow manufacturers
to use all the lovely bells and whistles they put on their machine over
the MIDI !

Since these are usually controlled by computer, I have set them out as
a computer/synthesizer dialogue. Note that the computer MUST wait for
replies before proceding, or all will fail !

Here we go, then

1) SET BEND RANGE
-----------------

Computer:       F0 44 00 00 70+channel 40 data F7

Eg to set bend range to 8 on channel 4, send
                F0 44 00 00 74 40 08 F7

2) KEY TRANSPOSE
----------------

Computer:       F0 44 00 00 70+channel 41 data F7

Data is as follows:
Key:    G  A  A# B  B# C  C# D  E  E# F  F#
Data:   45 44 43 41 41 00 01 02 03 04 05 06

Eg to set key on channel 0 to C#, send
                F0 44 00 00 70 41 01 F7

3) TONE MIX
-----------

Computer:       F0 44 00 00 70+channel 42 data F7

The data is 00 to turn tone mix off, or 41..49 for mix level 1..9

Eg to set tone mix on channel 0 to 7, send
                F0 44 00 00 70 42 47 F7

4) ASK ABOUT PROGRAMMER ( Send request 2 )
-----------------------

Computer:       F0 44 00 00 70+channel 19 00
CZ101/1000:     F0 44 00 00 70+channel 30
Computer:       70+channel 31
CZ101/1000:     data1 data2 F7
Computer:       F7

data1 is the program selected ( see PROGRAM CHANGE )
data2 returns the vibrato/portamento on/off setting:

data2   00      10      20      30
Vibrato OFF     OFF     ON      ON
Port'o  OFF     ON      OFF     ON

Eg an exchange such as
Computer:       F0 44 00 00 70 19 00            "Want data on channel 0"
CZ101:          F0 44 00 00 70 30               "Gotcha.. data ready"
Computer:       70 31                           "Ok, give it to me"
CZ101:          27 30 F7                        "Internal 8, v on, p on"
Computer:       F7

REMOTE PROGRAMMING
------------------
The send request 1 and receive request 1 messages.

These dump a lot of data across the MIDI, which is the same for both
messages, except that the data go the other way. The exchanges are:

Send request
Computer:       F0 44 00 00 70+channel 10 program
CZ101/1000:     F0 44 00 00 70+channel 30
Computer:       70+channel 31
CZ101/1000:      F7
Computer:       F7

Receive request
Computer:       F0 44 00 00 70+channel 20 program
CZ101/1000:     F0 44 00 00 70+channel 30
Computer:        F7
Cz101/1000:     F7

The program byte is the same as that set by the PROGRAM CHANGE
function, with the addition that you can request the temporary sound
area as well ( number is 60 ). This is the area that is used if you
have altered a preset and not saved it into internal memory.

 is a sequence of 256 bytes containing a LOT of info. Now
Casio have done a lot of funny things with these, like splitting bytes
in half and encoding things in wierd ways so please bear with me.

To keep everything this side of infinite length, I shall adopt the same
strategy as the manual, which is to write data in bytes, although they
are transmitted in half- bytes. For example, me writing a byte as 5F
requires you to transmit or receive as 0F 05 ( wierd, huh ? ). This
will obvoiusly save a lot of space.

So, here goes again :-)

There are 25 distinct sections within 

Sec#    Length  Symbol          Contents
        (bytes)

1       1       pflag           line select data, octave range
2       1       pds             detune up or down
3       2       pdl,pdh         detune range
4       1       pvk             vibrato wave number
5       3       pvdld,pvdlv     vibrato delay time
6       3       pvsd,pvsv       vibrato rate
7       3       pvdd,pvdv       vibrato depth
8       2       mfw             dco1 waveform
9       2       mamd,mamv       dca1 key follow
10      2       mwmd,mwmv       dcw1 key follow
11      1       pmal            end step number of dca1 envelope
12      16      pma             dca1 envelope rate/level
13      1       pmwl            end step number of dcw1 envelope
14      16      pmw             dcw1 envelope rate/level
15      1       pmpl            end step number of dco1 envelope
16      16      pmp             dco1 envelope rate/level
17      2       sfw             dco2 waveform
18      2       samd,samv       dca2 key follow
19      2       swmd,swmv       dcw2 key follow
20      1       psal            end step number of dca2 envelope
21      16      psa             dca2 rate/level
22      1       pswl            end step number of dcw2 envelope
23      16      psw             dcw2 rate/level
24      1       pspl            end step number of dco2 envelope
25      16      psp             dco2 rate/level

1) PFLAG
Looking at bits,
        0000  00   00
 Not used^   OCTV  LS

OCTV controls octave range: 00=octave 0, 01=+1, 10=-1

LS is the line select: 00=1, 01=2, 10=1+1', 11=1+2'

So, fo Octave +1, line select 1+1', PFLAG=00000110 = 06

2)PDS
For detune +, PDS is 0, for detune - it is 01

3)PDETL,PDETH

Two bytes controlling the depth of the detune.
The first byte is the FINE data.

FINE:   0..15   16..30  31..45  46..60
Byte:   00..0F  11..1F  21..2F  31..3F

The second contains both the octave and note data:

OCT:    0       1       2       3
NOTE:   0..11   0..11   0..11   0..11
Byte:   00..0B  0C..17  18..23  24..2F


4) PVK

This is the vibrato wave number, encoded as follows

WAVE NUMBER:    1       2       3       4
Byte:           08      04      20      02

5) PVDLD,PVDLV

This is the vibrato delay time, transmitted in three bytes.

Delay   Bytes           Delay   Bytes           Delay   Bytes
25      19 00 19        50      32 00 4B        75      4B 00 DF
26      1A 00 1A        51      33 00 4F        76      4C 00 E7
27      1B 00 1B        52      34 00 53        77      4D 00 EF
28      1C 00 1C        53      35 00 57        78      4E 00 F7
29      1D 00 1D        54      36 00 5B        79      4F 00 FF
30      1E 00 1E        55      37 00 5F        80      50 01 0F
31      1F 00 1F        56      38 00 63        81      51 01 1F
32      20 00 21        57      39 00 67        82      52 01 2F
33      21 00 23        58      3A 00 6B        83      53 01 3F
34      22 00 25        59      3B 00 6F        84      54 01 4F
35      23 00 27        60      3C 00 73        85      55 01 5F
36      24 00 29        61      3D 00 77        86      56 01 6F
37      25 00 2B        62      3E 00 7B        87      67 01 7F
38      26 00 2D        63      3F 00 7F        88      58 01 8F
39      27 00 2F        64      40 00 87        89      59 01 9F
40      28 00 31        65      41 00 8F        90      5A 01 AF
41      29 00 33        66      42 00 97        91      5B 01 BF
42      2A 00 35        67      43 00 9F        92      5C 01 CF
43      2B 00 37        68      44 00 A7        93      5D 01 DF
44      2C 00 39        69      45 00 AF        94      5E 01 EF
45      2D 00 3B        70      46 00 B7        95      5F 01 FF
46      2E 00 3D        71      47 00 BF        96      60 02 1F
47      2F 00 3F        72      48 00 C7        97      61 02 3F
48      30 00 43        73      49 00 CF        98      62 02 5F
49      31 00 47        74      4A 00 D7        99      63 02 7F

For delays in the range 0..31, just transmit 00..1F, 00, 00..1F eg for
delay of 12, send 0C 00 0C. This is convenient since it saves me typing
in another column of boring numbers ;-)

6) PVSD,PVSV

Again, here comes another table for conversions. The first column
(0..24) is omitted since the only difficult thing needed is to add 01
00 20 to each entry ( The first few go 00 00 20, 01 00 40, 02 00 60,
... 06 00 E0, 07 01 00, ..)

Rate    Bytes           Rate    Bytes           Rate    Bytes
25      19 03 40        50      32 09 E0        75      4B 1C E0
26      1A 03 60        51      33 0A 60        76      4C 1D E0
27      1B 03 80        52      34 0A E0        77      4D 1E E0
28      1C 03 A0        53      35 0B 60        78      4E 1F E0
29      1D 03 C0        54      36 0B E0        79      4F 20 E0
30      1E 03 E0        55      37 0C 60        80      50 23 E0
31      1F 04 00        56      38 0C E0        81      51 25 E0
32      20 04 60        57      39 0D 60        82      52 27 E0
33      21 04 A0        58      3A 0D E0        83      53 29 E0
34      22 04 E0        59      3B 0E 60        84      54 2B E0
35      23 05 20        60      3C 0E E0        85      55 2D E0
36      24 05 60        61      3D 0F 60        86      56 2F E0
37      25 05 A0        62      3E 0F E0        87      57 31 E0
38      26 05 E0        63      3F 10 60        88      58 33 E0
39      27 06 20        64      40 11 E0        89      59 35 E0
40      28 06 60        65      41 12 E0        90      5A 37 E0
41      29 06 A0        66      42 13 E0        91      5B 39 E0
42      2A 06 E0        67      43 14 E0        92      5C 3B E0
43      2B 07 20        68      44 15 E0        93      5D 3D E0
44      2C 07 60        69      45 16 E0        94      5E 3F E0
45      2D 07 A0        70      46 17 E0        95      5F 41 E0
46      2E 07 E0        71      47 18 E0        96      60 47 E0
47      2F 08 20        72      48 19 E0        97      61 4B E0
48      30 08 E0        73      49 1A E0        98      62 4F E0
49      31 09 60        74      4A 1B E0        99      63 53 E0

7) PVDD,PVDV

These are again encoded as three bytes in a most obscure way. Below 32,
the encoding is 00..1F, 00, 01..20 eg for depth 13, send 0D 00 0E.

Depth   Bytes           Depth   Bytes           Depth   Bytes
25      19 00 1A        50      32 00 4F        75      4B 00 E7
26      1A 00 1B        51      33 00 53        76      4C 00 EF
27      1B 00 1C        52      34 00 57        77      4D 00 F7
28      1C 00 1D        53      35 00 5B        78      4E 00 FF
29      1D 00 1E        54      36 00 5F        79      4F 01 07
30      1E 00 1F        55      37 00 63        80      50 01 1F
31      1F 00 20        56      38 00 67        81      51 01 2F
32      20 00 23        57      39 00 6B        82      52 01 3F
33      21 00 25        58      3A 00 6F        83      53 01 4F
34      22 00 27        59      3B 00 73        84      54 01 5F
35      23 00 29        60      3C 00 77        85      55 01 6F
36      24 00 2B        61      3D 00 7B        86      56 01 7F
37      25 00 2D        62      3E 00 7F        87      57 01 8F
38      26 00 2F        63      3F 00 83        88      58 01 9F
39      27 00 31        64      40 00 8F        89      59 01 AF
40      28 00 33        65      41 00 97        90      5A 01 BF
41      29 00 35        66      42 00 9F        91      5B 01 CF
42      2A 00 37        67      43 00 A7        92      5C 01 DF
43      2B 00 39        68      44 00 AF        93      5D 01 EF
44      2C 00 3B        69      45 00 B7        94      5E 01 FF
45      2D 00 3D        70      46 00 BF        95      5F 02 0F
46      2E 00 3F        71      47 00 C7        96      60 02 3F
47      2F 00 41        72      48 00 CF        97      61 02 5F
48      30 00 47        73      49 00 D7        98      62 02 7F
49      31 00 4B        74      4A 00 DF        99      63 03 00

8) MFW

These two bytes transmit the waveform for DCO1, and also the modulation
ie ring, noise or none.

                First byte      Second byte
                000 000 0  0  00 000 000

First=1         000        0  00
Fisrt=2         001        0  00
First=3         010        0  00
First=4         100        0  00
First=5         101        0  00
First=6         110        0  01
First=7         110        0  10
First=8         110        0  11
Second=1            000 1  0  00
Second=2            001 1  0  00
Second=3            010 1  0  00
Second=4            100 1  0  00
Second=5            101 1  0  00
Second=6            110 1  0  01
Second=7            110 1  0  10
Second=8            110 1  0  11
NO MODULATION                    000
RING MODULATION                  100
NOISE MODULATION                 011

So, for instance, to set first = 4, second= 2, ring modulation, we
have

        100 001 1 0 00 100 000 = 1000 0110 0010 0000 = 86 20

9) MAMD,MAMV

These two bytes set the DCA1 key follow:

Key follow:     0   1   2   3   4   5   6   7   8   9
1st byte:       00  01  02  03  04  05  06  07  08  09
2nd byte:       00  08  11  1A  24  2F  3A  45  52  5F


10) MWMD, MWMV

These two bytes set the DCW1 key follow

Key follow:     0   1   2   3   4   5   6   7   8   9
1st byte:       00  01  02  03  04  05  06  07  08  09
2nd byte:       00  1F  2C  39  46  53  60  6E  92  FF


11) PMAL

This sets the position of the end step on DCA1. Step 1..8 gives bytes
00..07.

12) PMA

This consists of 8 repetitions of Rate,Level.

Given that you wish to set rate r, the data you need to send is

        Byte= 119 x r
              -------
                99
Conversely, if byte=0, rate=0, if byte=7F, rate=99, otherwise

        r=99 x byte
          --------- + 1
             119

Add 80 hex if the level will be coming down on this step.

The level goes up linearly, with 0 being 00, up to 99 is 7F, so that

        Byte= 127 x level
              -----------
                 99

and     Level= 99 x byte
               --------- + 1
                  127

except at byte=0 where level=0, and byte=127, where level=99

In all these conversions, fractional parts are ignored, so a result of
byte=24.6987 would be taken as byte=24.

13) PMWL

End step number for DCW1. Same as PMAL


14) PMW

This sets the steps in DCW1, and consists of 8 repetitions of
Rate,Level. The format is similar to PMA, so that you add 80 ( 128 dec
) to the rate if the level is coming down this step, and that you add
80 to the level if you wish to set a sustain point. The level data is
handled the same as PMA, but for some strange reason the rate data is
encoded differently.

So      byte= 119 x level
              ----------- + 8
                  99

and     level= 99 x (byte-8)
               ------------- +1
                    119

except where byte=8, level=0, and where byte=77, level=99

15) PMPL

Another end step setting, this time for DCO1. Same as PMAL and PMWL

16) PMP

Another envelope setting, this time for the DCO1 rates and levels.
Again Casio uses a completely different encoding scheme.

        byte= 127 x rate
              ----------
                  99
and
        rate= 99 x byte
              --------- + 1
                 127

except where byte=00, rate=0, where byte=7F, rate=99

For the level, level data 0..63 translate as bytes 00..3F, and level
data 64..99 translate as bytes 44..67.

17) SFW

These two bytes set the waveform for DCO2. They use the same format as
MFW does for DCO1, except that the modulation bits are ignored ( it is
best to set these bits to zero , just in case ).

18) SAMD,SAMV
19) SWMD,SWMV
20) PSAL
21) PSA
22) PSWL
23) PSW
24) PSPL
25) PSP

All the above use the same formats as their counterparts for the first
set of DCO,DCW,DCA, and perform exactly the same functions on the
DCA2,DCW2,and DCO2.
