---
title: "Pathlib - An alternative to the os.path module"
published: true
date: "20190327"
tags: ["os", "pathlib", "python"]
---

# Introduction

Pathlib is a Python library used for creating and manipulating path objects. Working with paths with `os.path` module can be clunky sometimes and `pathlib` is something that is going to help you with that allowing you to handle paths in an object-oriented way.

`pathlib` module contains classes to handle filesystem paths using either MS Windows or POSIX standard syntax. The classes can be divided in two categories: _Pure_ paths and _concrete_ paths. Pure paths provide path-handling for strings but do not access the actual filesystem. They are a representation of a path without messing with the filesystem. To put it simple, if you need to create a path from a string, but you don't want to refer to an actual, existing path, you could use `PurePath` to create it.

Concrete paths extend the API and they access the filesystem and can be used, for example, to modify file permissions. So they always refer to an actual file or folder that resides in the memory.

# Pure Paths

`PurePath` is the base class for all classes in the `pathlib` module. It is also used as a base for the `Path` subclass which is surely the class that is the most useful one for a regular programmer.

Instantiating `pathlib.PurePath` returns `PurePosixPath` or `PureWindowsPath` depening on the operating system on which the code in run. This naturally facilitates writing cross-platform code, because the class will take care of the correct syntax to use.

Pure paths may be useful also if you need to manipulate Unix paths on a Windows system and vice versa. A path can be created from segments of strings or `os.PathLike` objects. Let's take a look at the example below.

```
from pathlib import PurePath

path = PurePath(first, second, third)
print(repr(path))
```

The code prints out `PureWindowsPath('first/second/third')`. If you run the code on Linux or MacOS, the `repr` should be `PurePosixPath('first/second/third')`.

The syntax for extending an existing path is not obvious, but once you get used to it, it will prove itself very readable. The division operator `/` is used for the extension.

```
from pathlib import PurePath

path = PurePath(first, second, third)
print(repr(path))

extended_path = path / fourth
print(repr(extended_path))
```

The code above prints out `PureWindowsPath('first/second/third/fourth')`.

You can also use the double-dot operator (ie. refer to parent folder) with it. The single-dot operator, which refers to the current folder, will get automatically trimmed.

```
from pathlib import PurePath

path = PurePath(first, second, third)
print(repr(path))

extended_path = path / fourth
print(repr(extended_path))

fifth_path = path / .. / fifth_not_fourth
print(repr(fifth_path))

fifth_path = path / .. / fifth_not_fourth / .
print(repr(fifth_path))
```

Both `fifth_path` reprs print out `PureWindowsPath('first/second/third/../fifth_not_fourth')`.

# Concrete paths

The concrete paths just like the pure paths are cross-platform compatible as they are subclassed form the same base class. `Path` is the class that you are most probably going to be using.

The usage of the Path class is pretty straight-forward. Let's delve into an example right away to make the concept of `Path` brighter.

```
from pathlib import Path

# Current folder
current_path = Path.cwd()
print(repr(current_path))

# Does the path exist?
isPath = current_path.exists()
print(isPath)

# Is the path a directory?
is_directory = current_path.is_dir()
print(is_directory)

# Is the path a file?
is_file = current_path.is_file()
print(is_file)

# Create a dir in the current path
new_dir_path = current_path / new_dir
try:
    Path.mkdir(new_dir_path)
except FileExistsError:
    print(Path already exists)

# Rename a path
modified_dir_path = current_path / newer_dir
try:
    new_dir_path.rename(modified_dir_path)
except FileExistsError:
    print(Path already exists)

# Remove a path
Path.rmdir(Path.cwd() / newer_dir)


# Users home folder
home_path = Path.home()
print(repr(home_path))
```

The methods that you are going to use the most are probably in the example above. They are the basic operations for manipulating paths.

Some more interesting methods are `Path.iterdir()` and `Path.glob()`.

`Path.iterdir()` creates a generator that yields path objects of the directory contents. It doesn't _walk_ in the directories that it yields, but you can immagine that with `iterdir()` and `is_dir()` you could quite easily create your own walk function.

Also `Path.glob()` create a generator that yield all files that match to your search term. You could use it to find all python files in a path like this:

```
current_path = Path.cwd()
for py_file in current_path.glob(*.py):
    print(py_file)
```
