---
layout: post
title: "Python venv: Virtual Environments"
author: skaczmarek
date: 2024-11-08
categories: [programming]
tags: [python]
---

Python virtual environments, also known as `venv`, are a way to isolate different Python environments. This is especially useful during development because you can install different packages with PIP for each project. This eliminates the risk of version conflicts that arise, for example, when one project requires a specific version of a package, while another project needs a newer version of the same package. Previously, you would have had to solve this issue by creating a separate virtual machine or container for each project, which is time-consuming and resource-intensive. Here’s how to set up `venv` easily and quickly.

# Creating a venv Environment

A `venv` environment is essentially a folder that you point Python to as the location for installing packages. You can create this folder anywhere you want, such as within the project directory you’re working on. However, remember to add this folder to .gitignore if you’re using version control. Personally, I prefer to keep all environments in a single directory.

The following example applies to Python 3.4 and higher. Prepare a folder for the development environment of your project:

```bash
py@e73a39cc4acc:~$ mkdir -p .venvs/pocket-tools

py@e73a39cc4acc:~$ ls -al .venvs/
total 16
drwxrwxr-x 4 py py 4096 Nov  8 18:58 .
drwxr-x--- 4 py py 4096 Nov  8 18:58 ..
drwxrwxr-x 2 py py 4096 Nov  8 18:58 pocket-tools
```

Now create the environment itself:

```bash
py@e73a39cc4acc:~$ python3 -m venv .venvs/pocket-tools
```

# Activating the venv Environment

Use the `source` command to activate the environment by specifying the `/bin/activate` file inside the environment directory:

```bash
py@e73a39cc4acc:~$ source .venvs/pocket-tools/bin/activate
(pocket-tools) py@e73a39cc4acc:~$
```

Note that the name of the active environment appears before the username in the bash prompt. Now let's install some packages for Python and compare them with those in another environment. Clone the `pocket-tools` project and install the dependencies defined in the `requirements.txt` file:

```bash
(pocket-tools) py@e73a39cc4acc:~$ git clone https://github.com/skaczmarek-dev/pocket-tools
(pocket-tools) py@e73a39cc4acc:~$ pip install -r pocket-tools/requirements.txt
```

List all the packages installed by PIP:

```bash
(pocket-tools) py@e73a39cc4acc:~$ pip freeze
certifi==2024.8.30
charset-normalizer==3.4.0
idna==3.10
requests==2.32.3
urllib3==2.2.3
```

Now exit the active `venv` environment and list the packages installed by PIP in the default Python environment:

```bash
(pocket-tools) py@e73a39cc4acc:~$ deactivate
py@e73a39cc4acc:~$ pip freeze
setuptools==68.1.2
wheel==0.42.0
```

As you can see, the default Python environment contains different packages than the `pocket-tools` environment, and the `deactivate` command is used to exit the active environment.
