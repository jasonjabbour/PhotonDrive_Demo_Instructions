-----------------
Demo Commands:
-----------------

Check for you timesync. It's a service that runs in the background:

sudo systemctl status ptp4l.service 

Start Docker Container

dev_start

dev_into

Running the stack 
bash scripts/startup/autonomy.sh

----------
-- Make your own Map
--------

GPS Module, Localization, CANBus

Start:
./scripts/localization.sh
./scripts/gps.sh
./scripts/canbus.sh

Start Recording:

What it means:
cyber_recorder record -c <where data is coming from (single topic)> -o <filename>

Command:
cyber_recorder record -c /apollo/localization/pose -o test_lot5.record

Convert to CSV
python modules/tools/map_gen/extract_path.py test_lost5.csv test_lot5.record.000000 test_lot5.record.000001 ...<add as many as you recorded. The longer the recording the more of these record files that would have been produced>

To create your actual final map. 
bash scripts/create_map_from_xy.sh --xy test_lot5.csv --map_name test_lot5
bash scripts/create_map_from_xy.sh --xy test_lot5.csv --map_name <what ever you want to name map>

Change the settings of your basemap:
cd apollo/modules/map/data/test_lot5
nano base_map.txt

	speed_limit change to <3>

If you make changes to your base map run this script! If you changed the default_end_way_point.txt you do not need to run the following script:

cd ~/apollo
./scripts/update_map_layers.sh test_lot5

Now that made a new map, you need to use that new map. 

nano /apollo/modules/common/data/global_flagfiles.txt
	
	Change --map_dir=<path_file>/test_lot5

Start Autonomy Stack

./scripts/startup/autonomy.sh stop && ./scripts/startup/autonomy.sh


Debugging---------

Inside Docker
 cyber_monitor

E-Stop
	- Kills commands by killing the CAN Bus. 


To fix:

*** Any time you Change Source Code!!!!!! (If change made in Config, pb.txt, .dag, then no need to run this only changes to the source code)

 cd apollo

./apollo.sh build_opt_gpu











	
