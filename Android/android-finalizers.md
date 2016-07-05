It is discouraged to use finalizers - http://www.slideshare.net/Xamarin/advanced-memory-management-on-ios-and-android-mark-probst-and-rodrigo-kumpera

They are not guaranteed to run within any deadline.
They don't run in a specific sequence.
They make objects live longer.
The GC doesn't know about unmanaged resources.

Using a finalizer to perform cleanup is the wrong way of doing things... If you want to ensure that  an `Activity` is destroyed, manually dispose the `Activity` in it's `OnDestroy()`:

```
protected override void OnDestroy ()
{
    base.OnDestroy ();
    this.Dispose ();
}
```

This will cause Mono to break the peer object connection and destroy the activity during the next garbage collection cycle (GC.Collect(GC.MaxGeneration)). From the docs:

To shorten object lifetime, Java.Lang.Object.Dispose() should be invoked. This will manually "sever" the connection on the object between the two VMs by freeing the global reference, thus allowing the objects to be collected faster.
Note the call order there, this.Dispose() must be called after any code that invokes back into Android land. Why? All the connections between Java and .NET are now broken to allow Android to reclaim resources so any code that uses Android-land objects (Fragment, Activity, Adapter) will fail.

One extremely useful tool to use here is `StrictMode`. You can enable it via the following code, or by the Android Developer Settings (`Settings -> Developer Options -> Monitoring -> Strict Mode enabled`). In this case, the code is much more precise, but you can learn more about StrictMode here:

https://developer.android.com/reference/android/os/StrictMode.html

```
var vmPolicy = new StrictMode.VmPolicy.Builder ();
StrictMode.SetVmPolicy (vmPolicy.DetectActivityLeaks().PenaltyLog().Build ());
```

This enables turns on StrictMode, an extremely useful debugging tool that happily informs you when you've leaked resources. When one of your apps activities isn't released correctly, it will dump something like this to the output stream:

```
[StrictMode] class activitydispose.LeakyActivity; instances=2; limit=1
[StrictMode] android.os.StrictMode$InstanceCountViolation: class activitydispose.LeakyActivity; instances=2; limit=1
[StrictMode]    at android.os.StrictMode.setClassInstanceLimit(StrictMode.java:1)
```

Combining this with the `Dispose()` call, you can check that activities are being released. Here is how you'd typically an Activity and its resources in Xamarin.Android:

```
protected override void Dispose (bool disposing)
{
    // TODO: Dispose logic here.
    base.Dispose (disposing);
    GC.Collect(GC.MaxGeneration); // Will force GEN 2 collection
}
```

```
protected override void OnDestroy ()
{
    if (myObj != null) { // Release any Java objects
        myObj.Dispose ();
        myObj = null; // Call either .Dipose or = null, both do the same thing
    }
    base.OnDestroy ();
    this.Dispose (); // Sever java binding.
}
```