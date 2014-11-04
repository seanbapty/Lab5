Lab5
====

Reverse Engineering a Remote Control

How long will it take for the timer to rollover?
The clock is 8 MHz TACCR0 goes to 0xFFFF, and the clock divider is 3 so 65.5 ms.

How long does each timer count last?
Each timer count lasts 1 us.

Annotate picture
```
TAR = 0;						// reset timer and
			while(IR_DECODER_PIN==0);		// wait while IR is logic 0
			time0[i] = TAR;					// and store timer A
```
