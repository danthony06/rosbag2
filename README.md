# rosbag2
![License](https://img.shields.io/github/license/ros2/rosbag2)
[![GitHub Action Status](https://github.com/ros2/rosbag2/workflows/Test%20rosbag2/badge.svg)](https://github.com/ros2/rosbag2/actions)

Repository for implementing rosbag2 as described in its corresponding [design article](https://github.com/ros2/design/blob/ros2bags/articles/rosbags.md).

## Installation instructions

## Debian packages

rosbag2 packages are available via debian packages and thus can be installed via

```
$ export CHOOSE_ROS_DISTRO=crystal # rosbag2 is available starting from crystal
$ sudo apt-get install ros-$CHOOSE_ROS_DISTRO-ros2bag ros-$CHOOSE_ROS_DISTRO-rosbag2*
```

Note that the above command installs all packages related to rosbag2.
This also includes the plugin for [reading ROS1 bag files](https://github.com/ros2/rosbag2_bag_v2), which brings a hard dependency on the [ros1_bridge](https://github.com/ros2/ros1_bridge) with it and therefore ROS1 packages.
If you want to install only the ROS2 related packages for rosbag, please use the following command:

```
$ export CHOOSE_ROS_DISTRO=crystal # rosbag2 is available starting from crystal
$ sudo apt-get install ros-$CHOOSE_ROS_DISTRO-ros2bag ros-$CHOOSE_ROS_DISTRO-rosbag2-transport
```

## Build from source

It is recommended to create a new overlay workspace on top of your current ROS 2 installation.

```
$ mkdir -p ~/rosbag_ws/src
$ cd ~/rosbag_ws/src
```

Clone this repository into the source folder:

```
$ git clone https://github.com/ros2/rosbag2.git
```
**[Note]**: if you are only building rosbag2 on top of a Debian Installation of ROS2, please git clone the branch following your current ROS2 distribution.

Then build all the packages with this command:

```
$ colcon build [--merge-install]
```

The `--merge-install` flag is optional and installs all packages into one folder rather than isolated folders for each package.

#### Executing tests

The tests can be run using the following commands:

```
$ colcon test [--merge-install]
$ colcon test-result --verbose
```

The first command executes the test and the second command displays the errors (if any).

## Using rosbag2

rosbag2 is part of the ROS 2 command line interfaces.
This repo introduces a new verb called `bag` and thus serves as the entry point of using rosbag2.
As of the time of writing, there are three commands available for `ros2 bag`:

* record
* play
* info

### Recording data

In order to record all topics currently available in the system:

```
$ ros2 bag record -a
```

The command above will record all available topics and discovers new topics as they appear while recording.
This auto-discovery of new topics can be disabled by given the command line argument `--no-discovery`.

To record a set of predefined topics, one can specify them on the command line explicitly.

```
$ ros2 bag record <topic1> <topic2> … <topicN>
```

The specified topics don't necessarily have to be present at start time.
The discovery function will automatically recognize if one of the specified topics appeared.
In the same fashion, this auto discovery can be disabled with `--no-discovery`.

If not further specified, `ros2 bag record` will create a new folder named to the current time stamp and stores all data within this folder.
A user defined name can be given with `-o, --output`.

#### Splitting recorded bag files

rosbag2 offers the capability to split bag files when they reach a maximum size or after a specified duration. By default rosbag2 will record all data into a single bag file, but this can be changed using the CLI options.

_Splitting by size_: `ros2 bag record -a -b 100000` will split the bag files when they become greater than 100 kilobytes. Note: the batch size's units are in bytes and must be greater than `86016`. This option defaults to `0`, which means data is written to a single file.

_Splitting by time_: `ros2 bag record -a -d 9000` will split the bag files after a duration of `9000` seconds. This option defaults to `0`, which means data is written to a single file.

If both splitting by size and duration are enabled, the bag will split at whichever threshold is reached first.

#### Recording with compression

By default rosbag2 does not record with compression enabled. However, compression can be specified using the following CLI options.

For example, `ros2 bag record -a --compression-mode file --compression-format zstd` will record all topics and compress each file using the [zstd](https://github.com/facebook/zstd) compressor.

Currently, the only `compression-format` available is `zstd`. Both the mode and format options default to `none`. To use a compression format, a compression mode must be specified, where the currently supported modes are compress by `file` or compress by `message`.

It is recommended to use this feature with the splitting options.

#### Recording with a storage configuration

Storage configuration can be specified in a YAML file passed through the `--storage-config-file` option.
This can be used to optimize performance for specific use cases.

See storage plugin documentation for more detail:
* [mcap](rosbag2_storage_mcap/README.md#writer-configuration)
* [sqlite3](rosbag2_storage_sqlite3/README.md#storage-configuration-file)


### Replaying data

After recording data, the next logical step is to replay this data:

```
$ ros2 bag play <bag_file>
```

The bag file is by default set to the folder name where the data was previously recorded in.

### Analyzing data

The recorded data can be analyzed by displaying some meta information about it:

```
$ ros2 bag info <bag_file>
```

You should see something along these lines:

```
Files:             demo_strings.db3
Bag size:          44.5 KiB
Storage id:        sqlite3
Duration:          8.501s
Start:             Nov 28 2018 18:02:18.600 (1543456938.600)
End                Nov 28 2018 18:02:27.102 (1543456947.102)
Messages:          27
Topic information: Topic: /chatter | Type: std_msgs/String | Count: 9 | Serialization Format: cdr
                   Topic: /my_chatter | Type: std_msgs/String | Count: 18 | Serialization Format: cdr
```

### Converting bags

Rosbag2 provides a tool `ros2 bag convert` (or, `rosbag2_transport::bag_rewrite` in the C++ API).
This allows the user to take one or more input bags, and write them out to one or more output bags with new settings.
This flexible feature enables the following features:
* Merge (multiple input bags, one output bag)
* Split top-level bags (one input bag, multiple output bags)
* Split internal files (by time or size - one input bag with fewer internal files, one output bag with more, smaller, internal files)
* Compress/Decompress (output bag(s) with different compression settings than the input(s))
* Serialization format conversion
* ... and more!

Here is an example command:

```
ros2 bag convert --input /path/to/bag1 --input /path/to/bag2 storage_id --output-options output_options.yaml
```

The `--input` argument may be specified any number of times, and takes 1 or 2 values.
The first value is the URI of the input bag.
If a second value is supplied, it specifies the storage implementation of the bag.
If no storage implementation is specified, rosbag2 will try to determine it automatically from the bag.

The `--output-options` argument must point to the URI of a YAML file specifying the full recording configuration for each bag to output (`StorageOptions` + `RecordOptions`).
This file must contain a top-level key `output_bags`, which contains a list of these objects.

The only required value in the output bags is `uri` and `storage_id`. All other values are options (however, if no topic selection is specified, this output bag will be empty!).

This example notes all fields that can have an effect, with a comment on the required ones.

```
output_bags:
- uri: /output/bag1  # required
  storage_id: ""  # will use the default storage plugin, if unspecified
  max_bagfile_size: 0
  max_bagfile_duration: 0
  storage_preset_profile: ""
  storage_config_uri: ""
  all: false
  topics: []
  rmw_serialization_format: ""  # defaults to using the format of the input topic
  regex: ""
  exclude: ""
  compression_mode: ""
  compression_format: ""
  compression_queue_size: 1
  compression_threads: 0
  include_hidden_topics: false
  include_unpublished_topics: false
```

Example merge:

```
$ ros2 bag convert -i bag1 -i bag2 -o out.yaml

# out.yaml
output_bags:
- uri: merged_bag
  all: true
```

Example split:

```
$ ros2 bag convert -i bag1 -o out.yaml

# out.yaml
output_bags:
- uri: split1
  topics: [/topic1, /topic2]
- uri: split2
  topics: [/topic3]
```

Example compress:

```
$ ros2 bag convert -i bag1 -o out.yaml

# out.yaml
output_bags:
- uri: compressed
  all: true
  compression_mode: file
  compression_format: zstd
```

### Overriding QoS Profiles

When starting a recording or playback workflow, you can pass a YAML file that contains QoS profile settings for a specific topic.
The YAML schema for the profile overrides is a dictionary of topic names with key/value pairs for each QoS policy.
Below is an example profile set to the default ROS2 QoS settings.

```yaml
/topic_name:
  history: keep_last
  depth: 10
  reliability: reliable
  durability: volatile
  deadline:
    # unspecified/infinity
    sec: 0
    nsec: 0
  lifespan:
    # unspecified/infinity
    sec: 0
    nsec: 0
  liveliness: system_default
  liveliness_lease_duration:
    # unspecified/infinity
    sec: 0
    nsec: 0
  avoid_ros_namespace_conventions: false
```

You can then use the override by specifying the `--qos-profile-overrides-path` argument in the CLI:

```sh
# Record
ros2 bag record --qos-profile-overrides-path override.yaml -a -o my_bag
# Playback
ros2 bag play --qos-profile-overrides-path override.yaml my_bag
```

See [the official QoS override tutorial][qos-override-tutorial] and ["About QoS Settings"][about-qos-settings] for more detail.

### Using in launch

We can invoke the command line tool from a ROS launch script as an *executable* (not a *node* action).
For example, to launch the command to record all topics you can use the following launch script:

```xml
<launch>
  <executable cmd="ros2 bag record -a" output="screen" />
</launch>
```

Here's the equivalent Python launch script:

```python
import launch


def generate_launch_description():
    return launch.LaunchDescription([
        launch.actions.ExecuteProcess(
            cmd=['ros2', 'bag', 'record', '-a'],
            output='screen'
        )
    ])
```

Use the `ros2 launch` command line tool to launch either of the above launch scripts.
For example, if we named the above XML launch script, `record_all.launch.xml`:

```sh
$ ros2 launch record_all.launch.xml
```

## Storage format plugin architecture

Looking at the output of the `ros2 bag info` command, we can see a field `Storage id:`.
Rosbag2 was designed to support multiple storage formats to adapt to individual use cases.
This repository provides two storage plugins, `mcap` and `sqlite3`.
The default is `mcap`, which is provided to code by [`rosbag2_storage::get_default_storage_id()`](rosbag2_storage/include/rosbag2_storage/default_storage_id.hpp) and defined in [`default_storage_id.cpp`](rosbag2_storage/src/rosbag2_storage/default_storage_id.cpp#L21)

If not specified otherwise, rosbag2 will write data using the default plugin.

In order to use a specified (non-default) storage format plugin, rosbag2 has a command line argument `--storage`:

```
$ ros2 bag record --storage <storage_id>
```

Bag reading commands can detect the storage plugin automatically, but if for any reason you want to force a specific plugin to read a bag, you can use the `--storage` option on any `ros2 bag` verb.

To write your own Rosbag2 storage implementation, refer to [this document describing that process](docs/storage_plugin_development.md)


## Serialization format plugin architecture

Looking further at the output of `ros2 bag info`, we can see another field attached to each topic called `Serialization Format`.
By design, ROS 2 is middleware agnostic and thus can leverage multiple communication frameworks.
The default middleware for ROS 2 is DDS which has `cdr` as its default binary serialization format.
However, other middleware implementation might have different formats.
If not specified, `ros2 bag record -a` will record all data in the middleware specific format.
This however also means that such a bag file can't easily be replayed with another middleware format.

rosbag2 implements a serialization format plugin architecture which allows the user the specify a certain serialization format.
When specified, rosbag2 looks for a suitable converter to transform the native middleware protocol to the target format.
This also allows to record data in a native format to optimize for speed, but to convert or transform the recorded data into a middleware agnostic serialization format.

By default, rosbag2 can convert from and to CDR as it's the default serialization format for ROS 2.

[qos-override-tutorial]: https://docs.ros.org/en/rolling/Guides/Overriding-QoS-Policies-For-Recording-And-Playback.html
[about-qos-settings]: https://docs.ros.org/en/rolling/Concepts/About-Quality-of-Service-Settings.html
