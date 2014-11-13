Lab5
====

#Reverse Engineering a Remote Control
##Purpose
The goal of this lab was to use your knowledge of interrupts and the Timer_A subsytem to reverse engineer a remote control.
##Preliminary Design
Although I did not understand how to implement the objectives, going into the lab I had a basic understanding that our Microcontroller should recieve remote control signals and use them to tun on off lights.
##Day 1
###Timer Counts
How long will it take for the timer to rollover?
The clock is 8 MHz TACCR0 goes to 0xFFFF, and the clock divider is 3 so 65.5 ms.

How long does each timer count last?
Each timer count lasts 1 us.

![alt tag](https://raw.githubusercontent.com/seanbapty/Lab5/master/timerpicuture.JPG)

###IR data packets
| Pulse                     | Duration (ms) | Timer A counts |
|---------------------------|---------------|----------------|
| Start logic 0 half-pulse  | 8.828         | 8828           |
| Start logic 1 half-pulse  | 4.375         | 4375           |
| Data 1 logic 0 half-pulse | 0.569         | 569            |
| Data 1 logic 1 half-pulse | 1.6296        | 1629.6         |
| Data 0 logic 0 half-pulse | 0.566         | 566            |
| Data 0 logic 1 half-pulse | 0.5332        | 533.2          |
| Stop logic 0 half-pulse   | 8.8164        | 8816.4         |
| Stop logic 1 half-pulse   | 2.2334        | 2233.4         |


|                   | Data 1 Logic 1 (ms) | Data 0 logic 0 (ms) | Data 0 Logic 1 (ms) |
|-------------------|---------------------|---------------------|---------------------|
|                   | 1.55                | 0.6                 | 0.45                |
|                   | 1.5                 | 0.6                 | 0.45                |
|                   | 1.55                | 0.6                 | 0.45                |
|                   | 1.55                | 0.6                 | 0.5                 |
|                   | 1.55                | 0.55                | 0.45                |
|                   | 1.5                 | 0.6                 | 0.5                 |
|                   | 1.55                | 0.55                | 0.5                 |
|                   | 1.55                | 0.55                | 0.5                 |
|                   |                     |                     |                     |
| mean              | 1.5375              | 0.58125             | 0.475               |
| sd                | 0.023145502         | 0.025877458         | 0.026726124         |
| timer range lower | 1421.772488         | 451.862708          | 341.369379          |
| timer range upper | 1653.227512         | 710.637292          | 608.630621          |


| Button | code (not including start and stop bits) |
|--------|------------------------------------------|
| 0      | 30DF20DF                                 |
| 1      | 30DFA05F                                 |
| 2      | 30DF609F                                 |
| 3      | 30DFE01F                                 |
| Power  | 30DFA857                                 |
| VOL +  | EA9A1A5A1A                               |
| VOL -  | EA9A1A5A1A                               |
| CH +   | 30DF40BF                                 |
| CH -   | 30DFC03F                                 |

##Day 2
The required functionality was achieved by implementing the "PWR" button to toggle the red LED on and off, and the "ZER" button to toggle the green LED on and off. A Functionality was not achieved.

#Code
##Pseudocode
Very little psudocode was needed, because there was very little code that had to be written. However, the trickiest part of the lab was getting the program to read and decode the message coming in from the remote. Normally this could be done with a simple comparison, but the data coming in was bit-by-bit stored in an array, and the comparison values defined in the header were one dimensional. Therefore, a comparison value was made with a shifting logic high bit and the result was either incremented or kept the same based on the comparison between the incoming data and the comparison array.

Below is the pseudocode that helped solve this dilema of adding an array to an integer.
```
comparison value of same length as incoming data with a 1 in the MSB
loop through all the bits coming in from the remote
				if there is a one coming in from the remote
					OR result with comparison value (a one in the comparison or'ed with anything will yield a one)
				}
				if there is a zero coming in from the remote
					not the comarison array changing the one to a zero and and then and it with the result value (0 and with anything is zero)
					}
				rotate comparison with a 1 in the MSB to the right
			}
```
##C Code
There were 3 main changes made to this lab. The first was to change the constants defined in the header to values specific to the remote used.
```
#define		ZER		0x30DF20DE //updated key values based on logic analyzer output
#define		PWR		0x30DFA856
#define		ONE		0x30DFA05F
#define		TWO		0x30DF609F
#define		THR		0x30DFE01F

#define		VOL_UP	0xEA9A1A5A1A
#define		VOL_DW	0xEA9A1A5A1A
#define		CH_UP	0x30DF40BF
#define		CH_DW	0x30DFC03F
```
Second, it was necessary to implement the pseudocode above to read the data coming into the program.
```
void main(void) {

	initMSP430();				// Setup MSP to process IR and buttons
	P1DIR |= BIT0 | BIT6;
	P1OUT &= ~(BIT0 & BIT6);

	while(1)  {
		int32 checkValue = 0x80000000; //comparison value with a 1 set in the MSB
		if (packetIndex == 34) {	   //start filling data at the appropriate time
			//newIrPacket = TRUE;
			int i = 2;
			for (i; i<33; i++){ //32 bit data packets
				if(packetData[i]>1500){	//logic one sent from remote
					result |= checkValue;
				}
				else if(packetData[i]<600){//logic zero sent from remote
					result &= ~checkValue;
				}
				checkValue >>=1; //rotate the bit right of the compare value
			}
			if (result==PWR){    //button press defined in header
				P1OUT ^= BIT6;

			}
			else if (result==ZER){
				P1OUT ^= BIT0;
			}
			//result=0;
			packetIndex++;
		} // end if new IR packet arrived
	} // end infinite loop
} // end main
```
Finally, it was necessary to set the interrupt flag so that after recieving data there was a delay, and then the program could begin recieving new data as soon as the user was ready to press the remote.
```
#pragma vector = PORT2_VECTOR			// This is from the MSP430G2553.h file

__interrupt void pinChange (void) {

	int8	pin;
	int16	pulseDuration;			// The timer is 16-bits

	if (IR_PIN)		pin=1;	else pin=0;

	switch (pin) {					// read the current pin level
		case 0:						// !!!!!!!!!NEGATIVE EDGE!!!!!!!!!!
			pulseDuration = TAR;
			TACTL = 0;
			packetData[packetIndex++] = pulseDuration;
			LOW_2_HIGH; 				// Setup pin interrupr on positive edge
			break;

		case 1:							// !!!!!!!!POSITIVE EDGE!!!!!!!!!!!
			TAR = 0x0000;				// time measurements are based at time 0
			TA0CCR0 = 0xFFFF;			//set up time delay between when new data will be recieved
			TACTL &= ~TAIFG;
			TACTL = ID_3 | TASSEL_2 | MC_1 | TAIE;
			HIGH_2_LOW; 						// Setup pin interrupr on positive edge
			break;
	} // end switch

	P2IFG &= ~BIT6;			// Clear the interrupt flag to prevent immediate ISR re-entry

} // end pinChange ISR
```
####Documentation
C2C Yarbourough explained that to sum an array I could or the ones and not and the zeros
