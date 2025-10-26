# How to set up VS Code to Run and Debug C++ code on MacOS

This is a short guide on how to work with C++ in VSCode on MacOS.
I wanted it to be just as convenient as using CLion (another IDE specifically for C++)
So this is what I came up with so far

## Extentions

Add these extentions:
1. The basic C++ extention that will allow you to use IntelliSense (code suggestions and lookup in the editor)  
https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools

2. This extention is for building executables from C++ code (so that a computer can actually run the program)  
https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools

3. This extention is for auto formatting code on save  
https://marketplace.visualstudio.com/items?itemName=xaver.clang-format

## Settings

Open settings using `cmd ,`

Change the following settings:

1. **clang-format.executable**  
We should put the path to our excutable here  
To find it out, open Terminal and run `which clang-format`     
For me it shows `/opt/homebrew/bin/clang-format`  
So paste this into the setting  

2. **@lang:cpp editor.formatOnSave = true**, **@lang:cpp editor.formatOnPaste = true**  
Now the files will be automatically reformatted on save and paste

4. **@lang:cpp editor.defaultFormatter: "ms-vscode.cpptools"**  
Specifying a formatter to use

5. **cmake.cmakePath**  
Paste the path to your `cmake` executable  
Again, use `which cmake` to find it out  
For me: `/opt/homebrew/bin/cmake`  

## Create a project

Open a new folder in VS Code and create a .cpp file

For example I created `helloworld.cpp` and put this code inside:

```
#include <iostream>

int main() {
  int a = 10;

  std::cin >> a;

  std::cout << a;

  std::cin >> a;

  return 0;
};
```

## Set up a project using CMake 

Open a Command Pallette using `cmd + shift + P`

Run the command `CMake: Quick Start`

Our installed CMake extention will guide you through creating `CMakePresets.txt` and `CMakeLists.txt` files 



`CMakePresets.txt` will look something like this:
```
{
  "version": 8,
  "configurePresets": [
    {
      "name": "test",
      "displayName": "Clang 15.0.0 arm64-apple-darwin23.5.0",
      "description": "Using compilers: C = /usr/bin/clang, CXX = /usr/bin/clang++",
      "binaryDir": "${sourceDir}/out/build/${presetName}",
      "cacheVariables": {
        "CMAKE_INSTALL_PREFIX": "${sourceDir}/out/install/${presetName}",
        "CMAKE_C_COMPILER": "/usr/bin/clang",
        "CMAKE_CXX_COMPILER": "/usr/bin/clang++",
        "CMAKE_BUILD_TYPE": "Debug"
      }
    }
  ]
}
```

It tells Cmake what complier to use when building executables from our C++ code.
In my case it uses clang++ compiler (it comes with MacOS)

`CMakeLists.txt` files tells Cmake what executables to build 

```
cmake_minimum_required(VERSION 3.10.0)
project(test VERSION 0.1.0 LANGUAGES C CXX)

add_executable(test helloworld.cpp)
```

Now I have only one executable `test` that is based on my file `helloworld.cpp`

Let's say I create another .cpp file in my project  
Let's call it `anotherprogram.cpp`

```
#include <iostream>
#include <queue>

int main() {
  int a;

  std::cin >> a;
}
```

To turn this new file into an executable I go to my `CmakeLists.txt` file and add a new line: 

```
cmake_minimum_required(VERSION 3.10.0)
project(test VERSION 0.1.0 LANGUAGES C CXX)

add_executable(test helloworld.cpp)

add_executable(another anotherprogram.cpp)
```

Now I have two independent executables and I will be able to run them separately!

## Configure a project using CMake 

Okay, now we have to configure our project 

It means that cmake will create a makefile that contains specific instructions on how to build our executables 

Basically, those are the same instructions we have in our `CMakeLists.txt` file, but written in a computer-readable format

To configure the project, use `CMake: Configure` in Command Pallette (`cmd + shift + P`)

You will see that Cmake will create a new `out` folder in our directory  

There's a Makefile inside!

## Build a project using CMake

Finally, let's build our project.
It means we want to create actual executables using Makefile that we now have 

Run `Cmake: Build Target` in Comand Pallete

<img width="1188" height="264" alt="image" src="https://github.com/user-attachments/assets/5de4ebf0-586f-4432-a8b9-ee4f75e6f512" />


You can see that we can choose a specific executable to build or build all executables

After doing that, `out` folder contains executables! 

