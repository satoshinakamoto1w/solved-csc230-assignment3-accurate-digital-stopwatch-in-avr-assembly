Download Link: https://assignmentchef.com/product/solved-csc230-assignment3-accurate-digital-stopwatch-in-avr-assembly
<br>
This assignment is the culmination of the assembly programming part of the course. You will implement an accurate digital stopwatch in AVR assembly, using a built-in timer and the LCD screen to display the time. As with assignment 2, you should start early, since this assignment requires you to apply virtually all of the techniques that the course has covered for embedded programming, and debugging assembly code is not easy.

The core of this assignment is a timer display which uses the LCD screen to display the number of minutes, seconds and tenths of a second since timing started. Implementing this aspect alone (with no user input functionality or other stopwatch-like features) is sufficient to obtain 13 out of 18 possible marks (and potentially up to 14 marks if you can justify that the timing is highly accurate). The remaining marks are obtained by implementing user input functionality to pause and reset the timer, along with the ability to compute lap (split) times.

Section 2 describes the basic timing functionality needed. Section 3 describes the user input functionality required for the basic stopwatch and Section 4 describes the functionality needed for the fully-featured stopwatch (which includes a lap timer)

<h1>1              Timer Display</h1>

The LCD screen has two lines, each with 16 columns, numbered from 0 to 15. The basic timer required will display the time on the LCD screen in the format below.

0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

Time: MM:SS.T

Where MM and SS are the two digit (base 10) counts of elapsed minutes and seconds (respectively) and T is the number of tenths of a second that have elapsed. If the text displayed, or its alignment (that is, the exact column numbers in which it appears), differs from the above, it will be considered incorrect.

When the timer is running (see below), the display will be continuously updated to reflect the current time in minutes, seconds and tenths of a second. Note that the value of T must always be in the range 0 through 9 (inclusive), the value SS must always be a two-digit base-10 number in the range 0 through 59 (inclusive) and the value MM must always be a two-digit base-10 number in the range 0 through 99 (inclusive). There is no need to add support for the timer to run beyond 100 minutes.

At the RESET signal, the timer will be cleared to zero and display

0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

Time: 00:00.0

If no user input features are implemented, the behavior of the program will be as follows. Note that this is <strong>not </strong>the expected behavior of the program once user input is implemented (for example, if you have correctly implement the start/stop button, the timer must not automatically start when the program starts). See the next section for details on the expected behavior when user inputs are supported.

<ul>

 <li>When the program starts (that is, at the RESET signal), the screen will be initialized to display the interface above, with the time initialized to 00:00.0.</li>

 <li>The program will <em>immediately </em>start the timer, which will then be updated in real time as time elapses. Since no user input is implemented, there will be no facility to start or stop timing, so the timer will continue running indefinitely.</li>

 <li>The timer is expected to be theoretically accurate to within 1 second every ten minutes (under the assumption that the system clock is precisely 16Mhz). There is no need to handle cases where the timer runs for longer than 100 minutes (so no more than two digits will ever be needed for the minute value).</li>

</ul>

<h1>2              Basic Inputs</h1>

A basic stopwatch normally has two input features: a button to start or stop the timer (which can also be used to pause the timer) and a button to reset the clock to zero. For this functionality, we will use the SELECT and LEFT buttons. The table below gives a formal description of the expected behavior of each button. These buttons affect the primary time displayed on the top line of the LCD screen (as shown in the previous section).

