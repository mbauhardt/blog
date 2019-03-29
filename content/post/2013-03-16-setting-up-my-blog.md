---
title: "Setting Up This Blog"
date: 2013-03-16
tags: ["git", "jekyll-bootstrap"]
---

As a first step i forked the [plusjade/jekyll-bootstrap](https://github.com/plusjade/jekyll-bootstrap) project into [MyUserName/jekyll-bootstrap](https://github.com/MyUserName/jekyll-bootstrap). This gives me the following options

* change something in the origin code and provide this as a pull request commited into the _jb-development_ branch
* create a separate branch for my blog
* cherry-pick my changes into the blog branch
* update my blog branch with the latest changes from master


Now it is time to create the *empty* user repository on [github](http://github.com), see [github help](https://help.github.com/articles/user-organization-and-project-pages).
And clone the content of my jekyll bootstrap fork into

    git clone git@github.com git@github.com:MyUserName/jekyll-bootstrap.git MyUserName.github.com
    cd MyUserName.github.com

Then change the github origin to the user repo.

    git remote set-url origin git@github.com git@github.com:MyUserName/MyUserName.github.com.git

And add my jekyll bootstrap fork as an upstream url

    git remote add upstream git://github.com/MyUserName/jekyll-bootstrap.git

When i run into the situation that i want to update my blog with the latest jekyll update, i do the following on my jekyll-bootstrap fork

    git checkout master
    git pull
    git checkout MyUserName.github.com
    git merge master

Now sync these changes into my blog

    git checkout master
    git fetch upstream
    git merge upstream/master

Thats it.

