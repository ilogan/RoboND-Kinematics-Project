## Project: Kinematics Pick & Place



[//]: # "Image References"

[image1]: ./misc_images/RvizModel.png
[image2]: ./misc_images/URDFReferenceFrame.jpg
[image3]: ./misc_images/DHReferenceFrame.jpg
[image4]: ./misc_images/ModelParameters.png
[image5]: ./misc_images/DHAlpha.png
[image6]: ./misc_images/IndivTransform.png
[image7]: ./misc_images/IndivTransformCode.png
[image8]: ./misc_images/HomogTransform.png
[image9]: ./misc_images/HomogTransformCode.png
[image10]: ./misc_images/Theta1.png
[image11]: ./misc_images/Theta23.png
[image12]: ./misc_images/RobotTop.jpg
[image13]: ./misc_images/RobotSide.jpg
[image14]: ./misc_images/R36Symbols.png
[image15]: ./misc_images/thetas456.png
[image16]: ./misc_images/10of10.png
[image17]: ./misc_images/IndivTransformResult.png
[image18]: ./misc_images/HomogTransformResult1.png
[image19]: ./misc_images/HomogTransformResult2.png
[image20]: ./misc_images/HomogTransformResult3.png
[image21]: ./misc_images/HomogTransformResult4.png

## 
### Kinematic Analysis

#### Summary

The goal of this project is to have a simulated Kuka KR210 robot perform the rotations necessary to pick up a blue cylinder and place it into a container given starting and ending coordinates.

#### 1. Run the forward_kinematics demo and evaluate the kr210.urdf.xacro file to perform kinematic analysis of Kuka KR210 robot and derive its DH parameters.

The kr210.urdf.xacro file contains all the necessary model information needed to derive DH parameters. Since it loads into Rviz, I ran the demo and (with all joint angles set to zero) used the positions to figure out distances between the different links.

![alt text][image1]



It's important to note that the reference frames from the URDF file do not coincide with the reference frames used for determining DH Parameters. The two images below illustrate that difference.

###### URDF Reference Frame Assignments


![alt text][image2]

###### DH Parameter Reference Frame Assignments


![alt text][image3]



Combining knowledge of the positions from Rviz and the location of DH reference frame assignments, I was able to sketch out a model to figure out values for creating a DH Parameter table.


![alt text][image4]

The alpha values were a bit tricky to figure out, but by looking down the x-axis for each link, it was easy to find the rotational values (alpha) needed to match them with the respective z-axis.


![alt text][image5]



From there I was able to figure out the values for the DH parameter table:


Links | alpha(i-1) | a(i-1) | d(i) | theta(i)
--- | --- | --- | --- | ---
0->1 | 0 | 0 | 0.75 | q1
1->2 | - pi/2 | 0.35 | 0 | -pi/2 + q2
2->3 | 0 | 1.25 | 0 | q3
3->4 | - pi/2 | -0.054 | 1.5 | q4
4->5 | pi/2 | 0 | 0 | q5
5->6 | - pi/2 | 0 | 0 | q6
6->EE | 0 | 0 | 0.303 | 0

Note that there is a -pi/2 constant added to q2. Similar to determining the alpha values, we have to rotate the x-axis -pi/2 in order to align x1 with x2.

#### 2. Using the DH parameter table you derived earlier, create individual transformation matrices about each joint. In addition, also generate a generalized homogeneous transform between base_link and gripper_link using only end-effector(gripper) pose.

To describe relative orientation and translation between links, DH conventions uses the following transformation matrix:

![alt text][image6]



The first three entries in the first three columns represent the change in orientation, while the first three entries of the fourth column represent the translation, so I plugged in the values from the DH table to figure out individual transformation matrices between each link.

![alt text][image7]

These were results for each of the individual matrices:

![alt text][image17]



Combining the individual transformation matrices together yields the homogenous transform, which is just a matrix showing the total transformation needed from one link to another:

![alt text][image8]

By multiplying the individual matrices together, I created the homogenous transform from the base to the gripper (aka end effector):

![alt text][image9]

And this resulted in the final homogenous transform. It is quite large, so I broke it into columns (clicking the images enlarges them as well):

###### Column 1

[![alt text](./misc_images/HomogTransformResult1.png)](./misc_images/HomogTransformResult1.png) 

###### Column 2

[![alt text](./misc_images/HomogTransformResult2.png)](./misc_images/HomogTransformResult2.png) 

###### Column 3
[![alt text](./misc_images/HomogTransformResult3.png)](./misc_images/HomogTransformResult3.png) 


###### Column 4
[![alt text](./misc_images/HomogTransformResult4.png)](./misc_images/HomogTransformResult4.png) 



#### 3. Decouple Inverse Kinematics problem into Inverse Position Kinematics and inverse Orientation Kinematics; doing so derive the equations to calculate all individual joint angles.

The last three joints of the robot consist of the parts needed to create a spherical wrist. As a result, we can decouple the inverse kinematics problem into one of position and orientation. Joints 1, 2, and 3 control the position of the gripper, while joints 4, 5, and 6 control the orientation. I started by tackling the position problem.

##### Position

The following image gives an idea for tackling the first theta value:

![alt text][image10]

We can think of the wrist center as having a location in space at [x,y,z] (labelled xc, yc, and zc in the image). Using simple trigonometry, I took the inverse tangent of yc and xc to pretty easily determine theta 1.



To find thetas 2 and 3 requires a bit more work. The following image displays a rough idea on how to work out the problem:

![alt text][image11]

From examining the earlier robot sketch, I was able to determine that A and C are constants with lengths of 1.501m and 1.25m, respectively. That leaves side B as our variable length. To get a better idea of how everything fits together and find length B, I sketched a more in-depth model. I started with the top-down view of a robot with arbitrary joint angles:

![alt text][image12]

With L5/WC having coordinates [x,y,z] again, I took WCx and WCy to find the length of the horizontal (from a side-view) line of the triangle used to find side B. This was achieved through noting that the robot has a constant 0.35m distance from its base to Link 2 along the x and y axes.

Then I moved on to making a top down view of the robot and expanding upon the earlier triangle model:

![alt text][image13]

Again, it is important to note there is a constant 0.75m distance from the base to Link 2 along the z-axis. From here I was able to find the length of side B as the hypotenuse of a right triangle and the law of cosines to find the triangles angles. 

As seen in the above image, theta 2 was then a matter of subtracting angle a and t2 from pi/2 (90 degrees) in order to find its value. 

Theta 3 was a similar problem, with the exception being that only angle b needs to be subtracted from pi/2, instead. The image does not take into account the 0.036 radian sag between link 3 and 5, so this was added in its calculation as seen in the code. Also to note is that my above sketch does not illustrate theta 3. Angle b is at pi/2, thus theta 3 is zero.

##### Orientation

The remaining theta values were a bit more straightforward. The new problem was one of orientation, which meant I needed to utilize the previously defined rotation matrices.  I printed out the output of the rotation matrix from joints 3 through 6 symbolically:

![alt text][image14]



Using this information, I just needed to isolate the theta values:

![alt text][image15]

Finally, I was able to utilize all 6 theta values to complete the project.

### Project Implementation

#### 1. Fill in the `IK_server.py` file with properly commented python code for calculating Inverse Kinematics based on previously performed Kinematic Analysis. Your code must guide the robot to successfully complete 8/10 pick and place cycles. Briefly discuss the code you implemented and your results. 

Coding for the project was very challenging, but it was an excellent learning experience. The most computationally intensive part of the code takes place in a loop for calculating the forward/inverse kinematics. Because of this, it was important to define the DH parameters, homogenous transformation matrices, and any other static variables outside of the loop. 

With that said, the robot could be faster. It was able to grab 10 out of 10 cylinders, but its path planning can work quite odd at times. As an example, it sometimes performs 360 degree rotations, which can be a costly maneuver in terms of time consumption. If I were to improve upon the robot, keeping its joint movements within pi would be a good start to avoid such unnecessary actions.

Overall, this was a great project for getting my feet wet in terms of understanding kinematics. I definitely plan to work on the robot a bit more in the future in order to have a smoother working robot!


![alt text][image16]


