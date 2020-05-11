---
layout: post
title: "[React-Native] How to write a react native library bridge/module"
description: "As a React Native developer I want to be able to invoke native code when needed."
comments: true
keywords: "react-native"
---

As a React Native developer I want to be able to invoke native code when needed. To accomplish this I wrote a React Native API bridge to invoke the native code using JavaScript. This post describes the process I used to accomplish this and contains native code examples in Java and Objective C.

Most guides instruct users to install the bridge within their application. Since I’m a library developer this bridge will be independent from applications.

Warning: `react-native` applications only! Per the React Native Documentation if you’re looking to consume native code in your React Native application you cannot use a project created using `create-react-native-app.` You need to use `react-native` as described in the next step.

## Overview

1. Install required packages
2. Create the bridge boilerplate
* Fixup the npm code
* Fixup the native Android/Java boilerplate
* Fixup the native iOS/Object C boilerplate
3. Add some native Java code
4. Add some native Objective C code
5. Consume the library from the React Native application
6. Deploy the library bridge
7. Link the library

## Install required packages
The following packages are required to proceed:

1. Nodejs – download and install v6.12.3. This package should include the required Node Package Manager (npm).
2. React Native Command Line Tools – run 
`npm install -g react-native-cli.`
3. Android Studio including emulator support or a physical device.
4. xCode including simulator support or a physical device.
5. Visual Code or your favorite Javascript / text editor.

During installation `npm` should warn you of any other packages you might need. I used `node` v6.12.3 because the latest version (at this time) of node wasn’t compatible with React Native.

## Create the bridge boilerplate
I used the frostney/react-native-create-library command line tool for creating the boilerplate code. This is an `npm` tool which can be installed using:
```
$ npm install -g react-native-create-library
```
Once the package has been installed switch to a new directory (which will hold the new library bridge). Use the package to generate the boilerplate we’ll use:

```
$ cd ~/Documents/react-native/exampleBridge
 
~/Documents/react-native/exampleBridge $ react-native-create-library ExampleBridge
While `RN` is the default prefix,
  it is recommended to customize the prefix.
 
📚  Created library ExampleBridge in `./ExampleBridge`.
🕘  It took 20ms.
➡️  To get started type `cd ./ExampleBridge`
```
This should create a tree of files:

```
$ tree
.
├── README.md
├── android
│   ├── build.gradle
│   └── src
│       └── main
│           ├── AndroidManifest.xml
│           └── java
│               └── com
│                   └── reactlibrary
│                       ├── RNExampleBridgeModule.java
│                       └── RNExampleBridgePackage.java
├── index.js
├── ios
│   ├── RNExampleBridge.h
│   ├── RNExampleBridge.m
│   ├── RNExampleBridge.podspec
│   ├── RNExampleBridge.xcodeproj
│   │   └── project.pbxproj
│   └── RNExampleBridge.xcworkspace
│       └── contents.xcworkspacedata
├── package.json
└── windows
    ├── RNExampleBridge
    │   ├── Properties
    │   │   ├── AssemblyInfo.cs
    │   │   └── RNExampleBridge.rd.xml
    │   ├── RNExampleBridge.csproj
    │   ├── RNExampleBridgeModule.cs
    │   ├── RNExampleBridgePackage.cs
    │   └── project.json
    └── RNExampleBridge.sln
 
12 directories, 19 files
```
This has created all the files needed to start working on the native bridge. Here’s a brief overview of what’s included:

* `README.md` – documentation
* `android/` – directory containing native code for Android/Java
* `ndex.js` – configuration file which exports your native code for consumption via Javascript
* `ios/` – directory containing native code for iOS/Objective C
* `package.json` – Specifics of npm’s package.json handling
* windows – directory containing native code for Windows (not covered in this post).

There’s some additional changes required before any native code can be written.

## Fixup the npm code
There’s two files which should be updated after creating the boilerplate:

* `package.json`
* `index.js`
Let’s start with `package.json` (specifics of npm’s package.json handling):

