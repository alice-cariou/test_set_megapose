# Test_set_megapose

Benchmarking of [megapose](https://github.com/agimus-project/happypose) using Tiago, objects of the [tless dataset](http://cmp.felk.cvut.cz/t-less/v) and mocap.

# Using this repository

The results can be found in the all_results.yaml file, with the average, median, minimum and maximum results.

The benchmarking data is located in the `tiago` directory. Each example subdirectory has :
- a `results.yaml` file, containing the transformation between the mocap mesurement (our truth) and the megapose results
- the image as `image_rgb.png`
- an `object_data.json` file, containing the label and bbox needed for megapose in the right format
- a `details.yaml` with all the mesurements used

To create new tests, or run megapose on the existing tests, scripts can be found in the `scripts` directory.

The following instructios describe how to use the scripts.

# Before use

Depending on your goals, you might need to install stuff or configure your environement.
- megapose relating scripts : you will need megapose, configure the environment.Too obvious ? TODO
- mocap : might be a few things to configure depending on the computer.If it is the first time you use optitrack, you might need to setup a few things [here](lien?)
- rviz : link to bottom 

## create a test using tiago and the mocap

### on a computer connected to tiago

To use the following commands, you need to be able to connect to the mocap.
At the root of the repository :
`source setup_tiago.sh`  -> will assume you have a ~/openrobots_ws directory
`optitrack-ros -b`  
`rosaction call /optitrack/connect '{host: "muybridge", host_port: "1510", mcast: "239.192.168.30", mcast_port: "1511"}'`

You can now get all the necessary mesurements.

### tiago position in the mocap frame
TODO: choose the best scripts
To make the following steps easier, the position detected, tiago_smth, corresponds to torso_lift_link in the robot model, on top of the robot.
`./get_mocap.py --name <example_name>`

### object position in the mocap frame

To obtain the object position without modifying its appearance, i created plank_gepetto, which TODO
`./get_tless.py --name <example_name>`

### 

You should now have access to the topics regarding tiago.  
You can now get the image and the position of the camera with:  
`./get_tiago_infos.py --name <example_name>`


## using megapose

To test a new example with megapose, you will need to :  

Create an inputs file for megapose, with the name of the example directory, the name of the object, and the coordinates of the bounding box around the object to detect. (temporary solution hopefully)
`./screate_inputs_file.py --name <example_name> --object <object_name> x1 y1 x2 y2`  
an example of use for this script would be :  
`./screate_inputs_file.py --name 004 --object tless23 300 286 425 336`

Create the megapose example using the informations you have in this directory :  
`./create_megapose_example.py --name <example_name>`
This will create the example in the datadir you chose for happypose, ready to be used by happypose.

You can now run megapose on this new example using :  
`python -m happypose.pose_estimators.megapose.scripts.run_inference_on_example <example_name> --run-inference --vis-outputs`  
If you are not familiar with happypose, you might need to check its repository to configure your environment correctly : [happypose](https://github.com/agimus-project/happypose)

After that, you should be able to access the outputs of megapose for this example with :  
`./get_outputs_megapose.py --name <example_name>`
This will add megapose informations in the details.yaml of your example directory.

## comparing the results

To visualize the results with rviz and calculate the things:
first -> check link to install rviz

`./process_mesurements.py --name <example_name>`
This will show the robot and the object from the mocap, and the megapose results

In another window:
`./process_tf_obj --name <example_name>`
This will calculate the transform between what was detected by megapose and the "truth" (mocap), and write in the results.yaml file of your example directory.

Finally, if you added a new example and want it to be taken into consideration in the all_results.yaml file, you can run:
`./process_results`

## rviz

The installation is detailled [here](http://wiki.ros.org/Robots/TIAGo/Tutorials/Installation/InstallUbuntuAndROS)
You should then have created a workspace (named tiago_public_ws if you kept the name from the tutorial)

copy ac_show.launch file in your tiago_public_ws/src/tiago_robot/tiago_description/robots/ directory

then, each time you need to use it :
source tiago_public_ws/devel/setup.bach
roslaunch tiago_description ac_show.launch