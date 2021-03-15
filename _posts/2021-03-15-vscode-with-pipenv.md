---
layout: post
title: Using VS Code with pipenv
---

I found out how to use pipenv environment with VS Code from the article ["Visual Studio Code, Python and pipenv"](https://www.benjaminpack.com/blog/vs-code-python-pipenv/). Thanks, Benjamin Pack! I saved the text for myself.

---

First, find out where pipenv has created your virtualenv setup and stashed the python executable you are using. From the command line in your project folder (where your Pipfile is), execute the following:

`pipenv --py`

This will give you the full path to your virtualenv python install. For my sample project it was `/Users/pack/.local/share/virtualenvs/astra-AQkAm5fD/bin/python`

Now, create a local settings file that VS Code will use for your project

`mkdir .vscode && touch .vscode/settings.json`

After executing that, you’ll want to open the `settings.json` file and paste in the following:

```
{
    "files.exclude": {
        "**/.git": true,
        "**/.svn": true,
        "**/.hg": true,
        "**/CVS": true,
        "**/.DS_Store": true,
        "**/*.pyc": true,
        "**/__pycache__": true
    },
    "python.pythonPath": "<VIRTUALENV_PYTHON_PATH_HERE>"
}
```

The `files.exclude` block takes the existing VS Code settings for files not displayed and adds `.pyc` files and `__pycache__` folders to the list. The `python.pythonPath` variable is where you need to include the virtualenv python location that you found earlier.

Now when you load up VS Code with your project, it will use the appropriate Python version. If you also have pylint installed in your virtualenv environment then it will use that version to display any linting problems. If not, you can do that with the following.

`pipenv install --dev pylint`
