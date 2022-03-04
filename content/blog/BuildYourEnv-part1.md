---
title: "Build your DevOps Environment - Part 1!"
date:  2022-02-08T00:00:00+02:00
draft: true
categories: ["NetApp","Docker","Kubernetes","Trident","DevOps","Azure","GCP","AstraControl"]
banner: /na_devops.png
layout: post
---

Im LinkedIn Post "NetApp in a Cloud DevOps Context" wurde aufgezeigt, wie eine optimierte Kubernetes Umgebung in der Cloud aussehen kann. Diese Artikelserie zeigt nun wie man automatisiert eine solche Umgebung aufbauen kann.

Es gibt bzw. wird vier Teile dieser Serie geben. 
	- Teil 1: Planung
	- Teil 2: Aufbau der benötigten Infrastruktur 
	- Teil 3: Aktivierung des Backups 
	- Teil 4: Monitoring und Optimierung der Umgebung


#Das wichtigste zu erst, die Planung

Für den Aufbau der Testumgebungen werden die Cloud Provider Microsoft Azure und Google Cloud Plattform genutzt. Liegt aber nur daran, dass NetApp's Astra Control Service Amazon Webservice nicht unterstützt. Sobald die Verfügbar wird dies sofort nachgeholt. 
	
Bevor es losgeht, noch ein kurzer Hinweis: Die hier verwendeten Skripte können zu testzwecken benutzt werden. Ich übernehme aber für den Einsatz in Produktiv Umgebungen keine Haftung. Zudem erzeugen diese auch Cloud Ressourcen, die Kosten erzeugen werden! 


Skripte deployen ist einfach. Allerdings muss man sich im Vorfeld Gedanken über verschiedene Punkte machen:
	- Aufbau des Netzwerkes in der Cloud - IP Adressen, Firewall Regeln etc.
	- Zugriffsrechte auf Cloud Ressourcen
	- Lizenzen 


Jeder Cloud Provider hat hier seine eigene Nomenklatur. Aber schauen wir mal
