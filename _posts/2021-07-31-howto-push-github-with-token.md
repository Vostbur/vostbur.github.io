---
layout: post
title: How to push to GitHub via HTTPS if you have enabled two-factor authentication
---

If you enabled two-factor authentication in your Github account you won't be able to push via HTTPS using your accounts password. Instead you need to generate a personal [access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/). This can be done in the application settings of your Github account. Using this token as your password should allow you to push to your remote repository via HTTPS. Use your username as usual.

You may also need to update the origin for your repository if set to https:

```
git remote -v 
git remote remove origin
git remote add origin https://username:access-token@github.com/username/repo.git
```

----
### Links:

- [1] [Creating a personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
- [2] [Git push results in “Authentication Failed”](https://stackoverflow.com/questions/17659206/git-push-results-in-authentication-failed)
