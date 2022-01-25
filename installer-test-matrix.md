25/1/2022 - This file contains numerous setup configurations that are recommended to run against each O3DE releases installers.

Current listed requirements: https://www.o3de.org/docs/welcome-guide/requirements/
----------------------------

### Windows

*   Microsoft Visual Studio 2019 version 16.9.2 through version 16.11.x are supported with O3DE.
    *   This includes manually adding desktop/game development with C++ workloads
*   Visual C++ Redistributable for Visual Studio 2019   
    
*   CMake 3.20.5 or later

### Linux

*   CMake 3.20.5 or later  
    
*   Clang 12
*   Vulkan Drivers (hardware specific)
*   libffi (an older version for Ubuntu, which doesn't currently support the most recent)
*   libglu1-mesa-dev
*   libxcb-xinerama0
*   libxcb-xinput0
*   libxcb-xinput-dev
*   libxcb-xfixes0-dev
*   libxcb-xkb-dev
*   libxkbcommon-dev
*   libxkbcommon-x11-dev
*   libfontconfig1-dev
*   libcurl4-openssl-dev
*   libsdl2-dev
*   zlib1g-dev
*   mesa-common-dev  
    

  

Requirement alternatives to test:
---------------------------------

### Windows

*   Visual Studio 2022

### Linux

*   Clang 13

  

Configurations:
---------------

Note: In all cases where a requirement is intentionally left out or incorrect, the goal is to test for clear error messaging related to the issue (in some cases these errors will happen at the OS level and it may not be in our power to update the error messaging, but these should be recorded for reference/possible Troubleshooting page)

### Windows

*   Prerequisites are met exactly as written
*   No CMake
*   No Visual Studio
*   No VS 2019 redistributable
*   No prerequisites met
*   Previous Installer already installed
*   Visual Studio 2022 used in place of 2019
*   Earlier version of CMake installed (earlier than 3.20.5)
*   Visual studio version other than 16.9.2 through version 16.11.x is used (while using 2019, 2022 will inherently be this)

### Linux

*   Prerequisites are met exactly as written
*   No CMake
*   No Clang
*   Most recent libffi version
*   None of the additional libs (these are generally installed all at once)
*   No prerequisites met
*   Clang 13 used in place of 12
*   Earlier version of CMake installed(earlier than 3.20.5)  

|     |     |     |     |
| --- | --- | --- | --- |
| OS  | Scenario | Workflow | Notes |
| Windows | Prerequisites are fully met | 1.  If not already installed, install all the prerequisites found at: [https://www.o3de.org/docs/welcome-guide/requirements/](https://www.o3de.org/docs/welcome-guide/requirements/)<br>2.  Install most recent release installer<br>3.  Launch O3DE.exe<br>4.  Create a project if one doesn't exist<br>5.  Build the project<br>6.  Launch the Editor |     |
| Windows | CMake isn't installed | 1.  If it is already installed, uninstall CMake<br>    1.  To uninstall:<br>2.  If not already installed, install the remaining prerequisites found at: [https://www.o3de.org/docs/welcome-guide/requirements/](https://www.o3de.org/docs/welcome-guide/requirements/)<br>3.  Install most recent release installer<br>4.  Launch O3DE.exe<br>5.  Create a project if one doesn't exist<br>6.  Build the project<br>7.  Launch the Editor |     |
| Windows | Visual Studio isn't installed | 1.  If it is already installed, uninstall Visual Studio<br>    1.  To uninstall:<br>2.  If not already installed, install the remaining prerequisites found at: [https://www.o3de.org/docs/welcome-guide/requirements/](https://www.o3de.org/docs/welcome-guide/requirements/)<br>3.  Install most recent release installer<br>4.  Launch O3DE.exe<br>5.  Create a project if one doesn't exist<br>6.  Build the project<br>7.  Launch the Editor |     |
| Windows | VS 2019 redistributable isn't installed | 1.  If it is already installed, uninstall VS2019 redistributable<br>    1.  To uninstall:<br>2.  If not already installed, install the remaining prerequisites found at: [https://www.o3de.org/docs/welcome-guide/requirements/](https://www.o3de.org/docs/welcome-guide/requirements/)<br>3.  Install most recent release installer<br>4.  Launch O3DE.exe<br>5.  Create a project if one doesn't exist<br>6.  Build the project<br>7.  Launch the Editor |     |
| Windows | None of the prerequisites are met | 1.  If they are already installed, uninstall all the prerequisites found at: [https://www.o3de.org/docs/welcome-guide/requirements/](https://www.o3de.org/docs/welcome-guide/requirements/)<br>    1.  To uninstall: See above for the individual items<br>2.  Install most recent release installer<br>3.  Launch O3DE.exe<br>4.  Create a project if one doesn't exist<br>5.  Build the project<br>6.  Launch the Editor |     |
| Windows Scenario 2 | Previous Installer already installed |     |     |
| Windows | Visual Studio 2022 used instead of 2019 | 1.  If it is already installed, uninstall Visual Studio 2019<br>    1.  To uninstall:<br>2.  Install Visual Studio 2022<br>3.  If not already installed, install the remaining prerequisites found at: [https://www.o3de.org/docs/welcome-guide/requirements/](https://www.o3de.org/docs/welcome-guide/requirements/)<br>4.  Install most recent release installer<br>5.  Launch O3DE.exe<br>6.  Create a project if one doesn't exist<br>7.  Build the project<br>8.  Launch the Editor |     |
| Windows | CMake version earlier than 3.20.5 is used | 1.  If it is already installed, uninstall CMake<br>    1.  To uninstall:<br>2.  Install a version earlier than 3.20.5<br>    1.  Can be found at [https://github.com/Kitware/CMake/releases](https://github.com/Kitware/CMake/releases)<br>3.  If not already installed, install the remaining prerequisites found at: [https://www.o3de.org/docs/welcome-guide/requirements/](https://www.o3de.org/docs/welcome-guide/requirements/)<br>4.  Install most recent release installer<br>5.  Launch O3DE.exe<br>6.  Create a project if one doesn't exist<br>7.  Build the project<br>8.  Launch the Editor |     |
| Windows | Unsupported Visual Studio version is used (something other than 16.9.2 - 16.11.x) | 1.  If it is already installed, uninstall Visual Studio<br>    1.  To uninstall:<br>2.  Install a version of Visual Studio that falls outside the supported version range<br>3.  If not already installed, install the remaining prerequisites found at: [https://www.o3de.org/docs/welcome-guide/requirements/](https://www.o3de.org/docs/welcome-guide/requirements/)<br>4.  Install most recent release installer<br>5.  Launch O3DE.exe<br>6.  Create a project if one doesn't exist<br>7.  Build the project<br>8.  Launch the Editor |  |
