# DroneKit记录

## API参考

https://dronekit-python.readthedocs.io/en/latest/automodule.html#dronekit.Vehicle.gimbal

Vehicle movement is primarily controlled using the Vehicle.armed attribute and `Vehicle.simple_takeoff()` and `Vehicle.simple_goto` in GUIDED mode.

Velocity-based movement and control over other vehicle features can be achieved using custom MAVLink messages (`Vehicle.send_mavlink()`, `Vehicle.message_factory()`).

It is also possible to work with vehicle “missions” using the `Vehicle.commands attribute`, and run them in AUTO mode.

gimbal

time.sleep(num)

## 连接

https://dronekit-python.readthedocs.io/en/latest/guide/connecting_vehicle.html#get-started-connect-string

1. `dronekit_sitl.start_default()`，返回sitl？
2. `sitl.connection_string()`，返回连接字符串
3. `connect（连接字符串，wait_ready，baud）`，返回vehicle

## 车辆状态和设置

https://dronekit-python.readthedocs.io/en/latest/guide/vehicle_state_and_parameters.html

```
print "Airspeed: %s" % vehicle.airspeed
vehicle.mode = VehicleMode("GUIDED")
vehicle.armed = False
```

不能确保成功修改，轮询法见网址

### 常用指令

1. ★`vehicle.location`。含
    1. `global_frame` (LocationGlobal)：绝对经纬度，绝对高度
    例：
        
        ```
        print "Global Location: %s" % vehicle.location.global_frame
        print "Sea level altitude is: %s" % vehicle.location.global_frame.alt
        ```
        
    2. `global_relative_frame` (LocationGlobalRelative)：绝对经纬度，相对home的高度
    例：
        
        ```
        vehicle.location.global_relative_frame.lat
        vehicle.location.global_relative_frame.lon
        vehicle.location.global_relative_frame.alt
        ```
        
    3. `local_frame` (LocationLocal)：相对home的位置和高度
2. `vehicle.attitude`，获取飞机姿态角。
    
    ```
    vehicle.attitude.pitch
    vehicle.attitude.yaw
    vehicle.attitude.roll
    ```
    

### 参数设置

★使用`Vehicle.parameters`，返回一个`Parameters`对象

## 起飞

1. 设置为GUIDED（通常）
2. 发出arm命令
3. 起飞

### 常用指令

1. `vehicle.is_armable`，判断是否能起飞
2. `vehicle.mode`
3. `vehicle.armed`
4. `vehicle.simple_takeoff`

## 导航和控制无人机

### 常用指令

1. `simple_goto(location, airspeed=None, groundspeed=None)`，指`LocationGlobal` 或 `LocationGlobalRelative`
2. ★先对信息进行编码`vehicle.message_factory.set_position_target_local_ned_encode(...)`或`vehicle.message_factory.command_long_encode(...)`等，返回message类，然后使用`vehicle.send_mavlink(msg)`发送。见：https://dronekit-python.readthedocs.io/en/latest/guide/copter/guided_mode.html#guided-mode-how-to-send-commands
3. ★`def get_location_metres(original_location, dNorth, dEast)`，相对位置转绝对坐标，返回LocationGlobal类型
4. ★`def get_distance_metres(aLocation1, aLocation2)`，返回两个LocationGlobal 或 LocationGlobalRelative 对象之间的地面距离（以米为单位）
5. ★`def get_bearing(aLocation1, aLocation2)`，返回作为参数传递的两个 LocationGlobal 对象之间的方位。

一些命令例如MAV_CMD_NAV_TAKEOFF已经集成在`Vehicle.simple_takeoff()`中，无需使用mavlink发送

## 任务（AUTO模式下）

### 分类

1. MAV_CMD_NAV_*，用于飞行器位置的移动
2. MAV_CMD_DO_*，用于执行辅助功能（例如伺服电机）
3. MAV_CMD_NAV_*MAV_CMD_CONDITION_DISTANCE，延迟DO命令的执行

？：在一次任务中，最多只能同时运行一条导航命令和一条 DO 或 CONDITION 命令。CONDITION 和 DO 命令与上次发送的 NAV 命令相关联：如果无人飞行器在执行这些命令之前到达航点，则会加载下一条 NAV 命令，并跳过这些命令。

### ★相关任务操作

1. 下载当前任务
2. 清除当前任务
3. 创建、添加任务命令
4. 修改任务

以上见网址 https://dronekit-python.readthedocs.io/en/latest/guide/auto_mode.html

★`Command(...)`创建类：

1. `cmd = Command(0,0,0, mavutil.mavlink.MAV_FRAME_GLOBAL_RELATIVE_ALT, mavutil.mavlink.MAV_CMD_NAV_WAYPOINT, 0, 0, 0, 0, 0, 0,-34.364114, 149.166022, 30)`，其中`mavutil.mavlink.MAV_CMD_NAV_WAYPOINT`表示任务命令。返回航点命令，各参数意义见网址
2. `cmd.command = mavutil.mavlink.MAV_CMD_NAV_TAKEOFF`，可修改任务命令

见：https://dronekit-python.readthedocs.io/en/latest/automodule.html#dronekit.Vehicle.commands

### 开始任务

1. 要开始任务，请将模式更改为 AUTO
2. 如果vehicle在空中，则只需将模式更改为自动即可启动任务。
3. Copter 3.3 版本及更高版本：如果vehicle（仅）在地面上，您还需要发送MAV_CMD_MISSION_START命令。
4. 您可以通过退出自动模式（例如进入引导模式）来停止/暂停当前任务。如果切换回 自动模式：任务将在开始时重新启动或在当前航点恢复 - 行为取决于MIS_RESTART参数的值（适用于所有车辆类型）。
5. 更改当前任务命令：
    
    ```
    vehicle.commands.next = 2
    print ("Current Waypoint: %s" % vehicle.commands.next)
    ```
    
6. At the end of the mission the vehicle will enter LOITER mode (hover in place for Copter, circle for Plane, stop for Rover).
7. ★`def upload_mission(aFileName)`，将航点文件上传至飞控（调用了`readmission()`）
8. ★`def readmission(aFileName)`，读取航点文件，返回一个列表，元素为cmd
9. `def save_mission(aFileName)`，保存任务为航点文件（调用了`save_mission(aFileName)`）
10. `def download_mission()`，下载飞控上的任务，返回一个列表，元素为cmd
11. `def distance_to_current_waypoint()`，返回到下一个航点的距离（以米为单位）

### 判断任务结束

1. ~~Add a dummy mission command and poll `Vehicle.commands.next` for the transition to the final command：在最后创建虚拟任务，轮询`Vehicle.commands.next`是否等于该任务。~~见下载的DroneKit例程，有解决该问题的方法。

## 其他

1. 监听器listener：属性监听器，可监听并返回vehicle的一些属性。对于消息监听器，每次收到指定消息时都会调用该函数。
2.