<table width="561">

 <tbody>

  <tr>

   <td width="68"><strong>Button</strong></td>

   <td width="493"><strong>Behavior</strong></td>

  </tr>

  <tr>

   <td width="68">SELECT</td>

   <td width="493"><strong>Start/stop timer</strong>: When support for this button has been implemented, the program will start (at the RESET signal) with the timer stopped (at 00:00.0). Pressing this button when the timer is stopped will start the timer (within 0<em>.</em>1 seconds of the button press) and pressing the button when the timer is running will stop the timer <strong>immediately</strong>. It is considered incorrect behavior for the timer to continue to tick even once after the button has been pressed to stop timing. When the timer is stopped, the button may be pressed again to restart it (without resetting the time to zero), so this button can be used as a ‘pause’ feature.</td>

  </tr>

  <tr>

   <td width="68">LEFT</td>

   <td width="493"><strong>Clear timer</strong>: When this button is pressed, the timer will immediately be cleared to 00:00.0. In cases where the start/stop button has also been implemented, the cleared timer must also enter a stopped state (such that the start/stop button can be used to start the timer). If the start/stop button has not been implemented, then the cleared timer will not be stopped (but continue counting starting from 00:00.0). In cases where lap timing support has been implemented (see Section 4), this button must <strong>not </strong>clear the displayed lap time or disable the display of lap times. However, it should clear the CURRENT_LAP_START variable.</td>

  </tr>

 </tbody>

</table>

Notice that the SELECT button is used to both start and stop the timer. Therefore, you will need logic similar to that used for the PAUSE feature on Assignment 2 to ensure that a single button press is not read as repeated presses.

<h1>3              Lap Timing</h1>

Digital stopwatches often support a “lap timer” (or “split timer”) which allows the timing of subintervals with respect to the running stopwatch. For this assignment, we will use a very simple lap timing mechanism: when a lap time is requested, the program will display the ‘start time’ of the lap (which will either be 00:00.0 or, if the lap timer has been used before, the time at which the last lap ended) and the ‘end time’ of the lap (which will be a copy of the current time on the timer). After a lap time has been captured, the timer will continue running but the displayed lap time will remain static until another lap time is requested.

Tracking lap times will require keeping three extra time variables, LAST_LAP_START, LAST_LAP_END and CURRENT_LAP_START. The LAST_LAP_START and LAST_LAP_END variables track the start and end times of the most recent lap which was captured with the UP button. The CURRENT_LAP_START tracks the beginning of the ‘current’ lap (which will become the lap start time when the UP button is next pressed). When the timer is cleared, CURRENT_LAP_START is set to 00:00.0.

When the RESET signal occurs, the lower line of the LCD (row 1) is cleared. Lap times are only displayed after the UP button is pressed. When the UP button is pressed, the LAST_LAP_START and LAST_LAP_END times will be displayed on the bottom row of the LCD, with the start time shown on the absolute left and the end time shown on the absolute right, in the following format.

0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

Time: MM:SS.T

MM:SS.T                                                                                               MM:SS.T

When lap timing is implemented, the UP and DOWN buttons will have the following behavior.

<table width="561">

 <tbody>

  <tr>

   <td width="68"><strong>Button</strong></td>

   <td width="493"><strong>Behavior</strong></td>

  </tr>

  <tr>

   <td width="68">UP</td>

   <td width="493"><strong>Set Lap</strong>: When this button is pressed, the LAST_LAP_START time is set to the CURRENT_LAP_START time, the CURRENT_LAP_START and LAST_LAP_END times are <strong>both </strong>set to the current stopwatch time, and the display of theLAST_LAP_START and LAST_LAP_END time on the bottom row of the LCD (using the format above) is enabled. The lap times will continue to be displayed until the UP button is pressed again (which will change the displayed values), the RESET signal occurs (which will restart the program and disable the display of lap times) or the DOWN button is pressed (which will disable the display of lap times). The UP button should exhibit this behavior regardless of whether the main timer is currently running or paused.</td>

  </tr>

  <tr>

   <td width="68">DOWN</td>

   <td width="493"><strong>Clear Lap</strong>: When this button is pressed, the CURRENT_LAP_START variable will be set to 00:00.0 and the display of lap times will be disabled (so row 1 will be blank) until the UP button is pressed again.</td>

  </tr>

 </tbody>

</table>

<h1>4              Implementation Suggestions</h1>

The sections below give some advice on how to implement various aspects of the timing program. You are free to ignore the advice if you have a different idea.

