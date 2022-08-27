---
layout: post
title:  "Fix a Memory Leak of Metal Backend of Leela Chess Zero"
date:   2022-08-20 22:56:00 +0800
categories: github lc0 metal
---
I fixed a [memory leak](https://en.wikipedia.org/wiki/Memory_leak) of [Metal](https://developer.apple.com/metal/) [backend](https://en.wikipedia.org/wiki/Frontend_and_backend) of [Leela Chess Zero](https://github.com/LeelaChessZero/lc0) (lc0). Let's see how I did it.

# Get lc0 Metal backend source code
The lc0 Metal backend is developed in [almaudoh](https://github.com/almaudoh)'s [`new-metal-backend-mpsgraph`](https://github.com/almaudoh/lc0/tree/new-metal-backend-mpsgraph) branch. I [cloned](https://git-scm.com/docs/git-clone) the source code with the following commands.

```
git clone -b new-metal-backend-mpsgraph --recurse-submodules https://github.com/almaudoh/lc0.git lc0-almaudoh
cd lc0-almaudoh
git checkout 055e19e4081a32fdb763569375eaa05c7a2be37f
```

It will create a subdirectory `lc0-almaudoh` in the current directory, and download the source code to `lc0-almaudoh/`. Then, it will [checkout](https://git-scm.com/docs/git-checkout) the revision [`055e19e4081a32fdb763569375eaa05c7a2be37f`](https://github.com/almaudoh/lc0/commit/055e19e4081a32fdb763569375eaa05c7a2be37f) that has a memory leak.

# Generate an Xcode project from lc0
I detect memory leaks by [Xcode Instruments](https://developer.apple.com/xcode/features/), so I need to generate an [Xcode project](https://developer.apple.com/documentation/xcode/projects-and-workspaces) from lc0. The commands are shown as below.

```
cd lc0-almaudoh
meson setup build --backend xcode
```

It will create a subdirectory `build` in the `lc0-almaudoh` directory, and generate Xcode project files into `build/lc0.xcodeproj`.

# Configure lc0 Xcode project
The generated Xcode project has not been configured with:

- [Scheme](https://developer.apple.com/documentation/xcode/customizing-the-build-schemes-for-a-project) of lc0 [executable](https://en.wikipedia.org/wiki/Executable)
- [Argument passed on launch](https://developer.apple.com/documentation/xcode/customizing-the-build-schemes-for-a-project#Specify-Launch-Arguments-and-Environment-Variables)

Therefore, I manually select the scheme of lc0 executable, and specify a launch argument. The steps are shown as follows.

In [Terminal](https://support.apple.com/guide/terminal/welcome/mac), open the generated Xcode project by the following command:
```
open build/lc0.xcodeproj
```
It will open Xcode graphical user interface (GUI).

## Scheme of lc0 executable
In Xcode GUI, click the following items.
- Product -> Scheme -> lc0@exe

![Xcode-Product](/assets/Xcode-Product.png)
![Xcode-Product-Scheme](/assets/Xcode-Product-Scheme.png)
![Xcode-Product-Scheme-lc0](/assets/Xcode-Product-Scheme-lc0.png)

It selects the scheme of lc0 executable.

## Argument passed on launch
In Xcode GUI, click the following items.
- Product -> Scheme -> Edit Scheme...

![Xcode-Product-Edit-Scheme](/assets/Xcode-Product-Edit-Scheme-lc0.png)

- Run -> Arguments -> +

![Xcode-Arguments-Add-Items](/assets/Xcode-Arguments-Add-Items.png)

- Enter "`-b metal`" -> Close

![Xcode-Argument-Metal](/assets/Xcode-Argument-Metal.png)

It will set lc0's backend to Metal.

# Hack universal chess interface

The memory leak can be easily detected by running lc0 infinitely. Unfortunately, Xcode Instruments cannot run lc0 infinitely because of the following facts:

- The lc0 waits for [commands](http://wbec-ridderkerk.nl/html/UCIProtocol.html) from [standard input stream](https://en.wikipedia.org/wiki/Standard_streams#Standard_input_(stdin)).
- Xcode Instruments does not seem to allow a user entering commands to standard input stream.
- [Passing a command file to standard input stream](https://stackoverflow.com/questions/25985639/passing-text-file-to-standard-input) makes lc0 terminate  after lc0 processed all commands in the command file.

Therefore, I hacked [universal chess interface](http://wbec-ridderkerk.nl/html/UCIProtocol.html), so that lc0 can receive a command that makes lc0 search infinitely on launch.

I modified the source code as follows.

```
src/chess/uciloop.cc:132:
void UciLoop::RunLoop() {
  std::cout.setf(std::ios::unitbuf);
  std::string line;

  line = "go infinite";
  LOGFILE << ">> " << line;
  try {
    auto command = ParseCommand(line);
    DispatchCommand(command.first, command.second);
  } catch (Exception& ex) {
    SendResponse(std::string("error ") + ex.what());
  }

  while (std::getline(std::cin, line)) {
    LOGFILE << ">> " << line;
    try {
      auto command = ParseCommand(line);
      // Ignore empty line.
      if (command.first.empty()) continue;
      if (!DispatchCommand(command.first, command.second)) break;
    } catch (Exception& ex) {
      SendResponse(std::string("error ") + ex.what());
    }
  }
}
```

It will let lc0 process an additional command `go infinite` before reading data from standard input stream.

# Build lc0 executable

The lc0 executable is ready to be built from the source code. I built lc0 executable by the following steps.

- Product -> Build

![Xocde-Product-Build](/assets/Xcode-Product-Build.png)

It will build lc0 executable to `build/debug/lc0`.

# Download leela chess zero network

The lc0 evaluates a [chess](https://en.wikipedia.org/wiki/Chess) move by [neural network](https://lczero.org/play/networks/bestnets/), so I need to download a neural network to let lc0 use. The steps are shown as below.

1. In Safari, download this file: `http://training.lczero.org/get_network?sha=82d14d7d8a4f00826f269901d5e31df1a7b2112c20604dc8bee4008271db4d88`.
2. Move the downloaded file to the directory `build/debug/`.

It downloads [last T60 320x24 network 606511](https://lczero.org/play/networks/bestnets/) to the directory that contains lc0 executable, so lc0 can find the network.

# Run Xcode Instruments

Xcode project has been configured correctly. And, lc0 can run Metal backend with neural network infinitely. Now, I can run Xcode Instruments to [find memory leaks](https://help.apple.com/instruments/mac/current/#/dev022f987b) of Metal backend of lc0. The steps are shown as follows.

- Product -> Profile

![Xcode-Product-Profile](/assets/Xcode-Product-Profile.png)

- All -> Leaks -> Choose

![Xcode-Product-Profile-Leaks](/assets/Xcode-Product-Profile-Leaks.png)

- Record

![Xcode-Leaks-Record](/assets/Xcode-Leaks-Record.png)

It runs lc0 and collect information of memory allocations and leaks. I saw the result as follows.

![Xcode-Leaks-Tree](/assets/Xcode-Leaks-Tree.png)

It shows that memory usage is infinitely increasing, and the function `runInferenceWithBatchSize` has used 1366.22 MB. I double-clicked the function `runInferenceWithBatchSize` to see more details inside it.

![Xcode-Leaks-Tree-Function](/assets/Xcode-Leaks-Tree-Function.png)

It shows that 100% memory is using by the following source code.

```
    _resultDataDictionary = [_graph runWithFeeds:@{_inputTensor : _inputTensorData}
                                   targetTensors:_targetTensors
                                targetOperations:nil];
```

# Fix memory leak

In the previous section, we know which source code generates a memory leak. In this section, I will fix the memory leak.

The memory leak can be fixed by my [pull request](https://github.com/almaudoh/lc0/pull/2), in which I made the following changes.

- [Release](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW1) `_inputTensorData` explicitly.
- Surround the function that has memory leaks by an [autorelease pool](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html).

To checkout my pull request locally, we can follow the steps below.

In Terminal, enter the following commands:

```
git stash
git fetch origin pull/2/head:fix-memory-leak
git checkout fix-memory-leak
git stash pop
```

It will [stash](https://git-scm.com/docs/git-stash) changes in the working directory. Then, it will [fetch](https://git-scm.com/docs/git-fetch) the pull request to a new branch `fix-memory-leak`. Finally, it will checkout the branch `fix-memory-leak`, and [pop](https://git-scm.com/docs/git-stash#Documentation/git-stash.txt-pop--index-q--quietltstashgt) the stashed changes.

I reran Xcode Profiler to see whether the memory leak is fixed. The result is shown as follows.

![Xcode-Leaks-Fixed](/assets/Xcode-Leaks-Fixed.png)

It shows that the memory usage doesn't increase infinitely now.

# Remarks

There is another memory leak that is reported by Xcode Profiler, but it has not been fixed in my pull request. Can you find the memory leak and fix it?
