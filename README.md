Play Services Resolver for Unity
========

# Overview

This library is intended to be used by any Unity plugin that requires:

   * Android specific libraries (e.g
     [AARs](https://developer.android.com/studio/projects/android-library.html)).
   * iOS [CocoaPods](https://cocoapods.org/).
   * Version management of transitive dependencies.

# Background

Many Unity plugins have dependencies upon Android specific libraries, iOS
CocoaPods, and sometimes have transitive dependencies upon other Unity plugins.
This causes the following problems:

   * Integrating platform specific (e.g Android and iOS) libraries within a
     Unity project can be complex and a burden on a Unity plugin maintainer.
   * The process of resolving conflicting dependencies on platform specific
     libraries is pushed to the developer attempting to use a Unity plugin.
     The developer trying to use you plugin is very likely to give up when
     faced with Android or iOS specific build errors.
   * The process of resolving conflicting Unity plugins (due to shared Unity
     plugin components) is pushed to the developer attempting to use your Unity
     plugin. In an effort to resolve conflicts, the developer will very likely
     attempt to resolve problems by deleting random files in your plugin,
     report bugs when that doesn't work and finally give up.

The Play Services Resolver plugin (the name comes from its origin of just
handling
[Google Play Services](https://developers.google.com/android/guides/overview)
dependencies on Android) provides solutions for each of these problems.

## Android Dependency Management

The *Android Resolver* component of this plugin will download and integrate
Android library dependencies and handle any conflicts between plugins that share
the same dependencies.

Without the Android Resolver, typically Unity plugins bundle their AAR and
JAR dependencies, e.g. a Unity plugin `SomePlugin` that requires the Google
Play Games Android library would redistribute the library and its transitive
dependencies in the folder `SomePlugin/Android/`.  When a user imports
`SomeOtherPlugin` that includes the same libraries (potentially at different
versions) in `SomeOtherPlugin/Android/`, the developer using `SomePlugin` and
`SomeOtherPlugin` will see an error when building for Android that can be hard
to interpret.

Using the Android Resolver to manage Android library dependencies:

   * Solves Android library conflicts between plugins.
   * Handles all of the various processing steps required to use Android
     libraries (AARs, JARs) in Unity 4.x and above projects.  Almost all
     versions of Unity have - at best - partial support for AARs.
   * (Experimental) Supports minification of included Java components without
     exporting a project.

## iOS Dependency Management

The *iOS Resolver* component of this plugin integrates with
[CocoaPods](https://cocoapods.org/) to download and integrate iOS libraries
and frameworks into the Xcode project Unity generates when building for iOS.
Using CocoaPods allows multiple plugins to utilize shared components without
forcing developers to fix either duplicate or incompatible versions of
libraries included through multiple Unity plugins in their project.

## Unity Plugin Version Management

Finally, the *Version Handler* component of this plugin simplifies the process
of managing transitive dependencies of Unity plugins and each plugin's upgrade
process.

For example, without the Version Handler plugin, if:

   * Unity plugin `SomePlugin` includes the `Play Services Resolver` plugin at
     version 1.1.
   * Unity plugin `SomeOtherPlugin` includes the `Play Services Resolver`
     plugin  at version 1.2.

The version of `Play Services Resolver` included in the developer's project
depends upon the order the developer imports `SomePlugin` or `SomeOtherPlugin`.

This results in:

   * `Play Services Resolver` at version 1.2, if `SomePlugin` is imported then
     `SomeOtherPlugin` is imported.
   * `Play Services Resolver` at version 1.1, if `SomeOtherPlugin` is imported
     then `SomePlugin` is imported.

The Version Handler solves the problem of managing transitive dependencies by:

   * Specifying a set of packaging requirements that enable a plugin at
     different versions to be imported into a Unity project.
   * Providing activation logic that selects the latest version of a plugin
     within a project.

When using the Version Handler to manage `Play Services Resolver` included in
`SomePlugin` and `SomeOtherPlugin`, from the prior example, version 1.2 will
always be the version activated in a developer's Unity project.

Plugin creators are encouraged to adopt this library to ease integration for
their customers.  For more information about integrating Play Services Resolver
into your own plugin, see the [Plugin Redistribution](#plugin-redistribution)
section of this document.

# Requirements

The Android Resolver and iOS Resolver components of the plugin only work with
Unity version 4.6.8 or higher.

The *Version Handler* component only works with Unity 5.x or higher as it
depends upon the `PluginImporter` UnityEditor API.

# Getting Started

Before you import the Play Services Resolver into your plugin project, you first
need to consider whether you intend to *redistribute* Play Services Resolver
along with your own plugin.

Redistributing the `Play Services Resolver` inside your own plugin will ease
the integration process for your users, by resolving dependency conflicts
between your plugin and other plugins in a user's project.

If you wish to redistribute the `Play Services Resolver` inside your plugin,
you **must** follow these steps when importing the
`play-services-resolver-*.unitypackage`, and when exporting your own plugin
package:

   1. Import the `play-services-resolver-*.unitypackage` into your plugin
      project by
      [running Unity from the command line](https://docs.unity3d.com/Manual/CommandLineArguments.html), ensuring that
      you add the `-gvh_disable` option.
   1. Export your plugin by [running Unity from the command line](https://docs.unity3d.com/Manual/CommandLineArguments.html), ensuring that
      you:
      - Include the contents of the `Assets/PlayServicesResolver` directory.
      - Add the `-gvh_disable` option.

You **must** specify the `-gvh_disable` option in order for the Version
Handler to work correctly!

For example, the following command will import the
`play-services-resolver-1.2.46.0.unitypackage` into the project
`MyPluginProject` and export the entire Assets folder to
`MyPlugin.unitypackage`:

```
Unity -gvh_disable \
      -batchmode \
      -importPackage play-services-resolver-1.2.46.0.unitypackage \
      -projectPath MyPluginProject \
      -exportPackage Assets MyPlugin.unitypackage \
      -quit
```

## Background

The *Version Handler* component relies upon deferring the load of editor DLLs
so that it can run first and determine the latest version of a plugin component
to activate.  The build of the `Play Services Resolver` plugin has Unity asset
metadata that is configured so that the editor components are not
initially enabled when it's imported into a Unity project.  To maintain this
configuration when importing the `Play Services Resolver` .unitypackage
into a Unity plugin project, you *must* specify the command line option
`-gvh_disable` which will prevent the Version Handler component from running and
changing the Unity asset metadata.

# Android Resolver Usage

The Android Resolver copies specified dependencies from local or remote Maven
repositories into the Unity project when a user selects Android as the build
target in the Unity editor.

   1. Add the `play-services-resolver-*.unitypackage` to your plugin
      project (assuming you are developing a plugin). If you are redistributing
      the Play Services Resolver with your plugin, you **must** follow the
      import steps in the [Getting Started](#getting-started) section!

   2. Copy and rename the `SampleDependencies.xml` file into your
      plugin and add the dependencies your plugin requires.

      The XML file just needs to be under an `Editor` directory and match the
      name `*Dependencies.xml`. For example,
      `MyPlugin/Editor/MyPluginDependencies.xml`.

   3. Follow the steps in the [Getting Started](#getting-started)
      section when you are exporting your plugin package.

For example, to add the Google Play Games library
(`com.google.android.gms:play-services-games` package) at version `9.8.0` to
the set of a plugin's Android dependencies:

```
<dependencies>
  <androidPackages>
    <androidPackage spec="com.google.android.gms:play-services-games:9.8.0">
      <androidSdkPackageIds>
        <androidSdkPackageId>extra-google-m2repository</androidSdkPackageId>
      </androidSdkPackageIds>
    </androidPackage>
  </androidPackages>
</dependencies>
```

The version specification (last component) supports:

   * Specific versions e.g `9.8.0`
   * Partial matches e.g `9.8.+` would match 9.8.0, 9.8.1 etc. choosing the most
     recent version.
   * Latest version using `LATEST` or `+`.  We do *not* recommend using this
     unless you're 100% sure the library you depend upon will not break your
     Unity plugin in future.

The above example specifies the dependency as a component of the Android SDK
manager such that the Android SDK manager will be executed to install the
package if it's not found.  If your Android dependency is located on Maven
central it's possible to specify the package simply using the `androidPackage`
element:

```
<dependencies>
  <androidPackages>
    <androidPackage spec="com.google.api-client:google-api-client-android:1.22.0" />
  </androidPackages>
</dependencies>
```

## Auto-resolution

By default the Android Resolver automatically monitors the dependencies you have
specified and the `Plugins/Android` folder of your Unity project.  The
resolution process runs when the specified dependencies are not present in your
project.

The *auto-resolution* process can be disabled via the
`Assets > Play Services Resolver > Android Resolver > Settings` menu.

Manual resolution can be performed using the following menu options:

   * `Assets > Play Services Resolver > Android Resolver > Resolve`
   * `Assets > Play Services Resolver > Android Resolver > Force Resolve`

## Deleting libraries

Resolved packages are tracked via asset labels by the Android Resolver.
They can easily be deleted using the
`Assets > Play Services Resolver > Android Resolver > Delete Resolved Libraries`
menu item.

## Android Manifest Variable Processing

Some AAR files (for example play-services-measurement) contain variables that
are processed by the Android Gradle plugin.  Unfortunately, Unity does not
perform the same processing when using Unity's Internal Build System, so the
Android Resolver plugin handles known cases of this variable substitution
by exploding the AAR into a folder and replacing `${applicationId}` with the
`bundleID`.

Disabling AAR explosion and therefore Android manifest processing can be done
via the `Assets > Play Services Resolver > Android Resolver > Settings` menu.
You may want to disable explosion of AARs if you're exporting a project to be
built with Gradle / Android Studio.

## ABI Stripping

Some AAR files contain native libraries (.so files) for each ABI supported
by Android.  Unfortunately, when targeting a single ABI (e.g x86), Unity does
not strip native libraries for unused ABIs.  To strip unused ABIs, the Android
Resolver plugin explodes an AAR into a folder and removes unused ABIs to
reduce the built APK size.  Furthermore, if native libraries are not stripped
from an APK (e.g you have a mix of Unity's x86 library and some armeabi-v7a
libraries) Android may attempt to load the wrong library for the current
runtime ABI completely breaking your plugin when targeting some architectures.

AAR explosion and therefore ABI stripping can be disabled via the
`Assets > Play Services Resolver > Android Resolver > Settings` menu.  You may
want to disable explosion of AARs if you're exporting a project to be built
with Gradle / Android Studio.

## Resolution Strategies

By default the Android Resolver will use Gradle to download dependencies prior
to integrating them into a Unity project.  This works with Unity's internal
build system and Gradle / Android Studio project export.

It's possible to change the resolution strategy via the
`Assets > Play Services Resolver > Android Resolver > Settings` menu.

## Dependency Tracking

The Android Resolver creates the
`ProjectSettings/AndroidResolverDependencies.xml` to quickly determine the set
of resolved dependencies in a project.  This is used by the auto-resolution
process to only run the expensive resolution process when necessary.

## Displaying Dependencies

It's possible to display the set of dependencies the Android Resolver
would download and process in your project via the
`Assets > Play Services Resolver > Android Resolver > Display Libraries` menu
item.

# iOS Resolver Usage

The iOS resolver component of this plugin manages
[CocoaPods](https://cocoapods.org/).  A CocoaPods `Podfile` is generated and
the `pod` tool is executed as a post build process step to add dependencies
to the Xcode project exported by Unity.

Dependencies for iOS are added by referring to CocoaPods.

   1. Add the `play-services-resolver-*.unitypackage` to your plugin
      project (assuming you are developing a plugin). If you are redistributing
      the Play Services Resolver with your plugin, you **must** follow the
      import steps in the [Getting Started](#getting-started) section!

   2. Copy and rename the SampleDependencies.xml file into your
      plugin and add the dependencies your plugin requires.

      The XML file just needs to be under an `Editor` directory and match the
      name `*Dependencies.xml`. For example,
      `MyPlugin/Editor/MyPluginDependencies.xml`.

   3. Follow the steps in the [Getting Started](#getting-started)
      section when you are exporting your plugin package.

For example, to add the AdMob pod, version 7.0 or greater with bitcode enabled:

```
<dependencies>
  <iosPods>
    <iosPod name="Google-Mobile-Ads-SDK" version="~> 7.0" bitcodeEnabled="true"
            minTargetSdk="6.0" />
  </iosPods>
</dependencies>
```

## Integration Strategies

The `CocoaPods` are either:
   * Downloaded and injected into the Xcode project file directly, rather than
     creating a separate xcworkspace.  We call this `Xcode project` integration.
   * If the Unity version supports opening a xcworkspace file, the `pod` tool
     is used as intended to generate a xcworkspace which references the
     CocoaPods.  We call this `Xcode workspace` integration.

The resolution strategy can be changed via the
`Assets > Play Services Resolver > iOS Resolver > Settings` menu.

### Appending text to generated Podfile
In order to modify the generated Podfile you can create a script like this:
```
using System.IO;
public class PostProcessIOS : MonoBehaviour {
[PostProcessBuildAttribute(45)]//must be between 40 and 50 to ensure that it's not overriden by Podfile generation (40) and that it's added before "pod install" (50)
private static void PostProcessBuild_iOS(BuildTarget target, string buildPath)
{
    if (target == BuildTarget.iOS)
    {

        using (StreamWriter sw = File.AppendText(buildPath + "/Podfile"))
        {   
            //in this example I'm adding an app extension
            sw.WriteLine("\ntarget 'NSExtension' do\n  pod 'Firebase/Messaging', '6.6.0'\nend");
        }
    }
}
```
# Version Handler Usage

The Version Handler component of this plugin manages:
* Shared Unity plugin dependencies.
* Upgrading Unity plugins by cleaning up old files from previous versions.

Unity plugins can be managed by the Version Handler using the following steps:

   1. Add the `gvh` asset label to each asset (file) you want Version Handler
      to manage.
   1. Add the `gvh_version-VERSION` label to each asset where `VERSION` is the
      version of the plugin you're releasing (e.g 1.2.3).
   1. Optional: Add `gvh_targets-editor` label to each editor DLL in your
      plugin and disable `editor` as a target platform for the DLL.
      The Version Handler will enable the most recent version of this DLL when
      the plugin is imported.
   1. Optional: If your plugin is included in other Unity plugins, you should
      add the version number to each filename and change the GUID of each asset.
      This allows multiple versions of your plugin to be imported into a Unity
      project, with the Version Handler component activating only the most
      recent version.
   1. Create a manifest text file named `MY_UNIQUE_PLUGIN_NAME_VERSION.txt`
      that lists all the files in your plugin relative to the project root.
      Then add the `gvh_manifest` label to the asset to indicate this file is
      a plugin manifest.
   1. Redistribute the `Play Services Resolver` Unity plugin with your plugin.
      See the [Plugin Redistribution](#plugin-redistribution) for the details.

If you follow these steps:

   * When users import a newer version of your plugin, files referenced by the
     older version's manifest are cleaned up.
   * The latest version of the plugin will be selected when users import
     multiple packages that include your plugin, assuming the steps in
     [Plugin Redistribution](#plugin-redistribution) are followed.

## Building from Source

To build this plugin from source you need the following tools installed:
   * Unity (with iOS and Android modules installed)

You can build the plugin by running the following from your shell
(Linux / OSX):

```
./gradlew build
```

or Windows:

```
./gradlew.bat build
```

### Releasing

Each time a new build of this plugin is checked into the source tree you
need to do the following:

   * Bump the plugin version variable `pluginVersion` in `build.gradle`
   * Update `CHANGELOG.md` with the new version number and changes included in
     the release.
   * Build the release using `./gradle release` which performs the following:
      * Updates `play-services-resolver-*.unitypackage`
      * Copies the unpacked plugin to the `exploded` directory.
      * Updates template metadata files in the `plugin` directory.
        The GUIDs of all asset metadata is modified due to the version number
        change. Each file within the plugin is versioned to allow multiple
        versions of the plugin to be imported into a Unity project which allows
        the most recent version to be activated by the Version Handler
        component.
   * Create the release commit and tag the release using
     `./gradle gitTagRelease` which performs the following:
     * `git add -A` to pick up all modified, new and deleted files in the tree.
     * `git commit --amend -a` to create a release commit with the release notes
       in the change log.
     * `git tag -a RELEASE -m "version RELEASE"` to tag the release.
