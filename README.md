# BreezySLAM-Webots Implementation (Python)
This is a guide on how to use the python version of BreezySLAM. BreezySLAM uses TinySLAM's CoreSLAM with a particle filter.
Please see the important notes at the end of the page for tips.
## Adding the package
For Python3, compile the package by navigating to the python folder, cd to `BreezySLAM/python` then type the following command:
`sudo python3 setup.py install`
Notice that anything updated in the  `BreezySLAM/python`  folder will require a new compilation.
## Running the demo
BreezySLAM comes with some sample data to test. This can used by running `make pytest` when in the  `BreezySLAM/examples` directory. This will generate a .pgm file showing the SLAM done.
## Adding a new LIDAR device
The list and configurations of the LIDAR devices are located in `BreezySLAM/python/breezyslam/sensors.py`
The generic class for a LIDAR device in BreezySLAM is as follows (this is an example):
```` 
class  my_lidar(Laser):
	def  __init__(self, detectionMargin = 0, offsetMillimeters = 0):
	Laser.__init__(self, scan_size, scan_rate, detection_angle, distance_no_detection_mm, detectionMargin, offsetMillimeters)
````
### Scan Size
This is how many readings the LIDAR does every iteration. In order to find out how many are being outputted, a calculation can be done or a simple print statement in Webots can be used instead.

**Calculation:**
The scan size (which is unitless) is determined by the scan angle (degrees) and the scan resolution (degrees per bits): `Scan size = scan angle/scan resolution`

**Print statement:**
In Webots, LIDAR sensor readings are passed through the `lidar.getRangeImage()` function. A single `print(lidar.getRangeImage)` will simply show the size of the LIDAR reading in the console.
### Scan Rate
The scan rate is how frequently the LIDAR scans in terms of frequency.
### Detection Angle
This is the field of view of the lidar device.
### Distance No Detection
This is the distance (in mm) before the LIDAR device can no longer detect an object properly.
### Example with the Hoyuko URG04LX
The Hoyuko URG04LX is a LIDAR device available in Webots and already has an implementation made by the author. However, due to the differences in some variables used by Webots and the author of the BreezySLAM package, we will simply recalculate them for Webots (see notes for additional information on the differences and why we need to recalculate).

First, it is important to look at the LIDAR's datasheet ([link to the Hoyuko URG04LX datasheet](https://www.hokuyo-aut.jp/dl/Specifications_URG-04LX_1513063395.pdf)). This gives all the information that we need to know about the device.

**Scan Size:** According to the datasheet angular resolution of the device is of 0.36 degrees.
From the datasheet itself, the detection angle is of 240 deg. This means that our scan size is: `240/0.36=667`.

**Scan Rate:** In our example, we are given the scan speed, which is 100msec/scan. Since we want a frequency, we have to take the inverse. Since `1msec=0.001sec`, we have: `1/0.001=10Hz`.

**Detection Angle:** 240 deg.

**Distance No Detection:** 4000 mm

Finally, we can create our new LIDAR class in the `sensors.py` file:
```` 
class  URG04LX(Laser):
	def  __init__(self, detectionMargin = 0, offsetMillimeters = 0):
	Laser.__init__(self, 667, 10, 240, 4000, detectionMargin, offsetMillimeters)
````
Don't forget to save and recompile the package by doing: `sudo python3 setup.py install` in the `BreezySLAM/python` folder.
## Setting Up the Data
The format of the data is the most important part of BreezySLAM. All data entered must be _integers_ or it will not be happy. This means that any values inputted from `lidar.getRangeImage()` that are `inf` must be converted to a 0.

**Time:** 1 column. The suggested value is the time being in microseconds. This doesn't really matter since it just requires for the time to be ascending. This can simply be the values of a counter placed in the `while robot.step(timestep) != -1` loop.

**Odometry:** 24 columns. Odometry in BreezySLAM is optional. This is set by a boolean statement when running BreezySLAM. (Note : this is something I still haven't quite figured out. I have instead just set the use_odometry boolean to false and 0's were insterted for the )

**LIDAR Data:** n columns (where n is the scan size). These are the values generated from `lidar.getRangeImage()`. Since Webots outputs them as floats and BreezySLAM prefers large values to take into account the decimals that would be otherwise truncated, we instead must multiply the  float by 1000 and then convert it into an integer. For example, if we had values of `[1.31,1.47,1.28]` we would need `[1310,1470,1280]`. Furthermore, we would need to get rid of the `inf` by setting them all to 0 with an if statement so that if we have `[1.38,inf,inf]`, we would then have `[1380,0,0]`.
The following is an example of how it is shaped:

| Time | Q1| Q2| Q2|...|Data0|Data1|...|Datan|
| ---- |:-:|:-:|:-:| :-:| :-:| :-:|  :-:|  :-:| 
| 1    | 0| 0  |0  | ...| 0  | 1240| ...|0
| 2    | 0| 0  |0  | ...| 0  | 0| ...|0
| 3    | 0| 0  |0  | ...| 1362  | 1360| ...|0
| ...  | 0| 0  |0  | ...| 0  | 0| ...|1560

Each column is seperated by a space (" ") and each row is seperated by a new line.
For a more precise example, please look at the `exp2.dat` file located in `BreezySLAM/examples`.

From the example with the Hoyuko, we would have the first column be the time, the next 24 being the odometry values, and finally the next 667 columns dedicated to the LIDAR data. This means that there would be a total of 692 columns.
## Selecting a LIDAR device
## Calling BreezySLAM externally
Since the original BreezySLAM package calls for inputs through the terminal using a make file, a small modification has to be done to the `log2pgm.py` file. The definition for `main()` can be removed and replaced with, for example `def RunBreezySlam(dataset,use_odometry,seed)`. dataset refers to the `.dat` file,  use_odometry the boolean value if odometry wants to be used, and the seed is a random number between 0 and 9999 which is used for the particle filter in order to allow for data to be reproducible.
## Getting BreezySLAM's returned data
 
## Additional Settings
### Map Resolution
### Map Size
## Uninstalling BreezySLAM


##  Notes and Issues
**Windows Installation:** According to the Github page for BreezySLAM, it is not recommended for use on Windows and is no longer maintained (at least for the python version).

**Differences in scan size of the URG04LX:** The reason why the scan size in Webots is different from that in the example is due to the way that the angular resolution was calculated. The author recalculated this resolution by doing `360/1024=0.35156`. On the other hand, Webots simply used the value of `0.36` provided by the documentation. We can see the impact of this difference by calculating the scan size:

Author's scan size: `240/0.35156=683`

Webot's scan size: `240/0.36=667`

**Scan Rate:** The number doesn't really matter in Webots since it updates every 16ms. BreezySLAM has the option of receiving data over USB and can tell you the 
##  Work in Progress

##  References
[BreezySLAM package](https://github.com/simondlevy/BreezySLAM)
