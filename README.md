# PhotonDrive_Demo_Instructions
Demo Instructions


----------------------
--- Sensors
----------------------


Two Key Sensors
	- Know where you are
		- GPS
	- Know what's around you
		- Lidar: for optical detection
	- Time
		- Everything in the vehicle needs to understand what point of time it is. 
		- if t = 0, lidar has output x need to make sure GPS is in sync
		- PTP: time sync protocol
			- uses GPS as your host
			- GPS access 3 different satilites and does trangulation
			- Lat, long, std, timestamp
			- We parse out the timestamp and send out to our computer
			- Computer uses this timestamp as ground truth 
			- LiDar has PPS (pulse) it sends pulse every x seconds for timesync protocol
			- All sensors use GPS timestamp as ground truth
			- If indoors GPS timestamp is all messed up (starts at like zero at 1842)
				- Only way to fix it is to drive it out and let it settle. 
				- PTP will correct itself
			- Apollo does not do detection unless there is timestamp from GPS
		 
TODO: ROS Drivers to Send and Receive Commands


Drive-By-Wire
	- Takes in CAN messages and can relay what is happening with the car
	- It's a bridge between the car and the software

Speed & Steering Controller
	- Give it an output and it will control it itself
	- No need for timesync of protocol for this
	- Reach ou to Sri to set this up 

	- Open ROS node, subsribe to steering command node
	- Send 0 to 20 radians as input


Cameras
	- Solve the problem of calibration
	- You need to calculate where the camera is in space.
	- The Lidar is ground zero
	- Compute position with a physical checker board to localize the camera 
	- Whole objective is to find the position of the camera
	- They use an autoware tool to do this
	- Just needs to be done once subject to the car not being moved
	- Creates a callibration matrix output to be read in any time. 
	- Or you can use cameras for motion classification but LiDar for object detection
		- Used an object detection module from ROS for the LiDar
	- Each camera has a different lense
		- You can easily swap that out

Sensors
	- All the sensors have ROS Drivers to make them actual publish messages through topics

GPS Sync Box:
	- Takes in the serial signal from the GPS
	- parsing Serial Signal and sending it out through Eth
	- has information about the Lat, Long, and Timestamp

Switch
	- Lidar, GPS, and Router are connected through ethernet to the switch
	- Switch as an extra port which connects to the CPU and that's how you can connect to each IP address
	- If you can't access the server wirelessly because of a VPN, you can connect an ethernet cable to the switch and connect the ethernet cable (convert it to a usbc) to your laptop. Then you can ssh into the server from your laptop through the ethernet. 

Connection
	- Has IP address
		- GPS: 192.168.100.150
		- Lidar: 192.168.100.10
		- Router: 192.168.0.1
	USB Connection
		- Cameras

CAN 
	- Cars have lots of ECUs (brake, windshield wiper, acceleration, door actuators, ... what ever it is)
		- Anything that is electronic has an ECU which makes decision making for that unit
	- Usually there's a central ECU to make decision making about maybe like throttle control
	- CAN is a Serial Port that acts as a bridge to the ECUs of the car
		- Serial ports have a different protocol 
		- RS232 (standard protocol for serial) Pinout Color Code
	- PacMod -> output User CAN -> Serial Port (has input male) -> Kavser
	- CAN bus is sending lots of serial messages (publishes information at 30 Hz) from each of these modules
	- Architecture: Lots of ECUs for anything electronic in the car <-> Vehicle CAN <--Serial to Serial--> PackMod CAN  translates to (DBC) <-- Kavser --> Apollo
		- Pacmod system is their drive by wiresystem
		- The CANalizer connects to the Kavser
		- Kavser is a USB wire
		- CAN utils can be used instead of CANalizer using Python
		- CANalizer converts hex to binary readable data


Fuse Box
	- The fuse box under the driver seat provides power to all the sensors
	- This is a separate fuse box from the one native to the vehicle
	- The fuse box under the driver seat does not provide power to the ECUs of the vehicle but only the sensors
	- You can visualize these from the front display

Router
	- The router box is located under the passanger seat. 
	- It has the mac address and password on the back. 


---------------------------------------------------------------

--------- Apollo Demo 

----------------------------------------------------------------

Check status of system:
$ sudo systemctl status pip4l.service

Start the Docker and Enter it:
$ dev_start

$ dev_into


In Apollo Docker Start the Autonomy Processes:
$ ./scripts/startup/autonomy.sh

localhost:8888


Perception

$ ./scripts/startup/preception.sh stop


Debugging:

$ sudo systemctl status pip4l.service

Nothing should be in red

If not in sync end all processes and restart the computer


------------------------------------------
--------- Apollo Classroom Session 
------------------------------------------


Apollo Version: 5.5


Sensors
	Localization
		- GNSS
	Perception
		- Velodyne LiDAR
		- Leopard Imaging Cameras
		- Continental Rader (NOT INCLUDED, used for large distances)

