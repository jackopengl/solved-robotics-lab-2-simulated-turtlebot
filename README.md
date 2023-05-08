Download Link: https://assignmentchef.com/product/solved-robotics-lab-2-simulated-turtlebot
<br>
This lab session will introduce you to the TurtleBot, as well as writing a publisher to give commands to the robot to get it to move in a specified direction. However, we will first test all our code in a simulation environment at this stage before using them on the real Turtlebot.

Set up the singularity environment

Using steps described in lab 1, create a singularity container and then a few Ubuntu terminals to use for later.

Making sure your git clone is up-to-date

Before you do anything else, make sure your git clone of lab2 is up-to-date:

cd ~/catkin_ws/src/lab2 git pull

<h1>Gazebo simulation environment</h1>

In a terminal, run the following command:

roslaunch turtlebot_gazebo turtlebot_world.launch

(The first time you run this command, it may take a few minutes to load. Please be patient and wait until a new window with a virtual environment opens up.)

This brings up the Gazebo simulator where you can see the Turtlebot and objects in a simulated environment.  Use your mouse’s scroll wheel to zoom in and out. Press and hold your mouse’s middle button, while moving the mouse. This will rotate your view in the world. Similarly, experiment with your mouse’s left and right buttons. Explore the three-dimensional virtual environment and the robot.

At the top of your screen, you will see a menu:

The leftmost pointer mode is currently chosen. In this mode, your mouse will simply move your viewer in the world. Now choose the second from the left ( ).  In this mode, go and hold on an object in the virtual world, and drag it around. In this mode you can move the objects in the world. You can move the robot as well just like any other object.

<h1>Turtlebot ROS nodes and topics</h1>

You now have running the nodes that provide basic functionalities of the TurtleBot. For example, this will bring up the controller for the TurtleBot’s mobile base, drivers for TurtleBot’s sensors, etc. If you want to see a list of all the ROS nodes that are running, in a new terminal do this:

rosnode list and inspect the result. Many of the nodes are named to explain their functionality, so try to guess what each one does. Similarly, to see all the ROS topics that are created, do this:

rostopic list

These are all the topics required for the complete functioning of the TurtleBot. Can you see the one named “mobile_base/commands/velocity”? As the name suggests, this topic is used to send velocity commands to the mobile base of the robot. We will use it today.

<h1>Understanding the mobile_base/commands/velocity topic</h1>

To see the publishers and subscribers to the topic, run:

rostopic info /mobile_base/commands/velocity

The subscriber of this topic controls the mobile base. To move the robot, we will write a new node that will publish to this topic.

The rostopic info command also shows us the type of the topic. It is geometry_msgs/Twist. If we want to publish to the topic, we need to use this type. What is in a geometry_msgs/Twist type? The command “rosmsg show” is useful for that: rosmsg show geometry_msgs/Twist

The result shows us that Twist is actually a type that contains two other types in it, specifically the Vector3 type. Each Vector3 type includes three float values. Notice that one of the Vector3 types are named ‘linear’ and the other ‘angular’. It is becoming more clear now: This is a message type to specify linear and angular velocities in three dimensions.

<h1>Handling dependencies</h1>

In our package we will write code that uses message types from geometry_msgs, which is simply another ROS catkin package. When one package uses types defined in another package, this is called a dependency: Our package needs to depend on geometry_msgs. We indicate dependencies through the package.xml file in a package. Go to the lab2 directory, open the package.xml file in it, and add this somewhere between the &lt;package&gt; tags:

&lt;build_depend&gt;geometry_msgs&lt;/build_depend&gt;

Go back to your catkin workspace and catkin_make.

<h1>Simple forward/backward motion</h1>

You will find the file firstwalk.py in lab2 directory. Open and inspect it. Particularly notice that since we need access to the geometry_msgs/Twist message type, the script imports it like this:

from geometry_msgs.msg import Twist

You always need to import a message type if you are going to use it in a script.

Also notice how we set the x component of the linear velocity to a desired value.

Run this script and see what happens (you might need to make the script executable before you run it) :

rosrun lab2 firstwalk.py

From inspecting the script you should notice that the reason that the robot only moves backwards and forwards is because we only assigned values to the linear x component of one of the vectors. You will have noticed that we did not assign any angular velocity at all. This is why the robot only moves backwards and forwards without turning. Experiment with publishing instructions while altering the values of the velocity. Note that each Vector3 has 3 components (x, y and z). See what the results of differing combinations are. Try to understand why.

