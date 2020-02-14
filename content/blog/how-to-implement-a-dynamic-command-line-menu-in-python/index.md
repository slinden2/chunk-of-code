---
title: "How to implement a dynamic command line menu in Python?"
published: true
date: "20190319"
tags: ["CLI", "how-to", "menu", "tutorial"]
---

In this post I will show you how to implement a dynamic command line menu in Python. In most tutorials and posts that I have seen online the usual way to implement a command line menu is a while loop with a bunch of if-elif statements. This implementation is okay for small menu's, but it can quickly get very tedious if you need to modify your menu afterwards. In this implementation it is not possible to create dynamic menu items, meaning that we need to hard code the menu structure in a separate function.

First I will show you the usual implementation so that the advantages of a dynamic controller become more obvious. For the sake of this example I won't worry about error handling or other things in order to keep the code as concise as possible. After the first example, I will also show what is the better way to do the menu dynamically.

## if/elif menu

The first thing we need is a function that prints out the menu, so that the user knows which commands are available.

```py
def print_menu():

    print((================================\n
           MENU\n
               ================================\n
               1 - First menu item\n
               2 - Second menu item\n
               3 - Third menu item\n
               5 - Fourth menu item\n
               6 - Exit\n
               ================================\n
           Enter a choice and press enter:), end= )
```

I prefer this kind of multiline syntax, because it enables you to write clean and aligned rows and it is surely more beautiful than the following version:

```py
def print_menu():

    print(================================
MENU
    ================================
    1 - First menu item
    2 - Second menu item
    3 - Third menu item
    5 - Fourth menu item
    6 - Exit
    ================================
Enter a choice and press enter:)
```

My preferred version needs more typing, but at least that way you don't have some strange unaligned string rows on the left. Then we create the actual menu functionality.

```py
def menu():

    print_menu()
    user_input = 0

    while user_input != 6:

        user_input = int(input())

        if user_input == 1:
            print(Executing menu item 1)

        elif user_input == 2:
            print(Executing menu item 2)

        elif user_input == 3:
            print(Executing menu item 3)

        elif user_input == 4:
            print(Executing menu item 4)

        elif user_input == 5:
            print(Executing menu item 5)

        elif user_input == 6:
            print(Exiting...)
```

First we call the previously created `print_menu()` function and create a `user_input` variable to store the inputs typed in by the user. Then we enter the `while` loop and we run the while loop over and over again until the user types in the number 6, which exits the program.

Now we have our `print_menu()` function and the basic functionality in place. The next step is to run the program.

```py
if __name__ == __main__:
    menu()
```

We could start developing the progam further by elaborating the `if` statements. We could create additional functions or modules to be called after a specific key press. We could add more `elif` statements to run more function. Of course, every time we add a new `elif` statement, we need to modify also our `print_menu()` function. This is where the dynamic menu controller starts making sense. Lets see next how it can be implemented.

## Dynamic controller menu

We start by creating a new `class` for the controller with some methods in it. Let's call the class `Controller` and the methods `do_X`.

```py
class Controller:

    @staticmethod
    def do_1():
        print(Doing 1)

    @staticmethod
    def do_2():
        print(Doing 2)

    @staticmethod
    def do_3():
        print(Doing 3)

    @staticmethod
    def do_4():
        print(Doing 4)

    @staticmethod
    def do_5():
        print(Doing 5)

    @staticmethod
    def do_6():
        print(Exiting...)
```

This is the basic structure of the class. We simulate the `if/elif` statements of the previous example with separate methods that are called `do_X` methods. The do-methods in this example must be named with a **do** prefix. You will understand why later. So these are the methods that the user will be able to run from the command line. In this example we do not need to create an instance of the `Controller` class and that is why I use static methods here.

But how can we run all these methods? How do we bind the user input to a specific do-method?

The answer is rather simple once you've learned it once. We will need a separate helper method that takes the user input as an argument.

