# Video Streaming


> Definition

```shell
# API call described below requires shell access, either login to the device using desktop or use ssh for remote login.

ROS-Service Name: /<namespace>/navigation/position_set
ROS-Service Type: core_api/PositionSet, below is its description

#Request : expects position setpoint via twist.twist.linear.x,linear.y,linear.z
#Request : expects yaw setpoint via twist.twist.angular.z (send yaw_valid=true)
geometry_msgs/TwistStamped twist
float32 tolerance
bool async
bool relative
bool yaw_valid
bool body_frame

#Response : success=true - (if async=false && if setpoint reached before timeout = 30sec) || (if async=true)
bool success
```

```cpp
// C++ API described below can be used in onboard scripts only. For remote scripts you can use http client libraries to call FlytOS REST endpoints from C++.

Function Definition: int Navigation::position_set(float x, float y, float z, float yaw=0, float tolerance=0, bool relative=false, bool async=false, bool yaw_valid=false, bool body_frame=false)
Arguments:
    :param x,y,z: Position Setpoint in NED-Frame (in body-frame if body_frame=true)
    :param yaw: Yaw Setpoint in radians
    :param yaw_valid: Must be set to true, if yaw setpoint is provided
    :param tolerance: Acceptance radius in meters, default value=1.0m
    :param relative: If true, position setpoints relative to current position is sent
    :param async: If true, asynchronous mode is set
    :param body_frame: If true, position setpoints are relative with respect to body frame
    :return: For async=true, returns 0 if the command is successfully sent to the vehicle, else returns 1. For async=false, returns 0 if the vehicle reaches given setpoint before timeout=30secs, else returns 1.
```

```python
# Python API described below can be used in onboard scripts only. For remote scripts you can use http client libraries to call FlytOS REST endpoints from Python.

Class: flyt_python.api.navigation

Function: position_set(self, x, y, z, yaw=0.0, tolerance=0.0, relative=False, async=False, yaw_valid=False,
                     body_frame=False):
```

```cpp--ros
// ROS services and topics are accessible from onboard scripts only.

Type: Ros Service
Name: /<namespace>/navigation/position_set()
call srv:
    :geometry_msgs/TwistStamped twist
    :float32 tolerance
    :bool async
    :bool relative
    :bool yaw_valid
    :bool body_frame
response srv: bool success
```

```python--ros
# ROS services and topics are accessible from onboard scripts only.

Type: Ros Service
Name: /<namespace>/navigation/position_set()
call srv:
    :geometry_msgs/TwistStamped twist
    :float32 tolerance
    :bool async
    :bool relative
    :bool yaw_valid
    :bool body_frame
response srv: bool success

```

```javascript--REST
This is a REST call for the API. Make sure to replace 
    ip: ip of the FlytOS running device
    namespace: namespace used by the FlytOS device.

URL: 'http://<ip>/list_streams'

JSON Response:
{   stream1: String,
    stream2: String,.... }

```

```javascript--Websocket
NA


```


> Example

```shell
rosservice call /<namespace>/navigation/position_set "twist:
  header:
    seq: 0
    stamp: {secs: 0, nsecs: 0}
    frame_id: ''
  twist:
    linear: {x: 1.0, y: 3.5, z: -5.0}
    angular: {x: 0.0, y: 0.0, z: 0.5}
tolerance: 0.0
async: false
relative: false
yaw_valid: true
body_frame: false"

#sends (x,y,z)=(1.0,3.5,-5.0)(m), yaw=0.12rad, relative=false, async=false, yaw_valid=true, body_frame=false
#default value of tolerance=1.0m if left at 0    
```

```cpp
#include <core_script_bridge/navigation_bridge.h>

Navigation nav;
nav.position_set(1.0, 3.5, -5.0, 0.12, 5.0, false, false, true, false);
#sends (x,y,z)=(1.0,3.5,-5.0)(m), yaw=0.12rad, tolerance=5.0m, relative=false, async=false, yaw_valid=true, body_frame=false
```

```python
# create flyt_python navigation class instance
from flyt_python import api
drone = api.navigation()
# wait for interface to initialize
time.sleep(3.0)

# command vehicle towards 5 meteres WEST from current location regardless of heading
drone.position_set(-5, 0, 0, relative=True)

```

