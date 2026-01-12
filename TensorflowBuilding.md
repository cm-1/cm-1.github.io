Finally got 2.19.1 building on Windows. Used LLVM 21.1.8, Python 3.10.19, MSYS\_NT-10.0-26200 version 3.6.5, VS2019 with VS2022 build tools installed. Some of that might not have been necessary.

The build command, which again may contain some unnecessary parameters, was:
```
bazelisk build -c opt --config=monolithic --config=win_clang --compiler=clang-cl --shell_executable="C:/msys64/usr/bin/bash.exe" --action_env=PATH="C:/msys64/usr/bin;C:/Windows/System32;C:/Windows;<venv_path>/Scripts/" //tensorflow/lite/delegates/flex:tensorflowlite_flex
```

This builds a `tensorflowlite_flex.dll` somewhere in `bazel-bin`. However, then you have to actually _use_ that .dll with your C++ program. And since the .dll was compiled with clang..…

Perhaps my issue was just release vs debug. See:

- https://stackoverflow.com/questions/73610444/tensorflow-lite-c-build

Other links to try still:

- https://github.com/tensorflow/tensorflow/issues/62227
- https://github.com/tensorflow/tensorflow/issues/47166
- https://github.com/tensorflow/tensorflow/issues/43367
- https://github.com/tensorflow/tensorflow/issues/62347

Can debug the .exe with https://stackoverflow.com/questions/15097610/debugging-an-executable-in-visual-studio.


# Building clang program with 

# Bazel notes:
***TODO:*** Move this to its own file.

## Specifying number of jobs:
`--jobs=n (-j)`

## Changing location bazel-bin and whatnot point to
Command looks like `bazel(isk) --output_base=<directory> build -c opt ...`



# Trying v2.15

`C:\PortablePrograms\cmake-3.17.2-win64-x64\bin\cmake.exe -E env CXXFLAGS="/DTFLITE_MMAP_DISABLED" C:\PortablePrograms\cmake-3.17.2-win64-x64\bin\cmake.exe ..\tensorflow_src\tensorflow\lite\examples\minimal\`



## Misc. Links:

- https://web.archive.org/web/20231123183352/https://www.tensorflow.org/install/source_windows
- https://web.archive.org/web/20231119132155/https://www.tensorflow.org/lite/guide/build_cmake