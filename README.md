# Drekar Launch Process C++

This library contains client utility functions for processes launched using `drekar-launch`, although they may also be used without `drekar-launch`. Currently this package provides a reliable way for processes to receive shutdown signals from a process manager or the user using `ctrl-c`. This package contains
C++ implementations, analogous to `drekar-launch-process` for Python. See https://github.com/johnwason/drekar-launch-process for more information.

## Building

Use cmake to build the library.

```
git clone https://github.com/johnwason/drekar-launch-process-process-cpp
mkdir build
cd build
cmake ..
cmake --build .
```

Alternatively build using vcpkg:

```
git clone https://github.com/microsoft/vcpkg.git
cd vcpkg
git clone https://github.com/robotraconteur/vcpkg-robotraconteur.git
vcpkg --overlay-ports=vcpkg-robotraconteur\ports install drekar-launch-process-cpp:x64-windows
```

## Usage

The class `CWaitForExit` provides utility functions to wait for program exit. The `WaitForExit()` function
will block until the exit signal is received, or the `CallbackWaitForExit()` function will asynchronously
trigger a callback when the exit signal is received.

Blocking example:

```cpp
#include "drekar_launch_process_cpp/drekar_launch_process_cpp.h"
#include <iostream>

int main()
{
    drekar_launch_process_cpp::CWaitForExit wait_for_exit;
    std::cout << "Press Ctrl+C to exit" << std::endl;
    wait_for_exit.WaitForExit();
    std::cout << "Exit" << std::endl;
    return 0;
}
```

Callback example:

```cpp
#include "drekar_launch_process_cpp/drekar_launch_process_cpp.h"
#include <iostream>
#include <condition_variable>
#include <functional>

std::condition_variable cv;
std::mutex cv_mutex;
bool exit_received = false;

void exit_callback()
{
    std::unique_lock<std::mutex> lock(cv_mutex);
    exit_received = true;
    cv.notify_all();
}

int main()
{
    drekar_launch_process_cpp::CWaitForExit wait_for_exit;
    wait_for_exit.CallbackWaitForExit(exit_callback);
    std::unique_lock<std::mutex> lock(cv_mutex);
    std::cout << "Press Ctrl+C to exit" << std::endl;
    cv.wait(lock, [] {return exit_received; });
    std::cout << "Exit" << std::endl;
    return 0;
}
```

