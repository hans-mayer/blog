---
layout: post
title:  source port 19000/TCP DDOS
date:   2025-02-19 14:00:00 CET
categories: security
---

I realised that I was under a DDOS attack yesterday. Now, this is nothing surprising nor unexpected because this happens every day and every time. The interesting part is that I realised it at all. <br>

Between 20:29:41 and 20:30:23 LT (=UTC+1) I had 450 attempts to reach different ports on my site but coming all from source port 19000. There were 122 distinct IP involved and each IP did exactly try 4 times an attempt with 2 different ports. Therefore each port was checked twice. But no port was checked by more than 1 source IP. Starting with dest port 22 ending at 65443 with big gaps between. So it was coordinated in my opinion. The source IP were coming from around the world. Major part from US with 42% and Canada with 9%. Interesting there is a large white area where normal most of the hacks are coming. All packets did have the SYN flag and it was not the return packet from a lost established session. <br>

It would never catch my eyes or trigger any alert because of four TCP probes per IP is nothing. But the high number of identical source ports was the crucial factor. I could imagine that the idea was to find a weak FW or OS, which opens the port to anywhere on 19000/tcp in the first attempt and the seconds one verifies if this really happened. This indicates that each IP address did probe the same port twice.

Btw, all of the attacking IPs have open ports 554/tcp and 555/tcp (rtsp-proxy) and several other ports too. Not verified all of them but it seems to be that the operating systems are Linux like. And none of them had open port 19000/tcp.

A new experience. Another interesting part is the fact that Russia was not seen as source. For me an indication that hackers from over there are maybe involved.

This is the timeline

![19000/TCP timeline](/images/tcp19000_timeline.png)

This is the geography

![19000/TCP map](/images/tcp19000_map.png)

Part of the FW log

