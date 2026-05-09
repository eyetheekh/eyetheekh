---
title: "Android APK's with Python and Buildozer"
date: 2025-12-28T14:00:00+05:30
draft: false
tags: ["android"]
---

My first Android phone was a Moto G5s Plus 4/64. I installed many apps, games, and camera mods, all as APK files. I used to download APKs and patch them using Lucky Patcher. Good times.

A while later, I started thinking about building an Android app myself. I still remember the day I installed Android Studio on my HP 15 i3 laptop with 4GB RAM.  
It was brutal. The studio would barely opened after 2mins. That just killed my Android dreams back then :(

I randomly got an idea for a camera based app while standing in a grocery store. I thought it would be useful to have an app that could scan a bill, verify the prices and totals, and also store metadata about which items I spend the most money on.

I wanted to see if this was possible with using Python  and came across **Kivy** and **Buildozer**, which can be used to create APKs using plain Python. Well, why not give it a try.
I started with documentation, articles, github repositories, and tutorials. Well, most of them were outdated and the torture began. 

Getting a basic installable APK was very difficult. I spent hours on Colab and Kaggle experimenting with different combinations of Python, Cython, virtual environments, OpenJDK versions, system libraires and dependencies. It took two weeks just to crack a medical miracle and generate a properly installable APK that did not instantly crash.

This path is not recommended for normal humans, But incase if you want to build an APK and you insist on doing it only with Python, and you also hear about Kivy Buildozer:

```bash
sudo apt install python3-pip
sudo apt install python3.12-venv

mkdir myapp
sudo chmod -R 777 myapp
cd myapp

mkdir .venv
sudo chmod -R 777 .venv
sudo python3 -m venv .venv
source .venv/bin/activate

pip install -U buildozer

sudo apt update
sudo apt install -y git zip unzip openjdk-17-jdk python3-pip autoconf \
libtool pkg-config zlib1g-dev libncurses5-dev libncursesw5-dev \
libtinfo5 cmake libffi-dev libssl-dev

pip3 install --upgrade Cython==0.29.33 virtualenv
sudo apt install nano
export PATH=$PATH:~/.local/bin/
nano ~/.bashrc
sudo apt install -y automake
sudo apt install -y libtinfo5
pip3 install setuptools

buildozer -v android debug
```

As for the grocery bill scanning idea, even getting the base environment working itself was difficult. Its just outdated today and the documentation alone is not enough. Hopefully, I will revisit the project properly when I have enough time.

## Resources:
- https://www.youtube.com/@laptopml
- https://www.youtube.com/watch?v=l8Imtec4ReQ
- https://stackoverflow.com/questions/60077141/building-an-android-app-with-kivybuildozer
- https://medium.com/data-science/python-for-android-start-building-kivy-cross-platform-applications-6cf867d44612
