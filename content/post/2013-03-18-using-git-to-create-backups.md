---
title: "Using Git To Create Backup's"
date: 2013-03-18
tags: ["git"]
---

When you want to use git to backup your local documents on an usb stick, you can do the following steps.

1) Create Empty Git Repository

    mkdir /Volumes/usb-stick/Documents.git
    cd /Volumes/usb-stick/Documents.git
    git --bare init

2) Create A First Backup Your Documents 

    cd ~/Documents
    git init
    git add ./
    git commit
    git remote add origin /Volumes/usb-stick/Documents.git
    git push -u origin master


3) Backup New Files

    cd ~/Documents
    git add .
    git commit
    git push


4) You Can Also Clone Your Backup On Another Machine Or Folder

    mkdir ~/Desktop/NewFolder
    cd ~/Desktop/NewFolder
    git clone /Volumes/usb-stick/Documents.git .

