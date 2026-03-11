# Timberwolf Server Kanon – Index

Dieser Index ist die Navigations- und Referenzstruktur des Timberwolf-Kanons.

Der ursprüngliche Kanon bestand aus 27 Einzeldateien.
Diese wurden thematisch konsolidiert und werden künftig in folgende Hauptbereiche gegliedert:

01_TWS_System_Model  
02_Custom_Logic_Syntax  
03_Module_Reference  
04_Design_Patterns  
05_Anti_Patterns  
06_Hardening_Rules

Dieser Index dient drei Zwecken:

1. Navigation innerhalb des Kanons
2. Zuordnung von Themen zu Kanon-Dateien
3. Retrieval-Anker für AI-Systeme

Alle technischen Definitionen befinden sich in den jeweiligen Fachdateien.

---

# Struktur des Kanons

## 01_TWS_System_Model

Beschreibung des grundlegenden Funktionsmodells des Timberwolf Servers.

Enthält:

- Architekturprinzipien
- Systemkomponenten
- Denkmodell der Automationslogik
- Objektmodell
- Kommunikationsprinzipien
- Integration externer Systeme

Typische Themen:

Timberwolf Architektur  
Logic Engine  
Objekt-System  
Zeit-Management  
MQTT Integration  
HTTP / REST Integration  
KNX Integration  
Modus-System  
Visualisierungssystem

---

## 02_Custom_Logic_Syntax

Formale Struktur und Syntaxregeln für Custom Logic.

Enthält:

- Struktur einer Logikzelle
- Pflichtfelder
- Reihenfolge der Schlüssel
- Datentypen
- Triggermechanismen
- Cron-Syntax
- Parserregeln
- Speicher-Validierungsregeln

Typische Themen:

Custom Logic Struktur  
Input / Output Definition  
Module Definition  
Level-Struktur  
JSON-Struktur  
Parserverhalten  
Speicherfehler  
Logik-Editor Verhalten

Wichtige Prinzipien:

Dogma der kanonischen Reihenfolge  
Speicher-Validierungs-Falle  
Editor vs. Speicherprüfung

---

## 03_Module_Reference

Technische Referenz aller verwendeten Module.

Jedes Modul wird mit folgenden Informationen beschrieben:

- Modulname
- Funktion
- Inputs
- Parameter
- Outputs
- Datentypen
- typisches Verhalten

Diese Datei dient als primäres Nachschlagewerk für:

Module  
Parameterdefinitionen  
Input / Output Struktur

---

## 04_Design_Patterns

Bewährte Architektur- und Logikmuster für stabile Automationen.

Enthält:

- typische Logikarchitekturen
- Zustandsmodelle
- Sequenzsteuerungen
- Zeitsteuerungen
- Eventaggregation

Typische Patterns:

Input-Gehirn-Muskeln-Pattern  
Zustandsautomat  
Sequencer  
Zeitsteuerung  
Aggregation mehrerer Trigger

Wichtige Prinzipien:

Schnecke & Rennwagen Prinzip  
Trigger-Konsolidierung  
Update-Trigger Strategie

---

## 05_Anti_Patterns

Bekannte Fehlermuster und instabile Logikstrukturen.

Enthält:

- typische Fehlerquellen
- instabile Logikdesigns
- Triggerprobleme
- Race Conditions
- unkontrollierte Kaskaden

Typische Anti-Patterns:

Mehrfach-Trigger durch gleiche Objekte  
instabile Zirkelbezüge  
Trigger-Stürme  
Race Conditions

Diese Datei beschreibt:

Problem  
Symptom  
Ursache  
empfohlene Alternative

---

## 06_Hardening_Rules

Verbindliche Regeln zur Erstellung stabiler Logiken.

Diese Regeln basieren auf:

Praxis-Erfahrung  
Fehleranalysen  
Systemverhalten des Timberwolf Servers

Beispiele:

Präzedenzfall-Dogma  
Praxis schlägt Dokumentation  
keine geratenen Modulparameter  
keine hypothetische Syntax  
keine instabilen Triggerarchitekturen

Diese Regeln haben Vorrang vor:

Design Patterns  
Allgemeinem Fachwissen  
externer Dokumentation

---

## 07_Dynamic_Hardening (reserved)

Dieser Bereich ist für zukünftige Erweiterungen des Kanons vorgesehen.

Er enthält dynamische Härtungsregeln, die aus realen Systemanalysen,
Fehlerfällen und neuen Praxisbeobachtungen entstehen.

Geplante Inhalte:

- neue Hardening-Regeln
- systematische Fehleranalysen
- Erweiterungen bestehender Anti-Patterns
- neue Stabilitätsregeln für Logikarchitekturen

Quellen:

Kanon-Dateien 28–29 (Dynamic Hardening).

Hinweis:

Solange diese Dateien nicht im Knowledge-System vorhanden sind,
dürfen keine Regeln aus diesem Bereich angenommen oder rekonstruiert werden.

---

## 08_TWS_Official_Documentation (reserved)

Dieser Bereich enthält die offizielle technische Dokumentation
des Timberwolf Servers und seiner Subsysteme.

Geplante Inhalte:

- vollständige Systemdokumentation
- Beschreibung aller Subsysteme
- Schnittstellenbeschreibungen
- interne Systemkomponenten
- Firmware-Funktionen

Typische Themen:

KNX Stack  
MQTT System  
HTTP / REST API  
Modbus Integration  
Zeitfunktionen  
Visu Engine  
Netzwerkfunktionen  
Subsystemarchitektur

Quellen:

Kanon-Dateien 30–83 (offizielle Systemdokumentation).

Hinweis:

Diese Dokumentation ergänzt den Kanon,
ersetzt aber nicht die Hardening-Regeln oder Anti-Patterns.



# Schlüsselprinzipien des Kanons

Der Timberwolf-Kanon basiert auf realen Tests und Praxisbeobachtungen.

Die Hierarchie der Wahrheit lautet:

1. bestätigte Praxis-Erfahrung
2. Kanon-Regeln
3. externe Dokumentation

---

# Typische Suchbegriffe

Diese Begriffe dienen als Retrieval-Anker für AI-Systeme.

Cron  
Comparator  
Trigger  
Race Condition  
Logik-Zelle  
JSON-Struktur  
Parser  
KNX Trigger  
MQTT Integration  
HTTP Integration  
Sequencer  
Zustandsautomat  
Timer  
Aggregation

---

# Verwendung des Kanons

Bei technischen Fragen arbeitet ein AI-System in folgender Reihenfolge:

1. Suche im Kanon
2. Verwende kanonische Definitionen
3. Nur wenn keine Definition existiert: allgemeines Fachwissen

Wenn eine Moduldefinition nicht eindeutig im Kanon vorhanden ist:

- keine Parameter raten
- keine Syntax erfinden
- Benutzer um Referenz bitten

---

# Ziel des Kanons

Der Kanon soll sicherstellen, dass:

Custom Logic

- syntaktisch korrekt
- speicherbar
- stabil
- reproduzierbar

erstellt werden kann.