Notice that the firstwalk.py script uses the velocity 0.2 to command the wheels. The units are in m/s. <strong>Please never command the robot to drive faster than 0.2 m/s, otherwise you can hurt the robot or some other item in the room, e.g. the glass displays in 9.21, when you are running the real robot.</strong>

<h1>Circle and square</h1>

To get the robot turning we must also supply a value for the angular velocity. More specifically the z component of the angular velocity.

<strong>Exercise 1.</strong> Create a script that will continuously drive the robot in a circle. You will need to use a loop to constantly publish the message, however only one Twist message will actually need to be created. It is important to remember that you can deliver angular and linear velocities at the same time. A well traced fluid circle comes from one motion of both linear and angular velocities combined. <strong>Important: Please read the section “Writing infinite loops in ROS” in the Appendix of this document, before starting to work on this exercise.</strong>

<strong>Exercise 2. </strong>Create a script that will trace a square by driving the robot. The unit of the angular velocity is in radians/sec, positive representing anti-clockwise movement. Take that into consideration when trying to program a turn for a nice square.

Great! You now know how to move the TurtleBot.

<h1>Bumper input</h1>

When you start up the TurtleBot software it does not only start the controller for the mobile base, but it also starts drivers for sensors. If you look at the output of ‘rostopic list’ again, you can see different sensor topics. Here we will mention only one, but you can experiment with others as well. The topic “/mobile_base/events/bumper” is a topic that outputs values based on the bumpers on the TurtleBot base. If you have the real Turtlebot in front of you, please locate the two bumpers on the left and right of the robot. Use rostopic info to inspect this topic.

rostopic info /mobile_base/events/bumper

You will see that the messages are of type kobuki_msgs/BumperEvent. You should use rosmsg show to inspect the details of the BumperEvent message type.

One quick way to make sure this topic works as expected is to listen to it on command line using “rostopic echo”:

rostopic echo /mobile_base/events/bumper (Ignore any warnings you might get about “/clock”) Now, if you are following this using the real robot, press on the bumpers with your hand, while you keep your eye on this terminal window. You should see that the state variable is set to 1, when bumper is pressed.

In the simulator, you will not be able to press the bumper with your hands. Instead, move the robot and/or the objects in the virtual world, such that there is an object in close proximity in front of the robot. Then run firstwalk.py to make the robot move forward. When the robot hits the obstacle, you should again see the bumber state being set to 1.

<strong>Exercise 3. </strong>Change your script that moves the robot on a square to stop when a bumper is pressed. You will need to add a subscriber to the correct topic.

<strong>For safety, in any program you write that commands the robot base, we strongly recommend that you include a code block to stop and move the robot backwards whenever the bumper is pressed.</strong> The bumper is a simple sensor. There are many other sensors on the TurtleBot, including cameras.

<h1>Cameras</h1>

There are two cameras on the TurtleBot. One camera is the 3D sensor’s camera. The 3D sensor provides both RGB and depth values which are published as separate ROS topics. The other camera is the laptop’s camera, which you are also welcome to use. The laptop camera exists only on the real Turtlebot, now in simulation.

Now we will explain how you can use the image streams from the 3D sensor.

<h1>3D sensor</h1>

Now look at the list of topics again (“rostopic list”). There should be plenty of /camera/ topics including /rgb/ and /depth/. For example, /camera/rgb/image_raw is the topic for the raw RGB image from the camera. You can try “rostopic echo” on it, but it will dump a large binary stream; i.e. “rostopic echo” is not that useful to inspect binary messages. But ROS developers provide a tool to listen to and display images. The tool is called image_view. Run it like this:

rosrun image_view image_view image:=/camera/rgb/image_raw

You should see the RGB image from the robot’s 3D sensor in a new window. This image is the “live” image: if you move your robot, for example by executing the first_walk.py, this image should reflect what your robot sees.

Quit image_view after you are done. In the next lab session, you will learn how to write a node that listens to an image topic and make the robot react to what it sees.

The 3D sensor also outputs depth (distance) values. The topic “/camera/depth/points” include the depth data. Execute “rostopic echo” on it and you will see lots of values being streamed. It is not binary like the RGB image, but still difficult to interpret. We will use yet another tool to inspect this data. This tool is called Rviz (for “ros visualizer”). Rviz is a powerful and useful tool that you should get familiar with. Here is the rviz user guide: <u>http://wiki.ros.org/rviz/UserGuide</u> . Run it:

rosrun rviz rviz

