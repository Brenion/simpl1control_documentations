| Function Block               | Parameter                            | Data type | Read/Write             | Parameter settings in LOGO!Soft Comfort | Parameter settings on a partner device |
| :--------------------------- | :----------------------------------- | :-------- | :--------------------- | :-------------------------------------- | :------------------------------------- |
| On-Delay                     | Current Time                         | VW        | R                      |                                         |                                        |
| On-Delay                     | VW                                   | R/W       |                        | Unit: Seconds                           |                                        |
|                              |                                      |           | Value range: 0 to 9999 |                                         |                                        |
|                              |                                      |           | Unit: Minutes or Hours |                                         |                                        |
|                              |                                      |           | Value range: 0 to 5999 |                                         |                                        |
| On-Delay                     | Remaining Time                       | VW        | R                      |                                         |                                        |
| On-Delay                     | Time Base                            | VB        | R/W                    | 10 milliseconds                         |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Off-Delay                    | Current Time                         | VW        | R                      |                                         |                                        |
| Off-Delay                    | VW                                   | R/W       | Unit: Seconds          |                                         |                                        |
|                              |                                      |           | Value range: 0 to 9999 |                                         |                                        |
|                              |                                      |           | Unit: Minutes or Hours |                                         |                                        |
|                              |                                      |           | Value range: 0 to 5999 |                                         |                                        |
| Off-Delay                    | Remaining Time                       | VW        | R                      |                                         |                                        |
| Off-Delay                    | Time Base                            | VB        | R/W                    | 10 milliseconds                         |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| On-/Off-Delay                | Current Time                         | VW        | R                      |                                         |                                        |
| On-/Off-Delay                | On Time (TH)                         | VW        | R/W                    | Unit: Seconds                           |                                        |
|                              |                                      |           |                        | Value range: 0 to 9999                  |                                        |
|                              |                                      |           |                        | Unit: Minutes or Hours                  |                                        |
|                              |                                      |           |                        | Value range: 0 to 5999                  |                                        |
| On-/Off-Delay                | Off Time (TL)                        | VW        | R/W                    | Unit: Seconds                           |                                        |
|                              |                                      |           |                        | Value range: 0 to 9999                  |                                        |
|                              |                                      |           |                        | Unit: Minutes or Hours                  |                                        |
|                              |                                      |           |                        | Value range: 0 to 5999                  |                                        |
| On-/Off-Delay                | On Time (TH) Remaining Time          | VW        | R                      |                                         |                                        |
| On-/Off-Delay                | Off Time (TL) Remaining Time         | VW        | R                      |                                         |                                        |
| On-/Off-Delay                | On Time (TH) Time Base               | VB        | R/W                    | 10 milliseconds                         |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| On-/Off-Delay                | Off Time (TL) Time Base              | VB        | R/W                    | 10 milliseconds                         |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| On-/Off-Delay                | Current Time Base                    | VB        | R/W                    | 10 milliseconds                         |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Retentive On-Delay           | Current Time                         | VW        | R                      |                                         |                                        |
| Retentive On-Delay           | VW                                   | R/W       | Unit: Seconds          |                                         |                                        |
|                              |                                      |           | Value range: 0 to 9999 |                                         |                                        |
|                              |                                      |           | Unit: Minutes or Hours |                                         |                                        |
|                              |                                      |           | Value range: 0 to 5999 |                                         |                                        |
| Retentive On-Delay           | Remaining Time                       | VW        | R                      |                                         |                                        |
| Retentive On-Delay           | Time Base                            | VB        | R/W                    | 10 milliseconds                         |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Edge Triggered Wiping Relay  | Current Time                         | VW        | R                      |                                         |                                        |
| Edge Triggered Wiping Relay  | Pulse Width (TH)                     | VW        | R/W                    | Seconds or Minutes                      |                                        |
|                              |                                      |           |                        | 0–9999                                  |                                        |
| Edge Triggered Wiping Relay  | Interpulse Width (TL)                | VW        | R/W                    | Seconds or Minutes                      |                                        |
|                              |                                      |           |                        | 0–9999                                  |                                        |
| Edge Triggered Wiping Relay  | Pulse Width (TH) Remaining Time      | VW        | R                      |                                         |                                        |
| Edge Triggered Wiping Relay  | Interpulse Width (TL) Remaining Time | VW        | R                      |                                         |                                        |
| Edge Triggered Wiping Relay  | Pulse Width (TH) Time Base           | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Edge Triggered Wiping Relay  | Interpulse Width (TL) Time Base      | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Edge Triggered Wiping Relay  | Current Time Base                    | VB        | R                      | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Asynchronous Pulse Generator | Current Time                         | VW        | R                      |                                         |                                        |
| Asynchronous Pulse Generator | Pulse Width                          | VW        | R/W                    | Seconds or Minutes                      |                                        |
|                              |                                      |           |                        | 0–9999                                  |                                        |
| Asynchronous Pulse Generator | Interpulse Width                     | VW        | R/W                    | Seconds or Minutes                      |                                        |
|                              |                                      |           |                        | 0–9999                                  |                                        |
| Asynchronous Pulse Generator | Pulse Remaining Time                 | VW        | R                      |                                         |                                        |
| Asynchronous Pulse Generator | Interpulse Remaining Time            | VW        | R                      |                                         |                                        |
| Asynchronous Pulse Generator | Pulse Width (TH) Time Base           | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Asynchronous Pulse Generator | Interpulse Width (TL) Time Base      | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Asynchronous Pulse Generator | Current Time Base                    | VB        | R                      | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Random Generator             | Current Time                         | VW        | R                      |                                         |                                        |
| Random Generator             | Max. On Delay (TH)                   | VW        | R/W                    | Seconds or Minutes                      |                                        |
|                              |                                      |           |                        | 0–9999                                  |                                        |
| Random Generator             | Max. Off Delay (TL)                  | VW        | R/W                    | Seconds or Minutes                      |                                        |
|                              |                                      |           |                        | 0–9999                                  |                                        |
| Random Generator             | Max. On Delay (TH) Remaining Time    | VW        | R                      |                                         |                                        |
| Random Generator             | Max. Off Delay (TL) Remaining Time   | VW        | R                      |                                         |                                        |
| Random Generator             | Max. On Delay (TH) Time Base         | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Random Generator             | Max. Off Delay (TL) Time Base        | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Random Generator             | Current Time Base                    | VB        | R                      | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Stairway Lighting Switch     | Current Time                         | VW        | R                      |                                         |                                        |
| Stairway Lighting Switch     | Off Delay                            | VW        | R/W                    | Seconds or Minutes                      |                                        |
|                              |                                      |           |                        | 0–9999                                  |                                        |
| Stairway Lighting Switch     | Pre-Warning Time (T!)                | VW        | R                      |                                         |                                        |
| Stairway Lighting Switch     | Pre-Warning Period (T!L)             | VW        | R                      |                                         |                                        |
| Stairway Lighting Switch     | Off Delay Remaining                  | VW        | R                      |                                         |                                        |
| Stairway Lighting Switch     | Pre-Warning Time (T!) Remaining      | VW        | R                      |                                         |                                        |
| Stairway Lighting Switch     | Pre-Warning Period (T!L) Remaining   | VW        | R                      |                                         |                                        |
| Stairway Lighting Switch     | Off Delay Time Base                  | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Multiple Function Switch     | Current Time                         | VW        | R                      |                                         |                                        |
| Multiple Function Switch     | Off Delay Time (T)                   | VW        | R/W                    | Seconds or Minutes                      |                                        |
|                              |                                      |           |                        | 0–9999                                  |                                        |
| Multiple Function Switch     | Permanent Light (TL)                 | VW        | R/W                    | Seconds or Minutes                      |                                        |
|                              |                                      |           |                        | 0–9999                                  |                                        |
| Multiple Function Switch     | Pre-Warning Time (T!)                | VW        | R                      |                                         |                                        |
| Multiple Function Switch     | Pre-Warning Period (T!L)             | VW        | R                      |                                         |                                        |
| Multiple Function Switch     | Off Delay Time (T) Remaining         | VW        | R                      |                                         |                                        |
| Multiple Function Switch     | Permanent Light (TL) Remaining       | VW        | R                      |                                         |                                        |
| Multiple Function Switch     | Pre-Warning Time (T!) Remaining      | VW        | R                      |                                         |                                        |
| Multiple Function Switch     | Pre-Warning Period (T!L) Remaining   | VW        | R                      |                                         |                                        |
| Multiple Function Switch     | Off Delay Time (T) Time Base         | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Multiple Function Switch     | Permanent Light (TL) Time Base       | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Multiple Function Switch     | Current Time Base                    | VB        | R                      | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Weekly Timer                 | Week Day 1                           | VB        | R/W                    | Sunday = Bit 0                          | If bit = 1 → day is active             |
|                              |                                      |           |                        | Monday = Bit 1                          |                                        |
|                              |                                      |           |                        | ...                                     |                                        |
|                              |                                      |           |                        | Saturday = Bit 6                        |                                        |
| Weekly Timer                 | On Time 1                            | VW        | R/W                    | h:m                                     | h:m                                    |
| Weekly Timer                 | Off Time 1                           | VW        | R/W                    | h:m                                     | h:m                                    |
| Weekly Timer                 | Week Day 2                           | VB        | R/W                    | Sunday = Bit 0                          | If bit = 1 → day is active             |
|                              |                                      |           |                        | ...                                     |                                        |
|                              |                                      |           |                        | Saturday = Bit 6                        |                                        |
| Weekly Timer                 | On Time 2                            | VW        | R/W                    | h:m                                     | h:m                                    |
| Weekly Timer                 | Off Time 2                           | VW        | R/W                    | h:m                                     | h:m                                    |
| Weekly Timer                 | Week Day 3                           | VB        | R/W                    | Sunday = Bit 0                          | If bit = 1 → day is active             |
|                              |                                      |           |                        | ...                                     |                                        |
|                              |                                      |           |                        | Saturday = Bit 6                        |                                        |
| Weekly Timer                 | On Time 3                            | VW        | R/W                    | h:m                                     | h:m                                    |
| Weekly Timer                 | Off Time 3                           | VW        | R/W                    | h:m                                     | h:m                                    |
| Weekly Timer                 | Pulsely                              | VB        | R/W                    | Off = 0                                 |                                        |
|                              |                                      |           |                        | On = 1                                  |                                        |
| Yearly Timer                 | On Time                              | VW        | R/W                    | Month:Day                               | Month:Day                              |
| Yearly Timer                 | Off Time                             | VW        | R/W                    | Month:Day                               | Month:Day                              |
| Yearly Timer                 | On Year                              | VB        | R/W                    | Year                                    | Year                                   |
| Yearly Timer                 | Off Year                             | VB        | R/W                    | Year                                    | Year                                   |
| Yearly Timer                 | Monthly                              | VB        | R/W                    | 0 = No                                  |                                        |
|                              |                                      |           |                        | 1 = Yes                                 |                                        |
| Yearly Timer                 | Yearly                               | VB        | R/W                    | 0 = No                                  |                                        |
|                              |                                      |           |                        | 1 = Yes                                 |                                        |
| Yearly Timer                 | Pulsely                              | VB        | R/W                    | 0 = Off                                 |                                        |
|                              |                                      |           |                        | 1 = On                                  |                                        |
| Astronomical Clock           | Longitude                            | VD        | R/W                    | VBx+0: W=1 / E=0                        |                                        |
|                              |                                      |           |                        | ° ' "                                   |                                        |
| Astronomical Clock           | Latitude                             | VD        | R/W                    | VBx+0: S=1 / N=0                        |                                        |
|                              |                                      |           |                        | ° ' "                                   |                                        |
| Astronomical Clock           | Time Zero (E+; W-)                   | VW        | R/W                    | -11 to 12                               |                                        |
|                              |                                      |           |                        | High bit = sign                         |                                        |
| Astronomical Clock           | SunRise Time                         | VW        | R                      | h:m                                     |                                        |
| Astronomical Clock           | SunSet Time                          | VW        | R                      | h:m                                     |                                        |
| Stop Watch                   | Time Base                            | VB        | R/W                    | 0 = 10 ms                               |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Stop Watch                   | Current Time                         | VD        | R                      |                                         |                                        |
| Stop Watch                   | Lap Time                             | VD        | R                      |                                         |                                        |
| Stop Watch                   | Output Time                          | VW        | R                      |                                         |                                        |
| Up/Down Counter              | Counter                              | VD        | R/W                    | 0 to 999999                             |                                        |
| Up/Down Counter              | On Threshold                         | VD        | R/W                    | 0 to 999999                             |                                        |
| Up/Down Counter              | Off Threshold                        | VD        | R/W                    | 0 to 999999                             |                                        |
| Up/Down Counter              | Start Value                          | VD        | R/W                    | 0 to 999999                             |                                        |
| Hours Counter                | Maintenance Interval (MI)            | VD        | R/W                    | 0 to 599999 (9999H 59M)                 |                                        |
| Hours Counter                | Time-to-Go (MN)                      | VD        | R                      |                                         |                                        |
| Hours Counter                | Total Time (OT)                      | VD        | R                      |                                         |                                        |
| Threshold Trigger            | Frequency                            | VW        | R                      |                                         |                                        |
| Threshold Trigger            | On Threshold                         | VW        | R/W                    | 0 to 9999                               |                                        |
| Threshold Trigger            | Off Threshold                        | VW        | R/W                    | 0 to 9999                               |                                        |
| Threshold Trigger            | Gate Time                            | VW        | R                      |                                         |                                        |
| Analog Threshold Trigger     | On                                   | VW        | R/W                    | -20000 to 20000                         |                                        |
| Analog Threshold Trigger     | Off                                  | VW        | R/W                    | -20000 to 20000                         |                                        |
| Analog Threshold Trigger     | Gain                                 | VW        | R/W                    |                                         |                                        |
| Analog Threshold Trigger     | Offset                               | VW        | R/W                    |                                         |                                        |
| Analog Threshold Trigger     | Ax, Amplified                        | VW        | R                      |                                         |                                        |
| Analog Differential Trigger  | On                                   | VW        | R/W                    | -20000 to 20000                         |                                        |
| Analog Differential Trigger  | Differential                         | VW        | R/W                    | -20000 to 20000                         |                                        |
| Analog Differential Trigger  | Gain                                 | VW        | R/W                    |                                         |                                        |
| Analog Differential Trigger  | Offset                               | VW        | R/W                    |                                         |                                        |
| Analog Differential Trigger  | Ax, Amplified                        | VW        | R                      |                                         |                                        |
| Analog Differential Trigger  | Off                                  | VW        | R                      |                                         |                                        |
| Analog Comparator            | On                                   | VW        | R/W                    | -20000 to 20000                         |                                        |
| Analog Comparator            | Off                                  | VW        | R/W                    | -20000 to 20000                         |                                        |
| Analog Comparator            | Gain                                 | VW        | R/W                    |                                         |                                        |
| Analog Comparator            | Offset                               | VW        | R/W                    |                                         |                                        |
| Analog Comparator            | Ax, Amplified                        | VW        | R                      |                                         |                                        |
| Analog Comparator            | Ay, Amplified                        | VW        | R                      |                                         |                                        |
| Analog Comparator            | Ax - Ay                              | VW        | R                      |                                         |                                        |
| Analog Watchdog              | Gain                                 | VW        | R/W                    |                                         |                                        |
| Analog Watchdog              | Offset                               | VW        | R/W                    |                                         |                                        |
| Analog Watchdog              | Aen (Comparison Value)               | VW        | R                      |                                         |                                        |
| Analog Watchdog              | Ax, Amplified                        | VW        | R                      |                                         |                                        |
| Analog Watchdog              | Differential (+)                     | VW        | R/W                    | 0 to 20000                              |                                        |
| Analog Watchdog              | Differential (-)                     | VW        | R/W                    | 0 to 20000                              |                                        |
| Analog Amplifier             | Gain                                 | VW        | R/W                    | -1000 to 1000                           |                                        |
| Analog Amplifier             | Offset                               | VW        | R/W                    | -10000 to 10000                         |                                        |
| Analog Amplifier             | Ax, Amplified                        | VW        | R                      |                                         |                                        |
| Analog Multiplexer           | AQ Amplified                         | VW        | R                      |                                         |                                        |
| Analog Multiplexer           | V1 (S1=0; S2=0)                      | VW        | R/W                    | -32768 to 32767                         |                                        |
| Analog Multiplexer           | V2 (S1=0; S2=1)                      | VW        | R/W                    | -32768 to 32767                         |                                        |
| Analog Multiplexer           | V3 (S1=1; S2=0)                      | VW        | R/W                    | -32768 to 32767                         |                                        |
| Analog Multiplexer           | V4 (S1=1; S2=1)                      | VW        | R/W                    | -32768 to 32767                         |                                        |
| PWM                          | Min.                                 | VW        | R/W                    | -10000 to 20000                         |                                        |
| PWM                          | Max.                                 | VW        | R/W                    | -10000 to 20000                         |                                        |
| PWM                          | Gain                                 | VW        | R/W                    | -1000 to 1000                           |                                        |
| PWM                          | Offset                               | VW        | R/W                    | -10000 to 10000                         |                                        |
| PWM                          | Ax, Amplified (Current Period)       | VW        | R                      |                                         |                                        |
| PWM                          | T                                    | VW        | R/W                    | Seconds: 0–9999                         |                                        |
|                              |                                      |           |                        | Minutes/Hours: 0–5999                   |                                        |
| PWM                          | Periodic Time Base                   | VB        | R/W                    | 10 ms                                   |                                        |
|                              |                                      |           |                        | 1 = Seconds                             |                                        |
|                              |                                      |           |                        | 2 = Minutes                             |                                        |
|                              |                                      |           |                        | 3 = Hours                               |                                        |
| Mathematic Instructions      | AQ Amplified                         | VW        | R                      |                                         |                                        |
| Mathematic Instructions      | V1                                   | VW        | R/W                    | -32768 to 32767                         |                                        |
| Mathematic Instructions      | V2                                   | VW        | R/W                    | -32768 to 32767                         |                                        |
| Mathematic Instructions      | V3                                   | VW        | R/W                    | -32768 to 32767                         |                                        |
| Mathematic Instructions      | V4                                   | VW        | R/W                    | -32768 to 32767                         |                                        |
| Mathematic Instructions      | Operator 1                           | VB        | R/W                    | 0 = +                                   |                                        |
|                              |                                      |           |                        | 1 = -                                   |                                        |
|                              |                                      |           |                        | 2 = *                                   |                                        |
|                              |                                      |           |                        | 3 = /                                   |                                        |
| Mathematic Instructions      | Operator 2                           | VB        | R/W                    | 0 = +                                   |                                        |
|                              |                                      |           |                        | 1 = -                                   |                                        |
|                              |                                      |           |                        | 2 = *                                   |                                        |
|                              |                                      |           |                        | 3 = /                                   |                                        |
| Mathematic Instructions      | Operator 3                           | VB        | R/W                    | 0 = +                                   |                                        |
|                              |                                      |           |                        | 1 = -                                   |                                        |
|                              |                                      |           |                        | 2 = *                                   |                                        |
|                              |                                      |           |                        | 3 = /                                   |                                        |
| Mathematic Instructions      | Priority1                            | VB        | R/W                    | 0 = L                                   |                                        |
|                              |                                      |           |                        | 1 = M                                   |                                        |
|                              |                                      |           |                        | 2 = H                                   |                                        |
| Mathematic Instructions      | Priority2                            | VB        | R/W                    | 0 = L                                   |                                        |
|                              |                                      |           |                        | 1 = M                                   |                                        |
|                              |                                      |           |                        | 2 = H                                   |                                        |
| Mathematic Instructions      | Priority3                            | VB        | R/W                    | 0 = L                                   |                                        |
|                              |                                      |           |                        | 1 = M                                   |                                        |
|                              |                                      |           |                        | 2 = H                                   |                                        |
| Mathematic Instructions      | Reset Mode                           | VB        | R/W                    | 0 = Reset to zero                       |                                        |
|                              |                                      |           |                        | 1 = Keep last value                     |                                        |
| Analog Ramp                  | Gain                                 | VW        | R/W                    |                                         |                                        |
| Analog Ramp                  | Offset                               | VW        | R/W                    |                                         |                                        |
| Analog Ramp                  | Current Level                        | VW        | R                      |                                         |                                        |
| Analog Ramp                  | Level 1 (L1)                         | VW        | R/W                    | -10000 to 20000                         |                                        |
| Analog Ramp                  | Level 2 (L2)                         | VW        | R/W                    | -10000 to 20000                         |                                        |
| Analog Ramp                  | Largest Output Value                 | VW        | R                      |                                         |                                        |
| Analog Ramp                  | Start/Stop Offset                    | VW        | R/W                    | 0 to 20000                              |                                        |
| Analog Ramp                  | Speed of Change                      | VW        | R/W                    | 1 to 10000                              |                                        |
| PI Controller                | Set Value (SP)                       | VW        | R/W                    | -10000 to 20000                         |                                        |
| PI Controller                | PV, Amplified                        | VW        | R                      |                                         |                                        |
| PI Controller                | Aq                                   | VW        | R                      |                                         |                                        |
| PI Controller                | Kc                                   | VW        | R/W                    | 0 to 9999                               |                                        |
| PI Controller                | Integration Time (TI)                | VW        | R/W                    | Unit: Minutes                           |                                        |
|                              |                                      |           |                        | 0 to 5999                               |                                        |
| PI Controller                | Direction                            | VB        | R/W                    | 0 = +                                   |                                        |
|                              |                                      |           |                        | 1 = -                                   |                                        |
| PI Controller                | Manual Output (Mq)                   | VW        | R/W                    | 0 to 1000                               |                                        |
| PI Controller                | min                                  | VW        | R/W                    | -10000 to 20000                         |                                        |
| PI Controller                | max                                  | VW        | R/W                    | -10000 to 20000                         |                                        |
| PI Controller                | Gain                                 | VW        | R/W                    | -1000 to 1000                           |                                        |
| PI Controller                | Offset                               | VW        | R/W                    | -10000 to 10000                         |                                        |
| Analog Filter                | Dialog Parameter Avg. Sample Number  | VB        | R/W                    | 3 to 8                                  |                                        |
|                              |                                      |           |                        | (8=256 samples)                         |                                        |
| Analog Filter                | Ax                                   | VW        | R                      |                                         |                                        |
| Analog Filter                | Aq                                   | VW        | R                      |                                         |                                        |
| Max/Min                      | Mode                                 | VB        | R/W                    | 0, 1, 2 (modes)                         |                                        |
| Max/Min                      | Ax                                   | VW        | R                      |                                         |                                        |
| Max/Min                      | Minimum Value                        | VW        | R                      |                                         |                                        |
| Max/Min                      | Maximum Value                        | VW        | R                      |                                         |                                        |
| Max/Min                      | Aq                                   | VW        | R                      |                                         |                                        |
| Max/Min                      | Reset                                | VB        | R/W                    | 0 = no reset                            |                                        |
|                              |                                      |           |                        | 1 = reset min/max                       |                                        |