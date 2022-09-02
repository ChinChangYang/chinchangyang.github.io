---
layout: post
title:  "Improve KataGo performance by CoreML backend on MacOS"
date:   2022-08-20 22:56:00 +0800
categories: github katago coreml apple m1 arm64
---
I improved KataGo performance on MacOS by running OpenCL and CoreML simultaneously. Let's see how I did it.
# My Environment
- Apple M1 Pro.
- MacBook Air (Intel).

# Generate a KataGo CoreML model
KataGo will run a [CoreML](https://developer.apple.com/documentation/coreml) model to utilize [Apple Neural Engine (ANE)](https://machinelearning.apple.com/research/neural-engine-transformers). First, we need to convert [KataGo network](https://katagotraining.org/networks/) to a CoreML model. The steps are shown as follows.
1. In MacBook Air, install [Tensorflow 1.5](https://www.tensorflow.org/install/pip).
2. Follow the comment of this [gist](https://gist.github.com/ChinChangYang/5e9d7ff5900f7be8af0f226689e84e7a).

It will convert KataGo network to a CoreML model, `KataGoModel.mlpackage`. Then, we need to copy the CoreML model to somewhere Apple M1 Pro can access.

Note that it is very hard to generate a KataGo CoreML model on Apple M1 Pro because of the following facts.
- [KataGo's Python scripts](https://github.com/lightvector/KataGo/tree/master/python) only supports Tensorflow 1.x.
- Apple doesn't support Tensorflow 1.x on Apple M1 Pro. See this [link](https://developer.apple.com/forums/thread/704041) for reference.

So the only computer that can generate a KataGo CoreML model is MacBook Air for me.
# Get KataGo CoreML backend source code
KataGo CoreML backend is done in my [`v1.11.0-coreml1`](https://github.com/ChinChangYang/KataGo/tree/v1.11.0-coreml1) tag. We need to clone the source code. The command is shown as below.
```
git clone https://github.com/ChinChangYang/KataGo.git -b v1.11.0-coreml1
```
It will create a subdirectory `KataGo` in the current directory, and download the source code to `KataGo/`.
# Generate an Xcode project from KataGo
KataGo with CoreML backend will be built in [Xcode](https://developer.apple.com/xcode/) project, so we need to generate an Xcode project by [CMake](https://formulae.brew.sh/formula/cmake). The commands are shown as below.
```
cd KataGo
mkdir xcode
cd xcode
cmake -G Xcode ../cpp -DUSE_BACKEND=OPENCL
```
It will create a subdirectory `xcode` in the `KataGo` directory, and generate Xcode project files into `xcode/`.

The terminal should show something like below:
```
...
-- -DUSE_BACKEND=OPENCL, using OpenCL+CoreML backend.
...
-- Build files have been written to: /path/to/katago/xcode
```
# Configure KataGo Xcode project
The generated Xcode project has not been configured with:
- Release mode
- CoreML backend Swift source file
- CoreML model

Therefore, we need to manually configure Xcode project to fix it. The steps are shown as follows.

In Terminal, open the generated Xcode project by the following command:
```
open katago.xcodeproj
```
It will open Xcode graphical user interface (GUI).
## Release mode
In Xcode GUI, click the following menu items, buttons, and lists.

- Product -> Scheme -> katago

![Xcode-Product](/assets/Xcode-Product.png)
![Xcode-Product-Scheme](/assets/Xcode-Product-Scheme.png)
![Xcode-Product-Scheme-katago](/assets/Xcode-Product-Scheme-katago.png)

- Product -> Scheme -> Edit Scheme

![Xcode-Product-Edit-Scheme](/assets/Xcode-Product-Edit-Scheme.png)

- Run -> Info -> Build Configuration -> Debug

![Xcode-Scheme-Dialog](/assets/Xcode-Scheme-Dialog.png)

- Change Debug to Release.

![Xcode-Release](/assets/Xcode-Release.png)

- Click Close

It will configure KataGo build to release mode that is much faster than debug mode.
## CoreML backend Swift source file
In Xcode GUI, click the following menu items, list items, and buttons.
1. File -> Add Files to "katago" -> cpp/neuralnet/coremlbackend.swift
2. "Add to targets:" -> katago
3. Add

![Xcode-File](/assets/Xcode-File.png)
![Xcode-Add-Files](/assets/Xcode-Add-Files.png)
![Xcode-Add-Files-Dialog](/assets/Xcode-Add-Files-Dialog.png)

It will show a dialog to ask you: "Would you like to configure an Objective-C bridging header?" We just need to answer yes by clicking "Create Bridging Header".

![Xcode-Create-Bridging-Header](/assets/Xcode-Create-Bridging-Header.png)

Then, it will add `coremlbackend.swift` and `katago-Briding-Header.h` to Xcode project.
## CoreML model
In Xcode GUI, click the following menu items, list items, and buttons.
1. File -> Add Files to "katago" -> KataGoModel.mlpackage
2. "Add to targets:" -> katago
3. Add

![Xcode-File](/assets/Xcode-File.png)
![Xcode-Add-Files](/assets/Xcode-Add-Files.png)
![Xcode-Add-Files-CoreML](/assets/Xcode-Add-Files-CoreML.png)

It will add the CoreML model to Xcode project.

In Xcode GUI, click the following list items.
1. Project navigator -> katago
2. TARGETS -> katago
3. Build Settings -> CoreML Model Class Generation Language -> Automatic

![Xcode-Build-Settings](/assets/Xcode-Build-Settings.png)

- Change Automatic to Swift.

![Xcode-CoreML-Model-Class](/assets/Xcode-CoreML-Model-Class.png)

It will configure Xcode project to generate a Swift source file of CoreML model when we build KataGo.
# Build KataGo
Xcode project has been configured correctly. Now we can build KataGo by clicking the menu items below.

1. Product -> Build

![Xcode-Product](/assets/Xcode-Product.png)
![Xocde-Product-Build](/assets/Xcode-Product-Build.png)

If build is successful, it will show "Build Succeeded"
# Run KataGo with CoreML config
The KataGo executable has been built to `Release/katago`. We need to specify a CoreML config file to run KataGo correctly. An example config file can be found in [`../cpp/configs/misc/coreml_example.cfg`](https://github.com/ChinChangYang/KataGo/blob/coreml-backend/cpp/configs/misc/coreml_example.cfg).

The CoreML example config file sets the following parameters that are different with the GTP example config file ([`gtp_example.cfg`](https://github.com/ChinChangYang/KataGo/blob/coreml-backend/cpp/configs/gtp_example.cfg)).
- `numNNServerThreadsPerModel = 2`
- `openclDeviceToUseThread0 = 0`
- `openclDeviceToUseThread1 = 1`

It increases the number of server threads by 1, and configures [OpenCL](https://www.khronos.org/opencl/) devices to use thread 0 and thread 1.

The KataGo that were built from the source code of my [`coreml-backend`](https://github.com/ChinChangYang/KataGo/tree/coreml-backend) branch simply replaces the first OpenCL device by the CoreML model, so KataGo will actually run CoreML and OpenCL backends in thread 0 and thread 1, respectively.

- CoreML backend -> Thread 0
- OpenCL backend -> Thread 1

In Terminal, enter the following command to run KataGo GTP.
```
./Release/katago gtp -config ../cpp/configs/misc/coreml_example.cfg
```

On MacBook M1 Pro, it showed:
```
KataGo v1.11.0
Using TrompTaylor rules initially, unless GTP/GUI overrides this
Creating context for OpenCL Platform: Apple (Apple) (OpenCL 1.2 (Jun 17 2022 18:58:05))
Creating context for OpenCL Platform: Apple (Apple) (OpenCL 1.2 (Jun 17 2022 18:58:05))
Using OpenCL Device 0: Apple M1 Pro (Intel) OpenCL 1.2  (Extensions: cl_APPLE_SetMemObjectDestructor cl_APPLE_ContextLoggingFunctions cl_APPLE_clut cl_APPLE_query_kernel_names cl_APPLE_gl_sharing cl_khr_gl_event cl_khr_fp64 cl_khr_global_int32_base_atomics cl_khr_global_int32_extended_atomics cl_khr_local_int32_base_atomics cl_khr_local_int32_extended_atomics cl_khr_byte_addressable_store cl_khr_int64_base_atomics cl_khr_int64_extended_atomics cl_khr_3d_image_writes cl_khr_image2d_from_buffer cl_APPLE_fp64_basic_ops cl_APPLE_fixed_alpha_channel_orders cl_APPLE_biased_fixed_point_image_formats cl_APPLE_command_queue_priority)
Using OpenCL Device 1: Apple M1 Pro (Apple) OpenCL 1.2  (Extensions: cl_APPLE_SetMemObjectDestructor cl_APPLE_ContextLoggingFunctions cl_APPLE_clut cl_APPLE_query_kernel_names cl_APPLE_gl_sharing cl_khr_gl_event cl_khr_byte_addressable_store cl_khr_global_int32_base_atomics cl_khr_global_int32_extended_atomics cl_khr_local_int32_base_atomics cl_khr_local_int32_extended_atomics cl_khr_3d_image_writes cl_khr_image2d_from_buffer cl_khr_depth_images )
Loaded tuning parameters from: /Users/chinchangyang/.katago/opencltuning/tune8_gpuAppleM1Pro_x19_y19_c256_mv10.txt
Loaded tuning parameters from: /Users/chinchangyang/.katago/opencltuning/tune8_gpuAppleM1Pro_x19_y19_c256_mv10.txt
Initializing board with boardXSize 19 boardYSize 19
Loaded config ../cpp/configs/misc/coreml_example.cfg
Loaded model /Users/chinchangyang/.katago/default_model.bin.gz
Model name: kata1-b40c256-s11840935168-d2898845681
GTP ready, beginning main protocol loop
```

Then, we can issue commands to KataGo. For example:
```
genmove b
= Q16

quit
= 
```

# Test KataGo Performance

In Terminal, enter the following command to test KataGo performance of OpenCL+CoreML backends.
```
./Release/katago benchmark -config ../cpp/configs/misc/coreml_example.cfg
```
On MacBook M1 Pro, it showed:
```
Ordered summary of results: 

numSearchThreads =  4: 10 / 10 positions, visits/s = 241.68 nnEvals/s = 197.04 nnBatches/s = 150.84 avgBatchSize = 1.31 (33.2 secs) (EloDiff baseline)
numSearchThreads =  6: 10 / 10 positions, visits/s = 278.96 nnEvals/s = 232.70 nnBatches/s = 130.09 avgBatchSize = 1.79 (28.9 secs) (EloDiff +43)
numSearchThreads =  8: 10 / 10 positions, visits/s = 287.02 nnEvals/s = 241.13 nnBatches/s = 115.68 avgBatchSize = 2.08 (28.1 secs) (EloDiff +43)
numSearchThreads = 10: 10 / 10 positions, visits/s = 308.15 nnEvals/s = 260.96 nnBatches/s = 93.74 avgBatchSize = 2.78 (26.3 secs) (EloDiff +60)
numSearchThreads = 12: 10 / 10 positions, visits/s = 305.89 nnEvals/s = 259.76 nnBatches/s = 86.50 avgBatchSize = 3.00 (26.5 secs) (EloDiff +47)
numSearchThreads = 20: 10 / 10 positions, visits/s = 333.71 nnEvals/s = 290.55 nnBatches/s = 58.40 avgBatchSize = 4.97 (24.5 secs) (EloDiff +43)


Based on some test data, each speed doubling gains perhaps ~250 Elo by searching deeper.
Based on some test data, each thread costs perhaps 7 Elo if using 800 visits, and 2 Elo if using 5000 visits (by making MCTS worse).
So APPROXIMATELY based on this benchmark, if you intend to do a 5 second search: 
numSearchThreads =  4: (baseline)
numSearchThreads =  6:   +43 Elo
numSearchThreads =  8:   +43 Elo
numSearchThreads = 10:   +60 Elo (recommended)
numSearchThreads = 12:   +47 Elo
numSearchThreads = 20:   +43 Elo

If you care about performance, you may want to edit numSearchThreads in ../cpp/configs/coreml_example.cfg based on the above results!
If you intend to do much longer searches, configure the seconds per game move you expect with the '-time' flag and benchmark again.
If you intend to do short or fixed-visit searches, use lower numSearchThreads for better strength, high threads will weaken strength.
If interested see also other notes about performance and mem usage in the top of ../cpp/configs/coreml_example.cfg

2022-08-20 14:34:41+0800: GPU 0 finishing, processed 25817 rows 11265 batches
2022-08-20 14:34:41+0800: GPU 1 finishing, processed 15095 rows 6945 batches
```

# Known Issues
- Only support 19x19 board size
- Not support contributing self-play games because CoreML model cannot be updated from [KataGo Distributed Training](https://katagotraining.org) server.
