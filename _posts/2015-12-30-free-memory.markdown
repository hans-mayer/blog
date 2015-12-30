---
layout: post
title:  "free memory GNU/Linux "
date:   2015-12-30 12:00:00 CET
categories: GNU/Linux memory free
---

Linux has a well working memory management. Generally it's not necessary to adjust something manually. You can see the used and free space with the **free** command. 

<pre>
# free -m
             total       used       free     shared    buffers     cached
Mem:           970        376        593          0         99        102
-/+ buffers/cache:        174        795
Swap:          511          0        511
</pre>

If there is free memory space it can be used as cache for faster access. Some times it's interesting how much memory is really used and how much could be free. In this case you can drop the cache.

<pre>
# echo 3 > /proc/sys/vm/drop_caches
</pre>


<pre>
# free -m
             total       used       free     shared    buffers     cached
Mem:           970         39        931          0          0          6
-/+ buffers/cache:         31        938
Swap:          511          0        511
</pre>

