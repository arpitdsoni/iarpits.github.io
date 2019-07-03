---
layout: post
title:  "Create OptionSet from JSON in Swift"
desc: "Have you ever run into a situation where you had many structs conforming to `OptionSet` from a framework and wanted to convert an array of strings from JSON into an OptionSet?"
date:   2017-08-09
categories: blog
---

Have you ever run into a situation where you had many structs conforming to `OptionSet` from a framework and wanted to convert an array of strings from JSON into an OptionSet?

This is how I did it 🤓

First, let's create a protocol

```swift
public protocol OptionSetStringIntializing {
    init?(_ stringValue: String)
}
```

then let's use this protocol to initialize OptionSet from the string value

```swift
extension String.CompareOptions: OptionSetStringIntializing {
    private enum StrCompareOptions: String {
        case caseInsensitive
        case backwards
    }
    
    public init?(_ stringValue: String) {
        switch stringValue {
        case StrCompareOptions.caseInsensitive.rawValue:
            self = .caseInsensitive
        case StrCompareOptions.backwards.rawValue:
            self = .backwards
        default:
            return nil
        }
    }
}
```

Finally, we will create a static function (it can be an initializer too) which will decode an array of string in JSON into an OptionSet.

```swift
public extension OptionSet {
    
    static func decode<T>(from strings: [String]) -> T where T: OptionSet, T: OptionSetStringIntializing {
        var options: T = []
        
        strings.forEach { (str) in
            if let option = T(str) {
                options.formUnion(option)
            }
        }
        return options
    }
}
```

```swift
//Example
let jsonArray = ["caseInsensitive", "backwards"]
let options: String.CompareOptions = String.CompareOptions.decode(from: jsonArray)

options.contains(.caseInsensitive) // true
options.contains(.backwards) // true
options.contains(.anchored) // false
```

I hope it helped you. Let me know if there is a better way. Constantly learning.

Have a great day!

