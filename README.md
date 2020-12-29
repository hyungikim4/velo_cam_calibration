# velo_cam_calibration

Obtain the calibration matrix, which is the transformation matrix from the LiDAR coordinate system to the image pixel coordinate system.

## 1. Save images and pointcloud

Place the planar board in several locations and save the images to `data` folder and rosbag of pointcloud.

## 2. Obtain the coordinates of the vertices of a planar board in pointcloud

### Setting parameters

`velo_cam_calibration/launch/velodyne_vertex.launch`

|Parameter| Type| Description|
----------|-----|--------
|`error_thresh`|*double* |Save `vertex.xml` only when error rate is less than error_thresh. Default `0.07`.|
|`left_line_real_length`|*double*|The length of the left line of the flat board used for calibration (m scale). Default `0.81`.|
|`right_line_real_length`|*double*|The length of the left line of the flat board used for calibration (m scale). Default `0.81`.|

### Subscribed topics

|Topic|Type|Description|
------|----|---------
|`/velodyne_points`|sensor_msgs::PointCloud2|raw Lidar pointcloud|


### Published topics

|Topic|Type|Description|
------|----|---------|
|`/xfiltered_points`|sensor_msgs::PointCloud2|Filtered pointcloud on x-axis (using xmin, xmax).|
|`/xyfiltered_points`|sensor_msgs::PointCloud2|Filtered pointcloud on xy-axes (using xmin, xmax, ymin, ymax).|
|`/xyzfiltered_points`|sensor_msgs::PointCloud2|Filtered pointcloud on xyz-axes (using xmin, xmax, ymin, ymax, zmin, zmax).|
|`/projected_points`|sensor_msgs::PointCloud2|The result of projecting `/xyzfiltered_points` points to the plane equation obtained through RANSAC from `/xyzfiltered_points`.|
|`/line_points`|sensor_msgs::PointCloud2|Points for each line in `/projected_points`. Red-top left line. Green-top right line. Blue-bottom left line. Yellow-bottom right line.|
|`/vertex_points`|sensor_msgs::PointCloud2|For each line of `/line_points`, the equation of a straight line is obtained through RANSAC, and the intersection of the equations of two straight lines is expressed as four points. Red-top point. Green-left point. Yellow-right point. Blue-bottom point. These four vertices are saved as `vertex.xml`.|

### Execution
**[Example](https://youtu.be/gRup9FKuB_c)**

`roslaunch velo_cam_calibration velodyne_vertex.launch`

Press the publish point button in rviz and click the vertex of the planar board with the mouse. 

Click the four vertices of the planar board in the order of top, left, bottom, and right. 

These are used to get the parameters to filter the planar board (xmin, xmax, ymin, ymax, zmin, zmax).

When the error rate for the lengths of all sides is smaller than the `error_thresh` previously set as a parameter, it is saved as `vertex.xml` and `vertex_date.xml`.

## 3. Obtain the coordinates of the vertices of the plane board in the image
### Execution

**[Example](https://youtu.be/Cym1WIXi82o)**

`rosrun velo_cam_calibration camera_click_vertex`

In the image, click the vertices of the planar board in the order of top, left, bottom, and right, and press the N button to change to the next image. 

When four vertices are clicked on an image, it is saved as `camera.xml` and `camera_date.xml`.

## 3. Obtain the calibration matrix using SVD
Repeat 2 and 3 in advance to acquire information on the vertices of the flat board at various angles and distances in the `data` folder.

At this time, the number of `camera_xx.xml` and the number of `vertices_xx.xml` should be the same.

### Execution
`rosrun velo_cam_calibration velo_cam_calibration`

The calibration matrix is obtained using all `camera_xx.xml` and `vertices_xx.xml`, and `calibration_matrix.xml` and `calibration_matrix_date.xml` are created.
