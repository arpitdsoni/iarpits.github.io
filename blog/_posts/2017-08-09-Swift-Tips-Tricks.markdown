---
layout: post
title:  "Swift Tips & Tricks"
date:   2017-08-09
categories: blog
---

A collection of swift tips and tricks


### #3 Create OptionSet from array of string.
Do you ever ran into a situation where you had many structs conforming to `OptionSet` from a framework and wanted to convert array of strings from JSON into a OptionSet?

This is how I did it 😃

```swift
public extension OptionSet {

    public static func decode<T>(from strings: [String]) -> T where T: OptionSet, T: OptionSetGenerating {
        var options: T = []

        strings.forEach { (str) in
            if let option = T.generate(str) {
                options.formUnion(option)
            }
        }
        return options
    }
}

public protocol OptionSetGenerating {
    static func generate(_ stringValue: String) -> Self?
}

extension ORKPredefinedTaskOption: OptionSetGenerating {
private struct Flags {
        static let excludeInstructions = "excludeInstructions"
        static let excludeConclusion = "excludeConclusion"
        static let excludeAccelerometer = "excludeAccelerometer"
        static let excludeDeviceMotion = "excludeDeviceMotion"
        static let excludePedometer = "excludePedometer"
        static let excludeLocation = "excludeLocation"
        static let excludeHeartRate = "excludeHeartRate"
        static let excludeAudio = "excludeAudio"
    }

    public static func generate(_ stringValue: String) -> ORKPredefinedTaskOption? {
        switch stringValue {
        case Flags.excludeInstructions : return .excludeInstructions
        case Flags.excludeConclusion : return .excludeConclusion
        case Flags.excludeAccelerometer : return .excludeAccelerometer
        case Flags.excludeDeviceMotion : return .excludeDeviceMotion
        case Flags.excludePedometer : return .excludePedometer
        case Flags.excludeLocation : return .excludeLocation
        case Flags.excludeHeartRate : return .excludeHeartRate
        case Flags.excludeAudio : return .excludeAudio
        default: return nil
        }
    }
}

//Example
let options: ORKPredefinedTaskOption = ORKPredefinedTaskOption.decode(from: json.value(forKey: TaskKeys.predefinedTaskOptions, defaultValue: []))
```
<br/>
### #2 Play with UIView or UIViewController
Do you want to quickly test any `UIVIew` or `UIViewController` or work on new custom `UIView` and see rendering instantly? If yes than `Playground` is the best way to get started instead of testing in current project or making a new project.

Here I am setting `liveView` to a custom `UIViewController` which will render the view instantly in Playground Assistant Editor.

```swift
let options = ORKPredefinedTaskOption.decodeOptions(from: ["excludeInstructions", "excludeConclusion"])

let fitnessCheckTask = ORKOrderedTask.fitnessCheck(withIdentifier: "fitnessCheck",
                                                   intendedUseDescription: "intendedUseDescription",
                                                   walkDuration: 10,
                                                   restDuration: 5,
                                                   options: options)

let fitnessCheckTaskVC = ORKTaskViewController(task: fitnessCheckTask, taskRun: nil)

// Isn't it 😎
PlaygroundPage.current.liveView = fitnessCheckTaskVC
```
<br/>
### #1 Optional SwiftyJSON object with a default value

Do you want to avoid syntax which has both casting and nil coalescing operator?

`let strArray = (json["testArray"].arrayObject as? [String]) ?? ["default"]`

Here is the better way to write it 😃

```swift
extension JSON {
  func value<T>(forKey key: Key, defaultValue: @autoclosure () -> T) -> T {
    guard let value = self[key].object as? T else {
        return defaultValue()
      }
    return value
  }
}

// Example
let json: JSON = ["testArray": ["1", "2"],
                    "testString": "Str"]

json.value(forKey: "testArray", defaultValue: [])

json.value(forKey: "testString", defaultValue: "default")

```