```py
    @staticmethod
    def execute(user_input):
        controller_name = fdo_{user_input}
        try:
            controller = getattr(Controller, controller_name)
        except AttributeError:
            print(Method not found)
        else:
            controller()
```

In the snippet you can see the helper method. I still need to show you where to call this method from, but let's go through the `execute()` method first.

First we create a string and store it to `controller_name`. The stored string will be formatted like `do_{user_input}`. Ideally the user input is a number that correnponds to one of our do-methods. If it doesn't, an `AttibureError` will be raised.

Then we create a new variable, `controller`. We use the `getattr()` function to look for the requested do-method from within the `Controller` class. The first argument of the `getattr()` function is the class and the second one is the method name to be searched in a string format.

If no error is raised, we run the `controller` method created by the `getattr()` function. Now we need something to tie all this together. In the `if/elif` example we used a `while` loop, and so we do also in this one. Let's create the `run()` method.

```py
    @staticmethod
    def run():
        user_input = 0
        while(user_input != 6):
            user_input = int(input())
            Controller.execute(user_input)
        print(Program stopped.)
```

Perfect. Now we've got ourselves a working program. While the user inserts something that is not 6, the while loop will keep calling the `execute()` method with the wanted user input. If the user gives 6 as an input. The program stops and prints out "Program stopped." notification.

Note that in this example we suppose that the exit method is the `do_6` method. It may be necessary to write some code to find the _last_ do-method from the class and use it as the exit method, so that you can add more do-methods without having to modify the `run()` method.

But there is no menu. We will need a `generate_menu()` method. But before writing the `generate_menu()` method. Add a docstring to all your do-methods. The docstring will be used for generating the menu dynamically.

Modify your do-methods as you please. I leave a one-method example here just for clarification.

```py
    @staticmethod
    def do_1():
        First menu item
        print(Doing 1)
```

Then create the `generate_menu()` method.

```py
    @staticmethod
    def generate_menu():
        print(================================)
        do_methods = [m for m in dir(Controller) if m.startswith('do_')]
        menu_string = "\n".join(
            [f{method[-1]}.  {getattr(Controller, method).__doc__}
             for method in do_methods])
        print(menu_string)
        print(================================)
        print(Insert a number:, end= )
```

So what is happening there. After the first line of equals signs we create a list of all methods (`do_methods`) that start with `do_`. These are the famous do-methods that will be used as our menu items.

After that we create a single string that contains the whole menu by using list comprehension and `join` method. This part is a bit hard to read so I will go through the list comprehension to make it more clear.

- `method[-1]` is the number in the do-method name. For example, do\_**1**.
- `getattr(Controller, method).__doc__` fetches the docstring of the method.

These steps are repeated for all the methods in the `do_methods` list. As a result we get a string that in this case would be _"1. First menu item"_. You may notice that done like this the menu work only when there are less than 10 do-methods, as `method[-1]` slices only the last character of the method name. This could be easily fixed, for example, by using a simple regular expression. Or maybe in some cases you could just grab everything that comes after the underscore.

Come to this point, we are ready to run our working controller.

```py
def main():
    Controller.run()


if __name__ == __main__:
    main()
```

## Conclusion

I showed you two ways to create menu structure for a command line application.

The first way was the simple way by using `if/elif` statements. This menu structure is fine for small menus that are not going to be modified afterwards. It is really easy to understand even for a beginner programmer. The downside of it is that if some revisions are needed, the modifying process can get tedious and error prone.

The second way was a dynamic controller. This is the way that I recommend for larger menus. It is a bit harder to understand, but it is important to get good grasp of it if you are planning on designing bigger applications. The advantage is that by adding more do-methods, the menu updates dynamically without having to revise it manually. This means that the menu is always up-to-date and the methods are accessible.

With the dynamic menu you can easily use other method ID's than numbers. Nothing prevents you from using something like _do_jump_ or _do_shoot_ as your method name. You just need to adapt the `generate_menu()` to your method names.

That's all for now. :)
