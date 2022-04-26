---
title: "Python Code Quality Tools I Use"
date: 2022-04-26
description:
  "A list of Python development environment and code quality tooling that I'm using."
---

# Python Code Quality Tools I Use

The goal is to have more or less **exhaustive code quality setup** for VS Code and Python with minimal overlapping between the tools.

**What I use**:

- `VS Code` modern web-based lightweight editor with optional IDE functionality (> PyCharm, Vim)

- `VS Code Python` (extension) for integration of a lot of other great tools into VS Code (bundle)
  
  - `black` for code formatting (> autopep8, yapf)
  
  - `isort` for import sorting/refactoring (not done by `black`) (as inner script)
  
  - `Pylance` (Python language server) for IntelliSense features while writing code (autocompletion, refactoring, static typing, ...) (> Jedi)
    
    - `pyright` for static type checker + `stubs` (> mypy, pytype, pyre-check)
  
  - `pylint` + `pylint-django`/`pylint-celery` for complete linting (stylistic, error, ...) (~`eslint`) (> flake8) 

**What I don't use**:

- Other CQ tools (pycodestyle, pydocstyle, pyflakes, mccabe, flake8 (wraps previous + plugins), prospector (wraps previous), pylama (wraps previous))

- `pre-commit` or CI code quality integration

- Security checking (bandit)
