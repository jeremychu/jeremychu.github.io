---
layout: post
title: "How do I use this bloody thing?"
date: 2014-07-13 09:41:00 +0100
comments: true
categories: git octopress
---

_Note to self..._
<!-- more -->
Clone the repo - this will get you Master:

```
> git clone https://github.com/jeremychu/jeremychu.github.io.git
> cd jeremychu.github.io
```

Now switch to the Source branch:

```
> git checkout -b source origin/source
```

Setup for deployment to Github:

```
> rake setup_github_pages
	Enter the read/write url for your repository
	(For example, 'git@github.com:your_username/your_username.github.io.git)
	           or 'https://github.com/your_username/your_username.github.io')
	Repository url: git@github.com:jeremychu/jeremychu.github.io.git
```

You can now run _rake generate_ and _rake deploy_ , however it doesn't work.  I have no idea why.  
Future Jeremy, go and find out why!

```
> rake generate
...
> rake deploy

## Pushing generated _deploy website
To git@github.com:jeremychu/jeremychu.github.io.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:jeremychu/jeremychu.github.io.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

To fix it I do the following:
```
> cd _deploy
> git pull origin master
> cd ..
> rake deploy
```
Success!


Create a post:

```
> rake new_post['a new lot of learnings']
```

Don't forget to commit the source:

```
> git add .
> git commit -m "Updates to blog source"
> git push
```

