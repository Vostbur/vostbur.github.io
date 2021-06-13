---
layout: post
title: Setup a basic workflow using GitHub Actions with Docker
---

In the continuation of the post ["Creating dockerized Django app with VSCode"](https://vostbur.github.io/docker-django-vscode-start/), I updated the [source code](https://github.com/Vostbur/vscode-django-docker) to work with GitHub Actions.

## Setup a basic workflow using GitHub Actions

Add to the **requirements.txt** file the following line to check with linter:

    flake8>=3.9.2,<3.10

Create a new file at **.github/workflows/cd.yml** and fill it with the following contents:

    ---
    name: Code checks

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]

    jobs:
    test:
        name: Test
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v2
        - name: Test
            run: docker-compose run --rm . sh -c "python manage.py test"

    lint:
        name: Lint
        runs-on: ubuntu-20.04
        steps:
        - uses: actions/checkout@v2
        - name: Lint
            run: docker-compose run --rm . sh -c "flake8"

For adding a cool badge you I updated the README.md with the following link:

    [![Build Status](https://github.com/Vostbur/vscode-django-docker/actions/workflows/cd.yml/badge.svg?branch=master)](https://github.com/Vostbur/vscode-django-docker/actions/workflows/cd.yml)

It looks like: [![Build Status](https://github.com/Vostbur/vscode-django-docker/actions/workflows/cd.yml/badge.svg?branch=master)](https://github.com/Vostbur/vscode-django-docker/actions/workflows/cd.yml)

----
### Links:

- [1] [GitHub Actions для автоматической проверки кода](https://www.youtube.com/watch?v=NijFSs03Pd4)
- [2] [HOW TO USE GITHUB ACTIONS](https://londonappdeveloper.com/how-to-use-github-actions/)