As described in Section 4 of the rviz user guide, add a new display for the PointCloud2 message type. On the left pane, enter the topic name for this display as /camera/depth/points. Change the value of the “Fixed Frame” field under “Global Options”, and select “odom”. You should now see the 3D point cloud from the 3D sensor. It is also “live”: if you move the robot, the point cloud should reflect what the robot sees. You can use your mouse buttons to move around in the 3D display in Rviz. After you are done, quit rviz.




Lab Session 2 – Real Turtlebot

Now we will work on the real Turtlebot after the successful completion of our trials in simulation and we will start with an introduction.

<h1>TurtleBot introduction</h1>

You should be reading this while in front of the TurtleBot.

<ol>

 <li>Cables:</li>

</ol>

On the TurtleBot base there will be two USB cables you should see. They need to be plugged into the TurtleBot’s laptop to be operational.

<ol start="2">

 <li>Charging:</li>

</ol>

A seperate cable and powerpack are used to charge the TurtleBot, they will be located near the TurtleBots. When you are finished with the TurtleBot, plug the robots charger into the base (near the power switch for the base). This will make sure that the robot is charged for the next group to use it.

Similarly, when you are done, please place the laptop into one of the lockers and plug it in so that it can be charged.

<ol start="3">

 <li>Laptop and seating the laptop on the TurtleBot:</li>

</ol>

The notebooks are seated on top of the TurtleBot. They should be seated so the the laptop screen is facing the opposite direction to the 3d sensor is facing. They are held in place by the velcro squares on top of the robot and the one on the base of the laptop, make sure that you line up the squares as best as possible so the laptop is seated securely on the top.

<ol start="4">

 <li>Turning the robot on:</li>

</ol>

Use the power switch on the mobile base to turn it on. Then, turn the laptop on.

<h1>Initializing your group’s git repo</h1>

If you do not remember how to do this, please read the section “Initializing your group’s git repo” from the lab1 worksheet, and initialize your group’s git repo for lab2.

<h1>Setup Turtlebot laptop for your group</h1>

If you do not remember how to do this, please read the section “Setup Turtlebot laptop for your group” from the lab1 worksheet.




<h1>Running TurtleBot software</h1>

To get the laptop to command the robot, you need to start the TurtleBot software. In a terminal window, run:

roslaunch turtlebot_bringup minimal.launch

This will bring up the nodes that provide the basic functionality of the TurtleBot. For example, this will bring up the controller for the TurtleBot’s mobile base, drivers for TurtleBot’s sensors, etc. like the simulation environment. You can again inspect the nodes and topics launched. Having launched the relevant nodes for the real robot, follow the instructions for the sections above, this time on the real robot:

<ul>

 <li>Turtlebot ROS nodes and topics</li>

 <li>Understanding the mobile_base/commands/velocity topic</li>

 <li>Simple forward/backward motion</li>

 <li>Circle and square</li>

 <li>Bumper input</li>

</ul>

<strong>Exercises </strong>Complete Exercises 1, 2 and 3 as in the simulation lab, but this time on the real Turtlebot. You can use the code developed in the simulation lab. You should not need to change much. That is the beauty of using a simulator: You can develop your code in the simulator, and it should <em>mostly</em> work on the real robot. However always remember that the robot’s sensor inputs and motor commands will always be a lot noisier in the real world.

<h1>3D sensor – Real Robot</h1>

Since the 3D sensor consumes lots of battery power, the minimal.launch does not start it. To start the 3D sensor, you need to run a new launch file:

roslaunch astra_launch astra.launch

Now look at the list of topics again (“rostopic list”). You should see topic names which did not exist before. There should be plenty of /camera/ topics including /rgb/ and /depth/. For example,

/camera/rgb/image_raw is the topic for the raw RGB image from the camera. Just like in simulation, run:

rosrun image_view image_view image:=/camera/rgb/image_raw

You should see the RGB image from the robot’s 3D sensor in a new window.

The 3D sensor also outputs depth (distance) values. The topic “/camera/depth_registered/points” include the depth data. Just like we did in the simulated environment, we will now use rviz to visualise depth data by running the following command:  rosrun rviz rviz

As described in Section 4 of the rviz user guide, add a new display for the PointCloud2 message type. On the left pane, enter the topic name for this display as /camera/depth_registered/points. Change the value of the “Fixed Frame” field under “Global Options”, and select “odom”. You should now see the 3D point cloud from the 3D sensor. You can use your mouse buttons to move around in the 3D display. After you are done, quit rviz.

<h1>Laptop camera</h1>