```
{
  "name": "react-native-example-bridge",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "react-native"
  ],
  "author": "",
  "license": "",
  "peerDependencies": {
    "react-native": "^0.41.2",
    "react-native-windows": "0.41.0-rc.1"
 
  }
}
```
Update the `name` and `description` as desired. The `name` is how you’ll interact with the package when installing it using `npm`. Choose a name that’s descriptive and unique. The template includes `peerDependencies` which are not required for the purpose of this guide. Let’s remove them for now. Update the other fields as desired (`author`, `license` etc).

```
{
  "name": "example-bridge",
  "version": "1.0.0",
  "description": "An example bridge to learn how to consume native code from a react native application",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "react-native"
  ],
  "author": "",
  "license": ""
}
```
Update `index.js:`

```
import { NativeModules } from 'react-native';
 
const { RNExampleBridge } = NativeModules;
 
export default RNExampleBridge;
```
Rename `RNExampleBridge` to something more descriptive. This name represents how you’ll `import` the native code using JavaScript.

## Fixup the native Android/Java boilerplate
Open the android directory using Android Studio. It should open and compile without any errors.

Let’s start with `RNExampleBridgeModule.java`:

```
package com.reactlibrary;
 
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Callback;
 
public class RNExampleBridgeModule extends ReactContextBaseJavaModule {
 
  private final ReactApplicationContext reactContext;
 
  public RNExampleBridgeModule(ReactApplicationContext reactContext) {
    super(reactContext);
    this.reactContext = reactContext;
  }
 
  @Override
  public String getName() {
    return "RNExampleBridge";
  }
}
```
Rename the `String` returned by `getName()` to match the name used above in `index.js.` Using the refactor tool (Refactor -> Rename) or (Shift + F6) rename `RNExampleBridgeModule.java` and `RNExampleBridgePackage.java` (while keeping the “Module”/”Package” suffix) also.

Add a new file (in the same directory as `RNExampleBridgeModule.java` and `RNExampleBridgePackage.java`) named `MainApplication.java` which should contain this:

```
package com.rileymacdonald.reactnative;
 
import android.app.Application;
 
import com.facebook.react.ReactPackage;
import com.facebook.react.shell.MainReactPackage;
 
import java.util.Arrays;
import java.util.List;
 
public class MainApplication extends Application {
    protected List<ReactPackage> getPackages() {
        return Arrays.<ReactPackage>asList(
                new MainReactPackage(),
                new RNExampleBridgePackage()); // rename this to match your "package" java file.
    }
}
```

