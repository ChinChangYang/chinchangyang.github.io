---
layout: post
title:  "Run Katago with Core ML in Apple silicon"
date:   2022-08-11 00:09:00 +0800
categories: github katago coreml apple m1 arm64
---
# Convert a Katago network to a Core ML model
Borrow an Intel-based Macbook, and install Tensorflow 1.5. Then, follow this [gist](https://gist.github.com/ChinChangYang/5e9d7ff5900f7be8af0f226689e84e7a) to convert a Katago network.
# Generate an Xcode project from KataGo
```
cmake -G Xcode cpp -DUSE_BACKEND=OPENCL
```
# Create coremlbackend.swift
This step enables Xcode create a Swift header file for a Core ML model.
# Import a Core ML model to the Xcode project
Drag the Core ML model file to the project navigator of Xcode project from Finder.
# Specifically select Core ML model class generation language to Swift
Project navigator -> katago -> Build Settings -> CoreML Model Class Generation Language -> Swift
# Build Xcode project to generate the Core ML model class
# Check the generated Core ML model class
Project navigator -> the imported Core ML model -> Click Model Class -> See the Editor for the generated Core ML model class
# Implement coremlbackend.swift to use the Core ML model
coremlbackend.swift
```
import Foundation
import CoreML

@objc
class CoreMLBackend : NSObject {
    @objc func doSomething(input: MLMultiArray) throws -> MLMultiArray {
        let swa_model_bin_inputs = input
        let swa_model_global_inputs = input
        let swa_model_include_history = input
        let swa_model_symmetries = input
        let model = try kata1_b40c256_s11840935168_d2898845681()
        
        let input = kata1_b40c256_s11840935168_d2898845681Input(
            swa_model_bin_inputs: swa_model_bin_inputs,
            swa_model_global_inputs: swa_model_global_inputs,
            swa_model_include_history: swa_model_include_history,
            swa_model_symmetries: swa_model_symmetries)
        
        let output = try model.prediction(input: input)
        
        return output.swa_model_policy_output
    }
}
```
# Make an Objective-C bridge to the Swift
coremlbackend.mm
```
#import <Foundation/Foundation.h>
#import <CoreML/MLMultiArray.h>
#import "katago-Swift.h"

void get_coremlbackend_policy_output() {
    NSError *error = nil;
    NSArray<NSNumber *> *shape3x3 = @[@3, @3];

    MLMultiArray *input = [[MLMultiArray alloc] initWithShape:shape3x3 dataType:MLMultiArrayDataTypeFloat error: &error];

    [[[CoreMLBackend alloc] init] doSomethingWithInput: input error: &error];
}
```
# Create the Objective-C header file for C++
coremlbackend.h
```
#ifndef coremlbackend_h
#define coremlbackend_h

void get_coremlbackend_policy_output();

#endif /* coremlbackend_h */
```
# Call the Objective-C bridge from C++
coremlbackend.cpp
```
#include "../neuralnet/coremlbackend.h"
...
    get_coremlbackend_policy_output();
...
```

# Update (2023-06-07)
- KataGo [v1.12.4-coreml1](https://github.com/ChinChangYang/KataGo/releases/tag/v1.12.4-coreml1) supports [model version 11](https://github.com/lightvector/KataGo/releases/tag/v1.12.0), which is trained on [PyTorch](https://pytorch.org). Notably, with Apple's support for PyTorch on the Apple M1 Pro, you can now convert a KataGo network of PyTorch to a Core ML model without needing an Intel-based Macbook.
