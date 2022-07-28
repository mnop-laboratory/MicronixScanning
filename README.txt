Hello and welcome to the micronix scanning package! This package is desinged to control and create scanning patterns with the micronix 
micropositioning devices. We'll cover the functionality of the scanners along with any known bugs and preferred options. If you have any questions 
please feel free to reach out to either Hayden Binger or Devon Uram at binge063@umn.edu or uram0283@umn.edu.

Upon running the code the user will be prompted with a new subVI window. In that window you will need to accurately put in the com-port that the 
micronix controller is connected to. There will be two other controls, one for the baud rate of the controller which should be initialized to the 
correct value and another for the controller number which is not currently utilized in the code. After the port is connected the code will run through 
a few set up stages and then will enter into the main state machine where you have access to the bulk of the features. 

IMPORTANT NOTES: 
-The micronix scanning package defaults to using millimeters as opposed to micrometers. When using the issue command make sure that
any positions are given in millimeters. For a similar reason most move command and positions are in millimeters, with the exception of the scanning 
parameters which are converted on the backend for ease of use. 
-As this scanning software is designed for 3D controller patterns by default it will ping three axes, known as axis 1, axis 2, and axis 3. The 
scanning patterns on the other hand only perform 2D plane scans. In theory the bulk of the code would work with only two axis with minor changes 
being made to the position subVI, initialize subVI, and the move all subVI. 
-Due to limited blocking within the micronix subVIs, we've implemented various blocking procedures across the code. In general, when you click a 
button the button will light up while the task is being performed. The button will remain lighted up until the task is finished during which other 
buttons cannot be pressed. If while an action is being performed you press another button that button will buffer and execute after the current action 
is being performed. However, this will lock you out of being able to press any other buttons so it is highly discouraged. 
-If the code is ended improperly and the port is not closed then you will have to close all of labVIEW as well as turning off the controller and 
then turning it back on. 

KNOWN BUGS:
- Read position reads a non-numeric value then stops reading the position correctly. 
- Heatmap Z value bar not updating properly during and after a scan. 
- Min value in angled scans.
- Code crashes if executing command to fast on start up. 

List of features and description of features:
=====================================================================================================================================================

Initialization:
First moves all of the axises as far in the negative direction as they can, known as the negative limit or MLN, and then sets that point as zero
(SZP). This has the effect of creating a reproducible reference point between scans and can help with consistency. 

=====================================================================================================================================================

Encoder Position vs. Calculated Position:
At the top of the screen there are two separate indicators, one for enc position and one for calc position. "enc position" is the position as 
given by the encoder while "calc position" is the position expected to be at. In closed loop mode these should be accurate within 50 nm. 

As an important note while the controller is being sent commands directly the position cannot be updated live. Thus, to maximize speed the position 
will not update live while in the scanning patterns as well as when you issue it a command directly. 

=====================================================================================================================================================

Motion Parameters:
To improve the stability of your scans and motion you can manually control the velocity, the acceleration, and the deceleration. 
Maximum Values: 
Velocity (mm/s): 2.56
Acceleration (mm/s^2): 100
Deceleration (mm/s^2): 100
In order to update the values you will need to input the desired values then press the "update parameters" which will change the indicator to match
the control. 

=====================================================================================================================================================

Move Commands: 
There are two separate types of move commands described below:
Absolute movement/ move to:
This will move the motors to the specified position at the speed as set by the motion parameters. This is most useful for covering large distances 
or moving to known spots. You can move each of the motors one at the time to move to the position set by the control next to the button or update 
all three at the same time by using the "move all" button.
Relative movement/ move by:
This will tell the motor to move by a set amount in the positive or negative directions. By inputing the step size and then clicking left or right 
for positive or negative movement one can incrementally move the motors. 

=====================================================================================================================================================

Issue Command:
Using the issue command you can tell the controller to do anything that is in the micronix controller database but is not implemented in the code. 
Select the controller number that you would like to use then type the command into the line beneath it. When you have the syntax correct click the 
issue command button and the command will be given to the controller. 
The general syntax is: (controller number)(three letter code for the command)(optional number if one is required)
Example: 1MLP - Tells axis one to move to the positive limit.
Example: 3MVR-1 - Tells axis three to move 1 mm in the negative direction.
The controller number is appended to the front of the command from the indictor meaning you don't need to add that in the text box. 

=====================================================================================================================================================

Loop type:
There are three loop types available. To switch between them type in the number corresponding to the desired loop type and click update loop type. 
Open Loop: In this mode the controller will not consider the encoder position and move with no feedback. 
Closed Loop: This mode will align the encoder position to match the calculated position. This mode is slower than clean closed loop mode.
Clean Closed Loop: Acts as the closed loop mode except at the the speed of open loop mode. Suggested loop type. 

=====================================================================================================================================================

Limits:
Soft limits are limits that stop the motors from moving past the specified point. If a motor would receive a movement command that would move it
beyond the soft limit instead the motor will ignore that command and not move at all. There are two types of soft limits, a  positive one and a 
negative one. 

