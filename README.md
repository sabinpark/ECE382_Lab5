ECE382_Lab5
===========

## Objective
Use knowledge of interrupts and the Timer_A subsytem to reverse engineer a remote control.

### The Basic Idea
* Lab day 1: learn the timing and bit patterns for your remote control
* Lab day 2: demonstrate your code can receive and decode button presses from the remote control
* Lab day 3: implement etch-a-sketch or pong

## Lab Day 1

#### Setup
Here is how I hooked up the IR sensor to the MSP430:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/lab_setup.PNG "IR setup")

#### Digital Logic Analyzer
I hooked up the digital logic analyzer to my IR sensor/MSP430 setup and got the following results on the DLA:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/digital_logic_analyzer.jpg "digital logic analyzer result")

For the DLA, I had to make sure that the proper ports were set up on the DLA itself. I also double checked to make sure the DLA would read in data on the negative edge. Everything seemed setup properly and when I ran the program, I got good results.

#### Code Composer Studio
From Code Composer Studio, I got the respective time0 and time1 count values.
##### time0
![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/time0_array.PNG "time0 count result")
##### time1
![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/time1_array.PNG "time1 count result")

#### Questions/Answers
###### Question 1
*How long will it take the timer to roll over?*
```
	TAR = 0x0000;                     // time measurements are based at time 0
	TA0CCR0 = 0xFFFF;					        // create a 16mS roll-over period
	TACTL &= ~TAIFG;					        // clear flag before enabling interrupts = good practice
	TACTL = ID_3 | TASSEL_2 | MC_1;		// Use 1:1 presclar off MCLK and enable interrupts
```
Given the information above, I used simple dimensional analysis techiniques and found the answer:
![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/question1.PNG "question 1")
* Answer: 65.535 ms

###### Question 2
*How long does each timer count last?*
Again, I used some dimensional analysis techniques to find how long each timer count lasts:
![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/question2.PNG "question 2")
* Answer: 1 microsecond

###### Question 3
For this next part, I looked at the code from lines 32-34 and 36-38. Everything was pretty intuitive. *NOTE:* the pulse durations are the while loops.
![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/wave_form_question.PNG "waveform/code")

#### Remote Timer A Counts
|Pulse|Duration(ms)|Timer A Counts|
|:-:|:-:|:-:|
|Start logic 0 half-pulse|8800|8741|
|Start logic 1 half-pulse|4400|4340|
|Data 1 logic 0 half-pulse|550|562|
|Data 0 logic 0 half-pulse|550|524|
|Data 0 logic 0 half-pulse|600|566|
|Data 1 logic 0 half-pulse|500|521|
|Stop logic 0 half-pulse|600|567|
|Stop logic 1 half-pulse|1600|1609|

#### Timer A Samples/Statistics
I used the "OK" button on the APEX remote (#8 DFEC) and got the following results:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/timerA_samples.PNG "timer A samples")

I obtained 8 samples and then found the average and standard deviation of each pulse type. Based on the prompt, I then found the range of values that would correctly classify 99.9999426697% of the pulses if I were to continue taking more samples. Looking up *standard deviation* on wikipedia, I found that 99.9999426697% of the pulses will be encompassed in my range if I set my upper boundary as 5 standard deviations above the mean, and the lower boundary as 5 standard deviations below the mean.

Next, by reading in the logic 1 half-pulses, I was able to translate the digital highs and lows into binary code. Using that binary code (8 bytes), I then found the corresponding hexadecimal values.

Here are the codes for the instructor-requested remote control buttons:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/remote_codes_initial.PNG "remote control button codes (initial)")

Notice how the VOLUME buttons have two extra bytes and a bit than expected. This was unexpected and strange. This was probably due to how the the remote was designed. As a result, I decided not to use the VOLUME buttons. 

For my own purposes, I wanted to find the hexadecimal code for different buttons. Thus, I used the DLA once again and found the corresponding codes for the following buttons:

* Power
* 2
* 4
* 5
* 6
* 8
* CH +
* CH -

Here are the codes for the buttons that I wanted to implement:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/remote_codes_final.PNG "remote control button codes (final)")

## Lab Day 2

### Required Functionality
I decided to use the *2* and *8* buttons to toggle the LED *ON* and *OFF*. *2* would turn the LED ON and *8* would turn the LED OFF.

To get required functionality, I followed the provided code diagram/image:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/lab5_schematic.PNG "provided code schematic")



### A Functionality

## Documentation
* JP Terragnoli told me about getting the upper and lower bounds of the timer A counts by using a standard deviation of 5 above and below the mean.