## Fixup the native iOS/Object C boilerplate
Almost everything is included in the boilerplate code to get up and running. The following outlines additional steps required to work with xCode and add Objective C React Native methods. Open the `RNExampleBridge.xcodeproj` (xCode project) using xCode. You’ll notice a few errors because xCode isn’t able to recognize any of the React Native dependencies. To get the code to compile in xCode (thanks to ![](https://stackoverflow.com/a/43340802)) you’ll need to:

From the xCode menu:

1. Product -> Scheme -> Manage Shemes
2. Double click on your application
3. Click the build tab (on the left) -> uncheck Parallelize Build
4. Since this library is independent from any application it’s required that `React.xcodeproj` be added to the project:
* When initializing a React Native application using `react-native [options] [command]` a `React.xcodeproj` file will be generated in the root of the application directory (`node_modules/react-native/React/React.xcodeproj`). If this `xcodeproj` isn’t found or you don’t have an application try checking `/usr/local/lib/node_modules/react-native/React`.
* Copy the `node_modules/React/React.xcodeproj` into the ios directory of the react native application – `cd /{path-to-react-native-bridge-module}/ && cp -R React.xcodeproj/ .`
* Select the project in the xCode Project Navigator and click the Build Phases Tab -> Target Dependencies -> + -> add React
xCode should resolve all the the dependencies and compile/build successfully.

Rename `RNExampleBridge.h` and `RNExampleBridge.m` file names (and every reference to them within the code of these two files) following the naming convention used above for Android. Be sure the naming conventions match the Android implementation. If the names are offset the React Native methods will only work on one platform (iOS/Android).

## Add some native Java code
It’s time to add some native Java code to the bridge. In this example we’ll add a `Toast` message to verify the native code is working. Using Android Studio open the subclass of `ReactContextBaseJavaModule` (which was renamed from `RNExampleBridgeModule.java` above). Methods designated for React Native are declared using the `@ReactMethod` annotation. You can access`Context` using the `ReactApplicationContext` provided from the `ReactPackage` implementation (included from creating the module as shown above). The following example exposes a method called `showMessage()` which shows a native Android `Toast` method.

```
@ReactMethod
  public void showMessage() {
    Toast.makeText(reactContext.getApplicationContext(), "NATIVE CODE IS WORKING", Toast.LENGTH_LONG).show();
}
```
## Add some native Objective C code
Using xCode open the implementation of `RNExampleBridge.h` (`RNExampleBridge.m`) and add the `showMessage()` method. Methods designated for React Native are declared using `RCT_EXPORT_METHOD`:

```
RCT_EXPORT_METHOD(showMessage) {
    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"UIAlertView"
                                                    message:@"NATIVE CODE IS WORKING!" delegate:self cancelButtonTitle:@"Cancel"
                                          otherButtonTitles:@"OK", nil];
    [alert show];
}
```
Now we have a `showMessage()` method exposed for React Native on both iOS and Android. Calls to this module/method will automatically be routed and invoked by React Native for the correct platform (iOS or Android)!

## Consume the native React Native methods
It’s time to consume the native React Native module/methods we exposed above. As noted above React Native applications created using `create-react-native-app` are unable to consume native code. Make sure the application was created using `react-native`as shown above.

The first step is to install the module as an npm package locally on your machine. This can be done globally from your native modules root directory by invoking:
```
# Install the native module to the local machine with global accessibility
/{path-to-react-native-bridge-module} $npm install . -g
/usr/local/lib
└── example-bridge@1.0.0
```

Notice `npm` has successfully installed the package globally using the name provided defined in the native modules `package.json` “name” and “version” values. The application can now consume the native module using the `link` command. Change directory back to the application consuming the native module and link the native module to the application:

```
# Link the native module to the application using the package name defined from package.json
/{path-to-react-native-app} $ react-native link example-bridge
```
Open the `index.js` file of your application and register the module we just linked. The name to register comes from the name defined in the native modules `index.js` file:
```
AppRegistry.registerComponent('RNExampleBridge', () => App);
```
Once the component is registered it can now be consumed by the application using these names (“ExampleBridge” and “example-bridge”). Open `App.js` and import the module / consume the code.
```
import ExampleBridge from 'example-bridge'
```
Now methods exposed for React Native can be invoked through Javascript. Let’s add a button to show the appropriate native alert defined on each platform as `showMessage()`:

```
export default class App extends Component {
  _showNativeMessage() { ExampleBridge.showMessage(); }
 
  render() {
    return (
      <View>
        <Button title="Show Native Message" onPress={() => { this._showNativeMessage() }}/>
      </View>
    );
  }
}
```
## Run the application
React Native applications created using `react-native` can be run by invoking (make sure you have an emulator or attached device instance running):
```
# Android
/{path-to-react-native-app} $ react-native run-android
 
#iOS
/{path-to-react-native-app} $ react-native run-ios
```
You should see each platform consume and build the application and `ExampleBridge` native module dependency. Another terminal should be launched for the `Metro Bundler`. This terminal is responsible for deploying the application to the device.

## Troubleshooting
I experienced some caching issues during development resulting in a cached version of the module being used instead of the latest. To remove cached modules:

* `rm -rf node_modules` in the app
* `rm -rf node_modules` globally
* delete the ios/build directory in the app
* clean each of the native code projects

## Consuming native code libraries
If you’re looking to consume native code from a third party library; you can include the library natively (build.gradle dependency for Android / xCode link with binary library for iOS). Once you have a reference to the dependency you can include it in your native code and execute as desired.