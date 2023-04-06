# Installer Test Matrix Workflows

## General Docs

* [O3DE Welcome Guide](https://www.o3de.org/docs/welcome-guide/)
* [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/)
* [Setup O3DE](https://www.o3de.org/docs/welcome-guide/setup/)

## Common Issues to Look out For

* Invalid install location (Directory isn't empty)
* Install already exists (side-by-side install not currently supported)

## Installer FTUE

**First Time User Experience(FTUE):** 

* Launch O3DE.exe.
* Create and build a new project.
* Launch the O3DE Editor.
* Create a quick sample level.
* Enter Game Mode.

## Workflows

### Area: Installer Requirements

This area is used to test the Installer Requirements for the **Windows Installer** & **Linux Installer (deb)**. 

**Platform Requirements**

25/1/2022 - This file contains numerous setup configurations that are recommended to run against each O3DE releases installers.

*Windows Requirements*

* Microsoft Visual Studio 2019 version 16.9.2 through version 16.11.x are supported with O3DE.
    *   This includes manually adding desktop/game development with C++ workloads
* Visual C++ Redistributable for Visual Studio 2019     
* CMake 3.20.5 or later

*Windows Requirement Alternatives*

* Visual Studio 2022

*Linux Requirements*

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

*Linux Requirement Alternatives*

* Clang 13


**Platform Configurations**

<details>
<summary>Test Matrix Generation Data</summary>

Note: In all cases where a requirement is intentionally left out or incorrect, the goal is to test for clear error messaging related to the issue (in some cases these errors will happen at the OS level and it may not be in our power to update the error messaging, but these should be recorded for reference/possible Troubleshooting page)

*Windows*

*   Prerequisites are met exactly as written
*   No CMake
*   No Visual Studio
*   No VS 2019 redistributable
*   No prerequisites met
*   Previous Installer already installed
*   Visual Studio 2022 used in place of 2019
*   Earlier version of CMake installed (earlier than 3.20.5)
*   Visual studio version other than 16.9.2 through version 16.11.x is used (while using 2019, 2022 will inherently be this)

*Linux*

* Prerequisites are met exactly as written
* No CMake
* No Clang
* Most recent libffi version
* None of the additional libs (these are generally installed all at once)
* No prerequisites met
* Clang 13 used in place of 12
* Earlier version of CMake installed(earlier than 3.20.5)

</details>

### Installer Requirements Workflows

| Scenario                                                                          | OS              | Workflow                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Things to Watch For                                                                                                  |
|-----------------------------------------------------------------------------------|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| Prerequisites are fully met                                                       | Windows & Linux | <ol><li>If O3DE is not already installed, install all the prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                                                                                                                                                                   | <ul><li>O3DE successfully installs in the default location.</li><li>User is able to complete the FTUE.</li></ul>     |
| CMake isn't installed                                                             | Windows & Linux | <ol><li>If CMake is already installed, uninstall CMake.</li><li>Install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                                                                                                                                       |                                                                                                                      |
| None of the prerequisites are met                                                 | Windows & Linux | <ol><li>If they are already installed, uninstall all the prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). <ul><li>To uninstall: See above for the individual items.</li></ul><li>Install most recent release installer.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                                                                                                      |                                                                                                                      |
| Previous Installer already installed                                              | Windows & Linux | <ol><li>If previous installer is not already installed, install it.</li><li>Install most recent release installer.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                                                                                                                                                                                                                                                              |                                                                                                                      |
| CMake version earlier than 3.20.5 is used                                         | Windows & Linux | <ol><li>If cmake version installed is greater than the scenario version, uninstall CMake.</li><li>Install a version earlier than 3.20.5 if not already installed from [CMake Releases](https://github.com/Kitware/CMake/releases). </li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol> |                                                                                                                      |
| Visual Studio isn't installed                                                     | Windows         | <ol><li>If it is already installed, uninstall Visual Studio.</li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                                                                                                        |                                                                                                                      |
| VS 2019 redistributable isn't installed                                           | Windows         | <ol><li>If it is already installed, uninstall VS2019 redistributable.</li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                                                                                               |                                                                                                                      |
| Visual Studio 2022 used instead of 2019                                           | Windows         | <ol><li>If it is already installed, uninstall Visual Studio 2019.</li><li>Install Visual Studio 2022.</li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                                                               | <ul><li>O3DE successfully installs in the default location.</li><li>User is able to complete the FTUE.</li></ul>                                                                                                                        |
| Unsupported Visual Studio version is used (something other than 16.9.2 - 16.11.x) | Windows         | <ol><li>If it is already installed, uninstall Visual Studio.</li><li>Install a version of Visual Studio that falls outside the supported version range.</li><li>If not already installed, install the remaining prerequisites found at: [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/). </li><li>Install most recent release installer.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                             |                                                                                                                      |

---

### Area: General Installer

This area will test General Installer Workflows for the **Windows Installer**, **Linux Installer (deb)**, and 
**Linux Snap Installer**.

**Project Requirements**

* The **Windows Installer**, **Linux Installer (deb)**, or **Linux Snap Installer** is present on your machine.
* [O3DE Requirements](https://www.o3de.org/docs/welcome-guide/requirements/) are satisfied on your machine.
* [Linux Snapd](https://snapcraft.io/docs/installing-snapd) is installed on your Linux machine.
* Linux Snap Package under test is available on the Snapcraft store.

**Product:** O3DE is installed and a new project with a sample level has been created.

**Suggested Time Box:** 2 hours per platform.

### General Installer Workflows

| Scenario                                         | OS                                  | Workflow                                                                                                                                                                                                                                                                                                                                                            | Things to Watch For                                                                                                              |
|--------------------------------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| Install O3DE in Default Location                 | Windows Installer & Linux Installer | <ol><li>Download latest Release or Development Installer.</li><li>Run the Installer.</li><li>Select **Install**.</li><li>Run through the [Installer FTUE](#installer-FTUE). </li></ol>                                                                                                                                                                              | <ul><li>O3DE successfully installs in the default location.</li><li>User is able to complete the FTUE.</li></ul>                 |
| Install O3DE in Custom Location                  | Windows Installer & Linux Installer | <ol><li>Download latest Release or Development Installer.</li><li>Run the Installer.</li><li>Select "Options" and select your non-default install location.</li><li>Finish Installation.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                                      | <ul><li>O3DE successfully installs in the user defined location.</li><li>User is able to complete the FTUE.</li></ul>            |
| Install O3DE Linux Snap from local file system   | Linux Snap Installer                | <ol><li>Run the snap installer on the locaL O3DE snap package.<ul><li>`sudo snap install /path/to/o3de_<version>_amd64.snap --classic --dangerous`</li><li>The classic flag is required as O3DE is a classic confinement snap, the dangerous is required to install from a file locally.</li></ul><li>Run through the [Installer FTUE](#installer-ftue). </li></ol> | <ul><li>O3DE successfully installs in the snap installed packages location.</li><li>User is able to complete the FTUE.</li></ul> |
| Install O3DE Linux Snap from the Snapcraft Store | Linux Snap Installer                | <ol><li>Run the snap installer on the Snapcraft Store O3DE package.<ul><li>`sudo snap install o3de`</li></ul><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                                                                                                                      | <ul><li>O3DE successfully installs in the snap installed packages location.</li><li>User is able to complete the FTUE.</li></ul> |
| Repair O3DE Install                              | Windows Installer & Linux Installer | <ol><li>O3DE has already been installed on your machine.</li><li>Corrupt the installation by removing required files.</li><li>Run the Installer.</li><li>Use the Installer's **Repair** functionality.</li><li>Run through the [Installer FTUE](#installer-ftue). </li></ol>                                                                                        | <ul><li>O3DE successfully repairs the O3DE installation.</li><li>User is able to complete the FTUE.</li></ul>                    |
| Uninstall O3DE                                   | Windows Installer & Linux Installer | <ol><li>With O3DE already installed, launch the installer.</li><li>Select "Uninstall".</li></ol>                                                                                                                                                                                                                                                                    | <ul><li>The installer successfully uninstalls O3DE from the previously defined location on disk.</li></ul>                       |
---
