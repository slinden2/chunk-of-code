---
title: "Converting .py to .exe"
published: true
date: "20190402"
tags: ["py-to-exe", "pyinstaller"]
---

In this short tutorial we are going to convert a Python program into a self-sufficient, stand-alone .exe file that can be run on a computer that has no Python interpreter installed on it. For the sake of this example I am going to use my dir-xray program that creates xrays of the file system at certain moments. You can use any of your own programs, but if you don't have one or you just for whatever reason want to use mine, please download it freely from my [GitHub repo](https://github.com/slinden2/dir-xray).

## Installation and conversion

Converting a Python program to `.exe` file can be done with `PyInstaller` module. You can install it by running `$ pip install pyinstaller` in the command prompt. At the time of writing this blog post the PyInstaller module is at its 3.4 version.

In this blog post I will just quickly show you how to do the conversion for a simple and small program to get you started, but if you want to study the subject more thoroughly, I suggest you to check the [module documentation](https://pyinstaller.readthedocs.io/en/stable/).

Now you should have the module installed. On the command prompt, navigate to the directory where your application to be converted resides. Then type in

`$ pyinstaller -F --distpath C:\Users\_Username_\Desktop\Dir-Xray dirxray.py`

and press enter. This command creates a new folder on your desktop called Dir-Xray and creates in the folder a single `.exe` file containing `dirxray.py` and all its dependencies. Now you are good to go. Double-click the .exe and the program runs. At the first run the program creates its own configuration json file in the same folder.

Now you could move the .exe file to a different computer and it should run like it does on your PC where you have the Python interpreter installed.

## PyInstaller commands

Let's take a closer look at the flags that we used with the `pyinstaller` command. I will list some other flags as well that I think that can result useful for you.

| Flag              | Description                                                                                                               |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------- |
| \-\-distpath PATH | This is the output directory where the .exe file will be created.                                                         |
| -F                | Create a single-exe file instead of a bundle folder.                                                                      |
| -i PATH           | Path to your icon file. The file must be in .ico format. The icon will be shown as the icon of the .exe file              |
| \-\-noconsole     | Remove console. If your application has a GUI, you probably don't want the console to open when you launch the .exe file. |
| -help             | If want to take a look at some additional flags, you can use this flag.                                                   |

## Summary

Creating an .exe file with PyInstaller is easy and straight-forward. I haven't tried this for a more complex application with more dependencies or GUI, but after a short research the process should be the same. You should always run the `pyinstaller` command to the main .py file of your program. The module itself knows how to handle all the dependencies.

I wrote this article mostly as a note for myself. If I ever need to convert a .py file to a format that can be run on any Windows computer surely may be useful in the future.
