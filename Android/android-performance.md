# Xamarin.Android Performance Issues

### 1. Investigation

One of the best ways to investigate a problematic Xamarin.Android Errors is to first ensure you have the proper tooling available:

* Diagnostic MSBuild Output Enabled([Instructions](https://developer.xamarin.com/guides/android/troubleshooting/troubleshooting/#Diagnostic_MSBuild_Output))
* Android SDK Installed
* Android API Level Documentation


### 2. Render

You typically have about 16 ms / frame to do all of your drawing logic. This number is typically found based on the hardware performance of 1000ms / 60hz = 16ms. If the window is missed in the 16ms window, you will experience what's called a "Dropped Frame". You will need to wait until the next frame to render an update. That means it will take roughly 32ms to see a new result.

#### Jank

Jank is a term for dropped or delayed frames.

#### Rendering Pipeline

![](http://i.imgur.com/ZL6mbWJ.png)

#### CPU

The CPU consists of 4 major items:

- Measure
- Layout
- Record
- Execute

There are two problematic areas:

- Layouts
- Invalidations

**Debugging Tools:**

Hierarchy Viewer - [http://developer.android.com/tools/help/hierarchy-viewer.html](http://developer.android.com/tools/help/hierarchy-viewer.html)

You'll want to click on the venn-diagram icon which will put 3 dots on each View.

- Left - Measure Phase
- Middle - Layout Phase
- Right - Draw Phase

Color (Relative Performance to other nodes)

- Green - Fastest performer compared to other nodes
- Yellow - Bottom 50% to other nodes
- Red - Potential problem / Red Flag / Slowest Performer(Should be expected on at least one node)

Best Practices:

Keep layouts simple and flat. Inflating layouts is expensive in which each nested layout will affect performance.

#### GPU

The GPU consists of 1 major item:

- Rasterization

There is one problematic area:

- Overdraw

**Debugging Tools:**

Show GPU Overdraw (Settings -> Developer Options -> Debug GPU Overdraw -> Show overdraw areas)

[http://developer.android.com/tools/performance/debug-gpu-overdraw/index.html](http://developer.android.com/tools/performance/debug-gpu-overdraw/index.html)

You will now see that your device has many different colors to it. You can use the following key to help diagnose problematic areas:

- 4x Overdraw Red
- 3x Overdraw Pink
- 2x Overdraw Green
- 1x Overdraw Blue

Ideally you want your application to only be showing Blue->Green.

When you identify problematic areas of overdraw, use `ClipRect()` / `QuickReject()` and/or remove backgrounds/drawables/views.

### 3. Compute

The compute section is mainly comprised of three major items:

- Profile
- Fix
- Repeat

**Debugging Tools:**

Traceview - [http://developer.android.com/tools/help/traceview.html](http://developer.android.com/tools/help/traceview.html)

[http://developer.android.com/tools/debugging/debugging-tracing.html](http://developer.android.com/tools/debugging/debugging-tracing.html)

systrace - [http://developer.android.com/tools/performance/systrace/index.html](http://developer.android.com/tools/performance/systrace/index.html)

`Trace.BeginSection()` - [http://developer.android.com/tools/debugging/systrace.html#app-trace](http://developer.android.com/tools/debugging/systrace.html#app-trace)

Analyze a systrace - [http://developer.android.com/tools/debugging/systrace.html#analysis](http://developer.android.com/tools/debugging/systrace.html#analysis)

Best Practices:

Make use of Batching/Caching to make less requests or have a locally saved cache to backup to.

Avoid blocking the UI/Main Thread by identifying pieces of work that can go on background threads.

### 4. Memory

Memory is a bit trickier with Xamarin.Android as there are two Garbage Collectors being used.

Mono

Dalvik/ART

**Debugging Tools:**

Memory Monitor - [http://developer.android.com/tools/performance/memory-monitor/index.html](http://developer.android.com/tools/performance/memory-monitor/index.html)

Heap Viewer - [http://developer.android.com/tools/performance/heap-viewer/index.html](http://developer.android.com/tools/performance/heap-viewer/index.html)

Allocation Tracker - [http://developer.android.com/tools/performance/allocation-tracker/index.html](http://developer.android.com/tools/performance/allocation-tracker/index.html)

Xamarin Profiler - [https://xamarin.com/profiler](https://xamarin.com/profiler)

**Memory Leaks:**

While looking in memory monitor, you may notice that you are allocating more memory than what is being collected by the GC. This is the first red flag of a potential memory leak.

### 5. Battery

**Battery Drain:**

Current draw over time is the key to what work is being done via your application.

What tasks in my application are draining the battery the fastest?

**Debugging Tools:**

Battery Historian - [http://developer.android.com/tools/performance/batterystats-battery-historian/index.html](http://developer.android.com/tools/performance/batterystats-battery-historian/index.html)

[https://github.com/google/battery-historian](https://github.com/google/battery-historian)

**Best Practices:**

Limit network access.

Use `JobScheduler` to schedule jobs against the framework that will be executed in your application's own process.