<strong>5.1       Storing Times</strong>

The stopwatch is expected to display times in the format MM:SS.T and to keep time with 1<em>/</em>10 second precision. It is possible to store the entire time as a single integer value equal to the number of tenths of a second elapsed, and doing so makes the process of incrementing the time comparatively easy (since it becomes a wide addition operation). However, since the time must be continuously converted to a string to be displayed on the LCD, this representation may be inconvenient, since converting a large number to a series of digits requires the implementation of division and modulus operations (as in assignment 1).

Instead, consider representing the 5-digit time value by using an array of 5 bytes, with one byte per digit. Converting such a representation to a string is relatively straightforward (since one-digit values can be converted to strings by a simple addition) and the increment operation (which advances the time by a tenth of a second) can be achieved using an algorithm like the one below (where the 5 byte array is called A and it is assumed that the five digits in the MM:SS.T representation are represented by indices 0 − 4 (with 0 containing the first M digit and 4 containing the T digit).

//Add one to the MM:SS.T representation void increment_time(A){

//Make an array containing the maximum value allowed for each

//digit in the representation. Note that the in the MM:SS.T

//representation, the T value can be between 0 and 9, the

//SS value can be between 00 and 59, and the MM value can //be between 00 and 99.

//(There is no need to “create” this array every time the function

// is run, since it can be placed in program memory in advance)

//

//                                                                                           M M S S T

unsigned char maximum_values[5] = {9, 9, 5, 9, 9};

unsigned char i;

//Add one to the last index (T)

A[4] += 1

//Now work backwards if any digit exceeded the limits //in the array above. for (i = 4; i &gt;= 0; i–){

if (A[i] &gt; maximum_values[i]){

A[i] = 0;

A[i-1] += 1;

}

}

//If A[0] exceeded 9, then wrap around to 0.

if (A[0] &gt; maximum_values[0]) A[0] = 0;

}

Similarly, functions can be used to copy time values (for use with saving lap times) or to set a time array to zero.

<strong>5.2         Mitigating Button Aliasing</strong>

As we noticed during the completion of assignment 2, constantly polling the buttons to detect button presses can result in unusual behavior. In particular, when buttons with low voltages (such as RIGHT) are pressed, polling the buttons might result in the voltage for a different button being read. As a result, you may have experienced “button aliasing” during assignment 2, where pressing one button gives the behavior for a different button. For this assignment, it is expected that your code will proactively prevent button aliasing to the greatest extent possible (although it may be impossible to prevent the aliasing in all cases). To reduce the impact of aliasing, consider modifying your button polling logic to use one of the following techniques (or a different technique of your own design):

<ul>

 <li>Add a short delay between consecutive polls to reduce the probability that an erroneous transient value will be read.</li>

 <li>Instead of polling ‘constantly’, poll the buttons at regular intervals (e.g. 10 times per second). As long as the buttons are polled more often than the timer ticks, there will be no distinguishable difference in behavior if the frequency of polling is reduced.</li>

 <li>Use a function to poll the ADC and return a number in the range [0<em>,</em>5] based on which button was detected. Instead of simply calling the function to poll the buttons, call the function twice (or three times) and only acknowledge the button press if the return value was the same for all calls.</li>

</ul>

<h1>5              Sample Run Illustration</h1>

The table below shows the state of the LCD screen before and after various inputs during a sample run of the program. Obviously, your code should work on all possible input sequences, not just the ones below.

<strong>Event                                            LCD (before)                                                   LCD (after)</strong>

<table width="623">

 <tbody>

  <tr>

   <td width="154">Program starts</td>

   <td width="235">N/A</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">10 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">SELECT pressed(Timer Starts)</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">0.1 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.1</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">15 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.1</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:15.1</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">65.5 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:15.1</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 01:20.6</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">LEFT pressed(Timer Clears)</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 01:20.6</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">5 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">SELECT pressed(Timer Starts)</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">1 second elapses</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:00.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:01.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">15 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:01.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:16.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

 </tbody>

