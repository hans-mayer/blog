---
layout: post
title:  Mein zweites PHP Programm
date:   2025-04-17 18:23:00 CET
categories: 
---

Keine Angst, es folgt nicht jeden Tag ein neuer Blog mit "Mein x-tes PHP Programm". 

Ich habe eine kleine sqlite3 DB, in der ich die Niederschlagswerte eintrage. Das habe ich bis dato mit einem Shell Script gemacht. Begeistert von meinem [ersten Erlebnis](/2025/04/15/mein-erstes-PHP-Progamm.html){:target="_blank"} 
 mit OpenAI und aider.chat möchte ich nun eine Webanwendung dafür machen. 

Ich setze mich also hin und fange an zu beschreiben:
<pre><code>
> Ich benötige ein PHP Programm für eine Webanwendung. Das Programm soll Daten von einer sqlite3 Datenbank lesen und schreiben. Die 
> Datenbank wurde bereits angelegt und das File heißt 'regenmeter.sqlite3'. Darin befindet sich eine Tabelle mit Namen 'regenmeter' 
> und sie wurde so angelegt: CREATE TABLE regenmeter 
>     ^I( id integer primary key, timestamp datetime , menge real ); 
> Die Webanwendung soll am Bildschirm 3 Bereiche darstellen. Der oberste Bereich 'BlockA' soll die letzten 5 Einträge auflisten sort
> iert nach dem timestamp. Im mittleren Bereich 'BlockB' soll die Summe über 'menge' gruppiert über die letzten 5 Monate dargestellt
>  werden. Im unteren Bereich 'BlockC' soll eine Eingabemöglichkeit bestehen. Es soll 6 Eingabefelder geben für Tag, Monat, Jahr, St
> unde, Minute und für 'menge'. Für Tag, Monat, Jahr, Stunde und Minute sollen die aktuellen werte vorausgefüllt sein. Es soll aber 
> möglich sein, sie zu überschreiben.
</code></pre>

Ich erhalte ein File "webapp.php" das ich auf den Webserver als index.php ablege. Und so sieht das Ergebnis aus, wenn man mit einem Browser darauf zugreift: 

![erster versuch](/images/regenmeter1.png)

Sieht ja schon mal gut aus. Allerdings funktionierte die Eingabe nicht. Was ich nicht bedacht habe, ist die Tatsache, dass der PDO Treiber Schreibrecht auf das File UND auf das Directory haben will. Ich erstelle also ein Unterverzeichnis, verschieben die DB und passe die Berechtigungen an. Außerdem gefällt mir der Aufbau nicht ganz. Das teile ich aider.chat mit:

<pre><code>
> Die Überschrift in BlockB soll lauten "Regen in den letzten 5 Monaten". Das Datenbankfile 'regenmeter.sqlite3' soll sich im Unterv
> erzeichnis 'db' befinden. Die Eingabe in BlockC soll geändert werden. Tag, Monat, Jahr soll in einer Zeile sein. Stunde, Minute un
> d menge in der nächsten Zeile. Der Button "Eintrag hinzufügen" soll sich darunter befinden.                                       
</code></pre>

File wieder kopiert und getestet:

![zweiter versuch](/images/regenmeter2.png)

Ja, und es funktioniert auch die Eingabe. Ich habe dann noch ein paar Kleinigkeiten ändern lassen, damit es auch auf Smartphones gut lesbar ist. Aber im Prinzip hat es mit 2 Anweisungen funktioniert. 

Insgesamt habe ich dafür ca 1 1/2 Stunden gebraucht. Wenn ich mich erst in PHP einlesen hätte müssen, um so ein Programm zu schreiben, hätte ich ein Vielfaches der Zeit dafür benötigt. 

Und was hat es gekostet ? Wieder 0.01 USD abgebucht bis zum zweiten Versuch, final dann 0.02 USD. 

Einziger Wermutstropfen, es sind nicht die letzten 5 Monate, sondern die letzten 6. Aber den Fehler hätte ich wahrscheinlich auch gemacht, wenn ich mir das SQL Statement überlegt hätte, das so aussieht: 

`SELECT strftime("%Y-%m", timestamp) as month, SUM(menge) as total FROM regenmeter WHERE timestamp >= datetime("now", "-5 months") GROUP BY month ; `

Aber das wollte ich dann nicht mehr ändern, denn ein halbes Jahr Übersicht ist auch OK. Man könnte natürlich auch die Headerline anpassen. Aber diese "aufwändig geschriebene" App ist nicht öffentlich verfügbar und sie wird ohnehin nur von mir verwendet. 

[Link zu meinem ersten PHP Programm](/2025/04/15/mein-erstes-PHP-Progamm.html){:target="_blank"} 
