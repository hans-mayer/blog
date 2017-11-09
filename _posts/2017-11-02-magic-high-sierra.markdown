---
layout: post
title:  magic macOS High Sierra 10.13 update
date:   2017-11-02 17:51:00 CET
categories:
---

Today Apple offered to make some updates. One of the updates was macOS High Sierra 10.13

Here in the update history you can still see it.

![high_sierra_history.png](/images/high_sierra_history.png)

But doing the upgrade and booting there is still the old version available.

![high_sierra_history.png](/images/high_sierra_current.png)

It's magic.

And here the magic solution. <br>
Remove the file "Install macOS Sierra" in the Application directory.

![high_sierra_old.png](/images/high_sierra_old.png)

After removing the old version upgrade was running successfully. It took more than one hour.

After the upgrade I tried to update my blog with "jekyll". But jekyll didn't run. This fixed the issue.

<pre>
brew update
brew upgrade  
brew install  ruby
brew install  rubygems
gem install jekyll
gem install jekyll-sitemap
jekyll server &
</pre>

And the same with iStats

<pre>
gem install iStats
</pre>
