# Timberwolf Server -- Custom Logic Syntax

Diese Datei beschreibt die **formale Syntax der Custom Logic im
Timberwolf Server (TWS)**. Sie definiert, wie eine Logik strukturiert
sein muss, damit sie im Logik‑Editor korrekt gespeichert und ausgeführt
werden kann.

Diese Regeln sind verbindlich für:

-   Aufbau einer Logik
-   Modulsyntax
-   Triggerverhalten
-   Datentypen
-   Cron‑Ausdrücke
-   Parser‑ und Speicherregeln

Wenn eine Logik von diesen Regeln abweicht, kann sie im Logik‑Editor
nicht gespeichert werden oder führt zu instabilem Verhalten.

------------------------------------------------------------------------

# Grundstruktur einer Custom Logic

Eine Custom Logic besteht aus einer **Liste von Modulen**, die
nacheinander ausgeführt werden.

Allgemeine Modulstruktur:

    [ "ModuleName", Input1, Input2, ..., Output, Parameter ]

Typische Elemente:

-   **ModuleName** -- Name des Moduls
-   **Inputs** -- Eingangsobjekte oder Variablen
-   **Outputs** -- Zielobjekte oder Variablen
-   **Parameter** -- Konstanten oder Variablen

Die Reihenfolge der Elemente ist verbindlich.

------------------------------------------------------------------------

# Variablen und Konstanten

## Variablen

Variablen werden mit `$` gekennzeichnet.

Beispiele:

    "$Temperatur"
    "$Sollwert"
    "$Zeit_s"

Variablen können aus folgenden Quellen stammen:

-   Objektwerte
-   vorherige Module
-   Systemvariablen

------------------------------------------------------------------------

## Literale

Konstanten dürfen als Literale angegeben werden, z. B.

    0
    1
    3.14

Best Practice:

Viele Hardening‑Regeln verlangen jedoch, dass Konstanten als Variablen
definiert werden (ANTI‑MAGIC‑NUMBERS).

------------------------------------------------------------------------

# Datentypen

Typische Datentypen im TWS:

  Typ      Beschreibung
  -------- ----------------
  bool     Wahr / Falsch
  int      Ganzzahl
  float    Gleitkommazahl
  string   Text
  json     JSON‑Objekt

Module erwarten spezifische Datentypen. Falsche Typen können zu
Laufzeitfehlern führen.

------------------------------------------------------------------------

# Triggermechanismus

Die Ausführung einer Logik wird durch **Trigger** ausgelöst.

Typische Triggerquellen:

-   Änderung eines Objektwerts
-   Zeitereignisse
-   externe Ereignisse (MQTT, KNX, HTTP)

Ein Modul kann intern ebenfalls Trigger erzeugen.

Beispiele:

-   Clocksignal
-   Triggered
-   Monoflop

------------------------------------------------------------------------

# Trigger‑Erkennung

Das Modul

    Triggered

kann verwendet werden, um zu erkennen, welches Objekt eine Logik
ausgelöst hat.

Syntax:

    [ "Triggered", "Input", "Touched_Output_Bool" ]

------------------------------------------------------------------------

# Cron‑Syntax im Timberwolf Server

Der Timberwolf Server verwendet eine **6‑feldrige Cron‑Syntax**.

Struktur:

    Sekunde Minute Stunde Tag Monat Wochentag

Beispiel:

    0 0 8 * * 1

Bedeutung:

Montag um 08:00:00

------------------------------------------------------------------------

## Operatoren

  Symbol   Bedeutung
  -------- -----------------
  \*       beliebiger Wert
  ,        Liste
  \-       Bereich
  /        Schrittweite

Beispiel:

    0/10 * * * * *

Alle 10 Sekunden.

------------------------------------------------------------------------

# JSON‑Verarbeitung

JSON‑Daten können mit Modulen verarbeitet werden.

Typisches Modul:

    JsonPathSelector

Syntax:

    [ "JsonPathSelector", "Input_Json", "JsonPath", "Output", "Error" ]

Dieses Modul extrahiert Daten aus JSON‑Strukturen.

------------------------------------------------------------------------

# String‑Formatierung

Strings können mit dem Modul

    Printf

formatiert werden.

Syntax:

    [ "Printf", "Input_Wert", "Format_String", "Output_String" ]

------------------------------------------------------------------------

# Reguläre Ausdrücke

Regex‑Operationen werden mit folgendem Modul durchgeführt:

    Regex

Syntax:

    [ "Regex", "Input_Str", "Pattern_Str", "Match_Bool", "Full_Match", "Grp1", "Grp2", "Grp3", "Grp4", "$Grp5" ]

------------------------------------------------------------------------

# Zeitmodule

Der TWS enthält mehrere Module zur Zeitverarbeitung.

Beispiele:

## Localtime

    [ "Localtime", 0, "UnixOut", "Sec_Out", "Min_Out", "Hour_Out" ]

## Wakeup

    [ "Wakeup", "UnixTimestamp_In", "Alarm_Out_Bool" ]

------------------------------------------------------------------------

# Parser‑Regeln

Der Logik‑Parser prüft beim Speichern:

1.  korrekte Modulnamen
2.  korrekte Parameteranzahl
3.  gültige Datentypen
4.  syntaktisch korrekte Struktur

Typische Fehler:

-   falsche Parameterreihenfolge
-   fehlende Anführungszeichen
-   falsche Klammern
-   ungültiger Modulname

------------------------------------------------------------------------

# Speicher‑Validierung im Logik‑Editor

Der Logik‑Editor führt beim Speichern mehrere Prüfungen aus.

Prüfungen:

-   JSON‑Struktur
-   Modulsyntax
-   Parameteranzahl
-   Typkompatibilität

Wenn eine Prüfung fehlschlägt, kann die Logik nicht gespeichert werden.

------------------------------------------------------------------------

# Best Practices

Für stabile Logiken gelten folgende Empfehlungen:

-   klare Modulstruktur
-   keine Magic Numbers
-   konsistente Datentypen
-   saubere Triggerstruktur
-   Vermeidung von Race Conditions

Diese Best Practices werden in den Dateien

-   Design_Patterns
-   Anti_Patterns
-   Hardening_Rules

weiter vertieft.

------------------------------------------------------------------------

# Zusammenhang mit anderen Kanon‑Dateien

Diese Datei beschreibt **nur die formale Syntax**.

Weitere Informationen befinden sich in:

-   03_Module_Reference → Moduldefinitionen
-   04_Design_Patterns → Architektur‑Patterns
-   05_Anti_Patterns → bekannte Fehlerstrukturen
-   06_Hardening_Rules → verbindliche Regeln
