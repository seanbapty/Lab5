Lab5
====

Reverse Engineering a Remote Control

How long will it take for the timer to rollover?
The clock is 8 MHz TACCR0 goes to 0xFFFF, and the clock divider is 3 so 65.5 ms.

How long does each timer count last?
Each timer count lasts 1 us.

Annotate picture
![alt tag](https://raw.githubusercontent.com/seanbapty/Lab5/master/timerpicuture.JPG)

|      | Data 1 Logic 1 | Data 0 logic 0 | Data 0 Logic 1 |
|------|----------------|----------------|----------------|
|      | 1.55           | 0.6            | 0.45           |
|      | 1.5            | 0.6            | 0.45           |
|      | 1.55           | 0.6            | 0.45           |
|      | 1.55           | 0.6            | 0.5            |
|      | 1.55           | 0.55           | 0.45           |
|      | 1.5            | 0.6            | 0.5            |
|      | 1.55           | 0.55           | 0.5            |
|      | 1.55           | 0.55           | 0.5            |
|      |                |                |                |
| mean | 1.5375         | 0.58125        | 0.475          |
| sd   | 0.023145502    | 0.025877458    | 0.026726124    |
