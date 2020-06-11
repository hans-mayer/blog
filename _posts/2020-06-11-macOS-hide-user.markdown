---
layout: post
title:  macOS - hide user
date:   2020-06-11 17:09:00 CET
categories:
---


# macOS - hide a user at login screen

## list all users

<pre>
dscl . list /Users
</pre>

## hide login "test"

<pre>
sudo dscl . create /Users/test IsHidden 1
sudo defaults write /Library/Preferences/com.apple.loginwindow SHOWOTHERUSERS_MANAGED -bool FALSE
</pre>

## reverse the hidden user

<pre>
sudo dscl . create /Users/test IsHidden 0
sudo defaults delete /Library/Preferences/com.apple.loginwindow SHOWOTHERUSERS_MANAGED
</pre>
