---
description: ' Best practices for how and when to create a Python package. '
title: Python Packages Best Practices
---

This page contains steps on when and how to create a Python package.

## When Can a Package be Made?

A Python package can be created when it meets the following conditions:

1. It contains multiple reusable modules.
1. Multiple tests running in the Open 3D Engine pipeline depend on the package.
1. The tests or modules cannot be moved to relatively import the package.

Consider adding a module to an already existing package instead of creating a new one.

## How to Make a Package

Follow the official Python [documentation](https://packaging.python.org/tutorials/packaging-projects/) to create the test package. Existing packages such as `Tools/LyTestTools` can be referenced as examples.

### Adding the Package to Bundled Python

Set up automatic installation of the newly created package during CMake configuration by editing `cmake/LYPython.cmake`. This adds a minor cost to installation time. Use the `ly_pip_install_local_package_editable()` function to install your new package.

```cmake
ly_pip_install_local_package_editable(${LY_ROOT_FOLDER}/<path-to-package> <package-name>)
```

## Risks of Path Hacks

A workaround for importing modules is to hard code relative paths via `sys.path.append()`. We highly discourage use of this method because the solution is not scalable when any involved files move.

```python
import sys

# Not recommended
sys.path.append('../relative-path')
import my_module
```
