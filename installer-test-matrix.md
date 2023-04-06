# Installer Test Matrix Workflows

## General Docs

* [O3DE Welcome Guide](https://www.o3de.org/docs/welcome-guide/)
* [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/)
* [Setup O3DE](https://www.o3de.org/docs/welcome-guide/setup/)

## Common Issues to Look out For

* Invalid install location (Directory isn't empty)
* Install already exists (side-by-side install not currently supported)

## Workflows

### Area: Installer Requirements

#### Platform Requirements

25/1/2022 - This file contains numerous setup configurations that are recommended to run against each O3DE releases installers.

##### Windows Requirements

* Microsoft Visual Studio 2019 version 16.9.2 through version 16.11.x are supported with O3DE.
    *   This includes manually adding desktop/game development with C++ workloads
* Visual C++ Redistributable for Visual Studio 2019     
* CMake 3.20.5 or later

**Windows Requirement Alternatives**

* Visual Studio 2022

##### Linux Requirements

* CMake 3.20.5 or later      
* Clang 12
* Vulkan Drivers (hardware specific)
* libffi (an older version for Ubuntu, which doesn't currently support the most recent)
* libglu1-mesa-dev
* libxcb-xinerama0
* libxcb-xinput0
* libxcb-xinput-dev
* libxcb-xfixes0-dev
* libxcb-xkb-dev
* libxkbcommon-dev
* libxkbcommon-x11-dev
* libfontconfig1-dev
* libcurl4-openssl-dev
* libsdl2-dev
* zlib1g-dev
* mesa-common-dev  

**Linux Requirement Alternatives**

* Clang 13


#### Platform Configurations

<details>
<summary>Test Matrix Generation Data</summary>

Note: In all cases where a requirement is intentionally left out or incorrect, the goal is to test for clear error messaging related to the issue (in some cases these errors will happen at the OS level and it may not be in our power to update the error messaging, but these should be recorded for reference/possible Troubleshooting page)

**Windows**

*   Prerequisites are met exactly as written
*   No CMake
*   No Visual Studio
*   No VS 2019 redistributable
*   No prerequisites met
*   Previous Installer already installed
*   Visual Studio 2022 used in place of 2019
*   Earlier version of CMake installed (earlier than 3.20.5)
*   Visual studio version other than 16.9.2 through version 16.11.x is used (while using 2019, 2022 will inherently be this)

**Linux**

* Prerequisites are met exactly as written
* No CMake
* No Clang
* Most recent libffi version
* None of the additional libs (these are generally installed all at once)
* No prerequisites met
* Clang 13 used in place of 12
* Earlier version of CMake installed(earlier than 3.20.5)

</details>

#### Installer Requirements Workflows

| Scenario                                                                          | OS              | Workflow                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Things to Watch For |
|-----------------------------------------------------------------------------------|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|
| Prerequisites are fully met                                                       | Windows & Linux | <ol><li>If O3DE is not already installed, install all the prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Launch O3DE.exe.</li><li>Create a project if one doesn't exist.</li><li>Build the project.</li><li>Launch the Editor.</li></ol>                                                                                                                                                                                                |                     |
| CMake isn't installed                                                             | Windows & Linux | <ol><li>If O3DE is already installed, uninstall CMake.</li><li>Install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Launch O3DE.exe.</li><li>Create and build a new project.</li><li>Launch the Editor.</li></ol>                                                                                                                                                                                                       |                     |
| None of the prerequisites are met                                                 | Windows & Linux | <ol><li>If they are already installed, uninstall all the prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). <ul><li>To uninstall: See above for the individual items.</li></ul><li>Install most recent release installer.</li><li>Launch O3DE.exe.</li><li>Create and build a new project.</li><li>Launch the Editor.</li></ol>                                                                                                                                                                     |                     |
| Previous Installer already installed                                              | Windows & Linux | <ol><li>If previous installer is not already installed, install it.</li><li>Install most recent release installer.</li><li>Launch O3DE.exe.</li><li>Create and build a new project.</li><li>Launch the Editor.</li></ol>                                                                                                                                                                                                                                                                                                                             |                     |
| CMake version earlier than 3.20.5 is used                                         | Windows & Linux | <ol><li>If cmake version installed is greater than the scenario version, uninstall CMake.</li><li>Install a version earlier than 3.20.5 if not already installed from [CMake Releases](https://github.com/Kitware/CMake/releases). </li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Launch O3DE.exe</li><li>Create and build a new project.</li><li>Launch the Editor.</li></ol> |                     |
| Visual Studio isn't installed                                                     | Windows         | <ol><li>If it is already installed, uninstall Visual Studio.</li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Launch O3DE.exe.</li><li>Create and build a new project.</li><li>Launch the Editor.</li></ol>                                                                                                                                                                       |                     |
| VS 2019 redistributable isn't installed                                           | Windows         | <ol><li>If it is already installed, uninstall VS2019 redistributable.</li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Launch O3DE.exe.</li><li>Create and build a new project.</li><li>Launch the Editor.</li></ol>                                                                                                                                                              |                     |
| Visual Studio 2022 used instead of 2019                                           | Windows         | <ol><li>If it is already installed, uninstall Visual Studio 2019.</li><li>Install Visual Studio 2022.</li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Launch O3DE.exe.</li><li>Create and build a new project.</li><li>Launch the Editor.</li></ol>                                                                                                                              |                     |
| Unsupported Visual Studio version is used (something other than 16.9.2 - 16.11.x) | Windows         | <ol><li>If it is already installed, uninstall Visual Studio.</li><li>Install a version of Visual Studio that falls outside the supported version range.</li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Launch O3DE.exe.</li><li>Create and build a new project.</li><li>Launch the Editor.</li></ol>                                                                            |                     |

---

### Area: Installer Requirements

#### Something SOmething 

| Scenario                                                                          | OS              | Workflow                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Things to Watch For |
|-----------------------------------------------------------------------------------|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|
| Install O3DE In Default Location                                                  | Windows & Linux | <ol><li></li></ol> 1.  Download latest Release or Development Installer<br>2.  Run the Installer<br>3.  Select "Install"                                                                                                                                                                                                                                                                                                                                                                                                                             | *   O3DE successfully installs in the default location (Generally the root of the C drive)<br>*   User is able to launch o3de.exe |
| Install O3DE in Custom Location                                                   | Windows & linux | <ol><li></li></ol> 1.  Download latest Release or Development Installer<br>2.  Run the Installer<br>3.  Select "Options"<br>4.  Browse to a new location<br>5.  Select "OK"<br>6.  Select "Install"                                                                                                                                                                                                                                                                                                                                                  | *   O3DE successfully installs in the user defined location<br>*   User is able to launch o3de.exe                                |
| Uninstall O3DE                                                                    | Windows & Linux | <ol><li></li></ol> 1.  With O3DE already installed, launch the installer<br>2.  Select "Uninstall"                                                                                                                                                                                                                                                                                                                                                                                                                                                   | The installer successfully uninstalls O3DE from the previously defined location on disk                                           |





