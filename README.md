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
Here is the how I hooked up the IR sensor to the MSP430:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/lab_setup.PNG "IR setup")

#### Digital Logic Analyzer
I hooked up the digitical logic analyzer to my IR sensor/MSP430 setup and got the following results:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/digital_logic_analyzer.jpg "digital logic analyzer result")

#### Code Composer Studio
From Code Composer Studio, I got time0 and time1 count values.
##### time0
![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/time0_array.PNG "time0 count result")
##### time1
![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/time1_array.PNG "time1 count result")

#### Questions/Answers
###### Question 1
*How long will it take the timer to roll over?*
* 65.535 ms

###### Question 2
*How long does each timer count last?*
* 1 microsecond

###### Question 3
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

Here are the codes for instructor-requested remote control buttons:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/remote_codes_initial.PNG "remote control button codes (initial)")

Notice how the VOLUME buttons have two extra bytes and a bit than expected. As a result, I decided not to use the VOLUME buttons. 

Here are the codes for the buttons that I wanted to implement:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/remote_codes_final.PNG "remote control button codes (final)")

## Lab Day 2

### Required Functionality
I decided to use the CHANNEL buttons to toggle the LED *ON* and *OFF*.

### A Functionality

## Documentation
* JP Terragnoli told me about getting the upper and lower bounds of the timer A counts by using a standard deviation of 5 above and below the mean.
