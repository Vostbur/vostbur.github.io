---
layout: post
title: A dynamic GitHub profile with GitHub Actions
---

Thanks Yannick Chenot for the article [How to Build a Dynamic GitHub Profile with GitHub Actions and PHP](https://hackernoon.com/how-to-build-a-dynamic-github-profile-with-github-actions-and-php-h5g34cr)! It describes how to set up automatic updating of links to the latest blog posts in the GitHub profile using GitHub Actions. I will not retell the article, and you can read about the profile on GitHub [here]((https://docs.github.com/en/github/setting-up-and-managing-your-github-profile/customizing-your-profile/managing-your-profile-readme?ref=hackernoon.com)).

Here is my python solution

    import requests
    from bs4 import BeautifulSoup


    url = "https://vostbur.github.io"
    main_page = BeautifulSoup(requests.get(url).content, "html.parser")

    content = f"### Last posts from [blog]({url}):\n\n"
    for article_div in main_page.find_all("div", {"class": "mb-4"}):
        article_url = article_div.find("a", href=True)
        content += f"  - [{article_url.text}]({article_url['href']})\n"

    with open("README.md", "w", encoding="utf-8") as f:
        f.write(content)

Yaml file for GitHub Actions

    name: Update README.md

    on:
    push:
    workflow_dispatch:
    schedule:
        - cron: '0 0 * * *'

    jobs:
    build:
        runs-on: ubuntu-latest

        steps:
        - name: Clone repository
            uses: actions/checkout@v2
        
        - name: Install python dependencies
            run: pip3 install requests bs4

        - name: Update README.md
            run: python3 get_last_posts.py

        - name: Push changes
            uses: stefanzweifel/git-auto-commit-action@v4
            with:
            commit_message: Updated latest blog posts

----
### Links:

- [1] [How to Build a Dynamic GitHub Profile with GitHub Actions and PHP](https://hackernoon.com/how-to-build-a-dynamic-github-profile-with-github-actions-and-php-h5g34cr)
- [2] [Managing your profile README](https://docs.github.com/en/github/setting-up-and-managing-your-github-profile/customizing-your-profile/managing-your-profile-readme?ref=hackernoon.com)
