ECE382_Lab5
===========

## Objective
Use knowledge of interrupts and the Timer_A subsytem to reverse engineer a remote control.

### *UPDATES*:
* *REQUIRED FUNCTIONALITY:* ACHIEVED
* *A FUNCTIONALITY:* ACHIEVED (although it is definitely glitchy)
* Fortunately, I was given an extension on the due date for this assignment (surgery and initial recovery)

### The Basic Idea
* Lab day 1: learn the timing and bit patterns for your remote control
* Lab day 2: demonstrate your code can receive and decode button presses from the remote control
* Lab day 3: implement etch-a-sketch or pong

## Lab Day 1

#### Setup
Here is how I hooked up the IR sensor to the MSP430:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/lab_setup.PNG "IR setup")

I used the DFEC-provided APEX remote controller, labeled as #8: 
![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/APEX_remote.jpg "APEX remote #8")

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

The c file I used for the required functionality was: *12_start5.c*

I decided to use the *2* and *8* buttons to toggle the LED *ON* and *OFF*. *2* would turn the LED ON and *8* would turn the LED OFF.

To get required functionality, I went through and followed the provided code diagram/image:

![alt test](https://github.com/sabinpark/ECE382_Lab5/blob/master/images/lab5_schematic.jpg "provided code schematic")

I first declared any of the missing global variables. Below are the global variables used:
```
int8	newIrPacket = FALSE;	// flag to check if there is a new IR packet
int16	packetData[48];		// array to hold packet data
int8	packetIndex = 0;
int32	irPacket;
```

Inside of the .h file, I defined the constants used for each of the buttons:
```
#define		POWER		0x30DFA857		// PWR
#define		UP_1		0x30DF609F		// 2
#define		LEFT_1		0x30DF10EF		// 4
#define		SELECT		0x30DF906F		// 5
#define		RIGHT_1		0x30DF50AF		// 6
#define		DOWN_1		0x30DF30CF		// 8
#define		UP_2		0x30DF40BF		// CH +
#define		DOWN_2		0x30DFC03F		// CH -
```

I then changed the timer subroutines. Following the diagram, I ensured the TimerA interrupt was off, I cleared the packet index, set the new packet flag, and then cleared the timer A flag.
```
#pragma vector = TIMER0_A1_VECTOR			// This is from the MSP430G2553.h file
__interrupt void timerOverflow (void) {

	// turn off Timer A interrupt
	TACTL &= ~TAIE;

	// clear PacketIndex
	packetIndex = 0;

	// set newPacket flag
	newIrPacket = TRUE;

	TACTL &= ~TAIFG;      // clear the flag
}
```

Next, I coded the Port 2 Vector. I did not have to add too much, but once again, I followed the diagram and ensured that everything matched. *NOTE*: I used the case-switch statements pre-written in the code file.

Negative Edge: I read the TAR, classified the logic 1 half-pulse and shifted the bits into the irPacket, stored the TAR in packetData, then enabled the positive edge interrupt.
```
		case 0:		// !!!!!!!!!NEGATIVE EDGE!!!!!!!!!!
			pulseDuration = TAR;    // read TAR

			irPacket <<= 1;

			if(pulseDuration > minLogic1Pulse && pulseDuration < maxLogic1Pulse) {
				irPacket += 1;
			}

			packetData[packetIndex++] = pulseDuration;     // store TAR in packet Data

			LOW_2_HIGH; 	// Setup pin interrupt on positive edge  // enable positive edge interrupt
			break;
```

Positive Edge: I set the TAR to 0, turned Timer A on, enabled Timer A interrupt, then enabled the negative edge interrupt.
```
		case 1:		// !!!!!!!!POSITIVE EDGE!!!!!!!!!!!
			// set TAR = 0
			TAR = 0x0000;	// time measurements are based at time 0

			// turn Timer A on
			TACTL |= MC_0;

			// enable Timer A interrupt
			TACTL |= TAIE;

			HIGH_2_LOW	// Setup pin interrupt on neg. edge  // enable neg. edge interrupt
			break;
```

Finally, I put in some code into main. Inside of the while loop, all I did was create if-statements that checked the value of irPacket. Since I had all of the necessary button codes defined in my .h file, I simply had to compare *irPacket* with each respective button I wanted to use.
```
	while(1)  {
		if(newIrPacket) {
			newIrPacket = FALSE;

			if(irPacket == UP_1) {
				P1OUT |= BIT6;		// green LED ON
			}
			if(irPacket == DOWN_1) {
				P1OUT &= ~BIT6;		// green LED OFF
			}
			if(irPacket == UP_2) {
				P1OUT |= BIT0;		// red LED ON
			}
			if(irPacket == DOWN_2) {
				P1OUT &= ~BIT0;		// red LED OFF
			}
		}
	} // end infinite loop
```

As shown above, it was not difficult at all to implement the red LEDs as well.

### A Functionality

The files I used for the A functionality were: *13_lab5_A.c* and *13_nokia.asm*

To start off I knew that I wanted to implement A functionality using the etch-a-sketch program from the required functionality of Lab 4. After some initial problems in creating a new file (see the debugging section below), I was able to add in some of the code and get my A functionality to work.

I literally took my required functionality code from lab 4 and started by adding in the global variables:
```
// defined constants
int8	newIrPacket = FALSE;	// flag to check if there is a new IR packet
int16	packetData[48];		// array to hold packet data
int8	packetIndex = 0;
int32	irPacket;

void initMSP430();		// initialized the msp430
```

*NOTE*: I also added in the code to initiate the MSP430, which I then called in main.

In main, inside of the infinite while-loop, I adjusted the code from the required functionality (lab 5) and instead of just toggling the LEDs ON and OFF, I also added in code two lines of code (for each button) that checked for any wall boundaries and then triggered a flag that would indicate if a *button* was pressed. For ease of debugging, I kept my LED code as they were.

```
	while(1) {
		if(newIrPacket) {
			newIrPacket = FALSE;
			_disable_interrupt();

			if(irPacket == POWER) {
			}
			if(irPacket == UP_1) {
				P1OUT |= BIT6;		// green LED ON
				if (y>=1) y=y-1;	// move up
				button_press = TRUE;
			}
			if(irPacket == LEFT_1) {
				P1OUT &= ~BIT6;		// green LED OFF
				if (x>=1) x=x-1;	// move left
				button_press = TRUE;
			}
			if(irPacket == SELECT) {
				if(c == 1) c = 0;
				else if(c == 0) c = 1;
				button_press = TRUE;
			}
			if(irPacket == RIGHT_1) {
				P1OUT |= BIT6;		// green LED ON
				if (x<=10) x=x+1;
				button_press = TRUE;
			}
			if(irPacket == DOWN_1) {
				P1OUT &= ~BIT6;		// green LED OFF
				if (y<=6) y=y+1;
				button_press = TRUE;
			}
			if(irPacket == UP_2) {
				P1OUT |= BIT0;		// red LED ON
			}
			if(irPacket == DOWN_2) {
				P1OUT &= ~BIT0;		// red LED OFF
			}
		}
			_enable_interrupt();
```

The program would then check the *button_press* flag and then appropriately draw (or not draw) the block.

The only other things I added to this new c file were the Timer A and Port 2 subroutines. Pretty self-explanatory. 

I ran the program and then got a nice clean initial view of the block as expected. I then used the remote control buttons to move the block around. The block did move around, but everything was glitchy in that the block seemed to move in random spots that corresponded to the general direction of the button inputs. Unfortunately, the program was glitchy, but I am satisfied in that the program did meet the requirements of actually reading the packet data and moving the block.

### Debugging
#### Required Functionality
I initially had trouble with Timer A. When I did turn off Timer A according to the diagram, the LEDs were unresponsive to the remote control inputs. After I got rid of all the code that turned off Timer A, I achieved my required functionality, and the LEDs turned ON and OFF as expected.

#### A Functionality
My first problem was that when I tried to copy over the  code from lab 4's required functionality, the code would not run. I had copied and pasted the contents of the code directly (and several times), but the program would bring up about 5 errors.

I then had the idea of simply copying and pasting the files themselves and renaming those copies. Thus, I did just that and everything worked out fine. 

After writing down my code, my next problem was the glitchiness of the block movements. Obviously there was a problem between the initiations of the LCD screen and the packet-reading. I tried to change the order of the initiations and various other snippets, but I did not get any good results. I have yet to figure out why this glitch occured, but because I did meet the bare minimum of the A functionality, I decided I should get on with my life... 

## Documentation
* JP Terragnoli told me about getting the upper and lower bounds of the timer A counts by using a standard deviation of 5 above and below the mean.