IMPORTANT NOTE:
-Setting a new zero point will NOT update the soft limits. For example, if you set +5 as the new zero point and previously had zero as a soft limit, 
the spot you are at will be at the soft limit. This means that it is possible to to move past the soft limits by setting zero points. If this happens,
or if you accidentally set the soft limit spot before your current position, you can still use the absolute movement to get back to the operable range
of motion. However you will NOT be able to use the relative movement even if it brings you back into range. 
-Upon running the code the limits will be set to whatever values are currently in the indicators 
=====================================================================================================================================================

Scanning patterns:
The bulk of the use of the code comes from the scanning patterns. There are many controls described below:
Port number: The acquisition of data is done through a DAQ task. In order to build the task correctly you will need to input both the port the DAQ is 
connected to as well as the channel the signal you care about is on. 
Number of samples: This sets the number of samples that will be averaged over at each scanning spot. 
Rate: This sets the rate at which the samples will be taken from the DAQ. 
Fast Axis: This determines which axis will be used as the fast axis for your scan. The fast axis is the axis in which most of the motion will be done
for the scan. Inputs include "1" "2" or "3" to determine the axis. 
Slow Axis: Similar to the fast axis except determines the axis which will be scanned across. That is the axis that will move once for each line scan
in the fast axis. 
Fast Axis Step Size: Determines the size of each movement in the fast axis. This will change the pixel size of the heat map in the x direction.
Slow Axis Step Size: Determines the size of each movement in the slow axis. This will change the pixel size of the heat map in the y direction.
Fast Axis Length: Determines the length of which the scan will be over for the fast axis. 
Slow Axis Length: Determines the length of which the scan will be over for the slow axis. 
Angle: Determines the angle at which the scan will be performed. The angles is the relative change in degrees to the right angle of a rectangular 
scan. For example if the angle is input as zero the result will be a rectangular scan. If the angle is input as 45 the scan will be staggered to the 
right by 45 degrees, generating an offset in the fast axis. If you input -45 the scan will be staggered to the left by 45 degrees instead. 
Abort scan: When pressed the scan will end and move back to the scan origin. 
Scan Origin: Before each measurement is started the code records the position it is at. It displays this position in the scan origin. At the end
of each scan the code moves back to the origin for convenience. 
Finally there are four unique type of scanning patterns: 
Center Serpent Scan: Starts a serpent scan centered at the scan origin. A serpent scan is characterized by the negative moving backstrokes in the 
scanning pattern. That is, the fast axis will be scanning in the positive directions on even numbered lines while scanning in the negative directions
in odd numbered lines. This scan process minimizes scanner movement at the cost of noticeable offsets in the scan lines. 
Center Typewriter Scan: Starts a typewriter scan centered at the scan origin. A typewriter scan is characterized by always moving in the positive
direction while taking measurements. During a typical scan the motors will reach the end of the line in the fast axis then move back to the start then
continue the scan. 
Start Point Serpent Scan: Starts a serpent scan where the starting point is the bottom left measurement.
Start Point Typewriter Scan: Starts a typewriter scan where the starting point is the bottom left measurement.

IMPORTANT NOTES ON SCANNING:
- All scanning patterns will fill in measurements starting at the bottom left of the scan. 
- The main difference between center scanning and start point scanning is that the start point scan will take the current point as the bottom left of 
the scan while the center scan will take the starting point as the middle of the scan. 
- The preferred scanning pattern is typewriter scanning as the movements add limited extra time while drastically improving the quality of the scans. 

=====================================================================================================================================================

Graph:
Data is shown live on a heat map. The X axis is the fast axis of the scan and the Y axis is the slow axis of the scan. Before data is taken the graph
is updated to correspond to the actual position and scale matching the scanning parameters. The measured and averaged voltage will be displayed for 
each pixel on the graph and the heat map should update live as the measurements are taken. The value of each measurement and the time for it to run a 
singular loops are displayed at each step. At the end of each fast axis line the maximum, the minimum, and the position at which the maximum occurs 
for the scan so far will be displayed. On the graph itself there are two cursors which will display a position and the Z value at that position to 
the right of the graph. 
Actions to be taken:
- Save Image to PNG will open up the file directory to save a PNG of the current image in the graph. 
- Export data to excel will open an excel sheet with all of the Z values of the data in it. 
- Move to cursor will move to the current position of cursor one. 
- Move to max value will move the motors to the position on the graph with the highest Z value. This position will match the one displayed as max
values position. 

IMPORTANT NOTES:
- All actions can only be performed when a scan is not in progress. 
- If the abort button is pressed in the middle of a scan the minimum value will be displayed as -inf. This is because we build an array and initialize
the entries to -inf to store the measurements in. 

=====================================================================================================================================================

End Button:
When pressed will end the code and close the port connecting to the micronix scanners. 

IMPORTANT NOTES: 
-If stop button is pressed in the middle of an action, due to code blocking the action will finish first then the code will end and 
the port will close.
 
=====================================================================================================================================================

