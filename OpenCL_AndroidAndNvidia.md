# OpenCL on Android and Nvidia

## Motivation (Warp Size)
Basically, I wanted to find the "warp size" (a.k.a. "wavefront size", a.k.a. "subgroup size", occasionally "SIMD width") on Android. OpenGL ES does not really have a way to find this , excepting some extensions that don't seem to be widely supported. OpenCL has some parameters/extensions that seem to contain this info, though. See [https://stackoverflow.com/questions/7093488/opencl-how-to-i-query-for-a-devices-simd-width](https://stackoverflow.com/questions/7093488/opencl-how-to-i-query-for-a-devices-simd-width) and take care to read the comments.

My other notes document why I originally wanted to find the warp size (and also why maybe it's actually _not_ useful after all).

## OpenCL on Android

### Trying to set up from scratch
OpenCL is not officially supported on Android, but a lot of chip manufacturers (including Qualcomm, which made the chips for my current smartphones) will support it and include a libOpenCL.so file somewhere under `/system/vendor/lib` or something like that.

How do you even access `/system/vendor/lib`, though? You can do `.\adb.exe shell` in a place with an adb.exe file (assuming it's not installed such that it's in the PATH), and then in this shell, you can run your ordinary linux ls, cd, etc. commands.

The only resources I found on how to _develop_ an OpenCL application for Android say that you need to include the libOpenCL.so file in your Android Studio project. This, of course, will be a device-specific file, so I'm not sure what the development process looks like for apps like [OpenCL-Z](https://play.google.com/store/apps/details?id=com.robertwgh.opencl_z_android&hl=en_CA&gl=US), which works on many different phones. How do you get the .so file onto your PC? Via `.\adb.exe pull /system/vendor/lib/libOpenCL.so`.

You then put that libOpenCL.so file somewhere. If you get a "More than one file was found" sort of error for libOpenCL.so when you build, it's likely because of ["Automatic packaging of prebuilt dependencies used by CMake"](https://developer.android.com/studio/releases/gradle-plugin#cmake-imported-targets). To quote the documentation:

> External native build now automatically packages \[prebuilt\] libraries, so explicitly packaging the library with `jniLibs` results in a duplicate. To avoid the build error, move the prebuilt library to a location outside `jniLibs` or remove the `jniLibs` configuration from your `build.gradle` file.

Speaking of CMake, you need to use this in your CMakeLists.txt to include a prebuilt library:
```
add_library(OpenCL SHARED IMPORTED)
set_target_properties(OpenCL PROPERTIES IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/<other stuff here>/libOpenCL.so)
```

What about the header files? For OpenCL, I got the C ones from [https://github.com/KhronosGroup/OpenCL-Headers/tree/main/CL](https://github.com/KhronosGroup/OpenCL-Headers/tree/main/CL) and the C++ ones from [https://github.com/KhronosGroup/OpenCL-CLHPP/tree/main/include/CL](https://github.com/KhronosGroup/OpenCL-CLHPP/tree/main/include/CL).

Now, libOpenCL.so will itself rely on certain libraries. You can see which ones in an Ubuntu shell by running the command `Ubuntu: readelf -dW ./libOpenCL.so`; I'm not sure if there's a way to do readelf on Windows without the WSL or similar.

Now, some of the dependencies, like `libcutils.so`, won't be "found" unless you edit your project to target lower SDK versions. After API level 23, they were made _private_ libraries. You can change the API level, as described at [stackoverflow.com/questions/19465049/changing-api-level-android-studio](stackoverflow.com/questions/19465049/changing-api-level-android-studio), then, to get around this. For more info, see [https://stackoverflow.com/questions/51825635/how-to-solve-unsatisfiedlinkerror](https://stackoverflow.com/questions/51825635/how-to-solve-unsatisfiedlinkerror) and [https://developer.android.com/about/versions/nougat/android-7.0-changes#ndk](https://developer.android.com/about/versions/nougat/android-7.0-changes#ndk). 

That won't help for the `libc++.so` dependency, though. More info on this library, and how it's different from things like `libc++_shared.so`, can be found at [https://stackoverflow.com/questions/47244362/is-aosps-libc-so-the-same-as-ndks-libc-shared-so](https://stackoverflow.com/questions/47244362/is-aosps-libc-so-the-same-as-ndks-libc-shared-so). To get around this problem, I found that also pulling and including the `libc++.so` into the project allowed the .apk to finally build. But even though it finally built… the OpenCL code I was trying wouldn't work. Maybe there's a bug in the code I wrote rather than the whole process above, but because of the hacks I had to do, particularly for `libc++.so`, I feel it may be a more difficult issue to fix.

Someone else got things to work using CMake, see [https://stackoverflow.com/a/60576005](https://stackoverflow.com/a/60576005), but when I follow the same steps they do, I can't get it to work. I wonder if this is partially because they mention the possibility of "Instead of linking and including libOpenCL.so in the .apk, I chose to **only link my c++ code** to it and then use libOpenCL.so the library locally installed on the phone." I think that this may no longer be possible with CMake and Android studio, as no matter what I tried, the libOpenCL.so would be bundled into my .apk, which I verified using the [APK Analyzer](https://developer.android.com/studio/debug/apk-analyzer), accessed via `Build > Analyze APK` in Android Studio. Meanwhile, using the below method (the modified Samsung tutorial) seemed to exclude the libOpenCL.so from the .apk, and the application then ran. Maybe one needs to use ndk-build instead of CMake when working with OpenCL on Android now?

### The way that actually worked (sort of)

I basically followed along this tutorial: [https://www.lantronix.com/blog/running-opencl-sample-application-open-q-820-system-module-powered-qualcomm-snapdragon-820-soc/](https://www.lantronix.com/blog/running-opencl-sample-application-open-q-820-system-module-powered-qualcomm-snapdragon-820-soc/). It involved downloading a web archived version of some old Samsung tutorial and then modifiying it.

I had to make a few modifications beyond what this new tutorial stated, but they were generally easy to Google or guess. E.g., I had to switch to Java 11, instead of 1.8, by going to `File > Settings... > Build, Execution, Deployment > Build Tools > Gradle > Gradle JDK`. See the accepted answer at: [https://stackoverflow.com/questions/66980512/android-studio-error-android-gradle-plugin-requires-java-11-to-run-you-are-cur](https://stackoverflow.com/questions/66980512/android-studio-error-android-gradle-plugin-requires-java-11-to-run-you-are-cur)

I also had to enable exceptions in by adding `LOCAL_CPP_FEATURES += exceptions` to the `Android.mk` file. See the top-voted (NOT the accepted!) answer at [https://stackoverflow.com/questions/3217925/enable-exception-c](https://stackoverflow.com/questions/3217925/enable-exception-c).

Note that, this time when I analyzed the built .apk, libOpenCL.so did _not_ seem to be included, though perhaps I simply didn't find it.

However, I couldn't get the extensions to work.


### Extensions
I could not get extensions to work in the modified Samsung code. At first, I thought that was because of the headers the Samsung code included, which were very old and for OpenCL 1.2. So I deleted those and used the new headers instead. They contained the definitions for the extension and everything, but the functionality still wouldn't work. I can't recall right now, as I type this, whether the version that used the extension in the C++ code built or not. If it didn't build, that seems to suggest to me that maybe there's still something not working right with the headers. If that's the case, perhaps I could further-confirm this by having code sort of like:
```cpp
#ifdef <Extension Name Here>
printf("Seems the header stuff for the extension is fine!");
#else
printf("Extension header issue, I think!");
#endif
```

When I printed out which OpenCL extensions are supported on Android, it _said_ that the subgroup one was. But the function calls wouldn't work. I even tried the versions with "KHR" in their names, e.g., [https://www.khronos.org/registry/OpenCL/sdk/2.0/docs/man/xhtml/clGetKernelSubGroupInfoKHR.html](https://www.khronos.org/registry/OpenCL/sdk/2.0/docs/man/xhtml/clGetKernelSubGroupInfoKHR.html). 

### Potential Fixes to Above Problems?

[This tutorial by ArrayFire](https://web.archive.org/web/20180311023343/https://arrayfire.com/getting-started-with-opencl-on-android/) mentions a different way of getting the header files and whatnot, which would involve getting the SDK made by whoever made those chips. It seems such an SDK might exist: [https://developer.qualcomm.com/blog/accelerate-your-models-our-opencl-ml-sdk](https://developer.qualcomm.com/blog/accelerate-your-models-our-opencl-ml-sdk)

## OpenCL on Nvidia GPUs
It comes packaged with CUDA. Unfortunately, it's generally only OpenCL 1.2 that's supported, whereas the subgroup functionality I wanted to test (to compare the results on Nvidia GPUs with the _known_ warp size of 32 and use this to assess the potential accuracy on Android) is in OpenCL 2.0, 2.1, or 2.2.

Newer drivers may allow OpenCL 3.0 support. See: [https://developer.nvidia.com/opencl](https://developer.nvidia.com/opencl)

However, OpenCL 3.0 is in some sense a _subset_ rather than a _superset_ of OpenCL 2.0. A lot of the stuff that was required in 2.0 was made optional in 3.0. Thus, I'm not sure if the subgroup functionality would even be supported in 3.0. It's possible it'd be considered redundant, since there are Nvidia-specific OpenCL extensions that have the same information, in addition to "common knowledge" that it'd be 32 consistently. I'd also need to install non-OEM drivers if I wanted to test this on my laptop.


