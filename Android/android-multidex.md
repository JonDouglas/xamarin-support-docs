If you're an Android developer, you're bound to run into a scenario where multidex will be needed. Simply put, multidex is needed when our DEX limit is met. That limit is ~65k references. For API levels < 21, we need to ensure we can handle this scenario correctly. **Note: API 21 or greater handles multidex automatically.**

Remember back to the days when Google Play Services was this behemoth package? This made multidex a first class citizen and thus a problem in the long run. However since that, Google has separated the Google Play Services libraries into separate packages to follow a more sane pattern.

Here are a few things that we can do as Android developers to prevent using multidex in our applications:

1.  Only use the libraries based on their functionality. This means you can use specific google play services package for different purposes(Maps, Location, etc). You no longer have to reference a giant package that contains everything even if we only use a subset of it.

2.  Enable Proguard so it can strip our unused code.

However what happens when we run into the limitation where these both do not suffice? This is where multidex comes into play.

**Tip: Use various tooling to keep an eye on your DEX method count:**

<https://github.com/mihaip/dex-method-counts> (Old)

<https://github.com/google/android-classyshark> (New)

Let's first take a look at how we can implement multidex. Given the following class:

<https://developer.android.com/reference/android/support/multidex/MultiDexApplication.html>

Off the bat we see three different methods of how we can include this in our applications:

To use the legacy multidex library there is 3 possibility:

1.  Declare this class as the application in your AndroidManifest.xml.

2.  Have your Application extends this class.

3.  Have your Application override attachBaseContext starting with

`protected void attachBaseContext(Context base) { super.attachBaseContext(base); MultiDex.install(this); }`

Great! We have a starting point. Let's talk about how this is handled in Xamarin.Android.

Given the two current builds at the time of this post (STABLE - 6.0.X and BETA/ALPHA - 6.1.X). Keep these in mind as I'll be referencing these later.

Xamarin.Android handles multidex in two different ways. Both of these ways are mentioned above in the possibilities.

*   STABLE - Follows #1 and #2, in which the `<application>` element will be injected with either `mono.android.app.Application`, or a Custom Application class that extends `mono.android.app.Application`.

*   BETA/ALPHA - Follows #1 and #2, in which the `<application>` element will be injected with either `android.support.multidex.MultiDexApplication`, or a Custom Application class that extends `android.support.multidex.MultiDexApplication`.

Now that we know how it's handled, we can follow a few diagnostic steps to figure out any potential issues.

*   Application.class or CustomApplication.class is not found on the dexPathList:

Typically indicates that our Application or CustomApplication class is being put on a different dex list other than the primary dex list. This can be extremely problematic as it's typically the entry to our application. Typically the easiest check here is to search for your application string inside of your `classes.dex` file (`obj\Release\android\bin` or `obj\Release\android` depending on the version of Xamarin.Android - STABLE vs. BETA/ALPHA respectfully)

*   Know the version of Android SDK `build-tools` that you are running.

You can enable diagnostic build output within your IDE to see statements displaying the version of `build-tools`. Otherwise you can always set a custom build-tools version via the `$(AndroidSdkBuildToolsVersion)` MSBuild property. (Hint you can add a `<AndroidSdkBuildToolsVersion>23.0.3</AndroidSdkBuildToolsVersion>` in your .csproj as well)

<https://developer.xamarin.com/guides/android/under_the_hood/build_process/#Packaging_Properties>

*   My `multidex.keep` file is empty! What do I do?

This is typically a bug within the Android SDK `build-tools`. It's typically best to ensure you're on the latest version of `build-tools`, and that you search google for any outstanding issues. I've found that on Windows, the `multidex.keep` file will always generate empty on `build-tools` <= 23.0.3. However we can fix this by manually changing the invoked `.bat` file.

We can go into our `android-sdk\build-tools\23.0.3` folder here and find a `mainClassesDex.bat` file. This does just what it's called; creates the main `classes.dex` file. We now know that there's an issue between the time Xamarin invokes this command, and the command itself.

We can take a look at the main bulk of the command here:

    if DEFINED output goto redirect
    call "%java_exe%" -Djava.ext.dirs="%frameworkdir%" com.android.multidex.MainDexListBuilder "%disableKeepAnnotated%" "%tmpJar%" "%params%"
    goto afterClassReferenceListBuilder
    :redirect
    call "%java_exe%" -Djava.ext.dirs="%frameworkdir%" com.android.multidex.MainDexListBuilder "%disableKeepAnnotated%" "%tmpJar%" "%params%" 1>"%output%"
    :afterClassReferenceListBuilder
    

Ideally we also want to see the final command to ensure any syntax issues/etc.

*   Is it a bug in Xamarin tooling or the Android SDK?

This can be tricky because with certain scenarios it can totally be Xamarin's fault. However in most cases, it seems to come down to a bug in the Android SDK, more specifically the `build-tools` we use to invoke this functionality.

However with the new BETA/ALPHA(Xamarin.Android 6.1.X) builds, we can now create our own custom `multidex.keep` file via the new `Build Action` of `MultiDexMainDexList`. You can now simply create a `multidex.keep` file in your project, set the build action to `MultiDexMainDexList`, add the respective classes that you want to keep, and boom! You're off to the races.