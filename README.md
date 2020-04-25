

# Lidar 3D  Cloud Projection

This Project present how to associate regions in a camera image with Lidar points in 3D space. 

![](https://williamhyin-1301408646.cos.ap-shanghai.myqcloud.com/img/20200425193128.png)

![](https://williamhyin-1301408646.cos.ap-shanghai.myqcloud.com/img/20200425152627.png)

## Dependencies for Running Locally
* cmake >= 2.8
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* Git LFS
  * Weight files are handled using [LFS](https://git-lfs.github.com/)
* OpenCV >= 4.1
  * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors.
  * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)

## Basic Build Instructions

1. Clone this repo.

3. Make a build directory in the top level project directory: `mkdir build && cd build`

4. Compile: `cmake .. && make`

5. Run it: `./project_lidar_to_camera`.

   

## Steps

1. Filtering Lidar Points

   The code below shows how a filter can be applied to remove Lidar points that do not satisfy a set of constraints, i.e. they are …

   1. … positioned behind the Lidar sensor and thus have a negative x coordinate.
   2. … too far away in x-direction and thus exceeding an upper distance limit.
   3. … too far off to the sides in y-direction and thus not relevant for collision detection
   4. … too close to the road surface in negative z-direction.
   5. … showing a reflectivity close to zero, which might indicate low reliability.

   ```c++
   for(auto it=lidarPoints.begin(); it!=lidarPoints.end(); ++it) {
       float maxX = 25.0, maxY = 6.0, minZ = -1.4; 
       if(it->x > maxX || it->x < 0.0 || abs(it->y) > maxY || it->z < minZ || it->r<0.01 )
       {
       	continue; // skip to next point
   	}
   }
   ```

   

2. Convert current Lidar point into homogeneous coordinates and store it in the 4D variable X.

   ```
    X.at<double>(0, 0) = it->x;
    X.at<double>(1, 0) = it->y;
    X.at<double>(2, 0) = it->z;
    X.at<double>(3, 0) = 1;
   ```

   

2. Then, apply the projection equation to map X onto the image plane of the camera and Store the result in Y.

   ```
   Y = P_rect_00 * R_rect_00 * RT * X;
   ```

   

3. Once this is done, transform Y back into Euclidean coordinates and store the result in the variable pt.

   ```
   cv::Point pt;
   pt.x = Y.at<double>(0, 0) / Y.at<double>(0, 2);
   pt.y = Y.at<double>(1, 0) / Y.at<double>(0, 2);
   ```

   