ECE382_Lab5
===========

## Objective
Use knowledge of interrupts and the Timer_A subsytem to reverse engineer a remote control.

### The Basic Idea
* Lab day 1: learn the timing and bit patterns for your remote control
* Lab day 2: demonstrate your code can receive and decode button presses from the remote control
* Lab day 3: implement etch-a-sketch or pong

# Lab 1

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


