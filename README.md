Lab5
====

Reverse Engineering a Remote Control

How long will it take for the timer to rollover?
The clock is 8 MHz TACCR0 goes to 0xFFFF, and the clock divider is 3 so 65.5 ms.

How long does each timer count last?
Each timer count lasts 1 us.

Annotate picture
![alt tag](https://raw.githubusercontent.com/seanbapty/Lab5/master/timerpicuture.JPG)

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


|                   | Data 1 Logic 1 | Data 0 logic 0 | Data 0 Logic 1 |
|-------------------|----------------|----------------|----------------|
|                   | 1.55           | 0.6            | 0.45           |
|                   | 1.5            | 0.6            | 0.45           |
|                   | 1.55           | 0.6            | 0.45           |
|                   | 1.55           | 0.6            | 0.5            |
|                   | 1.55           | 0.55           | 0.45           |
|                   | 1.5            | 0.6            | 0.5            |
|                   | 1.55           | 0.55           | 0.5            |
|                   | 1.55           | 0.55           | 0.5            |
|                   |                |                |                |
| mean              | 1.5375         | 0.58125        | 0.475          |
| sd                | 0.023145502    | 0.025877458    | 0.026726124    |
| timer range lower | 1.421772488    | 0.451862708    | 0.341369379    |
| timer range upper | 1.653227512    | 0.710637292    | 0.608630621    |


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