</table>

<strong>Event                                            LCD (before)                                                   LCD (after)</strong>

<table width="623">

 <tbody>

  <tr>

   <td width="154">SELECT pressed(Timer Stops)</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:16.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:16.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">20 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:16.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:16.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">SELECT pressed(Timer Starts)</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:16.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:16.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">1.5 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:16.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:17.5</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">UP pressed(Set Lap)</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:17.5</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:17.500:00.0                               00:17.5</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">4.5 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:17.500:00.0                               00:17.5</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:22.000:00.0                               00:17.5</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">UP pressed(Set Lap)</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:22.000:00.0                               00:17.5</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:22.000:17.5                               00:22.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">3 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:22.000:17.5                               00:22.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:25.000:17.5                               00:22.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">DOWN pressed(Clear Lap)</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:25.000:00.0                               00:17.5</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:25.0</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">4.6 seconds elapse</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:25.0</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:29.6</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

  <tr>

   <td width="154">UP pressed(Set Lap)</td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:29.6</td>

      </tr>

     </tbody>

    </table></td>

   <td width="235">0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

    <table width="187">

     <tbody>

      <tr>

       <td width="187">Time: 00:29.600:00.0                               00:29.6</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

 </tbody>

</table>

<h1>6              Evaluation</h1>

Submit all .asm and .inc files needed to assemble your assignment electronically via conneX (including all of the necessary LCD library files). Your code must assemble, upload and run correctly on the ATmega2560 boards in ECS 249 using the toolchain and methodology described in Lab 3.

If your code does not assemble as submitted, you will be allowed to spend some of your demo time attempting to fix it, but the demo will be limited to 10 minutes, and you will lose marks if you miss parts of the evaluation by spending time fixing your code. If your code cannot be assembled during the demo, you will receive a mark of at most 2.

This assignment is worth 9% of your final grade and will be marked out of 18 during an interactive demo with an instructor. Demos must be scheduled in advance (through an electronic system available on conneX). If you do not schedule a demo time, or if you do not attend your scheduled demo, you will receive a mark of zero.

The marks are distributed among the aspects of the assignment as follows.

<table width="561">

 <tbody>

  <tr>

   <td width="59"><strong>Marks</strong></td>

   <td width="502"><strong>Aspect</strong></td>

  </tr>

  <tr>

   <td width="59">11</td>

   <td width="502">The basic, non-interactive timer described in Section 2 functions correctly with reasonably accurate timing (such that a human observer cannot notice any discrepancy).</td>

  </tr>

  <tr>

   <td width="59">1</td>

   <td width="502">Functions are used as appropriate instead of duplicated code, and all functions exhibit correct register preservation using the stack.</td>

  </tr>

  <tr>

   <td width="59">1</td>

   <td width="502">Multi-byte persistent data (such as the current time) is stored primarily in data memory instead of being held continuously in registers. It is permissible to temporarily use registers to hold values while your code is working with them.</td>

  </tr>

  <tr>

   <td width="59">1</td>

   <td width="502">The timing is theoretically accurate to within 1 second out of 10 minutes when the 16Mhz system clock is assumed to be exact.</td>

  </tr>

  <tr>

   <td width="59">1</td>

   <td width="502">The SELECT button correctly implements the start/stop feature described in Section 3. The code must ensure that a single press of the button be read only once (regardless of how long the button is held down).</td>

  </tr>

  <tr>

   <td width="59">1</td>

   <td width="502">The LEFT button correctly implements the clear feature described in Section 3.</td>

  </tr>

  <tr>

   <td width="59">1</td>

   <td width="502">The UP button correctly implements the ‘set lap’ feature described in Section 4. Note that unless the display of lap times is correct, it is impossible to evaluate this component (and the component below).</td>

  </tr>

  <tr>

   <td width="59">1</td>

   <td width="502">The DOWN button correctly implements the ‘clear lap’ feature described in Section 4.</td>

  </tr>

 </tbody>

</table>