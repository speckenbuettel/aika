\# Timberwolf Server – Modulreferenz



Diese Datei enthält die \*\*kanonische Referenz aller validierten Kernmodule\*\* der Timberwolf Custom Logic.



Zweck dieser Referenz:



\- eindeutige Modulsyntax

\- Dokumentation der Parameterstruktur

\- Beschreibung kritischer Systemregeln

\- Referenz für AI-Systeme



Diese Datei ist \*\*die maßgebliche Quelle für Modulsyntax und Modulparameter\*\*.



Wenn ein Modul hier definiert ist, darf \*\*keine alternative Syntax verwendet werden\*\*.



---



\# Modulstruktur



Jedes Modul ist im Kanon wie folgt beschrieben:



Module Name  

Funktion  

Syntax  

kritische Regeln  

Härtungsregeln  

Hinweise



Die Syntaxzeile ist \*\*verbindlich\*\*.



---



\# Polynomial



\## Zweck



Berechnet eine Polynomfunktion:



Y = A0 + A1·X + A2·X² ...



Dieses Modul dient als universeller mathematischer Baustein.



Typische Anwendungen:



\- Addition

\- Multiplikation

\- Differenzbildung

\- lineare Skalierung



\## Syntax



\[ "Polynomial", "Input\_X", "Output\_Y", \[ "$A0", "$A1", ... ] ]



oder



\[ "Polynomial", "Input\_X", "Output\_Y", \[ "$VAR<Koeff!>" ] ]



\## Härtungsregeln



ANTI-MAGIC-NUMBERS



Numerische Literale in Koeffizientenarrays sind verboten.



Auch Werte wie:



0  

1  

-1



müssen als Variablen definiert werden.



---



\# Addition



Addition wird über das Polynomial-Modul implementiert.



Y = X + A0



\## Syntax



\[ "Polynomial", "Summand1\_X", "Summe\_Y", \[ "$Summand2\_A0", "$Konst\_1\_Float" ] ]



---



\# Multiplikation



Multiplikation wird über Polynomial emuliert.



\## Syntax



\[ "Polynomial", "FaktorA", "Produkt", \[ "$Konst\_0\_Float", "$FaktorB" ] ]



---



\# Subtraktion



Subtraktion wird über zwei Schritte realisiert:



1\. Negation

2\. Addition



Direkte Integration negativer Werte im Polynomial ist verboten.



---



\# Ratio (Division)



\## Funktion



Division:



Ergebnis = Zaehler / Nenner



\## Syntax



\[ "Ratio", "Zaehler", "Ergebnis", "$Nenner" ]



\## Kritische Regel



Division durch Null muss verhindert werden.



Empfohlene Absicherung:



Limiter auf Nenner:



>= 0.001



---



\# Limiter



\## Funktion



Begrenzt einen Wert auf ein Minimum und Maximum.



\## Syntax



\[ "Limiter", "Input", "Out\_Clamped", "Out\_IsInRange", \[ "$Min", "$Max" ] ]



---



\# And / Or / Xor



Logische Gatter.



\## Syntax



\[ "Or", \[ "In\_A", "In\_B", ... ], "Out" ]



oder



\[ "Or", \[ "$VAR<In!>" ], "Out" ]



Analog:



And  

Xor



\## Regel



Alle Inputs müssen vom Typ \*\*bool\*\* sein.



---



\# Comparator



Schwellwertschalter.



\## Syntax



\[ "Comparator", "A", "Out", "B" ]



Out wird TRUE wenn



A > B



\## Hysterese



\[ "Comparator", "A", "Out", \[ "$Schwelle\_Aus", "$Schwelle\_Ein" ] ]



---



\# Clocksignal



Taktsignal.



\## Syntax



\[ "Clocksignal", "Enable", "Output\_Clock", "$Perioden\_Halbe\_s" ]



Besitzt implizite Trigger-Eigenschaft.



---



\# Monoflop



Timer-Modul.



\## Syntax



\[ "Monoflop", "Set", "Reset", "Out", "Zeit\_s", Mode\_Int\_Literal ]



Mode bestimmt:



\- Pegel

\- Flanke

\- Retrigger



---



\# Latch



Speichert Werte.



\## Syntax



\[ "Latch", "Input", "Output", "Trigger", Mode\_Int\_Literal ]



Modes:



0 Pegel  

1 steigende Flanke  

2 fallende Flanke  

3 beide Flanken



---



\# Multiplexer



Wählt einen Wert aus einer Liste.



\## Syntax



\[ "Multiplexer", \[ "$Wert\_0", "$Wert\_1", ...], "Output", "Selektor\_Int" ]



oder



\[ "Multiplexer", \[ "$VAR<In!>" ], "Output", "Selektor\_Int" ]