```cpp--ros
#include <core_api/PositionSet.h>

ros::NodeHandle nh;
ros::ServiceClient client = nh.serviceClient<core_api::PositionSet>("navigation/position_set");
core_api::PositionSet srv;

srv.request.twist.twist.angular.z = 0.5;
srv.request.twist.twist.linear.x = 4,0;
srv.request.twist.twist.linear.y = 3.0;
srv.request.twist.twist.linear.z = 5.0;
srv.request.tolerance = 2.0;
srv.request.async = true;
srv.request.yaw_valid = true;
srv.request.relative = false;
srv.request.body_frame = false;
client.call(srv);
success = srv.response.success;
```

```python--ros
def setpoint_local_position(lx, ly, lz, yaw, tolerance= 0.0, async = False, relative= False, yaw_rate_valid= False, body_frame= False):
    rospy.wait_for_service('namespace/navigation/position_set')
    try:
        handle = rospy.ServiceProxy('namespace/navigation/position_set', PositionSet)
        twist = {'header': {'seq': seq, 'stamp': {'secs': sec, 'nsecs': nsec}, 'frame_id': f_id}, 'twist': {'linear': {'x': lx, 'y': ly, 'z': lz}, 'angular': {'z': yaw}}}
        resp = handle(twist, tolerance, async, relative, yaw_rate_valid, body_frame)
        return resp
    except rospy.ServiceException, e:
        rospy.logerr("pos set service call failed %s", e)

```

```javascript--REST

$.ajax({
    type: "GET",
    dataType: "json",
    url: "http://<ip>/list_streams",  
    success: function(data){
           console.log(data);
    }
};

```

```javascript--Websocket
NA
```


> Example response

```shell
success: true
```

```cpp
0
```

```python
True
```

```cpp--ros
success: True
```

```python--ros
Success: True
```

```javascript--REST
{
    stream1: <link to stream1>,
    stream2: <link to stream2>,
    stream3: <link to stream3>,
    stream4: <link to stream4>,
    stream5: <link to stream5>,
    .
    .
    .    
}

```

```javascript--Websocket
NA
```





###Description:
This API sends local position setpoint command to the autopilot. Additionally, you can send yaw setpoint (yaw_valid flag must be set true) to the vehicle as well. Some abstract features have been added, such as tolerance/acceptance-radius, synchronous/asynchronous mode, sending setpoints relative to current position (relative flag must be set true), sending setpoints relative to current body frame (body_frame flag must be set true).
This command commands the vehicle to go to a specified location and hover. It overrides any previous mission being carried out and starts hovering.

###Parameters:
    
    Following parameters are applicable for onboard C++ and Python scripts. Scroll down for their counterparts in RESTful, Websocket, ROS. However the description of these parameters applies to all platforms. 
    
    Arguments:
    
    Argument | Type | Description
    -------------- | -------------- | --------------
    x, y, z | float | Position Setpoint in NED-Frame (in body-frame if body_frame=true)
    yaw | float | Yaw Setpoint in radians
    yaw_valid | bool | Must be set to true, if yaw 
    tolerance | float | Acceptance radius in meters, default value=1.0m 
    relative | bool | If true, position setpoints relative to current position is sent
    async | bool | If true, asynchronous mode is set
    body_frame | bool | If true, position setpoints are relative with respect to body frame
    
    Output:
    
    Parameter | type | Description
    ---------- | ---------- | ------------
    success | bool | true if action successful

### ROS endpoint:
Navigation APIs in FlytOS are derived from / wrapped around the core navigation services in ROS. Onboard service clients in rospy / roscpp can call these APIs. Take a look at roscpp and rospy api definition for message structure. 

* Type: Ros Service</br> 
* Name: /namespace/navigation/position_set</br>
* Service Type: PositionSet

### RESTful endpoint:
FlytOS hosts a RESTful server which listens on port 80. RESTful APIs can be called from remote platform of your choice.

* URL: ````GET http://<ip>/list_streams````
* JSON Response:
{
    stream1: \<link to stream1\>,
    stream2: \<link to stream2\>,
    stream3: \<link to stream3\>,
    stream4: \<link to stream4\>,
    stream5: \<link to stream5\>,
    .
    .
    .    
}



### API usage information:
Note: To view the video of a particular stream from the list of streams you need to create an **img** tag add the link to its source.

````<img src='http://<ip>/stream?topic=<link to stream1>' > ````

Tip: You can add the following parameters as query string to the link for lighter or better resolution video quality.
width, height, quality, rate:1/2/3

rate:1 will send out every frame, 2 will send out every second frame, 3 every third and so on..

Tip: To stop the video stream you need to delete the **img** tag completely.

Tip: To take a snapshot of the stream replace the word stream with snapshot in the link.

````<img src='http://<ip>/snapshot?topic=<link to stream1>' >````


Note: **Keep an eye out, for this API needs a port at the end of the IP set to :8080.**