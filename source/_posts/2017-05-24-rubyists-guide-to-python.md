---
layout: post
title: "Rubyist's guide to Python"
permalink: "rubyists-guide-to-python"
date: 2017-05-24 07:30:00 -0500
categories: web-development
---

Ruby was my first programming language and I love it for the simplicity and the syntactic sugar. Python is another popular high-level language used a lot for its data crunching libraries. I wanted to take some time to compare the difference between the two and point out some differences between Ruby and Python.

## Version Managment
Ruby has tools to manage the version of the Ruby language you use like `rvm` or `rbenv`. These tools allow you to
change the Ruby version you are using globally and by project. Additionally, they isolate your gem libraries by Ruby version
so that you don't need to worry about Ruby version compatibility based on the library version.

Python has a similar tool called [`pyenv`](https://github.com/pyenv/pyenv). According to the repo, `pyenv` was actually forked
from `rbenv` so we should expect similar behavior. The easiest way to install `pyenv` on a [Mac is to use Homebrew](https://github.com/pyenv/pyenv#homebrew-on-mac-os-x).
The alternative is to use their [installer](https://github.com/pyenv/pyenv-installer).

Once you have it installed, you can use the documentation for the [various commands](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md).
I ended up doing two things. The first was to update my `~/.bash_profile` to initialize `pyenv`. You can see the changes in my dotfiles [here](https://github.com/snags88/dotfiles/blob/6fea99e9a9c59169adbe02cf69b3f9610cde8b34/bash/path.bash#L20-L21). Then I just installing the most recent version of Python at this time (v3.6.1) and set it as my global version using

{% highlight shell %}
$ pyenv global
system
$ pyenv install 3.6.1
$ pyenv global 3.6.1
$ exec $SHELL
# Refresh your shell environment
$ python --version
Python 3.6.1
{% endhighlight %}

## Libraries
In Ruby, we have gems. Gems are just little (well, sometimes big) libraries that we can install in our projects to make development faster. You usually hit the RubyGems hosting service to download the libraries.

In Python, the library manager is called `pip` and it gets installed with Python if you used the instructions above. You can test it out by typing in `$ which pip` in the command line. From there, you can just install various libraries by using the `install` command like `pip install flask` (Flask is Python's version of Sinatra in Ruby). `pip` goes out to Python Package Index (PyPI) to download all the hosted libraries.

## Project Dependencies
For each project, it's best practice to manage its dependencies in a source file so that the dependent libraries can be reinstalled regardless of what environment you're in. In Ruby, there's a handy gem called [`bundler`](https://bundler.io/) that creates a `Gemfile` in your project. Whenever you want to recreate the environment, you can run `bundle` and all your dependencies will be installed. Magic!

I looked for something similar in Python and it seems like there's 2 things we need:

1. Create an environment by project so that installed libraries do not leak into other projects.
2. Create a source file that lists are dependencies. We can use this later to install dependencies in other environments.

The solution to the first item was to use `virtualenv`. It allows you to spin up virtual environments at will and isolate your work. I'm a huge fan of [this article](https://python-guide-pt-br.readthedocs.io/en/latest/dev/virtualenvs/). It does a nice job of explaining `virtualenv` at a high-level, but enough detail to get me going. You can simply install this using `$ pip install virtualenv`

I also used [`pyenv-virtualenv`](https://github.com/pyenv/pyenv-virtualenv) to help create virtual environments with `pyenv`. From a high-level this tool takes a version of Python you want to use and creates an `envs` directory where you can spin up new virtual environments. Then you can just use that environment as if you're specifying a version of Python you want to use. This tool needs to be installed via Homebrew `$ brew install pyenv-virtualenv`. You'll also need to initialize this tool in your [`.bash_profile`](https://github.com/snags88/dotfiles/blob/ec1ea56dada60210a65f820916b30302481f51bd/bash/path.bash#L22).

Now to bring actually create a virtual environment for our project, we need to run the following (assuming you're already in the directory of your project):

{% highlight shell %}
$ pyenv virtual 3.6.1 my_project
# Creates a virtual environment for us. Check it out in ~/.pyenv/versions/
$ pyenv local my_project
# Sets the local directory's Python version to be the virtual env we just created inside .python-version
{% endhighlight %}

The solution to the second item (the source file) is that `pip` uses a `requirements.txt` file in the project to determine the dependencies. Seems like there's a `$ pip freeze` command that we just need to output into the file. When we need to install the dependencies, we just run an install command:

{% highlight shell %}
$ pip freeze > requirements.txt
# Output dependencies into file

$ pip install -r requirements.txt
# Install all dependencies
{% endhighlight %}

It would be nice for the requirements file to be updated automatically like `bundler`, but after some Googling around, I couldn't find any tools or libraries to help with that. I wonder why it doesn't get auto-generated after each install...

## Interactive Interpreter
One of the ways I like to get familar with or test out behavior of a programming language is through a interactive interpretor. In Ruby, there are a few options like `irb` and [`pry`](http://pryrepl.org/).

In Python, you can just run the `$ python` command to enter an interactive session. I haven't found any alternatives for a Python REPL but I'm sure there are alternatives to what comes out of the box.

## Syntax
This is a pretty large topic so I'll try to start small from working with more primative types and go into creating classes, etc.

### Numbers
Most things are pretty much the same as Ruby except for 2 things I noticed:

1. In Python, division always returns a float value even if it's perfectly divisible. Ruby always rounds down to the nearest integer.
2. In Python, there is a "floor division" operator `//` that will round the division down to the nearest integer, similar to how Ruby's regular division works.

### Strings
Strings operate pretty similar to Ruby as well. Concatenation is done with `+`. 2 major differences I think are important to note are string interpolation and printing strings:

- Starting with v3.6, Python allows string interpolation using "f-strings". [This Stack Overflow answer](https://stackoverflow.com/a/4450610) does a nice job of summarizing the various ways of string interpolation.
  {% highlight python %}
  interpolate_me = "I'm interpolated!"
  f"Interpolate a variable: {interpolate_me}"
  {% endhighlight %}
- In Python, printing things out uses the `print()` function. In Ruby, `puts` is used.

### Absence of a value
Ruby uses `nil` to represent the absence of a value. Python uses a singlton called `None`.

### Truthy & Falsy values
In Ruby, all values are truthy except for `false` and `nil`. In Python, there is a larger list of items that are falsy like `0`, any empty list, empty string, and `None`. There are more values in this list and you can check the entire list [here](https://stackoverflow.com/a/39984051).

### Arrays
Arrays pretty much work the same between the two languages, but the method names are a little different (i.e. `push` in Ruby is `append` in Python).

The only thing I noticed was that in Python, you cannot assign an element at an array's index if nothing exists at that index.

{% highlight python %}
>>> arr = []
>>> arr[0] = 1
ERROR
>>> arr.append(1)
>>> arr
[1]
>>> arr[0] = 2
>>> arr
[2]
{% endhighlight %}

### Hash dictionary
In Ruby, key-value pairs stored in a data structure (`{}`) are called hashes. In Python, they are called dictionaries. They seem to work the same. The only thing to be careful for in Python is that the hashes `{}` are also used to describe "sets" which are like arrays, but with no duplicate values.

### Variables
Variable assignments are the exact same in Ruby. No special keyword to declare it, but you need to declare a variable before you use it. Otherwise you get an exception.

### Defining functions/methods
In Ruby, methods are defined by using the `def` and `end` keyword:

{% highlight ruby %}
def some_method(argument)
  argument #=> implicit return of argument
end
{% endhighlight %}

In Python, functions are defined similarly, except it's more dependent on indentation and requires an explicit return value:

{% highlight python %}
def some_method(argument)
  return argument #=> explicit return of argument

{% endhighlight %}

The other things to note is that Ruby uses the colon `:` for keyword arguments, whereas Python uses the equal `=`.

### Scoping Rules
In my experience, Ruby's scoping rules follow a narrow set of rules and are pretty understandable. I learned about how to determine the scope by using the scope gate rules: any time a method, module, or class is defined a new scope is created. Also, there is a global scope that's at the top level.

For example the following code snippet makes sense:

{% highlight ruby %}
a = 'global variable'

def test_method
  a = 'local variable'
  puts a #=> prints 'local variable' due to shadowing. imagine it as a new "a" variable inside of this scope

  [1,2,3].each do |num|
    a = num
  end

  puts a #=> prints '3' because we reassigned the variable in the loop above
end

puts a #=> prints global variable from global scope
test_method
puts a #=> prints global variable because variable was modified inside of another scope
{% endhighlight %}

As I've been reading about Python's scoping rules, it works similarly, but Python really emphasizes the concept of "Namespaces" that maintains the various namespaces and variables within the namespaces using a dictionary (or a ruby hash). As variables are used, Python searches the appropriate namespaces in the order of **Local**, **Enclosed**, **Global**, and **Built-in** (commonly refered to as LEGB). If nothing is found in any of those namespaces, then a _NameError_ is raised. So at the end of the day, the below snippet written in Python acts the same as the one above in Ruby:

{% highlight python %}
a = 'global variable'

def test_method ():
    a = 'local variable'
    print(a) #=> prints 'local variable' due to shadowing. imagine it as a new "a" variable inside of this scope

    for num in [1,2,3]:
        a = num

    print(a) #=> prints '3' because we reassigned the variable in the loop above

print(a) #=> prints global variable from global scope
test_method()
print(a) #=> prints global variable because variable was modified inside of another scope
{% endhighlight %}

The more interesting part for Python is that there are keywords you can use to actually reference the variables declared in the global or enclosing functions. I'm actually not sure how useful this is due to potential confusion and modifying out of scope variables unintentionally, which could lead to unwanted side effects. In any case, you can do this by using `global` or `nonlocal`:

{% highlight python %}
a = 'global variable'

def test_method ():
    global a #=> Tells python to us the globally decalred "a" variable
    a = 'local variable'
    print(a) #=> prints 'local variable' due to reassignment of the variable

    for num in [1,2,3]:
        a = num

    print(a) #=> prints '3' because we reassigned the variable in the loop above

print(a) #=> prints global variable from global scope
test_method()
print(a) #=> prints 3 because the global "a" variable was used inside of "test_method"
{% endhighlight %}

### Function objects
In Ruby, methods can't really be passed around. They need to be wrapped in a Proc or Lamda object. In Python, functions need to be invoked, so if you just have a reference to a function or a method, you can actually pass that around or set them to other variables for access at a later point in time.

### Classes & Objects
I'm not going to get too much into the intricates of Ruby's classes here because this is assuming that we know Ruby pretty well. For Python, there are some similarities, but much more noticeable differences compared to some of the other things we looked into already.

- You can declare attributes in the body of the class like variables. You can reassign the attributes after class definition and object instantiation:
  {% highlight python %}
  class MyClass:
    i = 123

  c = MyClass #=> Class def
  print(c.i) #=> 123
  c.i = 789
  print(c.i) #=> 789
  o = MyClass() #=> Instance of class
  print(o.i) #=> 789
  o.i = 456
  print(o.i) #=> 456
  {% endhighlight %}
- The attributes declared in the class body are class variables. Attributes assigned to `self` are instance variables:
  {% highlight python %}
  class MyClass:
    class_variable = 'hello from class variable!'

    def __init__(self, name):
      self.instance_variable = name

  o1 = MyClass('first object!')
  o2 = MyClass('second object!')
  print(o1.class_variable) #=> 'hello from class variable!'
  print(o2.class_variable) #=> 'hello from class variable!'
  print(o1.instance_variable) #=> 'first object!'
  print(o2.instance_variable) #=> 'second object!'
  {% endhighlight %}
- The lifecycle hook for object instantiation is `def __init__(self)`
  {% highlight python %}
  class MyClass:
    def __init__(self, arg1):
      self.first_arg = arg1

  o = MyClass('first argument!')
  print(o.first_arg) #=> 'first argument!'
  {% endhighlight %}
- The first argument in a method definition is the `self` keyword (or `cls` for class methods). It can be used to get a reference to the current object or class in the method definition
  {% highlight python %}
  class MyClass:
    def __init__(self):
      self.set_b()

    def set_b(self):
      self.b = 10

  o = MyClass()
  print(x.b) #=> 10
  {% endhighlight %}
- Python lets you do multiple inheritance. The ways method calls are looked up is from left to right in the inheritance declaration and depth first. However, Python will make sure that each base class is only looked up once:
  {% highlight python %}
  class MyClass(BaseClass1, BaseClass2):
    # Define class here with `super()` and stuff
  {% endhighlight %}
- Python doesn't enforce private variables or methods. It's up to the developer to denote the private methods by using a single or double underscore before the method name. When this is done, it will actually namespace those attributes or methods with the name of the class. This is called _name mangling_:
  {% highlight python %}
  class MyClass:
    def public_method(self):
      print('public method')

    def __private_method(self):
      print('private method')

  o = MyClass()
  o.public_method #=> 'public method'
  o. __private_method #=> Throws exception
  o._MyClass__private_method #=> 'private method'
  {% endhighlight %}

## Modules
For code organization, it's nice to keep similar code like a class definition or utility functions in a single file. Ruby uses the keyword `require` to import the code into another file. Rails does a lot of magic behind the scenes to make things available to you everywhere in your code source.

For Python, a similar concept applies, but each file has its own namespace and we use the `import` keyword to get code from one file into another. The filename and the module name should match and there's a set of rules for how it finds the files in detail [here](https://docs.python.org/3/tutorial/modules.html#the-module-search-path). After you've imported a module, you can access its namespaced attributes by using dot notation:

{% highlight python %}
# module.py
def printer(string):
    print(string)

def multiplier(a, b):
    print(a * b)

class MyClass:
    def __init__(self, name):
        self.name = name

    def say_hello(self):
        print('Hello, my name is ' + self.name +'!')

# import.py
import module

module.printer('hello') #=> prints 'hello'
module.multiplier(4, 2) #=> prints '8'

o = module.MyClass('Seiji')
o.say_hello() #=> prints 'Hello, my name is Seiji!'
{% endhighlight %}

Also important to note that you can import specific names from your module by doing `from module_name import name1, name2` or just import everything into the current namespace by doing `from module_name import *`.

Extending this idea of module imports, you can imagine this is how we can share behavior across different classes and mix them into our class definition:

{% highlight python %}
import module

class MyOtherClass:
  printer = module.printer
  multiplier = module.multiplier
{% endhighlight %}

## Conclusion
There are actually more similarities between Python and Ruby than I imagined. Of course there are syntactic differences, but they both are easy to use. Python tends to lean on the side of more explicitness (which is not a bad thing), where as Ruby has some niceness in terms of the developer experience. At the end of the day, the choice of language depends on the problem you're trying to solve. My next step is to start playing with Django, the Rails equivalent web framework for the Python language.