You should be able to launch the laptop camera using this command: roslaunch libuvc_camera launchcam.launch

You should now see the topic /laptop_camera/image_raw being published. Use image_view to inspect the image.

<h1>Other topics</h1>

Earlier in this lab you will have noticed that we were instructing you to examine and analyse topics and the messages they use via the rostopic echo and rosmsg show commands. This is a very helpful method for understanding how the different topics communicate with each other over ROS. When writing publishers and subscribers you will definitely want to use this method of examination to find out about the messages you need to be handling or sending in your custom nodes. A follow on point to that method was that you can also use rostopic echo to examine messages arriving at the topic, allowing you see the kind of messages that arrive if you do particular things to the robot.

rostopic info &lt;topic_name&gt; rostopic echo &lt;topic_name&gt; rosmsg show &lt;message_type&gt;

Remember these commands, they will become <strong>very useful</strong> for your future work within ROS. You can also call “rostopic –help” to see other useful functionality within the rostopic command. Same applies to rosmsg and rosnode commands.




Below, we collect the topics on the TurtleBot that are worth knowing about and we encourage you to examine them using this method and anything else we mention:




<strong>/camera/depth_registered/points</strong> – This is a depth pointcloud produced by the 3dsensor that you can view in rviz visualiser rosrun rviz rviz

This will be a useful topic to subscribe to for when you want to program nodes that react to objects based on depth and how far away or close an object may be (following, for example).




<strong>/camera/rgb/image_raw </strong>– The raw image data being input from the sensor, you can view this using a package called image_view

rosrun image_view image_view image:=/camera/rgb/image_raw

This can be useful for identifying the colour of specific objects in the sensors field of view. <strong>/laptop_charge</strong> – Keeps track of the battery levels in the turtlebots laptop, you can inspect this topic using the rostopic info and rosmsg show method you used earlier.

rostopic info /laptop_charge rostopic echo /laptop_charge

You could write a node to monitor this level and give warnings to the user when the battery levels are running low.




<strong>/mobile_base/events/bumper</strong> – Records events concerning the bumpers on the robots base, you can inspect this topic using the rostopic info and rosmsg show method you used earlier.

rostopic info /mobile_base/events/bumper rosmsg show BumperEvent

rostopic echo /mobile_base/events/bumper




<strong>/mobile_base/events/cliff</strong> – Records events concerning when the robot can or cannot detect a floor beneath itself, you can inspect this topic using the rostopic info and rosmsg show method you used earlier.

rostopic info /mobile_base/events/cliff rosmsg show CliffEvent

rostopic echo /mobile_base/events/cliff

Try running rosotpic echo and examine the messages produced when you pick up and put down the robot.




<strong>/mobile_base/events/robot_state </strong>– Indicates the state of the robot, on or off, you can inspect this topic using the rostopic info and rosmsg show method you used earlier.

rostopic info /mobile_base/events/robot_state rosmsg show RobotStateEvent

rostopic echo /mobile_base/events/robot_state

Run rostopic echo on this topic, it should show you one message, you can guess what it means.




<strong>/mobile_base/events/wheel_drop </strong>– Records events concerning whether or not the wheels have dropped within their seating either due to the floor becoming angular or due to the robot being picked up.

You can inspect this topic using the rostopic info and rosmsg show method you used earlier.

rostopic info /mobile_base/events/wheel_drop rosmsg show WheelDropEvent

rostopic echo /mobile_base/events/wheel_drop

Run rostopic echo on this topic and try picking the robot up and placing it down again. You’ll notice that two seperate messages are produced when you do this, one for each wheel.

Try and figure out which wheel is wheel 0 and which wheel is wheel 1.







<strong>/mobile_base/sensors/imu_data</strong> – The imu collects data concerning linear acceleration and angular velocity and sends it to the main processor, you can inspect this topic using the rostopic info and rosmsg show method you used earlier.

rostopic info /mobile_base/sensors/imu_data rosmsg show

rostopic echo /mobile_base/sensors/imu_data

Run rostopic echo on this topic and try nudging the robot or turning it on the spot. You’ll notice how sensitive the robot is to changes in its own velocity.






































































Appendix.




Writing infinite loops in ROS




If you want to write infinite loops in your code, one way to do this in Python is: while True:

&lt;other commands&gt;

Please do <strong>not</strong> use the loop style above when using ROS. Instead, here is the correct way to do it: while not rospy.is_shutdown():

&lt;other commands&gt;

The code above will loop until rospy is shutdown, e.g. when you ctrl-c your code.