Apollo Architecture
	- Routing Expects a map. Routing sends information to planning
	- Canbus recieves and sends commands to control (feed back loop
	- Also a high level feedback loop wiht planning
	- Camera -> traffic light recognizer
	- Lidar, GNSS, Radar, for localization and prediction
	- Expectation is that the map already knows where the traffic light is. 
	- So the traffic light module gets triggered if it knows there's a traffic light there 
	- Extrapolation Prediction: Prediction module comes from pose detection
		- $ cat apollo/modules/prediction/conf/prediction_conf.pb.txt
		- It's hard coded in C
		- Recongizes theres an obstactle
		- What type of obstactle
		- Get relative pose to Ego and Map
		- Understands what the expected heading of the obstacle. 
		- If the road curves then the prediciton module will understand that it's expected to also move along that curve
		- Gives prediction about places that where the object might move
	- CyberRt
		- The Apollo version DDS
	
CANBus
	- Commands and reports include
	- Throttle percentage
	- Brake Percentage
	- Steering angle 

	...	


Map/Routing/Planning:

Map gives information to routing
Routing Modules gives shortest possible path to goal
Planning (Live plan using A*) to get to that destination
	- Has lots of parameters such as 
		- Obstacle
		- Intersection
Planning and Routing talk to each other continuously. 
Can do lane change if another waypoint in the other lane
Or use borrow lane change 

Speed and Steering Controller (SSC)
	- Calculate the percent brake/acc/steering from how much curvature & speed comes from Apollo
	- Depending on curvature and speed that comes from apollo, SSC will calculate the brake/acc/steering percentage
	- Speed: Uses PID Controller
	- Steering: Uses Stanley Controller




-----------------
Demo Commands:
-----------------

Check for you timesync. It's a service that runs in the background

$ sudo systemctl status ptp4l.service 
	
	- RMS is what you want to check. You want that to be very small. 
	- If RMS is really high then planning will be going a head of time and it won't work 
	- Once its at a small number then you are ready

Start Docker Container

$ dev_start
$ dev_into


Now we can Start our stack. There is one script that starts the whole stack

$ cat scripts/startup/autonomy.sh
	- This launches a bunch of scripts separately

$ bash scripts/startup/autonomy.sh

Dreamview is the Visualizer 

	- Go to localhost:8888
	- Look under console tab. The numebrs should not go over 00.00.500
	- Now that you're closer to our map
	- If you are outside the map, select two points at start and end of the map
	- Then click send routing request
	- Once you get close enough to the route it will automatically enable
	- Click Start Auto When you get on the route and you see the blue planning line. 
Outside of docker:
The network interface is down that the kavaser uses. 
$ sudo ip link set can0 type can bitrate 500000
$ sudo ip link set up can0


Make a Map:

Options:
1. OSN
2. 3rd Party
3. Make your own lane and route
	- takes an x and y in space and makes its own lane


----------
-- Make your own Map
--------

GPS Module, Localization, CANBus

Start:
$ ./scripts/localization.sh
$ ./scripts/gps.sh
$ ./scripts/canbus.sh

Start Recording:

What it means:
cyber_recorder record -c <where data is coming from (single topic)> -o <filename>

Command:
$ cyber_recorder record -c /apollo/localization/pose -o test_lot5.record

Start driving, then ctrl+C when you're done recording your map


Convert to CSV
$ python modules/tools/map_gen/extract_path.py test_lost5.csv test_lot5.record.000000 test_lot5.record.000001 ...<add as many as you recorded. The longer the recording the more of these record files that would have been produced>

View CSV
$ cat test_lot5.csv

These are the x,y waypoints

To create your actual final map. 
$ bash scripts/create_map_from_xy.sh --xy test_lot5.csv --map_name test_lot5
bash scripts/create_map_from_xy.sh --xy test_lot5.csv --map_name <what ever you want to name map>

How to know map was created?

$ cd apollo/modules/map/data

You should now see your map_name like test_lot5
 $ cp ../test_lot4/default_end_way_point.txt
 $ cat default_end_way_point.txt
This should the final way point that you want to navigate to. You can change the numbers of this to change where the car navigates to. 

Change the settings of your basemap:
$ cd apollo/modules/map/data/test_lot5
$ nano base_map.txt

	speed_limit change to <3>


If you make changes to your base map run this script! If you changed the default_end_way_point.txt you do not need to run the following script:

$ cd ~/apollo
$ ./scripts/update_map_layers.sh test_lot5

Now that made a new map, you need to use that new map. 

$ nano /apollo/modules/common/data/global_flagfiles.txt
	
	Change --map_dir=<path_file>/test_lot5

Start Autonomy Stack

$  ./scripts/startup/autonomy.sh stop && ./scripts/startup/autonomy.sh

Wait to finish

Go to DreamView (localhost:8.8.8.8)

Will autonmatically load your map because you changed your global map file. 

Route Editing > Click end point destination > Send Routing Request

Then Start driving close to the lane until you see the blue planning route. Then click start auto. 



*** Any time you Change Source Code!!!!!! (If change made in Config, pb.txt, .dag, then no need to run this only changes to the source code)
$ cd apollo
$ bash apollo.sh build_opt_gpu



Debugging---------



Inside Docker
$ cyber_monitor
	This shows individual modules running. These are all the scrips launched by the autonomy.sh script. You can click on them and see the topics being sent


	- If car isn't moving forward. Check apollo/canbus/chassis
	- If position is off apollo/loclization/pose
	- apollo/snesor/gnss/best_pose (this updates ever 1 second so you should see things changing
	- apollor/sensor/velodyne32/pointcloud

Any time you Change Source Code!!!!!! (If change made in Config, pb.txt, .dag, then no need to run this only changes to the source code)
$ cd apollo
$ bash apollo.sh build_opt_gpu

Outside Docker
$ cd apollo/data/log
$ tail -f  perception.info
$ tail -f  planning.info
$ tail -f  control.info
$ ls to see all the logs that are available. These info files could have errors that are helpful. 

IP Addresses
	- Lidar, GNSS, Router
	- Router and GNSS are going through single dns route points (ens1f1)
		- Go to settings. Should be 192.168.100.126 (IP OF THE INTERFACE)	
		- GNSS: 192.168.100.150
		- Router: 192.168.100.1
	- ens1f0 (the other interface) (for the lidar alone)
		- Assigned the interface 192.168.1.100
		- Lidar: 192.168.1.201
	- 

E-Stop
	- Kills commands by killing the CAN Bus. 


Connect to Novatel (GNSS)

$ telnet 192.168.100.150 3004

$ log loglist
$ log bestpos
$ docs.novatel.com/oem7


--------------------------------------
---- Simulation 
--------------------------------------


JOSM - Java OpenStreetMap (Software)
- Make the lanes using a Google Map
- Export as OSM
- Go to apollo/tools/lanelet2_convert
-bash convert_ll2_map.bash ~/Download/map_you_made.osm mapping <- this is the foler

Once you have converted the osm map:

$ dev_start

$ dev_into

Launch Dream View
$ ./scripts/dreamview.sh

In Dream View  > Turn on Sim Control  

Turn on Setup 

Click on the two way points you want to navaigate to. 


Any time you Change Source Code!!!!!! (If change made in Config, pb.txt, .dag, then no need to run this only changes to the source code)
$ bash apollo.sh build_opt_gpu


--------------------------------------
Apollo Setup 
---------------------------------------


Apollo 
- Modules
- Scripts 

Dag files are used ot launch in Apollo

Change max speed 
 in 
apollo/modules/calibratin/data/gem/planning


Todo:
- Make copy of apollo directory and github branch
- How we put additional sensors 
- Pacmod system
	- you need to install it on the computer then plug in the video game controller. 
	- Then plug in the joy stick directly to the computer. 
- How to create new maps
	- OSN Map: used for bigger maps without driving along that route. Just use google maps
	- How to set waypoints dynamically 
	- How to bridge data to ROS
- Get slides from Sri
- How to local install
- They are able to bridge the apollo topics to ROS topcis 
- RTK how to configure
	- Get scripts from Sri
	- 4 commands 
	- Is there any RTK Base stations ask HUIT
- Lane Change
- Get GEM SSC documentation 
- Get a USB hub and plug in to our computer




Storage Procedure After Testing Vehicle:

- Turn Off the sensor switch underneath the driver seat
- Turn off Vehicle
- Place keys out of ignition 
- Turn off wireless keyboard
- Engage Vehicle Hand brake
- Press the E-Stop Botton In to ensure no autonomous actions will be taken
- Put in gear in neutral
- Plug in UPS to AC power
- Plug in ethernet cord to PCIe Card
- Plug in vehicle to Charge
- Open Windows
- Keep Server Running
- Make sure external UPS batteries are daisy changed into the the UPS to ensure they are charging


Connect the UPS to a power outlet to charge its internal and daisy changed external batteries





-------------------------------------------------------
----------- JoyStick --------------------------------
--------------------------------------------------------

Joystick on personal Laptop -------------------------

source /opt/ros/noetic/setup.bash

catkin_make



source devel/setup.bash


roslaunch ssc_gesture ssc_joystick.launch

sudo apt-get update && sudo apt-get install -y usbutils


/dev/input/js0


Joystick on Vehicle -----------------------------------

I have SSC and its respective dependencies installed. these are the instructions to launch and test out the controller using the SSC Joystick ROS node:

In the first terminal type: roslaunch pacmod pacmod.launch​
in the second terminal type: roslaunch ssc_pm_gem_e4 speed_steering_control.launch​
in the third terminal type: roslaunch ssc_joystick ssc_joystick.launch​

If you don't see any errors on any of the terminal screens, you should be able to enable using the controller. the controls for using it are available in the README here (NOTE: Make sure you have the E-stop release before you test). Let me know if you have any issues with this and I can help you out. 


Fix Namespace:
Within the opt/ros/melodic/share/pacmod/launch file add ns="pacmod" to the node pkg="socketcan_bridge" and pkg="pacmod" elements. 








	
