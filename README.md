<h1 align="center">
  Amazon Kinesis Video Streams C++ Producer, kvssink GStreamer Plugin, and Java Native Interface
  <br>
</h1>

<h4 align="center"> Amazon Kinesis Video Streams | Secure Video Ingestion for Analysis &amp; Storage </h4>

<p align="center">
  <a href="https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp/actions/workflows/ci.yml"> <img src="https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp/actions/workflows/ci.yml/badge.svg"> </a>
  <a href="https://codecov.io/gh/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp"> <img src="https://codecov.io/gh/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp/branch/master/graph/badge.svg" alt="Coverage Status"> </a>
</p>

<details open>
  
  <summary><h2>Table of Contents</h2></summary>

* [Key Features](#key-features)
* [Quick Start](#quick-start)
* [Build Options](#build-options)
* [Using kvssink](#using-kvssink)
* [Debugging](#debugging)
* [Development](#development)
* [Related](#related)
* [License](#license)
  
</details>

<br>

## Key Features
* C++ SDK with sample programs
* GStreamer Plugin (kvssink)
* Java Native Interface (JNI)

Amazon Kinesis Video Streams Producer SDK for C/C++ makes it easy to build an on-device application that securely connects to a video stream, and reliably publishes video and other media data to Kinesis Video Streams. It takes care of all the underlying tasks required to package the frames and fragments generated by the device's media pipeline. The SDK also handles stream creation, token rotation for secure and uninterrupted streaming, processing acknowledgements returned by Kinesis Video Streams, and other tasks.

<br>

## Quick Start
### Required Tools
The following packages are required to build the SDK libraries. Using a package manager such as _Homebrew_ (Mac), _APT_ (Linux), and _Chocolatey_ (Windows) is the prefered method of installation.
* C++ Compiler (GNU or Clang recommended)
* `git`
* `CMake`
* `pkg-config` for _Mac_/_Linux_, `pkgconfiglite` for _Windows_
* `m4`
* _Windows_ only: `nasm` and `strawberryperl`

<br>

### Install GStreamer
If building the samples or the kvssink GStreamer plugin, GStreamer libraries are required.

_Mac_
```bash
brew install gstreamer gst-plugins-base gst-plugins-good gst-plugins-bad gst-plugins-ugly gst-libav
```
_Linux_
```bash
sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-base-apps gstreamer1.0-plugins-bad gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-tools
```
_Windows_
```bat
choco install gstreamer --version=1.22.8
choco install gstreamer-devel --version=1.22.8
```
#### Verify GStreamer Installation
Run the following command to display the GStreamer version to confirm the installation was successful:
```bash
gst-launch-1.0 --gst-version
```

<br>

### Download
```bash
git clone https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp.git
```

<br>

### Build
#### Prepare a build directory in the newly checked out repository:
```bash
mkdir -p amazon-kinesis-video-streams-producer-sdk-cpp/build
cd amazon-kinesis-video-streams-producer-sdk-cpp/build
```
#### Configure the build:

_Mac and Linux_
```bash
cmake -DBUILD_GSTREAMER_PLUGIN=TRUE ..
```
_Windows_ (may need to run twice)
```bat
cmake -G "NMake Makefiles -DBUILD_GSTREAMER_PLUGIN=TRUE" ..
```


> [!NOTE]
> For more build configuration options, see [Cmake Arguments](#cmake-arguments).

#### Compile:
_Mac and Linux_
```bash
make
```
_Windows_
```bat
nmake
```

<br>

### Run the Samples
Included are sample applications for streaming and ingesting to a KVS stream. To stream to KVS, the GStreamer samples use an implemented `appsink` sink element, and the kvssink samples use the custom `kvssink` sink element.
> [!TIP]
> More on [GStreamer elements](https://gstreamer.freedesktop.org/documentation/application-development/basics/elements.html?gi-language=c).

<br>

#### GStreamer appsink Samples
To stream media from the device's camera and audio sources, create a stream in the [AWS KVS console](https://console.aws.amazon.com/kinesisvideo/home) and run the following sample:
```bash
./kvs_gstreamer_audio_video_sample <your-stream-name>
```

<br>

#### kvssink Samples
The SDK comes with two programmatic GStreamer samples: `kvssink_gstreamer_sample` and `kvssink_intermittent_sample`. For more use cases, see the CLI pipeline examples at [Example: Kinesis Video Streams Producer SDK GStreamer Plugin](https://docs.aws.amazon.com/kinesisvideostreams/latest/dg/examples-gstreamer-plugin.html).

The programmatic samples require the AWS region to be set with the `AWS_DEFAULT_REGION` environment variable. For example:
```bash
export AWS_DEFAULT_REGION=us-west-2
```

To load kvssink plugin into GStreamer, set the following environment variable from the `build` directory.

_Mac and Linux_
```bash
export GST_PLUGIN_PATH=`pwd`/.
```

_Windows_
```bat
set GST_PLUGIN_PATH=%CD%\.
```

Running the following command should now display information on the kvssink plugin:
```
gst-inspect-1.0 kvssink
```
If the build failed or GST_PLUGIN_PATH is not properly set, you may instead see the following output:<br>
`No such element or plugin kvssink`

After building the SDK, loading kvssink into the GStreamer plugin path, and setting a region, the sample executables, which are located in the `build` directory, can be run.

<br>

**Running the kvssink Intermittent Sample**
```bash
./kvssink_intermittent_sample <stream-name (optional)> <testsrc or devicesrc (optional)>
```
Setting the source to `testsrc` will use [videotestsrc](https://gstreamer.freedesktop.org/documentation/videotestsrc/?gi-language=c) and to `devicesrc` will use [autovideosrc](https://gstreamer.freedesktop.org/documentation/autodetect/autovideosrc.html?gi-language=c). By default, kvssink uses "DEFAULT_STREAM" as the stream name, and the sample uses videotestsrc as the source. If a KVS stream with the provided or default name does not exist, the stream will automatically be created.

The intermittent kvssink sample will stream video for 20 seconds, then pause for 40 seconds, and repeat until an interrupt signal is received. To manually adjust the streaming and paused intervals, you can change the `KVS_INTERMITTENT_PLAYING_INTERVAL_SECONDS` and `KVS_INTERMITTENT_PAUSED_INTERVAL_SECONDS` values in the *kvssink_intermittent_sample.cpp* file.

<br>

#### Running Samples in offline mode
By default, the samples run in near realtime mode. To set offline mode, set streamInfo.streamCaps.streamingType to `STREAMING_TYPE_OFFLINE`, where, `streamInfo` is of type `StreamInfo`, `streamCaps` is of type `StreamCaps` and `streamingType` is of type `STREAMING_TYPE`.

<br>

## Build Options
### Considerations
- The **kvssink** GStreamer plugin and samples, and the **JNI** are _not_ built by default. To build them, include their corresponding cmake command arguments: `cmake .. -DBUILD_GSTREAMER_PLUGIN=ON -DBUILD_JNI=TRUE`
- By default, the **dependency libraries** are installed from GitHub and built locally. To instead link to pre-installed libraries on the device, include the following cmake command argument: `cmake .. -DBUILD_DEPENDENCIES=OFF`
  - The depedency libraries are Curl, OpenSSL, and Log4Cplus.
 
### CMake Arguments
You can pass the following options to `cmake ..`.

* `-DBUILD_GSTREAMER_PLUGIN` -- Build kvssink GStreamer plugin
* `-DBUILD_JNI` -- Build C++ wrapper for JNI to expose the functionality to Java/Android
* `-DBUILD_DEPENDENCIES` -- Build depending libraries from source
* `-DBUILD_TEST=TRUE` -- Build unit/integration tests, may be useful for confirm support for your device. `./tst/producerTest`
* `-DCODE_COVERAGE` --  Enable coverage reporting
* `-DCOMPILER_WARNINGS` -- Enable all compiler warnings
* `-DADDRESS_SANITIZER` -- Build with AddressSanitizer
* `-DMEMORY_SANITIZER` --  Build with MemorySanitizer
* `-DTHREAD_SANITIZER` -- Build with ThreadSanitizer
* `-DUNDEFINED_BEHAVIOR_SANITIZER` Build with UndefinedBehaviorSanitizer
* `-DALIGNED_MEMORY_MODEL` Build for aligned memory model only devices. Default is OFF.
* `-DBUILD_LOG4CPLUS_HOST` Specify host-name for log4cplus for cross-compilation. Default is OFF.
* `-DCONSTRAINED_DEVICE` Set the thread stack size to 0.5MB, needed for Alpine builds


### Setting the Log Level
To set a log level, update the log level value [here](https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp/blob/master/kvs_log_configuration#L1).

The log levels currently available with `log4cplus` are:
1. `TRACE`
2. `DEBUG` 
3. `INFO`   
4. `WARN`   
5. `ERROR`   
6. `FATAL`   

Note: The default log level is `DEBUG`.

The SDK also tracks entry and exit of functions which increases the verbosity of the logs. This will be useful when you want to track the transitions within the codebase. To do so, you need to set log level to TRACE and add the following to the cmake file:
`add_definitions(-DLOG_STREAMING)`
Note: This log level is extremely VERBOSE and could flood the files if using file based logging strategy.


### Installing the Library
If the SDK library needs to be installed on your system rather than the local `build` directory, run `make install`. This will install in the default directory such as `usr/local/lib/`, based on the system. To install in another directory, run `cmake` with the `-DCMAKE_INSTALL_PREFIX` option with the desired directory before running `make install`

### Cross-Compilation
If you wish to cross-compile `CC` and `CXX` are respected when building the library and all its dependencies. See the [ci.yml](https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp/blob/develop/.github/workflows/ci.yml) for an example. Every commit is tested for cross compilation.
Please note that GStreamer is not cross-compiled as a part of the cross-compilation of the KVS-SDK, you will have to cross-compile it separately.

### Docker Scripts
The sample docker scripts for RTSP plugin, raspberry pi and linux can be found in the [Kinesis demos repository](https://github.com/aws-samples/amazon-kinesis-video-streams-demos/tree/master/producer-cpp).

<br>

## Using kvssink
The kvssink element includes the following parameters:

* `stream-name` -- The name of the destination Kinesis video stream. If not set, kvssink will use "DEFAULT_STREAM" as the stream name. If a KVS stream with the provided or default name does not exist, the stream will automatically be created.
* `storage-size` -- The storage size of the device in megabytes. For information about configuring device storage, see StorageInfo. If not set, it will default to 128 MB.
* `access-key` -- The AWS access key that is used to access Kinesis Video Streams. You must provide either this parameter or credential-path, or set the AWS_ACCESS_KEY_ID environment variable.
* `secret-key` -- The AWS secret key that is used to access Kinesis Video Streams. You must provide either this parameter or credential-path, or set the AWS_SECRET_ACCESS_KEY environment variable.
* `credential-path` -- A path to a file containing your credentials for accessing Kinesis Video Streams. For example credential files, see Sample Static Credential and Sample Rotating Credential. For more information on rotating credentials, see Managing Access Keys for IAM Users. You must provide either this parameter or access-key and secret-key, or set the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY environment variables.

To see all kvssink parameters, see [AWS Docs - kvssink Paramters](https://docs.aws.amazon.com/kinesisvideostreams/latest/dg/examples-gstreamer-plugin-parameters.html#kvssink-optional-parameters)

For examples of common use cases, see [Example: Kinesis Video Streams Producer SDK GStreamer Plugin](https://docs.aws.amazon.com/kinesisvideostreams/latest/dg/examples-gstreamer-plugin.html)

<br>

## Debugging
* When building the JNI, if you run into a cmake error `Could NOT find JNI (missing: JAVA_INCLUDE_PATH JAVA_INCLUDE_PATH2 JAVA_AWT_INCLUDE_PATH)`, make sure Java is installed and your environment variables are set correctly:  
`export JAVA_INCLUDE_PATH2=/Library/Java/JavaVirtualMachines/<YOUR_JDK_VERSION>/Contents/Home/include` or `export JAVA_INCLUDE_PATH2=$JAVA_HOME/include` for Mac OS.  
`export JAVA_INCLUDE_PATH2='/usr/java/<JDK_VERSION>/include'` for Linux.
* If you are successfully streaming but run into issue with playback. You can do `export KVS_DEBUG_DUMP_DATA_FILE_DIR=/path/to/directory` before streaming. Producer will then dump MKV files into that path. The file is exactly what KVS will receive. You can use [MKVToolNIX](https://mkvtoolnix.download/index.html) to check that everything looks correct. You can also try to play the MKV file in compatible players.
* If you are running into issues building libcurl on M1 Mac, you can try `brew unlink openssl`.
* If you would like to visualize the GStreamer pipeline being constructed in a GStreamer application, include the following after the elements have been linked:
`GST_DEBUG_BIN_TO_DOT_FILE(<gst-bin-object>, GST_DEBUG_GRAPH_SHOW_ALL, <file-name>);`
For example, if the application created a pipeline object `GstPipeline* pipeline = gst_pipeline_new("test-pipeline")`, and you would like to see the visualized pipeline with filename pipeline, add:
`GST_DEBUG_BIN_TO_DOT_FILE((GstBin*) pipeline, GST_DEBUG_GRAPH_SHOW_ALL, "pipeline");`. Also ensure to set the path to where you would like the file to be stored. `export GST_DEBUG_DUMP_DOT_DIR=.`. The file generated would be a `.dot` format. Convert to PDF to check the visualized pipeline. Also, this requires `graphviz` to be installed. So make sure to install that.

### FAQ
* Is CPP-SDK and GStreamer supported on Mac/Windows/Linux (Supported Platforms)?  
Yes! We have FAQs and platform specific instructions for [Windows](docs/windows.md), [MacOS](docs/macos.md) and [Linux](docs/linux.md)

<br>

## Development

The repository is using develop branch as the aggregation and all of the feature development is done in appropriate feature branches. The PRs (Pull Requests) are cut on a feature branch and once approved with all the checks passed they can be merged by a click of a button on the PR tool. The master branch should always be build-able and all the tests should be passing. We are welcoming any contribution to the code base. The master branch contains our most recent release cycle from develop.

### Release
The repository is under active development and even with incremental unit test coverage where some of the tests are actually full integration tests, we require more rigorous internal testing in order to 'cut' release versions. The release is cut against a particular commit that gets approved. The general philosophy is to cut a release when a set of commits contribute to a self-containing feature or when we add major internal functionality improvements.

### Versioning
We deploy 3 digit version strings in a form of 'Major.Minor.Revision' scheme.
* Major version update - Major functionality changes. Might not have direct backward compatibility. For example, multiple public API parameter changes.
* Minor version update - Additional features. Major bug fixes. Might have some minor backward compatibility issues. For example, an extra parameter on a callback function.
* Revision version update - Minor features. Bug fixes. Full backward compatibility. For example, an extra fields added to the public structures with version bump.

## Related
* [What Is Amazon Kinesis Video Streams](https://docs.aws.amazon.com/kinesisvideostreams/latest/dg/what-is-kinesis-video.html)
* [C SDK](https://github.com/awslabs/amazon-kinesis-video-streams-producer-c)
* [Example: Kinesis Video Streams Producer SDK GStreamer Plugin](https://docs.aws.amazon.com/kinesisvideostreams/latest/dg/examples-gstreamer-plugin.html)

## License

This library is licensed under the Apache 2.0 License.
