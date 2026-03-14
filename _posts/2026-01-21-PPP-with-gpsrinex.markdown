---
layout: post
title:  PPP with gpsrinex, CSRS-PPP and ECTT 
date:   2026-01-21 17:31:00 CET
categories: gps 
---

In my previous blog [PPP Precise Point Positioning](/2023/06/03/PPP-Precise-Point-Positioning.html){:target="_blank"} 3 years ago I used the method "averaging" to get a precise position. 

This time I used [this Canadian service CSRS-PPP](https://webapp.csrs-scrs.nrcan-rncan.gc.ca/geod/tools-outils/ppp.php){:target="_blank"} for post-processing the data. 

So I prepared my GNSS receiver ZEF-F9P for this job. This is well documented in the man page for "gpsrinex". 

So I run over several days commands like this: <BR>
`gpsrinex -i 30 -n 1440 -f gpsrinex2025322205923.obs ` <BR>
This would collect the information for a period of 12 hours. <BR>
2025322205923 means: year 2025 day number 322 of this year at 20:59:23 h

If this job is finished it's theoretical possible to upload this file immediately to the [Canadian service](https://webapp.csrs-scrs.nrcan-rncan.gc.ca/geod/tools-outils/ppp.php){:target="_blank"}. But then you get a result as product type "ultra rapid". If you wait two days or so it's "rapid" and if you wait more than two weeks you get the "final" version. Because it takes some time for them to get all the correction data to make a qualitativ high post processing with my collected data. 

If one submits this file one will get the result per e-mail. This e-mail may take several hours but it could also be available after several minutes. It depends on how much requests are in the queue. 

As processing mode I selected "Static" and "ITRF" (International Terrestrial Reference Frame)

This is the final result

<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        padding: 2px;
        }
</style>
| &nbsp;&nbsp;date&nbsp;&nbsp;          | &nbsp;&nbsp;latitude&nbsp;&nbsp;        | &nbsp;&nbsp;longitude&nbsp;&nbsp;       | &nbsp;&nbsp;high&nbsp;&nbsp;             |
|---------------------------------------|-----------------------------------------|-----------------------------------------|------------------------------------------|
| &nbsp;&nbsp;2023117170821&nbsp;&nbsp; | &nbsp;&nbsp;48.1492922571&nbsp;&nbsp;   | &nbsp;&nbsp;16.2838437976&nbsp;&nbsp;   | &nbsp;&nbsp;286.6270&nbsp;&nbsp;         |
| &nbsp;&nbsp;2025322205923&nbsp;&nbsp; | &nbsp;&nbsp;48.1492917165&nbsp;&nbsp;   | &nbsp;&nbsp;16.2838443722&nbsp;&nbsp;   | &nbsp;&nbsp;287.0627&nbsp;&nbsp;         |
| &nbsp;&nbsp;2025323203540&nbsp;&nbsp; | &nbsp;&nbsp;48.1492922225&nbsp;&nbsp;   | &nbsp;&nbsp;16.2838446130&nbsp;&nbsp;   | &nbsp;&nbsp;286.7583&nbsp;&nbsp;         |
| &nbsp;&nbsp;2025324091707&nbsp;&nbsp; | &nbsp;&nbsp;48.1492910139&nbsp;&nbsp;   | &nbsp;&nbsp;16.2838430433&nbsp;&nbsp;   | &nbsp;&nbsp;286.9125&nbsp;&nbsp;         |
| &nbsp;&nbsp;2025327220019&nbsp;&nbsp; | &nbsp;&nbsp;48.1492924369&nbsp;&nbsp;   | &nbsp;&nbsp;16.2838438271&nbsp;&nbsp;   | &nbsp;&nbsp;286.7396&nbsp;&nbsp;         |
| &nbsp;&nbsp;2025330191355&nbsp;&nbsp; | &nbsp;&nbsp;48.1492919858&nbsp;&nbsp;   | &nbsp;&nbsp;16.2838448611&nbsp;&nbsp;   | &nbsp;&nbsp;286.8052&nbsp;&nbsp;         |
| &nbsp;&nbsp;average&nbsp;&nbsp;       | &nbsp;&nbsp;48.1492919387&nbsp;&nbsp; | &nbsp;&nbsp;16.2838440857&nbsp;&nbsp; | &nbsp;&nbsp;286.817&nbsp;&nbsp; |
{: .tablelines}

<pre>N: 48 8 57.45097  E: 16 17 1.83870  H: 286.817 m 
X: 4092523.2481   Y: 1195484.9891   Z: 4728181.4834 
</pre>

If I compare this result with the result of "averaging" then there is a gap of about 47 cm in direction 156 deg South-Southeast (SSE) 

This is the result as graph 

![gnss_position_gpsrinex](/images/gnss_position_gpsrinex.png)

Now using the [ETRF/ITRF Coordinate Transformation Tool ECTT](https://www.epncb.oma.be/_productsservices/coord_trans/index.php){:target="_blank"} to convert from ITRF2020 ( over ITRF2000 ) to ETRF2000 for epoche 2026.0 I get

<pre>4092523.9026 1195484.3949 4728181.0565
48.1492862849 16.2838339544 286.8070
48 8 57.430625640  16 17 1.8022358400
</pre>


A commandline tool to [transform ecef wgs84](https://github.com/hans-mayer/transform_ecef_wgs84){:target="_blank"} data.


