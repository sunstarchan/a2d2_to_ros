# A2D2 to ROS

Utilities for converting [A2D2 data sets](https://www.a2d2.audi/) to ROS bags.

The idea is that there is an executuable for each sensor modality: camera, lidar, and bus. Bag files are generated for these modalities independently.

NOTE: Currently, there is only a converter for the [Sensor Fusion > Bus Signal](https://www.a2d2.audi/a2d2/en/download.html) data sets.

## FAQ

[FAQ.md](FAQ.md) contains common questions about the A2D2 data set.

## Converter: Sensor Fusion > Bus Signal

This converter parses a bus signal data JSON file and outputs the data into a bag file.

### JSON parsing and validation

[RapidJSON](https://rapidjson.org/) is used to load, parse, and validate the JSON data files.

A [JSON Schema](http://json-schema.org/) file is provided in [schemas/sensor\_fusion\_bus\_signal.schema](schemas/sensor_fusion_bus_signal.schema) to perform validation.

### Usage

An example invocation is given below. For the example, assume the following locations:

* Package: `~/catkin_ws/src/a2d2_to_ros`
* Data set: `~/data/a2d2/Munich/Bus\ Signals`

```
$ rosrun a2d2_to_ros sensor_fusion_bus_signals --schema-path ~/catkin_ws/src/a2d2_to_ros/schemas/sensor_fusion_bus_signal.schema --json-path ~/data/a2d2/Munich/Bus\ Signals/camera_lidar/20190401_121727/bus/20190401121727_bus_signals.json
```

This command will create the following bag file:

```
~/catkin_ws/src/a2d2_to_ros/20190401121727_bus_signals.bag
```

To get a full list of usage options, run with the `--help` switch:

```
$ rosrun a2d2_to_ros sensor_fusion_bus_signals --help
Convert sequential bus signal data to rosbag for the A2D2 Sensor Fusion data set. See README.md for details.
Available options are listed below. Arguments without default values are required:
  -h [ --help ]                              Print help and exit.
  -s [ --schema-path ] arg                   Path to the JSON schema.
  -j [ --json-path ] arg                     Path to the JSON data set file.
  -o [ --output-path ] arg (=.)              Optional: Path for the output bag file.
  -b [ --bus-frame-name ] arg (=bus)         Optional: Frame name to use for bus signals.
  -v [ --include-original-values ] arg (=1)  Optional: Include the original data set values in their original units.
  -r [ --include-converted-values ] arg (=1) Optional: Include data set values converted to ROS standard units.
```

### Bag file conventions

* Each field in the JSON file corresponds to up to four topics in the generated bag:
    * `original_value`: The original value recorded in the JSON file as a [std\_msgs::Float64](http://docs.ros.org/api/std_msgs/html/msg/Float64.html) message
    * `original_units`: The units of the original value recorded in the JSON file as a [std\_msgs::String](http://docs.ros.org/api/std_msgs/html/msg/String.html) message (this is published only once per bag file).
    * `value`: The value in the JSON file converted to ROS standard units as a [std\_msgs::Float64](http://docs.ros.org/api/std_msgs/html/msg/Float64.html) message
    * `header`: A [std\_msgs::Header](https://docs.ros.org/melodic/api/std_msgs/html/msg/Header.html) message that contains the timestamp of the recorded data
* The `original_value` and `original_units` topics are not included if the converter is run with `--include-original-values false`
* The `value` topic is not included if the converter is run with `--include-converted-values false`
* The message time in the bag file is the same as the timestamp in the header message.
* Each bag file contains a `/clock` topic that has a [rosgraph\_msgs::Clock](http://docs.ros.org/api/rosgraph_msgs/html/msg/Clock.html) message for each unique timestamp in the data set.
* The output bag file is given the same basename as the input JSON file.
* Each of the topics in the bag file (except for `/clock`) is prefixed with `/a2d2/[JSON_FILE_BASENAME]`

## Compatibility

This code is built and tested under:

* [ROS Melodic](https://wiki.ros.org/melodic) with [Ubuntu 18.04.4](http://releases.ubuntu.com/18.04/).
* [Clang 6.0.0](https://releases.llvm.org/6.0.0/tools/clang/docs/ReleaseNotes.html) with `-std=c++14`.

There is nothing very platform specific, so other reasonably similar system configurations should work.
