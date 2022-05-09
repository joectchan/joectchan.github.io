---
title: "How to use personal access token from github"
categories:
  - blog
tags:
  - github
  - security
---
Github does not allow me to push my existing repo using username/password anymore. I need to create personal access token.

1. I need to login to my github account.
2. Follow this [link](https://techglimpse.com/git-push-github-token-based-passwordless/) to create a personal access token.
3. Save the token in a password manager to avoid losing it.
4. Show remote origin of my local git repo.
```
 git remote show origin
* remote origin
  Fetch URL: https://github.com/joectchan/joectchan.github.io.git
  Push  URL: https://github.com/joectchan/joectchan.github.io.git
```
5. The link in step 2 tells you how to git push using personal access token. i.e. for my repo shown in step 4, I need to type:
```
git push https://<personal_access_token>@github.com/joectchan/joectchan.github.io.git
```
