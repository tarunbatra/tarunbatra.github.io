---
title: PyPi packages decoded for npm developers
date: 2020-08-31 20:00:00
tags:
  - python
  - npm
  - pip
category:
  - python
photos: /data/images/PyPi-packages-decoded-for-npm-developers/cover.jpg
description: Step by step guide to develop Python packages and comparisions with npm ecosystem. I deployed my package on PyPi and here's what I've learned.
---


If you are a developer in Javascript / Node.js ecosystem, you have used `npm`. It's the world's largest software registry and the ecosystem around it is advanced and mature. I realized it when I tried to port `password-validator`, an open source npm package I maintain, to Python. I spent weeks to figure out things which were very intuitive to me in the Javascript world. This blog will list down some of the key differences I found between the two ecosystems for anybody else who treads the same path. By the end of this post, we should be able to install our package using:
```sh
pip install <my-awesome-package>
```

## What is a package?
In `npm`, any folder which has a `package.json` file is a package. In Python world, we need a file called `setup.py` in the root folder to make our project a package for [PyPi][pypi-link], a common repository for Python packages. This file is used to declare meta data like name, version, author, and even dependencies (with a catch). Look at this [sample setup.py file][sample-setup-py-link] to get an idea.

Normally, the actual package code goes inside a directory with the package name and it is required to have a `__init__.py` file in it to indicate that it is a module. So the directory structure would look more or less like:

<pre>
package-project
├── setup.py
└── my-package
    ├── __init__.py
    └── lib.py
</pre>

In this structure your actual package code will go in `__init__.py` and/or any other python file in this directory like `lib.py`.

## Dependency management
Dependency management is one area where I found Python ecosystem lacking and primordial. `npm` has concepts like `dependencies` and `devDependencies` and `package-lock` which make dependency management easier. The manifest file `setup.py` has a section called `install-requires` which is a list of the dependencies required to run the project. These dependencies are downloaded while installing the package using `pip`. It's not very widely used to define the exact dependencies and dependencies of dependencies.

Instead, a combination of a virtual environment and requirements file is used.

### Virtual environment
`npm` allows installing packages globally or local to a project. In `pip`, all packages are installed globally.  To separate the local dependencies from system-wide packages, a virtual environment is created which contains all the dependencies local to the project and even a local Python distribution. Code running in a virtual environment doesn't require any system dependency, so one can be assured that the code should run in all systems wherever the local virtual environment is set up correctly.

There are multiple ways of setting up a virtual environment, like `venv` and `virtualenv`. I found former to be the easiest to use.

```sh
python3 -m venv <dir>
```

The above command creates a virtual environment in directory `<dir>` which means this directory will house everything required to run the program including Python and pip distributions. This directory needs not be committed to version control. To use the virtual environment, it has to be "activated".

```sh
source <dir>/bin/activate
```

### Requirements file
Every virtual environment has its own pip and Python distribution. So it's a clean slate when you start. When we install dependencies, we can make a log of them using:
```sh
pip freeze > requirements.txt
```
This will log all the packages and their versions installed by pip in the file `requirements.txt`. The file name can be anything but it's a convention in the Python community to use this name. There is no concept of devDependencies in Python but it's not uncommon for some projects to have a `requirements-dev.txt` to log devDependencies of the project. Here's a [sample requirements file][sample-requirements-file-link].


Installing a project's dependencies now can be done with:
```sh
pip install -r requirements.txt
```

## Documentation
`JSDocs` is the most common way to document code in JavaScript and the list of documentation tools alomst stops there. Documentation in Python is taken very seriously.

### Sphinx
[`Sphinx`][sphinx-link] is a documentation generator for Python which can be used in various ways thanks to a variety of extensions available. To get started with sphinx, we need a `conf.py` file which will have configuration for sphinx. The file can be generated using `sphinx-quickstart` CLI utility. Here's a [sample conf.py file][sample-conf-file-link] to get started.

Sphinx and Python ecosystem prefers using `reStructuredText` (reST) for documentation instead of Markdown which is common in the JavaScript world. M↓ can be used with Sphinx too, but it's a bit of work and reST is very similar to M↓. Here's a sample [README in reST][rest-readme-link].

## Docstrings
Sphinx can look through special comments in Python code and include them in the documentation. These comments which are called `Docstrings` are used to describe functions, classes etc. and are very similar to JSDocs in JavaScript and JavaDocs in Java. Here's a method documented with Docstrings:

```py
def add(a, b):
  ''' Performs addition of 2 numbers

  Example:
    >>> res = add(1, 2)
    3
  Returns:
    int: Result of addition
  '''
  return a + b
```

