In order to see what a CMake variable is set to (so that you can decide whether or not you want to delete the cache, override a default, etc.), you can use `cmake -L /path/to/CMakeLists.txt | grep MY_CMAKE_VARIABLE` to do so from the command line without having to search through a bunch of long files. Source: [https://stackoverflow.com/questions/8474753/how-to-get-a-cmake-variable-from-the-command-line](https://stackoverflow.com/questions/8474753/how-to-get-a-cmake-variable-from-the-command-line)

To change the install location of a library without having to rebuild it (which may take a long time), you can re-run CMake with the option `-DCMAKE_INSTALL_PREFIX=<somewhere different to last time>`, which should not require the project being re-built, and then just install with `cmake --install` or `make install`. **If you are _moving_ an already-installed library**, you'll probably want to uninstall first, before you change the CMake setup that would allow an easy uninstall….

# Parallel Compilation
Running `make` with the `-j4` (replace "4" with however many processors your computer has or you want to use) is _sooo_ much faster than the default. However, if you want to use this while still using `cmake --build`, you can do this by using `-- <build-tool-options go here>` at the end of your CMake call. E.g., `cmake --build . --target install -- -j12`.

**Update:** It seems you can now use `--parallel <num jobs>` instead on Linux. On Windows, however, the behaviour of `--parallel` may not be what you expect, as explained in Nikita's answer at https://stackoverflow.com/questions/10688549/how-do-i-configure-portable-parallel-builds-in-cmake. 
Instead, on Windows, you may want `-- /p:CL_MPcount=<num jobs>`.

# GLEW
GLEW does not really come with a config .cmake file, unless you download the GitHub source and build from it. But since CMake comes with FindGLEW functionality, all you have to do is download the windows binary, extract it somewhere, and pass in the location as part of `CMAKE_PREFIX_PATH`. E.g.:
```
cmake -T host=x64 -DCMAKE_BUILD_TYPE=DEBUG -DCMAKE_PREFIX_PATH="D:\PortableLibraries\glew-2.2.0-win32\glew-2.2.0" ..
```

# C++ Versions

It seems that Visual Studio 2022 doesn't support C++11. See: https://learn.microsoft.com/en-us/answers/questions/903797/c-11-in-2022-(please-dont-tell-me-about-c11-or-c-1
(This isn't _exactly_ a CMake note, but it's close enough.)

# Windows C Runtime Library (CRT)
Was getting errors like `error LNK2019: unresolved external symbol __imp___stdio_common_vsscanf referenced in function sscanf` when linking some code from GitHub with glfw3 install.

GLFW's CMake had `USE_MSVC_RUNTIME_LIBRARY_DLL` set by default, meaning `/MD` flag is used instead of `/MT`. 
Flag options: https://learn.microsoft.com/en-us/cpp/build/reference/md-mt-ld-use-run-time-library?view=msvc-170&viewFallbackFrom=vs-2019
Note that this has nothing to do with whether GLFW itself is static or shared. As explained at https://stackoverflow.com/questions/35116437/errors-when-linking-to-protobuf-3-on-ms-visual-c, you can have static or dynamic libs use either `/MD` or `/MT`.

Meanwhile, the GitHub code had:
```cmake
if (WIN32)
    set(GLEW_STATIC true)
    set(OpenCV_STATIC true)
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -MTd")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -MT")
endif ()
```
As explained at https://stackoverflow.com/questions/35116437/errors-when-linking-to-protobuf-3-on-ms-visual-c, you can't "mix" `/MT` and `/MD` in this way. The SO answer seemed to suggest it's generally better to use `/MD`, so I changed the GitHub code rather than GLFW's CMake options.

To see whether an existing .lib file was compiled with `/MD` or `/MT`, you can follow https://stackoverflow.com/questions/25284513/check-what-run-time-static-library-or-dll-uses and open the .lib file in a text editor, then search for `cl.exe` or `-defaultlib` and see what comes after. For the latter, "msvcrt" means `/MD` and "libcmt" means `/MT`, as explained at https://stackoverflow.com/questions/3007312/resolving-lnk4098-defaultlib-msvcrt-conflicts-with.