For example, you can run `out/build/test/another` to run our code from `anotherprogram.cpp` file

## Run our project 

To run a project, just use `CMake: Run Without Debugging`

<img width="956" height="92" alt="image" src="https://github.com/user-attachments/assets/5a8adfed-c934-457e-a24d-2b5255eea9eb" />

It will prompt you an executable to run. 

If you later want to change an executable, just use `Cmake: Set Launch/Debug Target`

You can also set a shortcut for `CMake: Run Without Debugging` 

(just click on the gear button next to it)

<img width="1164" height="44" alt="image" src="https://github.com/user-attachments/assets/cfd3a5aa-b216-4adb-9f90-efd2fe6dc177" />

## Debug our project 

This is a tricky part.

The standard tools in Visual studio don't support a native MacOS C++ debugger "lldb"

So for that reason we will install another extention:  
https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb

It will allow us to use the native `lldb` MacOS debugger

After installing the extention we need to do is to create `launch.json` file in `.vscode` folder in our project directory

```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug with CodeLLDB",
      "type": "lldb",
      "request": "launch",
      "program": "${command:cmake.launchTargetPath}",
      "args": [],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal"
    }
  ]
}
```

This will create a new debugging configuration called "Debug with CodeLLDB"  
This configuration tells our extention to use `lldb` when debugging

You can also create multiple configurations for each executable.  
For each configuration, specify "program" parameter - it is the path to an executable that we want to debug 

In the confuguration above I set "program" parameter to `"${command:cmake.launchTargetPath}"`  
It means that we will use the executable that Cmake built for us 

Let's say I first set an executable to be "test"  
(remember, this is done using `Cmake: Set Launch/Debug Target`)  
And then the debugger will use this executable and we will debug `helloworld.cpp` file.

And if I later change the executable to be "another"  
Then we will debug `anotherprogram.cpp` file 

So you don't have to manually create a debuging configuration in `launch.json` every time you want to debug a different file

-------

Okay, Now open a VSCode Debugger View on the left panel (or use `cmd + shift + D`)

Choose our debugging configuration

<img width="944" height="578" alt="image" src="https://github.com/user-attachments/assets/25660495-8b9a-4d4f-9226-3941267fb4fa" />

And then press the green play button. 

Hopefully it works!

## Note

Every time you open a .cpp file in VSCode, you might see something like this:

<img width="1780" height="698" alt="image" src="https://github.com/user-attachments/assets/80d7cd2c-c6c4-4512-9f27-da5c473ad807" />

There's a play button with 3 or 2 options (I have three)

1. Actions "Debug C/C++ File" and "Run C/C++ File" are provided by our C/C++ extention. 

2. If you have action "Run", it is most likely provided by an extention called Code Runner  
https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner

As you can see, we use none of these actions 

The reason is that C/C++ extention doesn't support native `lldb` debugger for MacOS.  
So if you try to run a program, you won't be able to type any input in a terminal

You can fix this by creating a specific configuraion in `launch.json` that looks something like this:

```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "(lldb) Launch",
      "type": "cppdbg",
      "request": "launch",
      "program": "${command:cmake.launchTargetPath}",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}",
      "environment": [
        {
          "name": "PATH",
          "value": "${env:PATH}:${command:cmake.getLaunchTargetDirectory}"
        },
        {
          "name": "OTHER_VALUE",
          "value": "Something something"
        }
      ],
      "externalConsole": true,
      "MIMode": "lldb"
    }
  ]
}
```

This configuration uses a native "cppdbg" debugging wrapper for VisualStudio  
(But under the hood it actually connects to "lldb" debugger using "MIMode")  
This whole process creates some problem when trying to use a terminal for the input

So we set `"externalConsole": true`  
It means that the termianal will open in an external window  
That might work, but it's very inconvenient for me  
Using intergrated terminal in VSCode is much more convenient  

And the Run button by "Code Runner" extention actually works with intergrated terminal  
But there are some cons:  
1. It only works with one .cpp file. So you can't build a project using many files that import from each other  
2. It compiles the file every time
3. You cannot debug with it (At least I don't know how)

So we actually don't any of these options  

## This is how the project looks by the end of the guide

<img width="598" height="440" alt="image" src="https://github.com/user-attachments/assets/9dc6dddf-8522-4404-86c8-4193da5aa3e5" />

It's okay if you don't have `settings.json`

Those are the workspace specific settings in VSCode that are not neccessary