|________time|__________________IP|_____dport|___ISO|___________country|
|-----------:|-------------------:|---------:|-----:|-----------------:|
|    20:29:41|     109.241.221.153|     10048|    PL|            Poland|
|    20:29:41|        193.37.83.54|      8888|    GB|    United Kingdom|
|    20:29:41|       98.98.143.146|       523|    GB|    United Kingdom|
|    20:29:42|      101.78.214.242|     10086|    HK|         Hong Kong|
|    20:29:42|      170.76.164.246|      9080|    US|     United States|
|    20:29:42|      173.219.41.139|      1974|    US|     United States|
|    20:29:42|      174.86.201.204|       441|    US|     United States|
|    20:29:42|        184.67.57.42|      8008|    CA|            Canada|
|    20:29:42|      188.155.25.168|      8445|    CH|       Switzerland|
|    20:29:42|        190.4.184.51|     20443|    CW|           Curaçao|
|    20:29:42|      216.224.227.14|      1964|    US|     United States|
|    20:29:42|        69.159.190.8|     10444|    CA|            Canada|
|    20:29:42|        70.27.96.136|      8010|    CA|            Canada|
|    20:29:42|       84.205.40.144|      8447|    NO|            Norway|
|    20:29:42|        89.255.58.72|     58443|    NL|   The Netherlands|
|    20:29:42|       96.224.240.47|      1010|    US|     United States|
|    20:29:42|      98.199.189.159|      3000|    US|     United States|
|    20:29:43|        107.0.204.18|      8006|    US|     United States|
|    20:29:43|       111.248.97.33|      5060|    TW|            Taiwan|
|    20:29:43|      118.142.38.162|      8880|    HK|         Hong Kong|
|    20:29:43|      140.186.62.119|      8084|    US|     United States|
|    20:29:43|       149.75.156.25|       448|    US|     United States|
|    20:29:43|       154.45.228.44|      9600|    FR|            France|
|    20:29:43|      180.173.61.181|      8005|    CN|             China|
|    20:29:43|      181.104.24.183|      9999|    AR|         Argentina|
|    20:29:43|     187.122.109.102|      1443|    BR|            Brazil|
|    20:29:43|      187.18.115.195|      1983|    BR|            Brazil|
|    20:29:43|       187.33.161.84|      1030|    BR|            Brazil|
|    20:29:43|         2.136.0.159|      7654|    ES|             Spain|
|    20:29:43|       201.6.108.105|      9191|    BR|            Brazil|
|    20:29:43|      208.38.251.179|      8000|    US|     United States|
|    20:29:43|      217.76.195.244|      8899|    UA|           Ukraine|
|    20:29:43|      218.157.37.253|      1984|    KR|       South Korea|
|    20:29:43|        24.106.219.6|        84|    US|     United States|
|    20:29:43|        24.106.219.6|        85|    US|     United States|
|    20:29:43|         24.65.94.39|     12443|    CA|            Canada|
|    20:29:43|       45.25.125.209|      5080|    US|     United States|
|    20:29:43|        58.38.25.226|      8444|    CN|             China|
|    20:29:43|       66.212.63.229|      4434|    LC|       Saint Lucia|
|    20:29:43|      66.245.254.218|       100|    CA|            Canada|
|    20:29:43|        66.76.57.209|     44301|    US|     United States|
|    20:29:43|        68.113.82.60|      9696|    US|     United States|
|    20:29:43|        70.71.86.179|      3389|    CA|            Canada|
|    20:29:43|        72.139.71.98|     20110|    CA|            Canada|
|    20:29:43|      74.141.164.146|      6061|    US|     United States|
|    20:29:43|        74.195.87.34|      5002|    US|     United States|
|    20:29:43|        75.138.1.102|      8787|    US|     United States|
|    20:29:43|        78.194.35.58|      1188|    FR|            France|
|    20:29:43|       84.247.88.162|      1025|    RO|           Romania|
|    20:29:43|       85.14.121.150|     50005|    PL|            Poland|
|    20:29:43|        85.26.26.123|      8016|    BE|           Belgium|
|    20:29:43|       88.95.222.128|      8070|    NO|            Norway|
|    20:29:43|        89.255.39.69|      9595|    NL|   The Netherlands|
|    20:29:43|        93.42.231.58|      8282|    IT|             Italy|
|    20:29:43|       98.165.110.67|      8123|    US|     United States|
|    20:29:43|       98.98.143.146|        23|    GB|    United Kingdom|
|    20:29:43|       98.98.143.146|       523|    GB|    United Kingdom|
|    20:29:44|        1.36.177.143|      5555|    HK|         Hong Kong|
|    20:29:44|      108.46.137.167|      8091|    US|     United States|
|    20:29:44|      120.28.188.254|      6969|    PH|       Philippines|
|    20:29:44|     150.129.144.147|      4711|    IN|             India|
|    20:29:44|     151.236.206.148|      2105|    SE|            Sweden|
|    20:29:44|       162.247.23.26|      8843|    CA|            Canada|
|    20:29:44|      170.76.164.246|      9080|    US|     United States|
|    20:29:44|         171.6.96.64|      6580|    TH|          Thailand|
|    20:29:44|      173.219.41.139|      1974|    US|     United States|
|    20:29:44|      174.86.201.204|       441|    US|     United States|
|    20:29:44|      180.65.213.179|       666|    KR|       South Korea|
|    20:29:44|       186.73.69.146|      9076|    PA|            Panama|
|    20:29:44|      188.155.25.168|      8445|    CH|       Switzerland|
|    20:29:44|        190.4.184.51|     20443|    CW|           Curaçao|
|    20:29:44|        193.37.83.54|      8888|    GB|    United Kingdom|
|    20:29:44|      202.22.227.179|       447|    NC|     New Caledonia|
|    20:29:44|      203.129.25.134|      8012|    AU|         Australia|
|    20:29:44|      203.205.35.215|      8446|    VN|           Vietnam|
|    20:29:44|       206.192.81.22|      6868|    US|     United States|
|    20:29:44|      207.136.99.239|      9898|    CA|            Canada|
|    20:29:44|      211.176.84.143|      7443|    KR|       South Korea|
|    20:29:44|       212.50.191.10|       555|    GB|    United Kingdom|
|    20:29:44|      216.224.227.14|      1964|    US|     United States|
|    20:29:44|      216.36.168.168|      5272|    CA|            Canada|
|    20:29:44|      220.135.125.95|     55443|    TW|            Taiwan|
|    20:29:44|      223.197.136.27|      8765|    HK|         Hong Kong|
|    20:29:44|        24.229.57.68|      2200|    US|     United States|
|    20:29:44|       24.76.119.247|      1605|    CA|            Canada|
|    20:29:44|        42.98.213.22|     20107|    HK|         Hong Kong|
|    20:29:44|        45.85.208.74|       255|    PL|            Poland|
|    20:29:44|       47.206.202.61|      4040|    US|     United States|
|    20:29:44|        50.239.83.38|      3128|    US|     United States|
|    20:29:44|      58.153.207.122|       442|    HK|         Hong Kong|
|    20:29:44|       61.219.165.49|      9030|    TW|            Taiwan|
|    20:29:44|      62.232.150.162|      7441|    GB|    United Kingdom|
|    20:29:44|       66.209.63.242|      8089|    CA|            Canada|
|    20:29:44|      68.132.165.218|     50001|    US|     United States|
|    20:29:44|      70.172.137.135|      3060|    US|     United States|
|    20:29:44|       72.12.122.239|      8072|    US|     United States|
|    20:29:44|      83.228.108.120|      9016|    BG|          Bulgaria|
|    20:29:44|       84.205.40.144|      8447|    NO|            Norway|
|    20:29:44|        89.255.58.72|     58443|    NL|   The Netherlands|
|    20:29:44|       96.224.240.47|      1010|    US|     United States|
|    20:29:44|      96.249.219.181|      7071|    US|     United States|
|    20:29:44|       98.150.220.77|      8050|    US|     United States|
|    20:29:44|       98.155.101.80|      8585|    US|     United States|
|    20:29:44|        98.172.76.20|      8800|    US|     United States|
|    20:29:44|       98.98.143.146|      8182|    GB|    United Kingdom|
|    20:29:45|      101.78.214.242|     10086|    HK|         Hong Kong|
|    20:29:45|        107.0.204.18|      8006|    US|     United States|
|    20:29:45|       111.248.97.33|      5060|    TW|            Taiwan|
|    20:29:45|      140.186.62.119|      8084|    US|     United States|
|    20:29:45|       149.75.156.25|       448|    US|     United States|
|    20:29:45|       154.45.228.44|      9600|    FR|            France|
|    20:29:45|      167.98.154.219|       445|    GB|    United Kingdom|
|    20:29:45|        184.67.57.42|      8008|    CA|            Canada|
|    20:29:45|     187.122.109.102|      1443|    BR|            Brazil|
|    20:29:45|      187.18.115.195|      1983|    BR|            Brazil|
|    20:29:45|       201.6.108.105|      9191|    BR|            Brazil|
|    20:29:45|       202.21.110.22|     22533|    MN|          Mongolia|
|    20:29:45|      218.157.37.253|      1984|    KR|       South Korea|
|    20:29:45|       218.161.62.42|      8090|    TW|            Taiwan|
|    20:29:45|        24.106.219.6|        84|    US|     United States|
|    20:29:45|        24.106.219.6|        85|    US|     United States|
|    20:29:45|       45.25.125.209|      5080|    US|     United States|
|    20:29:45|        58.38.25.226|      8444|    CN|             China|
|    20:29:45|      61.238.217.162|      1989|    HK|         Hong Kong|
|    20:29:45|      66.245.254.218|       100|    CA|            Canada|
|    20:29:45|        68.113.82.60|      9696|    US|     United States|
|    20:29:45|        70.27.96.136|      8010|    CA|            Canada|
|    20:29:45|        74.195.87.34|      5002|    US|     United States|
|    20:29:45|        78.194.35.58|      1188|    FR|            France|
|    20:29:45|       84.247.88.162|      1025|    RO|           Romania|
|    20:29:45|        85.26.26.123|      8016|    BE|           Belgium|
|    20:29:45|       88.95.222.128|      8070|    NO|            Norway|
|    20:29:45|      89.197.228.162|      3443|    GB|    United Kingdom|
|    20:29:45|        89.255.39.69|      9595|    NL|   The Netherlands|
|    20:29:45|        93.42.231.58|      8282|    IT|             Italy|
|    20:29:45|      98.199.189.159|      3000|    US|     United States|
|    20:29:45|        98.6.254.142|      8881|    US|     United States|
|    20:29:45|       98.98.143.146|        23|    GB|    United Kingdom|
|    20:29:46|       108.52.245.64|     33333|    US|     United States|
|    20:29:46|      118.142.38.162|      8880|    HK|         Hong Kong|
|    20:29:46|     150.129.144.147|      4711|    IN|             India|
|    20:29:46|       162.247.23.26|      8843|    CA|            Canada|
|    20:29:46|         171.6.96.64|      6580|    TH|          Thailand|
|    20:29:46|      180.173.61.181|      8005|    CN|             China|
|    20:29:46|      181.104.24.183|      9999|    AR|         Argentina|
|    20:29:46|       187.33.161.84|      1030|    BR|            Brazil|
|    20:29:46|         2.136.0.159|      7654|    ES|             Spain|
|    20:29:46|      202.22.227.179|       447|    NC|     New Caledonia|
|    20:29:46|      203.129.25.134|      8012|    AU|         Australia|
|    20:29:46|      203.132.196.87|     10080|    HK|         Hong Kong|
|    20:29:46|      207.136.99.239|      9898|    CA|            Canada|
|    20:29:46|      208.38.251.179|      8000|    US|     United States|
|    20:29:46|      210.177.235.39|      8082|    HK|         Hong Kong|
|    20:29:46|      211.176.84.143|      7443|    KR|       South Korea|
|    20:29:46|       212.82.69.194|      5443|    GB|    United Kingdom|
|    20:29:46|      216.36.168.168|      5272|    CA|            Canada|
|    20:29:46|      217.76.195.244|      8899|    UA|           Ukraine|
|    20:29:46|        24.229.57.68|      2200|    US|     United States|
|    20:29:46|         24.65.94.39|     12443|    CA|            Canada|
|    20:29:46|       24.76.119.247|      1605|    CA|            Canada|
|    20:29:46|       47.206.202.61|      4040|    US|     United States|
|    20:29:46|      58.153.207.122|       442|    HK|         Hong Kong|
|    20:29:46|      62.232.150.162|      7441|    GB|    United Kingdom|
|    20:29:46|       66.209.63.242|      8089|    CA|            Canada|
|    20:29:46|       66.212.63.229|      4434|    LC|       Saint Lucia|
|    20:29:46|      68.132.165.218|     50001|    US|     United States|
|    20:29:46|      70.172.137.135|      3060|    US|     United States|
|    20:29:46|       72.12.122.239|      8072|    US|     United States|
|    20:29:46|      75.109.210.112|     10000|    US|     United States|
|    20:29:46|        75.138.1.102|      8787|    US|     United States|
|    20:29:46|      83.228.108.120|      9016|    BG|          Bulgaria|
|    20:29:46|       85.14.121.150|     50005|    PL|            Poland|
|    20:29:46|      87.242.235.220|      3118|    GB|    United Kingdom|
|    20:29:46|      96.249.219.181|      7071|    US|     United States|
|    20:29:46|       98.150.220.77|      8050|    US|     United States|
|    20:29:46|       98.155.101.80|      8585|    US|     United States|
|    20:29:46|       98.165.110.67|      8123|    US|     United States|
|    20:29:46|        98.172.76.20|      8800|    US|     United States|
|    20:29:46|       98.98.143.146|      8182|    GB|    United Kingdom|
|    20:29:47|        1.36.177.143|      5555|    HK|         Hong Kong|
|    20:29:47|      108.46.137.167|      8091|    US|     United States|
|    20:29:47|      120.28.188.254|      6969|    PH|       Philippines|
|    20:29:47|     151.236.206.148|      2105|    SE|            Sweden|
|    20:29:47|       186.73.69.146|      9076|    PA|            Panama|
|    20:29:47|      200.159.74.194|      7373|    BR|            Brazil|
|    20:29:47|      203.205.35.215|      8446|    VN|           Vietnam|
|    20:29:47|       206.192.81.22|      6868|    US|     United States|
|    20:29:47|       212.50.191.10|       555|    GB|    United Kingdom|
|    20:29:47|      220.135.125.95|     55443|    TW|            Taiwan|
|    20:29:47|      223.197.136.27|      8765|    HK|         Hong Kong|
|    20:29:47|       42.118.144.56|      1971|    VN|           Vietnam|
|    20:29:47|        42.98.213.22|     20107|    HK|         Hong Kong|
|    20:29:47|        45.85.208.74|       255|    PL|            Poland|
|    20:29:47|        50.239.83.38|      3128|    US|     United States|
|    20:29:47|       61.219.165.49|      9030|    TW|            Taiwan|
|    20:29:47|       96.224.240.47|     50443|    US|     United States|
|    20:29:47|        98.6.254.142|      8881|    US|     United States|
|    20:29:48|      101.78.214.242|      9292|    HK|         Hong Kong|
|    20:29:48|        107.0.204.18|     37443|    US|     United States|
|    20:29:48|      109.225.90.172|      7771|    SE|            Sweden|
|    20:29:48|      114.32.175.177|      8564|    TW|            Taiwan|
|    20:29:48|      140.186.62.119|      8181|    US|     United States|
|    20:29:48|       149.75.156.25|     47808|    US|     United States|
|    20:29:48|       151.84.99.133|        86|    IT|             Italy|
|    20:29:48|      167.98.154.219|       445|    GB|    United Kingdom|
|    20:29:48|      173.219.41.139|     40001|    US|     United States|
|    20:29:48|      180.65.213.179|       666|    KR|       South Korea|
|    20:29:48|        184.67.57.42|      5566|    CA|            Canada|
|    20:29:48|     187.122.109.102|      4430|    BR|            Brazil|
|    20:29:48|        190.4.184.51|      9000|    CW|           Curaçao|
|    20:29:48|       201.6.108.105|       168|    BR|            Brazil|
|    20:29:48|       202.21.110.22|     22533|    MN|          Mongolia|
|    20:29:48|      216.224.227.14|       161|    US|     United States|
|    20:29:48|      218.157.37.253|      3522|    KR|       South Korea|
|    20:29:48|       218.161.62.42|      8090|    TW|            Taiwan|
|    20:29:48|        24.106.219.6|     12459|    US|     United States|
|    20:29:48|        24.106.219.6|        79|    US|     United States|
|    20:29:48|        58.38.25.226|      8889|    CN|             China|
|    20:29:48|      61.238.217.162|      1989|    HK|         Hong Kong|
|    20:29:48|      66.245.254.218|      4439|    CA|            Canada|
|    20:29:48|        66.76.57.209|      2020|    US|     United States|
|    20:29:48|        70.27.96.136|     60443|    CA|            Canada|
|    20:29:48|        70.71.86.179|      8180|    CA|            Canada|
|    20:29:48|       84.205.40.144|      3690|    NO|            Norway|
|    20:29:48|       84.247.88.162|      6588|    RO|           Romania|
|    20:29:48|        85.26.26.123|      8343|    BE|           Belgium|
|    20:29:48|      87.242.235.220|      3118|    GB|    United Kingdom|
|    20:29:48|      89.197.228.162|      3443|    GB|    United Kingdom|
|    20:29:48|        89.255.58.72|      8032|    NL|   The Netherlands|
|    20:29:48|        95.67.36.243|        80|    UA|           Ukraine|
|    20:29:48|      98.199.189.159|      9443|    US|     United States|
|    20:29:49|       108.52.245.64|     33333|    US|     United States|
|    20:29:49|     109.241.221.153|      1009|    PL|            Poland|
|    20:29:49|       111.248.97.33|     44300|    TW|            Taiwan|
|    20:29:49|      118.142.38.162|      1024|    HK|         Hong Kong|
|    20:29:49|       154.45.228.44|      4431|    FR|            France|
|    20:29:49|      159.100.64.117|     28443|    NL|   The Netherlands|
|    20:29:49|       162.247.23.26|     44322|    CA|            Canada|
|    20:29:49|      168.195.168.88|      1880|    BR|            Brazil|
|    20:29:49|         171.6.96.64|     51004|    TH|          Thailand|
|    20:29:49|      180.173.61.181|      8042|    CN|             China|
|    20:29:49|      181.104.24.183|      8833|    AR|         Argentina|
|    20:29:49|     186.136.129.168|        99|    AR|         Argentina|
|    20:29:49|       187.33.161.84|     65443|    BR|            Brazil|
|    20:29:49|      200.110.188.58|     43221|    AR|         Argentina|
|    20:29:49|      203.129.25.134|      8020|    AU|         Australia|
|    20:29:49|      203.132.196.87|     10080|    HK|         Hong Kong|
|    20:29:49|      207.136.99.239|      9876|    CA|            Canada|
|    20:29:49|      208.38.251.179|        22|    US|     United States|
|    20:29:49|      210.177.235.39|      8082|    HK|         Hong Kong|
|    20:29:49|       212.82.69.194|      5443|    GB|    United Kingdom|
|    20:29:49|      216.36.168.168|      8003|    CA|            Canada|
|    20:29:49|      217.76.195.244|       443|    UA|           Ukraine|
|    20:29:49|        24.229.57.68|     25000|    US|     United States|
|    20:29:49|         24.65.94.39|       540|    CA|            Canada|
|    20:29:49|       24.76.119.247|     47001|    CA|            Canada|
|    20:29:49|       45.25.125.209|     44443|    US|     United States|
|    20:29:49|        61.216.52.72|     44334|    TW|            Taiwan|
|    20:29:49|       66.209.63.242|      8083|    CA|            Canada|
|    20:29:49|       66.212.63.229|      8887|    LC|       Saint Lucia|
|    20:29:49|        68.113.82.60|      9081|    US|     United States|
|    20:29:49|      70.172.137.135|      3400|    US|     United States|
|    20:29:49|       72.12.122.239|     18081|    US|     United States|
|    20:29:49|        72.139.71.98|      5000|    CA|            Canada|
|    20:29:49|        74.195.87.34|      7000|    US|     United States|
|    20:29:49|        75.138.1.102|      4433|    US|     United States|
|    20:29:49|        78.194.35.58|     32443|    FR|            France|
|    20:29:49|      83.228.108.120|       449|    BG|          Bulgaria|
|    20:29:49|      96.249.219.181|      9877|    US|     United States|
|    20:29:49|       98.165.110.67|      7302|    US|     United States|
|    20:29:49|        98.172.76.20|        81|    US|     United States|
|    20:29:50|        1.36.177.143|     51000|    HK|         Hong Kong|
|    20:29:50|      101.78.214.242|      9292|    HK|         Hong Kong|
|    20:29:50|        107.0.204.18|     37443|    US|     United States|
|    20:29:50|      108.46.137.167|     12345|    US|     United States|
|    20:29:50|      120.28.188.254|      1112|    PH|       Philippines|
|    20:29:50|      170.76.164.246|      6004|    US|     United States|
|    20:29:50|      173.219.41.139|     40001|    US|     United States|
|    20:29:50|      174.86.201.204|      5001|    US|     United States|
|    20:29:50|        184.67.57.42|      5566|    CA|            Canada|
|    20:29:50|      188.155.25.168|      4343|    CH|       Switzerland|
|    20:29:50|        190.4.184.51|      9000|    CW|           Curaçao|
|    20:29:50|        193.37.83.54|      2050|    GB|    United Kingdom|
|    20:29:50|      200.159.74.194|      8085|    BR|            Brazil|
|    20:29:50|       212.50.191.10|       500|    GB|    United Kingdom|
|    20:29:50|      216.224.227.14|       161|    US|     United States|
|    20:29:50|        24.106.219.6|     12459|    US|     United States|
|    20:29:50|        24.106.219.6|        79|    US|     United States|
|    20:29:50|        42.98.213.22|      8051|    HK|         Hong Kong|
|    20:29:50|        45.85.208.74|     10180|    PL|            Poland|
|    20:29:50|       47.206.202.61|      3090|    US|     United States|
|    20:29:50|        50.239.83.38|      8443|    US|     United States|
|    20:29:50|      68.132.165.218|      8087|    US|     United States|
|    20:29:50|       84.205.40.144|      3690|    NO|            Norway|
|    20:29:50|        89.255.58.72|      8032|    NL|   The Netherlands|
|    20:29:50|        95.67.36.243|        80|    UA|           Ukraine|
|    20:29:50|       96.224.240.47|     50443|    US|     United States|
|    20:29:50|        98.6.254.142|      8118|    US|     United States|
|    20:29:51|      109.225.90.172|      9001|    SE|            Sweden|
|    20:29:51|       116.234.1.221|       981|    CN|             China|
|    20:29:51|      118.142.38.162|      1024|    HK|         Hong Kong|
|    20:29:51|      140.186.62.119|      8181|    US|     United States|
|    20:29:51|       149.75.156.25|     47808|    US|     United States|
|    20:29:51|       151.84.99.133|      8086|    IT|             Italy|
|    20:29:51|       154.45.228.44|      4431|    FR|            France|
|    20:29:51|      159.100.64.117|     28443|    NL|   The Netherlands|
|    20:29:51|      180.173.61.181|      8042|    CN|             China|
|    20:29:51|      181.104.24.183|      8833|    AR|         Argentina|
|    20:29:51|     187.122.109.102|      4430|    BR|            Brazil|
|    20:29:51|      187.18.115.195|      8500|    BR|            Brazil|
|    20:29:51|         2.136.0.159|     48888|    ES|             Spain|
|    20:29:51|       201.6.108.105|       168|    BR|            Brazil|
|    20:29:51|       202.21.110.22|      1987|    MN|          Mongolia|
|    20:29:51|      208.186.217.68|        88|    US|     United States|
|    20:29:51|      208.38.251.179|        22|    US|     United States|
|    20:29:51|      216.36.168.168|      8003|    CA|            Canada|
|    20:29:51|      217.76.195.244|       443|    UA|           Ukraine|
|    20:29:51|      218.157.37.253|      3522|    KR|       South Korea|
|    20:29:51|      223.197.136.27|      8097|    HK|         Hong Kong|
|    20:29:51|       42.118.144.56|      9072|    VN|           Vietnam|
|    20:29:51|       45.25.125.209|     44443|    US|     United States|
|    20:29:51|        58.38.25.226|      8889|    CN|             China|
|    20:29:51|       66.209.63.242|      8083|    CA|            Canada|
|    20:29:51|       66.212.63.229|      8887|    LC|       Saint Lucia|
|    20:29:51|      66.245.254.218|      4439|    CA|            Canada|
|    20:29:51|        68.113.82.60|      9081|    US|     United States|
|    20:29:51|        70.27.96.136|     60443|    CA|            Canada|
|    20:29:51|        70.71.86.179|      8180|    CA|            Canada|
|    20:29:51|        72.139.71.98|      5000|    CA|            Canada|
|    20:29:51|        74.195.87.34|      7000|    US|     United States|
|    20:29:51|        75.138.1.102|      4433|    US|     United States|
|    20:29:51|       84.247.88.162|      6588|    RO|           Romania|
|    20:29:51|       85.14.121.150|      2550|    PL|            Poland|
|    20:29:51|        85.26.26.123|      8343|    BE|           Belgium|
|    20:29:51|      87.242.235.220|      9035|    GB|    United Kingdom|
|    20:29:51|        89.255.39.69|      3012|    NL|   The Netherlands|
|    20:29:51|        93.42.231.58|      1080|    IT|             Italy|
|    20:29:51|       98.165.110.67|      7302|    US|     United States|
|    20:29:51|      98.199.189.159|      9443|    US|     United States|
|    20:29:51|       98.98.143.146|      6080|    GB|    United Kingdom|
|    20:29:52|        1.36.177.143|     51000|    HK|         Hong Kong|
|    20:29:52|       103.249.34.94|      4445|    HK|         Hong Kong|
|    20:29:52|      108.46.137.167|     12345|    US|     United States|
|    20:29:52|       108.52.245.64|     10082|    US|     United States|
|    20:29:52|       111.248.97.33|     44300|    TW|            Taiwan|
|    20:29:52|     151.236.206.148|       999|    SE|            Sweden|
|    20:29:52|       162.247.23.26|     44322|    CA|            Canada|
|    20:29:52|      168.195.168.88|      1880|    BR|            Brazil|
|    20:29:52|         171.6.96.64|     51004|    TH|          Thailand|
|    20:29:52|      180.65.213.179|      3333|    KR|       South Korea|
|    20:29:52|     186.136.129.168|        99|    AR|         Argentina|
|    20:29:52|        186.4.231.69|     20000|    EC|           Ecuador|
|    20:29:52|       186.73.69.146|      4444|    PA|            Panama|
|    20:29:52|       187.33.161.84|     65443|    BR|            Brazil|
|    20:29:52|      200.110.188.58|     43221|    AR|         Argentina|
|    20:29:52|      200.159.74.194|      8085|    BR|            Brazil|
|    20:29:52|      202.22.227.179|        83|    NC|     New Caledonia|
|    20:29:52|      203.129.25.134|      8020|    AU|         Australia|
|    20:29:52|      203.205.35.215|      8088|    VN|           Vietnam|
|    20:29:52|      207.136.99.239|      9876|    CA|            Canada|
|    20:29:52|      211.176.84.143|     50051|    KR|       South Korea|
|    20:29:52|       218.161.62.42|      8067|    TW|            Taiwan|
|    20:29:52|      220.135.125.95|      8350|    TW|            Taiwan|
|    20:29:52|        24.229.57.68|     25000|    US|     United States|
|    20:29:52|       24.76.119.247|     47001|    CA|            Canada|
|    20:29:52|       47.206.202.61|      3090|    US|     United States|
|    20:29:52|        50.239.83.38|      8443|    US|     United States|
|    20:29:52|      58.153.207.122|     19443|    HK|         Hong Kong|
|    20:29:52|      61.238.217.162|      8001|    HK|         Hong Kong|
|    20:29:52|      62.232.150.162|     49160|    GB|    United Kingdom|
|    20:29:52|      68.132.165.218|      8087|    US|     United States|
|    20:29:52|      70.172.137.135|      3400|    US|     United States|
|    20:29:52|       72.12.122.239|     18081|    US|     United States|
|    20:29:52|        78.194.35.58|     32443|    FR|            France|
|    20:29:52|      83.228.108.120|       449|    BG|          Bulgaria|
|    20:29:52|      89.197.228.162|     42424|    GB|    United Kingdom|
|    20:29:52|      96.249.219.181|      9877|    US|     United States|
|    20:29:52|       98.150.220.77|      3010|    US|     United States|
|    20:29:52|       98.155.101.80|      1111|    US|     United States|
|    20:29:52|        98.172.76.20|        81|    US|     United States|
|    20:29:53|      114.32.175.177|     55055|    TW|            Taiwan|
|    20:29:53|       116.234.1.221|       981|    CN|             China|
|    20:29:53|      120.28.188.254|      1112|    PH|       Philippines|
|    20:29:53|      14.241.186.234|      2888|    VN|           Vietnam|
|    20:29:53|     150.129.144.147|     15002|    IN|             India|
|    20:29:53|       202.21.110.22|      1987|    MN|          Mongolia|
|    20:29:53|       212.50.191.10|       500|    GB|    United Kingdom|
|    20:29:53|       212.82.69.194|      6443|    GB|    United Kingdom|
|    20:29:53|       42.118.144.56|      9072|    VN|           Vietnam|
|    20:29:53|        42.98.213.22|      8051|    HK|         Hong Kong|
|    20:29:53|        45.85.208.74|     10180|    PL|            Poland|
|    20:29:53|      60.250.219.241|      8043|    TW|            Taiwan|
|    20:29:53|        61.216.52.72|      9095|    TW|            Taiwan|
|    20:29:53|      87.242.235.220|      9035|    GB|    United Kingdom|
|    20:29:53|        95.67.36.243|      9093|    UA|           Ukraine|
|    20:29:53|        98.6.254.142|      8118|    US|     United States|
|    20:29:54|       108.52.245.64|     10082|    US|     United States|
|    20:29:54|      109.225.90.172|      9001|    SE|            Sweden|
|    20:29:54|     151.236.206.148|       999|    SE|            Sweden|
|    20:29:54|       151.84.99.133|      8086|    IT|             Italy|
|    20:29:54|      159.100.64.117|     10081|    NL|   The Netherlands|
|    20:29:54|       186.73.69.146|      4444|    PA|            Panama|
|    20:29:54|         2.136.0.159|     48888|    ES|             Spain|
|    20:29:54|      203.132.196.87|     51003|    HK|         Hong Kong|
|    20:29:54|      208.186.217.68|      3525|    US|     United States|
|    20:29:54|      210.177.235.39|      4443|    HK|         Hong Kong|
|    20:29:54|      223.197.136.27|      8097|    HK|         Hong Kong|
|    20:29:54|      61.238.217.162|      8001|    HK|         Hong Kong|
|    20:29:54|      75.109.210.112|      8080|    US|     United States|
|    20:29:55|       103.249.34.94|      4445|    HK|         Hong Kong|
|    20:29:55|      114.32.175.177|     55055|    TW|            Taiwan|
|    20:29:55|      14.241.186.234|      2888|    VN|           Vietnam|
|    20:29:55|      168.195.168.88|      3550|    BR|            Brazil|
|    20:29:55|      180.65.213.179|      3333|    KR|       South Korea|
|    20:29:55|        186.4.231.69|     20000|    EC|           Ecuador|
|    20:29:55|      200.110.188.58|        82|    AR|         Argentina|
|    20:29:55|       218.161.62.42|      8067|    TW|            Taiwan|
|    20:29:55|      220.135.125.95|      8350|    TW|            Taiwan|
|    20:29:55|       46.164.155.24|      8132|    UA|           Ukraine|
|    20:29:55|       61.219.165.49|      6002|    TW|            Taiwan|
|    20:29:55|      89.197.228.162|     42424|    GB|    United Kingdom|
|    20:29:56|      159.100.64.117|     10081|    NL|   The Netherlands|
|    20:29:56|      203.132.196.87|     51003|    HK|         Hong Kong|
|    20:29:56|      203.205.35.215|      8088|    VN|           Vietnam|
|    20:29:56|      210.177.235.39|      4443|    HK|         Hong Kong|
|    20:29:56|       212.82.69.194|      6443|    GB|    United Kingdom|
|    20:29:56|      60.250.219.241|      2600|    TW|            Taiwan|
|    20:29:56|        61.216.52.72|      9095|    TW|            Taiwan|
|    20:29:56|        95.67.36.243|      9093|    UA|           Ukraine|
|    20:29:57|       116.234.1.221|      1935|    CN|             China|
|    20:29:57|      168.195.168.88|      3550|    BR|            Brazil|
|    20:29:57|      200.110.188.58|        82|    AR|         Argentina|
|    20:29:57|      208.186.217.68|      3525|    US|     United States|
|    20:29:57|       46.164.155.24|      8132|    UA|           Ukraine|
|    20:29:58|       103.249.34.94|     20030|    HK|         Hong Kong|
|    20:29:58|      167.98.154.219|      7070|    GB|    United Kingdom|
|    20:29:58|        186.4.231.69|       543|    EC|           Ecuador|
|    20:29:59|       116.234.1.221|      1935|    CN|             China|
|    20:29:59|      14.241.186.234|       777|    VN|           Vietnam|
|    20:29:59|      60.250.219.241|      2600|    TW|            Taiwan|
|    20:30:00|       103.249.34.94|     20030|    HK|         Hong Kong|
|    20:30:01|      14.241.186.234|       777|    VN|           Vietnam|
|    20:30:01|      167.98.154.219|      7070|    GB|    United Kingdom|
|    20:30:01|        186.4.231.69|       543|    EC|           Ecuador|
|    20:30:01|       46.164.155.24|      8099|    UA|           Ukraine|
|    20:30:03|       46.164.155.24|      8099|    UA|           Ukraine|
|    20:30:13|        59.120.52.26|      8081|    TW|            Taiwan|
|    20:30:16|        59.120.52.26|      8081|    TW|            Taiwan|
|    20:30:20|        59.120.52.26|     10001|    TW|            Taiwan|
|    20:30:23|        59.120.52.26|     10001|    TW|            Taiwan|

[FW log](/images/tcp19000.csv)
