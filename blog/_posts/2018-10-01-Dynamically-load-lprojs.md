---
layout: post
title: "Dynamically load lprojs"
desc: "It loads lprojs into the app bundle during build time. So, you avoid hardcoding languages in xcode project settings"
date: 2018-10-01
categories: blog
---

At work, I ran into a problem where there was a need to dynamically load lprojs folders into the iOS app. You may ask why you want to do that? So, the app is architected in such a way that any number of apps can be created by just changing the configuration files (including lprojs folders). So, for each client we just change configuration files and app is ready to build. 

You may ask, why not just add all the languages in project settings?

Firstly, we can do that but it messes up with `Bundle.main.preferredLocalizations`, even if the client doesn't support any particular language that will appear in preferredLocalizations. 

Secondly, if you are using any third-party framework that supports localizations, it will load some content in that language, even if the other content is loaded in English language as second preferred language.

To overcome all this, I can up with the idea of  copying the lprojs into app bundle.

Here we go...

Let's first create a shell script which copies all config folders into our app.

```sh
echo "Copying files from AppConfig"
cp -a AppConfig/*.lproj ${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app

cp -a AppConfig/JSON ${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app
```

Go to your app target > click on `Build Phases` tab > add a new run script.

![Copy Script phase in build phases](/img/blogs/2018-10-01-dynamically-load-lprojs/build_phases.png)

Thats it! config folders will be copied into the app bundle during build time.

Here is the simple API to look for any localized file based on preferred languages set by the user in their device settings.

```swift
class Localization {
        
    private init() {}
    
    static let defaultLanguage = "en"
    
    static var currentLanguage: String {
        return Bundle.main.preferredLocalizations.first ?? defaultLanguage
    }
    
    static var folderName: String {
        return "\(currentLanguage).lproj"
    }
    
    static var folderPath: String {
        return "\(Bundle.main.bundlePath)/\(folderName)"
    }
}
```

I hope it helped you. Have a great day!