---



\# JsonPathSelector



Extrahiert Daten aus JSON.



\## Syntax



\[ "JsonPathSelector", "Input\_Json", "JsonPath", "Output", "Error" ]



---



\# HEX->INT



Konvertiert Hex-String in Integer.



\## Syntax



\[ "HEX->INT", "Input\_Hex\_String", "Output\_Integer", "$ByteSwap\_Bool", "$WordSwap\_Bool" ]



---



\# STR->FLOAT / STR->INT



Konvertiert Strings in Zahlen.



\## Syntax



\[ "STR->FLOAT", "Input\_String", "Output\_Float" ]



---



\# Printf



Formatiert Strings.



\## Syntax



\[ "Printf", "Input\_Wert", "Format\_String", "Output\_String" ]



---



\# Regex



Wendet reguläre Ausdrücke auf Strings an.



\## Syntax



\[ "Regex", "Input\_Str", "Pattern\_Str", "Match\_Bool", "Full\_Match", "Grp1", "Grp2", "Grp3", "Grp4", "$Grp5" ]



---



\# Stopwatch



Misst Laufzeit.



\## Syntax



\[ "Stopwatch", "StartStop\_Bool", "Time\_s\_Float" ]



---



\# PID controller



\## Syntax



\[ "PID controller", "Soll\_float", "Ist\_float", "Stell\_float", "Kp\_float", "Ki\_float", "Kd\_float" ]



---



\# PID\_awu



PID mit Anti-Windup.



\## Syntax



\[ "PID\_awu", "Soll\_f", "Ist\_f", "Stell\_f", "Kp\_f", "Ki\_f", "Kd\_f", "Untergrenze\_f", "Obergrenze\_f" ]



---



\# Statemachine



Zustandsautomat.



\## Syntax



\[ "Statemachine", \[ \[Transition1], \[Transition2], ... ], "Output\_State\_Int"]



---



\# Statistic



Statistikmodul.



\## Syntax



\[ "Statistic", \[ "Input1", ... ], "Out\_Min", "Out\_Max", "Out\_Mean", "$Out\_Median" ]



---



\# Ramp



Begrenzt Änderungsrate.



\## Syntax



\["Ramp", "$Ziel", "$Out", "$Active", "$Step", "$Period"]



---



\# Astro



Astronomische Berechnungen.



\## Syntax



\[ "Astro", \[ "$Latitude", "$Longitude" ], "Altitude", "Azimute", "Transit", "Sunrise", "Sunset", "Civildawn", ... ]



---



\# Localtime



Zeitstempel-Konverter.



\## Syntax



\[ "Localtime", 0, "UnixOut", "Sec\_Out", "Min\_Out", "Hour\_Out", ... ]



---



\# Wakeup



Einmaliger Timer.



\## Syntax



\[ "Wakeup", "UnixTimestamp\_In", "Alarm\_Out\_Bool" ]



---



\# SendToSimple



Sendet Nachrichten.



\## Syntax



\[ "SendToSimple", "Inhibit", "Channel", "Title", "Message", "Category", "Priority" ]



---



\# SendExplicit



Erzwingt Sendeaktion.



\## Syntax



\[ "SendExplicit", "Sende\_Bedingung\_Bool", "Variable\_mit\_Wert", Sende\_Option\_Int\_Literal ]



---



\# Triggered



Identifiziert Triggerquelle.



\## Syntax



\[ "Triggered", "Input", "Touched\_Output\_Bool" ]



---



\# Interpolation



Berechnet Werte zwischen Stützpunkten.



\## Syntax



\[ "Interpolation", "In\_X", "Out\_Y", \[ \["$X1", "$Y1"], \["$X2", "$Y2"], ... ] ]



---



\# Lowpass



Digitaler Tiefpassfilter.



\## Syntax



\[ "Lowpass", "Input", "Output", "$Zeitkonstante\_s" ]



---



\# SingleDigitCounter



Histogramm für Werte 0-9.



\## Syntax



\["SingleDigitCounter", \["$In1", "$In2"...], "$Cnt0", "$Cnt1", ..., "$Cnt9"]



---



\# Dim-Aktor



Relativ → Absolut Dimmer.



\## Syntax



\["Dim-Aktor", "$In\_Relativ", "$Out\_Absolut", "$Inhibit"]



---



\# Verschluss-Status / Verschluss-Statistik



Fensterstatusmodule.



\## Syntax



\["Verschluss-Status", "$Kipp", "$Offen", "$StatusInt", "$OffenBool", "$Text"]



\["Verschluss-Statistik", \["$StatusInt1", ...], "$AnzahlOffen", "$AnzahlGekippt", "$AllesZu"]

