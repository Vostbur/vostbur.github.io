---
categories:
- Programming
date: "2022-03-13"
description: How to backup (clone) all your own gists
slug: backup-all-your-own-gists
tags:
- python
- GitHub
title: Backup (clone) all your own gists
---

You need to have a personal access token to GitHub. In this script I use my token saved in `~/.github_access_token`.

Also you need to have the `git` utility on your system.

Install `PyGithub` package to work with GitHub API

    pip install PyGithub

The gist is actually a git repository.  All your gists you can clone as regular repositories into a directory called `repos`. Script clones gists with names contained the IDs. Also script creates `index.json` which contains an array of objects describing what you got.


    #!/usr/bin/env python -B

    # pip install PyGithub
    from github import Github
    import pathlib
    import json
    import os


    gists = []

    g = Github(open(pathlib.Path.home() / ".github_access_token").read())

    for gist in g.get_user().get_gists():
        gists.append({
            "id": gist.id,
            "description": gist.description,
            "public": gist.public,
            "clone": gist.git_pull_url,
            "updated": gist.updated_at.isoformat(),
            "url": gist.url,
        })

        os.system("git clone {0} repos/{1}".format(gist.git_pull_url, gist.id))

    with open("index.json", "w") as f:
        f.write(json.dumps(gists, indent=4) + "\n")
