---
layout: post
title:  PPP with gpsrinex
date:   2026-01-21 17:31:00 CET
categories: gps 
---

In my previous blog [PPP Precise Point Positioning](/2023/06/03/PPP-Precise-Point-Positioning.html){:target="_blank"} 3 years ago I used the method "averaging" to get a precise position. 

This time I used [this Canadian service](https://webapp.csrs-scrs.nrcan-rncan.gc.ca/geod/tools-outils/ppp.php){:target="_blank"} for post-processing the data. 

So I prepared my GNSS receiver ZEF-F9P for this job. This is well documented in the man page for "gpsrinex". 

So I run over several days commands like this: <BR>
`gpsrinex -i 30 -n 1440 -f gpsrinex2025322205923.obs ` <BR>
This would collect the information for a period of 12 hours. <BR>
2025322205923 means: year 2025 day number 322 of this year at 20:59:23 h

If this job is finished it's theoretical possible to upload this file immediately to the [Canadian service](https://webapp.csrs-scrs.nrcan-rncan.gc.ca/geod/tools-outils/ppp.php){:target="_blank"}. But then you get a result as product type "ultra fast". If you wait a day or so it's "fast" and if you wait more than a week you get the "final" version. Because it takes some time for them to get all the correction data to make a qualitativ high post processing with my collected data. 

If one submits this file one will get the result per e-mail. This e-mail may take several hours but it could also be available after several minutes. It depends on how much requests are in the queue. 

As processing mode I selected "Static" and "ITRF" 

This is the final result

<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        padding: 2px;
        }
</style>

| &nbsp;&nbsp;date&nbsp;&nbsp;                  | &nbsp;&nbsp;latitude&nbsp;&nbsp;        | &nbsp;&nbsp;longitude&nbsp;&nbsp;       |
| ----------------------------------------------- | ----------------------------------------- | ----------------------------------------- |
| &nbsp;&nbsp;gpsrinex2023117170821&nbsp;&nbsp; | &nbsp;&nbsp;48.14929225833&nbsp;&nbsp;  | &nbsp;&nbsp;16.28384379721&nbsp;&nbsp;  |
| &nbsp;&nbsp;gpsrinex2025322205923&nbsp;&nbsp; | &nbsp;&nbsp;48.14929171666&nbsp;&nbsp;  | &nbsp;&nbsp;16.28384437221&nbsp;&nbsp;  |
| &nbsp;&nbsp;gpsrinex2025323171908&nbsp;&nbsp; | &nbsp;&nbsp;48.14928167499&nbsp;&nbsp;  | &nbsp;&nbsp;16.28384415555&nbsp;&nbsp;  |
| &nbsp;&nbsp;gpsrinex2025323203540&nbsp;&nbsp; | &nbsp;&nbsp;48.14929222499&nbsp;&nbsp;  | &nbsp;&nbsp;16.28384461388&nbsp;&nbsp;  |
| &nbsp;&nbsp;gpsrinex2025324091707&nbsp;&nbsp; | &nbsp;&nbsp;48.14929101388&nbsp;&nbsp;  | &nbsp;&nbsp;16.28384304444&nbsp;&nbsp;  |
| &nbsp;&nbsp;gpsrinex2025327220019&nbsp;&nbsp; | &nbsp;&nbsp;48.14929243888&nbsp;&nbsp;  | &nbsp;&nbsp;16.28384382777&nbsp;&nbsp;  |
| &nbsp;&nbsp;gpsrinex2025330191355&nbsp;&nbsp; | &nbsp;&nbsp;48.14929198610&nbsp;&nbsp;  | &nbsp;&nbsp;16.28384486110&nbsp;&nbsp;  |
| &nbsp;&nbsp;average&nbsp;&nbsp;               | &nbsp;&nbsp;48.149291939807&nbsp;&nbsp; | &nbsp;&nbsp;16.283844086102&nbsp;&nbsp; |
{: .tablelines}

If I compare this result with the result of "averaging" then there is a gap of about 47 cm. 

This is the result as graph without the outlier in line 3 at 2025323171908

![gnss_position_gpsrinex](/images/gnss_position_gpsrinex.png)

