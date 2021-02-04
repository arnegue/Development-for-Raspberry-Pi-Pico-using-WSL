# About
This manual shows how to develop C/C++ Software for the [Raspberry Pi Pico](https://www.raspberrypi.org/documentation/pico) on a Windows-Machine using the WSL (Windows Subsystem for Linux) (in this specific example Ubuntu is used).
Usually Raspberry's manual advises you to develop on a Raspberry Pi itself. But my Pi 3B Did not have cmake 3.13. Yes, I could've build it myself BUT, why shouldn't I use the power of my desktop-PC (which only has Windows installed, but WSL)?

This repository shows:
* How to install the Pico-SDK on your Desktop-PC using the Linux-Subsystem
* How to set up the project with [CLion](https://www.jetbrains.com/de-de/clion/) on Windows (not WSL)
* Develop a new project
* Remote-compile it on your WSL (not Raspberry)
* Transfer it on your Pico
* Debug it (currently i see no other way but doing it with another Raspberry Pi (non-Pico)

The content of this repository is only example code. If you want to get this running instead of following all these steps you also have to get the pico-sdk-submodule with:

* `git submodule init` 
* `git submodule update`

# Setup SDK
* In WSL follow all steps in step 2 of the [manual](https://datasheets.raspberrypi.org/pico/getting-started-with-pico.pdf).
* If your cmake-version is still < 3.13 (check with `cmake --version`). Follow these steps (it will install CMake 3.19, but that works too):
    1. `wget http://www.cmake.org/files/v3.19/cmake-3.19.4.tar.gz`
    2. `tar -xvzf cmake-3.19.4.tar.gz`
    3. `./configure`
    4. `make`
    5. `sudo make install`

# Make it blink (local compiling)
## Compile
Follow step 3.1 of the manual

> You may come into the situation that Python3 is not installed and therefore won't compile. To install it run `sudo apt install python3`

## Copy
To finally copy the binary to your Raspberry Pi Pico, you have to copy `blink.uf2` to a directory Windows can access.
1. E.g on your WSL: `cp blink.uf2 /mnt/c/temp/`. This copies your file to Windows `C:\temp`.
2. Now on your Windows system simply copy the binary by hand to your Raspberry Pi Pico's mass storage
3. After the copy was complete the raspberry will restart itself automatically and start blinking

Now you've got your first blinking. 

## Remount
If you want to remount your Pico to load another program:
1. Unplug your Pico
2. Press and hold the BOOTSEL-button
3. Plug in your Pico
4. Release the BOOTSEL-button

> Your uf2-file will be removed when remounting!


# Make it blink (remote compiling)

## SSH (Communication between WSL and Windows)

The most common way to communicate with Linux systems over network is SSH.
To setup a SSH-Server on your WSL, you need to edit the SSH-Daemon-configuration file at `/etc/ssh/sshd_config`: Change these value (may need to remove `#`)
* `ListenAddress` to `127.0.0.1`. You can keep it like it is, but this way only your Windows can access it
* `PasswordAuthentication` to `yes`
* `AllowUsers` to <your_user_name>
* `sudo ssh restart` (sometimes `sudo sshd restart`)

## Setup CLion

* Create a new project in CLion with File -> New Project -> C++ Executable
* In project's directory add Pico-SDK as a git-submodule: `git submodule add https://github.com/raspberrypi/pico-sdk pico-sdk`
* File -> Settings -> Build, Execution, Development -> ToolChains
    * \+ -> WSL
        * Credentials -> (gear ->) add your user name and password and maybe test the connection
        * Usually CLion automatically selects the correct cmake. In my case it was the old one. You can identify the correct cmake with `whereis cmake` and execute with either ``cmake --version`` (mine was located in `/usr/local/bin/cmake`)
        * The other ones should be detected without a problem.
        * There might be a warning that currently no cmake 3.19.4 is supported, it does work though
        * GDB is not needed since we do not debug the program via WSL
    * After adding your ToolChain, ensure that in the ToolChains-List your WSL-Toolchain is the first one
* File -> Settings -> Build, Execution, Development -> CMake
    * \+ 
    * Set the name as you want 
    * Build Type: Debug
    * Environment: `PICO_SDK_PATH=<sdk-path of step 2 in manual>` (mentioned in "Setup SDK", mine was placed in `/home/pi/pico/pico-sdk`)
    * After adding your CMake-Profile, ensure that in the CMake-Profile-list your profile is the first one in the list
* Edit CMakeList.txt (see example in repository)
    * Include SDK: `include(pico-sdk/pico_sdk_init.cmake)`
    * Initialize it: `pico_sdk_init()`
    * Link SDK to your project: `target_link_libraries(pico_example pico_stdlib)`
    * Get your `.uf2`-file: `pico_add_extra_outputs(pico_example)`

    

  