There are a number of formats for writing Docstrings, which can be a topic discussion in itself. I found myself going back to [this guide by RealPython][docstrings-link] and I recommend giving it a read for a deeper understanding of Docstrings. Using the combination of reSt files and Docstrings, we can generate documentation using Sphinx in a form like PDF and HTML. Here's a [sample HTML documentation][sample-doc-link] generated using Sphinx.

There are less popular alternatives to Sphinx which can also be used for documentation, most notable being [ReadTheDocs][read-the-docs-link].

## Tests
I found writing unit tests in Python easier than in JavaScript due to the availability of mature and built-in tools.

### unittest
Python has a built-in module [`unittest`][unittest-link] which can be used to run the tests. The syntax is very similar to Jest or Mocha. Every "test" is a class and every class method is a test case. Methods like `setUp()` and `teardown()` have special meanings and are equivalent to `before` and `after` methods seen in JavaScript testing frameworks. Here's a [sample test file][sample-test-link]. Tests are run by:
```sh
python3 -m unittest discover -s <test_directory>
```
[Pytest][pytest-link] and [nose2][nose2-link] are some of the alternatives but I found `unittest` adequate.

### Doctests
Earlier in Documentation section, we discussed about Docstrings. The Examples section in a Docstring provide the user an understanding of how to use the function. What's great is we can also run the example against an automated unit test, using [doctest][doctest-link] (another built-in module). This can keep the documentation and implementation in sync.

## Publishing

Packages can be publushed to PyPi after building. Earlier the builds were made in Egg format, however now it is being replaced by Wheel format. To create a wheel build, we need to install `setuptools` and `wheel`.
```sh
python3 -m pip install --user --upgrade setuptools wheel
```
Now the build can be created by running:
```sh
python3 setup.py sdist bdist_wheel
```
This will create two files in `dist` folder:

- One with `.tar.gz` extension which is a source archive.
- Another with `.whl` extension which is the build file ready to be installed.

Now we need to upload the package build to PyPi. We need `twine` for that.
```sh
python3 -m pip install --user --upgrade twine
```
Final step, the uploading of the build:
```
python3 -m twine upload --repository testpypi dist/*
```
`--repository` here lets you select a specific package repository like custom or private npm registeries. It can be configured with a [`.pypirc`][pypirc-link] file.

## Additional resources
Two resources which helped me a lot in fuguring out my issues were:
1. The official [Python Packaging User Guide][python-packagin-guide-link]. I recommend going through it once.
2. [The Real Python][real-python-link]. I recommend clicking on any link from this website which comes up in search results.


It was an interesting experience for me to tackle all the challenges faced when I was on a journey to publish my first package on PyPi. A mature testing frameworks was a breath of fresh air while dependency management and documentation generation was a daunting task due to all the different tools to manage. In the end I published [password_validator][password-validator-python-link] and I am happy with the result.

Do reach out in comments or on my [Twitter][twitter-link] if you have any issues with the steps I have mentioned here. Happy publishing!

[pypi-link]: https://pypi.org/
[sample-setup-py-link]: https://github.com/pypa/sampleproject/blob/master/setup.py
[sample-requirements-file-link]: https://github.com/tarunbatra/password-validator-python/blob/master/requirements-dev.txt
[sphinx-link]: https://www.sphinx-doc.org/en/master/
[sample-conf-file-link]: https://github.com/tarunbatra/password-validator-python/blob/master/docs/source/conf.py
[rest-readme-link]: https://raw.githubusercontent.com/tarunbatra/password-validator-python/master/README.rst
[docstrings-link]:https://realpython.com/documenting-python-code/#documenting-your-python-code-base-using-docstrings
[sample-doc-link]: https://tarunbatra.com/password-validator-python/api_reference.html
[read-the-docs-link]: https://readthedocs.org/
[unittest-link]: https://docs.python.org/3/library/unittest.html
[sample-test-link]: https://github.com/tarunbatra/password-validator-python/blob/master/tests/test_password_validator.py
[pytest-link]: https://docs.pytest.org/en/stable/
[nose2-link]: https://docs.nose2.io/en/latest/
[doctest-link]: https://docs.python.org/3/library/doctest.html
[pypirc-link]: https://packaging.python.org/specifications/pypirc/
[python-packagin-guide-link]: https://packaging.python.org/
[real-python-link]: https://realpython.com/
[password-validator-python-link]: https://github.com/tarunbatra/password-validator-python/
[twitter-link]: https://twitter.com/tarunbatra/