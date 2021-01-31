# Flutter (for Windows)

## Installation

_These steps are majorly taken from the [official Flutter Docs for Installation](https://flutter.dev/docs/get-started/install/windows) but include other smaller details that may have been glossed over or may cause errors. The installation assumes that you already have __Android Studio__ installed with the SDKs needed for development._

1. Download the latest stable version of Flutter from [here](https://flutter.dev/docs/development/tools/sdk/releases) & unzip it to your preferred location
2. From the Start search bar, enter `env` and select __Edit environment variables for your account__
3. Under __User variables__ check if there is an entry called __Path__ & add the full path to `flutter\bin` to the list of paths
4. Fire up a terminal & type in `flutter doctor` _(checks your environment and displays a report of the status of your Flutter installation)_
   
   > If you see `Android license status unknown`, run the following: `flutter doctor --android-licenses`. This should check the license status of all SDK packages, and if not accepted, will prompt you to accept them.
   >
   > While doing this, if you come across the error `Error: Unknown argument --licenses`, it is possibly due to the absense of some SDK Tools. To install the required tools, head over to Android Studio & go to __Tools > SDK Manager > SDK Tools__ & check the following:
   > - Android SDK Command-line Tools (latest)
   > - Android SDK Platform-Tools
   >
   >
   > Also check that the _Android Emulator_ & _Intel x86 Emulator Accelerator (HAXM installer)_ are checked. Now, click __OK__ to complete the installation of the above tools & then again run `flutter doctor --android-licenses`.
   
   > If it shows the error that Flutter & Dart plugins are not installed in Android Studio, head over to Android Studio & go to __File > Settings > Plugins__, search for _Flutter_ & click __Install__. This will automatically install the Dart plugin as well.
   
5. Run `flutter doctor` again to check the status of installation
