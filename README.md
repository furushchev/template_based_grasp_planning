template_based_grasp_planning
- - -

## What is this

This is fork from usc-clmc-ros-pkg.
Support only hydro.

The `template_based_grasp_planning` stack provides a grasp planner that generates a library of templates from kinesthetic teaching and from that computes grasp hypotheses on unknown objects.
An implementation of the framework on the PR2 is provided together with visualization tools and user interfaces in the `pr2_template_grasping` package.

A consise description of the algorithm is provided in:  

```text
A. Herzog, P. Pastor, M. Kalakrishnan, L. Righetti, T. Asfour, S. Schaal, Template-Based Learning of Grasp Selection, Int. Conf. on Robotics and Automation (ICRA 2012)
```

## How to use

### GRASP DEMONSTRATION

First, you will need to demonstrate some grasps to the robot and generate a grasp library.
Therefore, a tabletop scenario is required.
Move the robot in front of a table with one object on it and orient the camera such that it points on the object.
Further, free the view on the object, such that there is no occlusion.
Before you can start the demonstration, it is required that the `tabletop_object_detector` and a stereo camera system is running.
The simplest way to achieve this, is by starting the `pr2_manipulation_pipeline` with

```bash
roslaunch pr2_tabletop_manipulation_launch pr2_tabletop_manipulation.launch stereo:=false
```

which should start the manipulation pipeline.
If your pr2 does not have a Kinect, you need to leave out the `stereo:=false`. For more details on how to start the pr2_tabletop_manipulation pipeline, please refer to
[pr2_tabletop_manipulation_apps](http://www.ros.org/wiki/pr2_tabletop_manipulation_apps/Tutorials/Starting%20the%20Manipulation%20Pipeline).

In order to record a grasp, you will need to move the robots hand around freely.
Thus, you might want to turn off the controllers of the right arm or put the robot into mannequin mode with:

```bash
roslaunch pr2_mannequin_mode pr2_mannequin_mode.launch
```

as described here: http://ros.org/wiki/pr2_mannequin_mode
Now you should be ready do demonstrate a grasp using:

```bash
roslaunch grasp_template_planning user_demonstration_recorder.launch filename:=[DEMONSTRATION_FILE_NAME]
```

which will provide you with a procedure to record a grasp demonstration.
Follow the instructions that are written in the output of the program.
The result of a demonstration will be written into the file `grasp_template_planning/data/grasp_demonstrations_data/[DEMONSTRATION_FILE_NAME]`.
Feel free to add more demonstration files or delete some of them. When you are done, you will need to generate a grasp library from the demonstrations that you recorded before.
Therefore, run

```bash
roslaunch grasp_template_planning generate_grasp_library.launch
```

This will generate a file `grasp_template_planning/data/grasp_library.bag`.
Now the algorithm is initialized and ready to be used.

### EXECUTION OF GRASPS

In order to execute a grasp on a new object, we provide the `pr2_template_grasping` package.
Before running the grasp execution make sure that the `pr2_tabletop_manipulation` is running and the controllers of the arm are turned on.
Also you should leave the `pr2_mannequin_mode`, if you used it before.
The grasp planning service that will compute grasp configurations, is run by

```bash
roslaunch pr2_template_based_grasping template_grasp_planning_server.launch
```

This node provides you with some services to plan grasps, visualize them and take feedback from grasp executions. We provide a modified pick and place application which you can refer to in order to see how to use the services. The unmodified pick and place application is described on http://www.ros.org/wiki/pr2_tabletop_manipulation_apps/Tutorials/Writing%20a%20Simple%20Pick%20and%20Place%20Application
In order to execute our modified version that uses the template-based grasp planner, run

```bash
oslaunch pr2_template_based_grasping template_grasp_execution_ui.launch
```

After the node connected to all the services, it should ask you to enter "go" followed by enter, which will start the planning and grasping procedure. The grasp planning might take up to half of a minute depending on how large your library is.

There might be issues with finding feasible grasps due to workspace limitations of the robot. In order to guarantee maximum workspace usage, drive the robot close to the edge of the table and drive its spine all the way up. Also make sure that the height of the table allows for approaching the object from various directions without hitting joint limits.
For deeper analysis of errors try to visualize the outcome of the grasp planner as described below and refer to [pr2_tabletop_manipulation_apps](http://www.ros.org/wiki/pr2_tabletop_manipulation_apps/).

### VISUALIZATION

After a grasp was (tried to be) executed, the planning node starts publishing several point-clouds and markers that can be visualized in rviz. They show the executed and matched grasp poses, templates, point clouds and also visualize the convex hull and extracted height-axes. In order to make the planning node publish one of its other resulting grasps rather than the executed one, you can call

```bash
rosservice call pr2_template_grasp_planner_visualization [RANK]
```

where `[RANK]` is the rank of the grasp you would like to be visualized.

### FEEDBACK

In order to make the algorithm consider feedback, you can return the result of a grasp to the planning service using the feedback user interface that is also provided in the `pr2_template_based_grasping` package. You can start the interface in an additional shell with

```bash
roslaunch pr2_template_based_grasping template_grasp_feedback_ui.launch
```

After the robot executed a grasp (as described before) you can give feedback by following the instructions in the feedback interface. This will make the planning service write a `*.bag` file into `grasp_template_planning/data/library_negatives` and use it in future planning requests.
Although the interface also provides you with the option to return feedback from successful trials, this option should not be used, yet, because it has never been tested.