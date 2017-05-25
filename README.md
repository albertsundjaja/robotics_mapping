# Included Files
* drive_rover.py containing the script to do autonomous driving
* supporting_functions.py containing helper functions
* decision.py containing script to decide on what to do
* perception.py containing script to sense surroundi

# How To
To begin using the model, execute the command
`python drive_rover.py`
Run the simulator and choose autonomous
https://s3-us-west-1.amazonaws.com/udacity-robotics/Rover+Unity+Sims/Windows_Roversim.zip

# Identifying obstacles and rock samples
Obstacles are identified by reversing navigable terrain. Thus, it includes the cliffs and blocking rocks. 
```
not_obstacle_index = thresholded_terrain.nonzero()
obstacle = np.ones_like(img[:,:,0])
obstacle[not_obstacle_index] = 0
```
Rock samples are yellowish in appearence, therefore a threshold which identify yellowish colors is applied to find rock samples. By trial and error, it is found that a range in RGB values of [100,100,0] and [200,200,70] will identify rock samples.

```
def rock_thresh(img, boundary =([100,100,0], [200,200,70])):
    """apply thresholding to find the rock sample"""
    lower = np.array(boundary[0], dtype = "uint8")
    upper = np.array(boundary[1], dtype = "uint8")
    # create mask
    mask = cv2.inRange(img, lower, upper)
    # apply image masking
    output = cv2.bitwise_and(img, img, mask = mask)
    # convert result to gray
    output = cv2.cvtColor(output, cv2.COLOR_RGB2GRAY)
    # create temp image for our rock
    zeros = np.zeros_like(img[:,:,0])
    try:
        # get the closest coordinate of the transformed rock
        closest_x = max(output.nonzero()[1])
        closest_y = max(output.nonzero()[0])
        # make the rock look bigger instead of just dot
        zeros[closest_y:closest_y+5,closest_x:closest_x+5] = 1
        return zeros
    except:
        #if no rock is in image, return zeros
        return zeros
```


# Image Processing
Images are first warped to get a bird eye view, then thresholded to identify navigable terrain. In addition, non obstacle and rock samples are also identified after the warp process. The thresholded images are then transformed so that the coordinates are from the rover's perspective. The resulting coordinates are then rotated and translated so that it correspond to the world map of 200x200 pixels (which correspond to 1m per 1px). A demo video is also included in this repo, test_mapping.mp4

# Perception Step
The perception step of the rover is similar to the image processing described above. In rover's perception, however, the navigable parts are converted into polar coordinates for decision making purpose

# Decision step
The decision step involves stopping and moving mode. When moving, the rover will sense whether there are enough 'clear' points forward, if it is not, then it will brake and enter stopping mode. During stopping mode, it will steer left by 15degrees a time until it can see enough 'clear' points to go forward, which then a throttling command will be issued


# Further Improvement
The rover sometimes keep going into the path that is already explored which is inefficient, a memory of where the rover has passed should be implemented so that it remembers where it has explored and find another path instead.

Anothe possible approach is to use a maze solving technique called right hand rule, in which it will keep turning right until it finally reaches the starting point again


