Software Structure
==================

The package from ``PAL Robotics`` are provided in `/opt/pal/alum/share/`. In here are the packages presented. Most of the packages only provided launch files and config files. Those files can only be editted as sudo. The basic functions consist of the following packages:

	- ``tiago_dual_bringup``: Launches all the needed configs

	- ``tiago_dual_description``: Provides the definition of the Tiago model, the meshes and also the physical constraints.

	- ``tiago_dual_gazebo``: This launches gazebo in a fixed world from PAL with the Tiago model.

	- ``tiago_dual_moveit_config``: Houses all the information about the manipulators and controllers. This package is needed if you want to move Moveit2 for controlling the manipulators.

	- ``pmb2_2dnav``: Houses the navigation for the Tiago that determine the best route and update to the current environmental situation.

Docker container
----------------
A Docker container is available from PAL Robotics. This container includes all the provided software packages and simulation worlds. These are client-specific. In other words, when you buy a robot from PAL Robotics, they provide you with the necessary software. With this container, you can work with Tiago as you would in real life. 

First you need to do is download the docker file. Download it from **LINK!!!** Then create the Docker container with the following command:

.. code-block::
   
   bash <client-name>-build-docker.sh --create

The create container can be started with the command:

.. code-block::
 
   docker start -ai tiago-dual-115-dev

Now you are in the docker container called the `Development Docker` from PAL. To run all the simulation code, make sure you are running it in the docker container. The Docker container was used for the research assignment because Tiago was unfortunately not working.

Simulation
----------

There is a package that contains a simulation in the supermarket provided by KAS lab. First clone the repository

.. code-block::

   git clone git@github.com:RichardFierkens/tiago_dual_supermarket.git

.. note::

   Improvement are needed (17 October 2025) for adjusting the map. But for now do this:


Now you need to change the path for the map in the file `/opt/pal/alum/share/pal_navigation_cfg/params/nav2_map_server/00_map_server.yaml` with the path of the world of the supermarket located.

After that, the simulation including moveit and nav2 can be run. 

.. code-block::

   ros2 launch tiago_planning supermarket_tiago.launch.py


If you want to make some changes in the coordination of this packages. Feel free to change in the packages. 

	- `tiago_skills/detect_object_server.cpp`: This provides the vision code. Using an apriltag detector for receiving the object coordinates
	- `tiago_skills/move_skill_server.cpp`: Here, the moveit is created using MoveGroupInterfaces. So far, here is not really anything needed to change.
	- `tiago_skills/navigate_skill_server.cpp`: This file make sure that the coordinates for nav2 is working. This files take care of making the Tiago move.
	- `tiago_skills/pick_place_skill_server.cpp`: This package make sure that the gripper will open and close. In here, an attach_object function is used. this is only needed for the simulation.
	- `BehaviorTree.ROS2/btcpp_ros2_samples_tiago_dual`: This package provides the behavior tree of the simple pick and place mission. 
	
	
	
Create map
----------

For now (28 October 2025), there are two maps created with SLAM. If you want to create a new map do the following:

1. Comment in the `supermarket_tiago.launch.py` the two launch arguments
   `loc_robot_launch` and `nav2_robot_launch` and uncomment `slam_bringup_launch`.
2. Select the world in the same launch file you want to create a map from.
3. Launch the file.
4. Open rviz and check the /map topic so that you can see the map.
5. Open in another terminal:

   .. code-block:: bash

      ros2 run teleop_twist_keyboard teleop_twist_keyboard cmd_vel:=/cmd_vel

6. When you are done with mapping, save the map to your folder with:

   .. code-block:: bash

      ros2 run nav2_map_server map_saver_cli -f /home/user/maps/AH_store_map \
         --ros-args -p image_format:=pgm

7. Now change this .yaml file as described above.

Pick and Place mission
----------------------

A simple pick and place mission is created for the Tiago in Gazebo Classic. Before running the simulation, you need to make sure that the collision in the arms of Tiago are disabled. Otherwise a plugin for grasping the object is not possible.

1. Comment the collision in the `pal_gripper_description/gripper.urdf.xacro` file. Do this for the wrist and arm urdf in the `tiago_description` as well.
2. Launch the simulation as before:

   .. code-block:: bash

      ros2 launch tiago_planning supermarket_tiago.launch.py
      
3. Open another terminal and launch the behavior tree:

   .. code-block:: bash

      ros2 launch btcpp_ros2_samples_tiago_dual sample_bt_executor.launch.xml
      
4. Now send the goal with the desired mission in a third terminal:

   .. code-block:: bash

      ros2 action send_goal /behavior_server btcpp_ros2_interfaces/action/ExecuteTree "{target_tree: PickPlaceMission}"

5. This will run the pick and place mission.


