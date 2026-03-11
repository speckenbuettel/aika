# Design Patterns

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_21.txt ---

// ==========================================================================
// 4. DESIGN PATTERNS, BEST PRACTICES
// ==========================================================================

// --- Adaptiver Schwellwert (Baseline-Pattern) ---
// Muster: Erzeugt einen dynamischen Schwellwert, der sich an einen langsam ändernden Grundwert ("Baseline") anpasst.
// Dies ist die kanonische Methode, um auf signifikante Abweichungen von einem Normalzustand zu reagieren,
// ohne durch feste Schwellen bei langsamen Drifts (z.B. saisonale Temperaturänderungen) Fehlauslösungen zu erzeugen.
// Prinzip:
// 1. Baseline-Bildung: Ein Lowpass-Filter mit einer hohen Zeitkonstante glättet den Eingangswert ($P_Istwert) und erzeugt so einen trägen, gleitenden Mittelwert ($State_Baseline).
// 2. Schwellwert-Berechnung: Ein Polynomial-Modulbaustein (oder CalcFormula) addiert einen festen Offset ($P_Offset) auf die Baseline, um den eigentlichen Schaltschwellwert ($Lgc_Schwellwert) zu berechnen.
// 3. Vergleich: Ein Comparator vergleicht den aktuellen Ist-Wert mit dem dynamisch berechneten Schwellwert.
Pattern:
// [ "Lowpass", "P_Istwert", "State_Baseline", "P_Zeitkonstante_s" ],
// [ "Polynomial", "State_Baseline", "Lgc_Schwellwert", [ "$P_Offset", "$Konst_1_Float" ] ],
// [ "Comparator", "P_Istwert", "O_Alarm", "Lgc_Schwellwert" ]

// --- Aktor-Rückmeldungs-Validierung (Regelkreis-Integrität) ---
// **Muster:** Überwacht die tatsächliche Ausführung eines Aktor-Befehls durch dessen Rückmeldung (Status, Drehzahl, Leistung) und reagiert bei Abweichungen vom Soll-Zustand.
// **Prinzip:** Vergleicht den von der Logik gesendeten Soll-Befehl mit der Ist-Rückmeldung des Aktors. Bei einer signifikanten oder zeitlich verzögerten Abweichung wird ein Fehlerzustand ausgelöst oder eine alternative Strategie eingeleitet.
// **KORREKTES PATTERN (Beispiel):**
/*
// Level-Block (Ausschnitt)
// ["$O_Pumpe_Soll", "bool", false],
// ["$P_Pumpe_Ist_Status", "bool", false], // Rückmeldung der Pumpe
// ["$Lgc_Pumpe_Fehler", "bool", false],
// ["$Konst_Timeout_Pumpe_s", "float", 10.0], // Max. Zeit bis Rückmeldung kommt
// ["$Konst_False", "bool", false],

// Module-Block (Ausschnitt)
// Prüfe, ob Pumpe laufen soll, aber nicht läuft (oder umgekehrt)
["Xor", ["$O_Pumpe_Soll", "$P_Pumpe_Ist_Status"], "$Lgc_Pumpe_Soll_Ist_Abweichung"],
// Starte Timer, wenn Abweichung besteht
["Monoflop", "$Lgc_Pumpe_Soll_Ist_Abweichung", "$Konst_False", "$Lgc_Pumpe_Fehler_Timer", "$Konst_Timeout_Pumpe_s", 0],
// Setze Fehler-Flag, wenn Timer abläuft
["Triggered", "-$Lgc_Pumpe_Fehler_Timer", "$Lgc_Pumpe_Fehler"]
*/

// --- Aktor-Zwangslaufzeit (Mindestbetriebsdauer) ---
// **Muster:** Stellt sicher, dass ein Aktor nach dem Einschalten für eine definierte Mindestdauer aktiv bleibt, selbst wenn die ursprüngliche Startbedingung nicht mehr erfüllt ist oder eine höhere Priorität eine Abschaltung fordern würde.
// **Prinzip:** Ein `Monoflop` (Mode 0 oder 2, nicht retriggerbar) wird beim Einschalten des Aktors gestartet. Solange dieser `Monoflop` läuft, wird die Abschaltbedingung des Aktors ignoriert oder übersteuert. Erst nach Ablauf des `Monoflop`s kann der Aktor wieder abgeschaltet werden.
// **KORREKTES PATTERN (Beispiel):**
/*
// Level-Block (Ausschnitt)
// ["$P_Aktor_Soll_Ein", "bool", false], // Soll-Befehl von der Logik
// ["$P_Aktor_Mindestlaufzeit_s", "float", 300.0], // 5 Minuten
// ["$State_Aktor_Zwangslauf_Aktiv", "bool", false], // Monoflop-Ausgang
// ["$O_Aktor_Final_Ein", "bool", false], // Finaler Aktor-Output
// ["$Konst_False", "bool", false],
// ["$Konst_True", "bool", true],

// Module-Block (Ausschnitt)
// Starte Zwangslauf-Timer bei Aktor-Einschalten (steigende Flanke)
["Monoflop", "$P_Aktor_Soll_Ein", "$Konst_False", "$State_Aktor_Zwangslauf_Aktiv", "$P_Aktor_Mindestlaufzeit_s", 2],
// Aktor bleibt an, wenn Soll-Ein ODER Zwangslauf aktiv
["Or", ["$P_Aktor_Soll_Ein", "$State_Aktor_Zwangslauf_Aktiv"], "$O_Aktor_Final_Ein"]
*/

// --- Architektonische Entkopplung (Das Zwei-Zellen-Prinzip) ---
// Muster: Dies ist die einzig validierte Methode zur Implementierung von stabilen,
// geschlossenen Regelkreisen und zur Auflösung des "Geschlossenen Regelkreis-Paradoxons".
// Prinzip: Anstatt die gesamte Logik in einer einzigen, komplexen Datei zu bündeln,
// wird die Verantwortung physisch auf zwei (oder mehr) separate Logik-Zellen aufgeteilt.
//
// Zelle A - "Der Wächter":
// - Verantwortung: Überwacht einen Zustand und meldet ein Ereignis.
// - Input: Der zu überwachende Zustand (z.B. der Ausgang einer anderen Logik-Zelle).
// - Logik: Minimalistisch, oft nur ein Monoflop (z.B. für eine Inaktivitäts-Prüfung).
// - Output: Ein einfaches Signal (z.B. ein boolescher Impuls) auf ein separates Objekt.
//
// Zelle B - "Die Steuerung":
// - Verantwortung: Führt die eigentliche Steuerungslogik aus.
// - Input: Nimmt das Signal von Zelle A als einen von vielen normalen Eingängen entgegen.
// - Logik: Enthält die komplexe Entscheidungsfindung.
// - Output: Steuert den Aktor.
//
// Ergebnis: Der logische Zirkelbezug wird aufgebrochen, da die Validierungs-Engine
// jede Zelle einzeln als lineare, stabile Logik bewertet. Der Regelkreis wird
// physisch über das Objektsystem geschlossen, was vom Server erlaubt wird.

// **HÄRTUNG (PATTERN: CIRCADIAN MASTER / Astro-HCL) [NEU V7.02.04]:**
// Prinzip: Nutzung der Sonnen-Elevation anstatt fester Uhrzeiten für die Lichtsteuerung.
// Vorteil: Automatische Anpassung an Sommer/Winter ohne manuelle Korrektur.
// Bausteine: Astro -> Interpolation -> Lowpass (als „digitales Schleifpapier“ für sanfte Übergänge).

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_21.txt ---

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_22.txt ---

// --- Automatischer Impuls-Reset (Multiplexer-Selbst-Reset) ---
// Muster: Setzt eine boolesche Impuls-Variable (z.B. einen von Cron ausgelösten Trigger)
// im selben Zyklus wieder auf 'false' zurück, in dem sie auf 'true' gesetzt wurde.
// Dies ist die kanonische Methode, um einen sauberen, nur einen Zyklus andauernden
// Impuls zu erzeugen und zu verarbeiten.
// Prinzip: Ein Multiplexer wird so konfiguriert, dass er sich selbst überschreibt.
// Wenn die Variable ($Impuls) 'false' (0) ist, wählt sie sich selbst als neuen Wert aus.
// Wenn die Variable 'true' (1) ist, wählt sie eine Konstante mit 'false' als neuen Wert.
// Da der ursprüngliche 'true'-Wert im selben Zyklus noch für andere Logiken
// verwendet werden kann (Regel 1.12 beachten), wird der Reset effektiv am Ende des
// Zyklus durchgeführt.
// Validiertes Pattern aus dem Betriebsstundenzähler:
Pattern: [ "Multiplexer", [ "$Impuls", "$Konst_False" ], "$Impuls", "$Impuls" ]

// --- Automatischer Trigger-Reset (Taster-Emulations-Pattern) ---
// Muster: Behandelt einen booleschen Eingang (z.B. von einem KNX-Taster) wie einen einmaligen Impuls, indem er nach der Verarbeitung sofort auf false zurückgesetzt wird.
// Dies verhindert, dass der Eingang über mehrere Zyklen true bleibt und die Logik-Zelle mehrfach auslöst, selbst wenn der Eingang auf Trigger "c" (on change) steht.
// Prinzip: Ein Latch-Modulbaustein, der am Ende des Module-Blocks platziert wird, überschreibt den Wert der Eingangsvariablen mit 'false'.
// Da dies am Ende der Logik-Abarbeitung geschieht, kann der ursprüngliche 'true'-Wert im selben Zyklus noch für die Flankenerkennung oder andere Bedingungen verwendet werden. Im nächsten Zyklus startet die Logik-Zelle garantiert mit einem 'false' auf diesem Eingang.
// Voraussetzung: Variablen $Konst_False (bool, false) und $Konst_True (bool, true) müssen deklariert sein.
Pattern: [ "Latch", "$Konst_False", "$P_Trigger_In", "$Konst_True", 0 ] // AM ENDE des Module-Blocks platzieren!

// --- Batterie-Verlustberechnung und Bilanzierung ---
// **Muster:** Berücksichtigt die physikalischen Umwandlungsverluste beim Laden und Entladen von Batteriespeichern, um eine realistische Energiebilanz zu erstellen.
// **Prinzip:** Die gemessenen Lade- und Entlademengen werden mit spezifischen Verlustfaktoren korrigiert.
// *   `Netto_Ladeenergie = Brutto_Ladeenergie * (1 - Ladeverlustfaktor)`
// *   `Netto_Entladeenergie = Brutto_Entladeenergie * (1 - Entladeverlustfaktor)`
// **Implikation:** Für die Berechnung von Autarkiegraden oder die Optimierung der Ladestrategie ist es entscheidend, mit den *tatsächlich nutzbaren* Energiemengen zu rechnen.
// **KORREKTES PATTERN (Beispiel):**
/*
// Level-Block (Ausschnitt)
// ["$P_Brutto_Ladeenergie_kWh", "float", 0.0],
// ["$P_Brutto_Entladeenergie_kWh", "float", 0.0],
// ["$Konst_Ladeverlustfaktor", "float", 0.10], // 10% Verlust
// ["$Konst_Entladeverlustfaktor", "float", 0.05], // 5% Verlust
// ["$Lgc_Netto_Ladeenergie_kWh", "float", 0.0],
// ["$Lgc_Netto_Entladeenergie_kWh", "float", 0.0],
// ["$Konst_1_Float", "float", 1.0],
// ["$Konst_Minus1_Float", "float", -1.0], // Für Subtraktion
// ["$Konst_0_Float", "float", 0.0], // Für Polynomial

// Module-Block (Ausschnitt)
// Netto-Ladeenergie = Brutto_Ladeenergie * (1 - Ladeverlustfaktor)
["Polynomial", "$Konst_Ladeverlustfaktor", "$Lgc_Ladeverlust_Faktor_Komplement", ["$Konst_1_Float", "$Konst_Minus1_Float"]], // 1 - Verlustfaktor
["Polynomial", "$P_Brutto_Ladeenergie_kWh", "$Lgc_Netto_Ladeenergie_kWh", ["$Konst_0_Float", "$Lgc_Ladeverlust_Faktor_Komplement"]],

// Netto-Entladeenergie = Brutto_Entladeenergie * (1 - Entladeverlustfaktor)
["Polynomial", "$Konst_Entladeverlustfaktor", "$Lgc_Entladeverlust_Faktor_Komplement", ["$Konst_1_Float", "$Konst_Minus1_Float"]], // 1 - Verlustfaktor
["Polynomial", "$P_Brutto_Entladeenergie_kWh", "$Lgc_Netto_Entladeenergie_kWh", ["$Konst_0_Float", "$Lgc_Entladeverlust_Faktor_Komplement"]]
*/

// --- Befehlspriorisierung & Manuell/Automatik-Umschaltung ---
// Muster: Alle Befehlsquellen (Sicherheit, Manuell, Automatik) in eine Logik-Zelle führen, die nach fester Prioritätenregel (Sicherheit > Manuell > Automatik) entscheidet.
// **HÄRTUNG: HOLD- ODER OVERRIDE-LOGIK FÜR MANUELLE EINGRIFFE**
// Dieses Pattern sollte explizit die Integration einer **"Hold"- oder "Override"-Logik** für manuelle Eingriffe (z.B. "Fernsehen"-Szene) beschreiben.
// **Prinzip:** Ein `Multiplexer` dient als zentraler "Arbitrator" für den Aktor-Sollwert (Höhe/Lamelle). Dessen `Selektor`-Eingang wird durch einen hochpriorisierten manuellen/Szenen-Befehl gesteuert. Solange dieser `true` ist, wird der manuelle Wert durchgereicht. Ist er `false`, wird das Ergebnis der `Automatik-Logik` durchgereicht.
// **Wichtig:** Um einen sanften Rücksprung zu gewährleisten, muss die Automatik den aktuellen Ist-Wert des Aktors lesen und als Startpunkt nutzen. Alternativ: Automatik-Output nach Ende der manuellen Übersteuerung explizit senden.

// --- BEST PRACTICE: KOMMENTAR-CODE-KONSISTENZ (Anti-Fehlinterpretation) ---
// **Beschreibung:** Jeder Kommentar im `Module`-Block MUSS zu 100% die exakte Funktion des darunterliegenden Codes widerspiegeln. Ein Kommentar darf niemals nur die *Absicht* beschreiben, wenn die Implementierung davon abweicht oder eine andere Logik erzeugt.
// **Regel:** Bei der Überprüfung des Codes ist explizit zu prüfen, ob Kommentare und Code semantisch übereinstimmen. Im Falle einer Diskrepanz ist entweder der Code oder der Kommentar zu korrigieren, um eine absolute Eindeutigkeit zu gewährleisten.

// --- Bool-zu-Wert Selektor (Das universelle Weichen-Pattern) ---
// Muster: Nutzt einen booleschen Eingang, um zwischen zwei Werten eines beliebigen Typs
// (String, Float, Integer) zu wählen. Dies ist die kanonische Methode für alle
// "Wenn-Dann-Sonst"-Zuweisungen und ersetzt eine Vielzahl spezifischer Konverter-Logiken.
// Prinzip: Ein boolescher Wert ($Input_Bool) wird als Selektor (0 oder 1) für ein
// Multiplexer-Modulbaustein mit zwei Eingängen verwendet. ACHTUNG: Literale wie 'true' oder 'false'
// dürfen nicht direkt im Multiplexer verwendet werden; es MÜSSEN Variablen sein.
Pattern: [ "Multiplexer", [ "$Wert_fuer_False", "$Wert_fuer_True" ], "Output", "$Input_Bool" ]

// --- Daten-Lookup-Tabelle (Parallele-Multiplexer-Pattern) ---
// Muster: Implementiert eine Wertetabelle (Lookup-Table), um aus einem numerischen Index
// einen Satz von zusammengehörigen Werten (z.B. RGB-Farbwerte) zu holen. Dies ist die
// kanonische Methode zur Abbildung von Arrays oder diskreten Datenstrukturen.
// Prinzip: Für jede Spalte der Datentabelle wird ein eigener Multiplexer-Modulbaustein verwendet.
// Alle diese Multiplexer werden vom selben Index-Eingang ($P_Index) gesteuert. Die
// Eingangslisten der Multiplexer enthalten die konstanten Werte für jede Spalte.
// Praxis-Beispiel (RGB-Farbwähler):
// [ "Multiplexer", [ "$Konst_R0", "$Konst_R1", ... ], "$O_Out_R", "$P_Index" ],
// [ "Multiplexer", [ "$Konst_G0", "$Konst_G1", ... ], "$O_Out_G", "$P_Index" ],
// [ "Multiplexer", [ "$Konst_B0", "$Konst_B1", ... ], "$O_Out_B", "$P_Index" ]

// **HÄRTUNG (PATTERN: OFFSET-ENTKOPPLUNG / Odometer-Prinzip) [NEU V7.02.04]:**
// Prinzip: Manuelle Korrekturwerte (Offsets) dürfen niemals direkt in die Differenzberechnung von Verbräuchen einfließen.
// Umsetzung: Der Offset wird erst am letzten Ausgang auf den internen, sauberen Zählerstand addiert. Die Statistik-Module greifen den Wert vor der Addition ab.

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_22.txt ---

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_23.txt ---

// --- Differenziator (Rate-of-Change-Detektor) ---
// Muster: Berechnet die Änderungsrate (dy/dt) eines Eingangswertes über die Zeit. Dies ist fundamental für
// die Detektion von plötzlichen Änderungen (z.B. schneller Temperaturanstieg, Feuchtigkeitsanstieg beim Duschen),
// um darauf zu reagieren, bevor absolute Schwellenwerte erreicht werden.
// Prinzip: Die Berechnung erfolgt in vier Schritten, die die strikt sequentielle Abarbeitung der Logik-Zelle nutzen.
// 1. Differenz bilden: Berechne die Differenz zwischen dem aktuellen Wert ($P_Wert_Aktuell) und dem Wert aus dem letzten Zyklus ($State_Wert_Alt). Ergebnis: $Lgc_Delta_Wert.
// 2. Zeitstempel-Differenz bilden: Berechne die Differenz zwischen dem aktuellen Zeitstempel ($Lgc_TS_Aktuell, geholt mit Localtime) und dem Zeitstempel des letzten Zyklus ($State_TS_Alt). Ergebnis: $Lgc_Delta_Zeit.
// 3. Änderungsrate berechnen: Dividiere die Wert-Differenz durch die Zeit-Differenz (Division-Modul). Das Ergebnis ($O_Rate_per_Second) ist die Änderungsrate in "Einheiten pro Sekunde".
// 4. Werte speichern (für nächsten Zyklus): Speichere am Ende des Module-Blocks den aktuellen Wert und den
// aktuellen Zeitstempel in die "Alt"-Variablen ($State_Wert_Alt, $State_TS_Alt) mittels Latch oder Multiplexer.

// --- Effiziente Bereichsumwandlung (Interpolations-Pattern) ---
// Muster: Wandelt einen Eingangswert aus einem Quellbereich (z.B. 0-200 km/h) in einen Zielbereich (z.B. 0-12 Beaufort) um.
// Dies ist die kanonische, elegante und effizienteste Methode für die Skalierung zwischen zwei kontinuierlichen Wertebereichen.
// Sie ist dem "Daten-Lookup-Tabelle"-Pattern überlegen, wenn es nicht um diskrete Indexwerte, sondern um stufenlose Umrechnungen geht.
// Prinzip: Der [Interpolation]-Modulbaustein wird mit einer Reihe von Stützpunkten konfiguriert, die Paare von Quell- und Zielwerten darstellen. Die Engine berechnet linear alle Werte zwischen diesen Stützpunkten.
// Praxis-Beispiel (km/h -> Beaufort):
// [ "Interpolation", "$P_Kmh_In", "$O_Bft_Out", [,,,, ... ] ]

// --- Effiziente String-Verkettung ---
// Muster: Das robuste und performante Zusammenfügen mehrerer Datenpunkte zu einem einzigen String.
// Dies ist kritisch für die Erstellung von MQTT-Befehlen, API-Anfragen oder formatierten Nachrichten.
// Prinzip:
// 1. Deklarieren: Alle temporären String-Variablen, die als Teile für die Verkettung dienen,
//    sollten als dynamischer `string` deklariert werden. Siehe Regel 1.2.
// 2. Formatieren: Wandle alle Nicht-String-Werte mit individuellen [Printf]-Aufrufen in diese
//    temporären String-Variablen um. Hierbei ist es eine zwingende Best Practice, für Zahlen Format-Spezifizierer mit führenden Nullen (`%02d`, `%04d`) zu verwenden, um unerwünschte Leerzeichen zu vermeiden.
// 3. Verketten: Führe einen einzigen [Concat]-Aufruf am Ende durch, der alle Teile
//    zusammenfügt.

// --- Formatierte Nachrichten (Printf + Concat Pattern) ---
// Muster: Kombination aus Printf (Werte formatieren) und Concat (Strings zusammensetzen).
// Siehe dazu das übergeordnete Pattern "Effiziente String-Verkettung" für die robusteste Implementierung.

// **HÄRTUNG (PATTERN: Gbglace Zähler-Statistik / Die "3-Sekunden-Regel") [NEU V7.02.04]:**
// Problem: Wenn man Zählerstände exakt um 00:00:00 Uhr schreibt, ordnet Grafana/InfluxDB sie oft schon dem neuen Tag zu.
// Lösung (gbglace): Der Trigger für den Perioden-Abschluss (15min, Stunde, Tag) wird 3 Sekunden VOR dem Zeitraster gesetzt.
// Cron-String: "57 14,29,44,59 * * * *" (Sekunde 57!)
// Effekt: Die Berechnung und das Speichern erfolgen noch innerhalb der alten Periode. Grafana zeigt den Balken korrekt im alten Zeitfenster an.
// Reset-Sicheres Zählen:
// Erkennung: Comparator (Alt > Neu = Reset).
// Formel (Ternär): Reset ? (Alt_Total + Neu_Raw) : (Alt_Total + Neu_Raw - Alt_Raw).
// Die Perioden-Kaskade:
// Man baut nicht 5 einzelne Timer. Man baut einen Taktgeber (den 15-Minuten Cron).
// Höhere Perioden (Stunde, Tag) werden durch AND-Verknüpfung abgeleitet:
// Stunden_Ende = 15Min_Ende AND Minute_59
// Tages_Ende = Stunden_Ende AND Stunde_23

// --- Gegenseitiger Ausschluss (Interlock) ---
// Muster: Stellt sicher, dass von mehreren Ausgängen zu jedem Zeitpunkt nur maximal einer aktiv sein kann.
// Dieses Pattern ist kritisch für die Ansteuerung von Motoren (z.B. Links/Rechts-Lauf), Ventilen oder mehrstufigen Geräten.
// Methode A (Best Practice - Der Zentralisierte Arbitrator): Die überlegene Methode besteht darin, alle Steuerbefehle
// in EINER Logik-Zelle zu sammeln. Diese Zelle verwaltet den Zustand zentral und agiert als "Schiedsrichter" (Arbitrator),
// der entscheidet, welcher Ausgang aktiv sein darf. Dies vermeidet Race Conditions und komplexe Querverschaltungen.
// Beispiel-Prinzip: "Wenn Befehl für Stufe 2 kommt, schalte Stufe 1 explizit aus und Stufe 2 ein."
// Methode B (Workaround - Der Gekreuzte Interlock): Wenn eine fehlerhafte Architektur mit getrennten Logik-Zellen
// bereits existiert, kann eine Verriegelung durch Rückführung erreicht werden. Der Ausgang von Logik A wird als
// negierter Eingang in Logik B geführt (und umgekehrt) und dort mit einer AND-Bedingung verknüpft. Dies ist
// architektonisch unterlegen, fehleranfällig und skaliert schlecht.
// HINWEIS: Für komplexe Szenarien mit mehr als zwei Zuständen (z.B. AUS, STUFE_1, STUFE_2, AUTO) ist ein dedizierter
// Zustandsautomat die professionellste Lösung. Siehe das Pattern 'Zustandsautomat (State Machine)' für eine detaillierte Anleitung.

// --- Gleichheitsprüfung (für Zahlen) ---
// Muster: Implementiert eine exakte Gleichheitsprüfung (==) für Zahlen oder Variablen. Der Comparator ist für
// den Vergleich > oder < vorgesehen, aber nicht für eine robuste Gleichheitsprüfung.
// Prinzip: Ein Limiter-Modulbaustein wird so konfiguriert, dass der minimale und maximale Grenzwert identisch sind.
// Der Ausgang "OutIsInRange" ist dann genau dann 'true', wenn der Eingangswert exakt diesem Wert entspricht.
// Der zweite Ausgang des Limiters (Out_Clamped) wird nicht benötigt und kann mit dem Literal 0 belegt werden.
// Voraussetzung: $Wert_zum_Vergleich muss den Wert enthalten, mit dem verglichen werden soll.
Pattern: [ "Limiter", "Variable_To_Test", 0, "Ergebnis_Bool", [ "$Wert_zum_Vergleich", "$Wert_zum_Vergleich" ] ]

// --- Index-basierter Demultiplexer (Sequenzer-Pattern) ---
// Muster: Sequentielles Aktivieren eines von vielen Ausgängen basierend auf einem numerischen Index. Dies ist die kanonische Methode zur Abarbeitung von Listen oder zur Implementierung von Sequenzern, da die Logik-Engine keine "For-Loops" oder Multivariablen-Iteration in der Ausgabe unterstützt.
// Prinzip:
// 1. Zähler: Eine Integer-Variable (z.B. $State_CurrentStep) dient als Zähler für den aktuellen Schritt. Sie wird bei jedem Schritt inkrementiert (z.B. mit Polynomial) und bei Bedarf zurückgesetzt. Der Wert 0 repräsentiert typischerweise einen inaktiven Zustand.
// 2. Auswahl: Für jeden potenziellen Ausgang (z.B. $O_Circuit_1, $O_Circuit_2, ...) wird ein eigener Multiplexer-Modulbaustein verwendet.
// 3. Ansteuerung: Der $State_CurrentStep wird als Selektor-Eingang für ALLE diese Multiplexer verwendet.
// 4. Logik: Der Eingangs-Array des Multiplexers wird so konstruiert, dass er nur am Index des Ziel-Ausgangs einen true-Wert (oder einen anderen Steuerwert) enthält. Der Index ist 1-basiert, passend zum Zählerwert.
Pattern:
// -- Annahmen: $State_CurrentStep zählt 0 (inaktiv), 1 (Kreis 1), 2 (Kreis 2)
// [ "Multiplexer", [ "$Konst_False", "$Konst_True", "$Konst_False" ], "$O_Circuit_1", "$State_CurrentStep" ],
// [ "Multiplexer", [ "$Konst_False", "$Konst_False", "$Konst_True" ], "$O_Circuit_2", "$State_CurrentStep" ]

// --- Impuls-basierte Positionssteuerung (Rollladen-Pattern) ---
// Muster: Bildet die Funktionalität eines Rollladenaktors für 'dumme' Antriebe nach, die über Start/Stop-Impulse gesteuert werden. Dies ist ein komplexes, stateful Pattern.
// Prinzip:
// 1. State Management: Ein Latch speichert die zuletzt bekannte Position ($State_pos_cur). Bei jeder abgeschlossenen Fahrt wird er mit der neuen Zielposition ($P_pos) aktualisiert.
// 2. Berechnung: Bei einem neuen Positionsbefehl werden die Fahrtrichtung ($Lgc_up/down) und die benötigte Fahrzeit ($Lgc_delay) mittels Comparator und CalcFormula berechnet.
// 3. Grenzfallerkennung: Ein Limiter prüft, ob eine Vollfahrt (Position 0 oder 100) angefordert wird. In diesem Fall wird der Stop-Timer unterdrückt.
// 4. Start-Impuls: Ein SendExplicit sendet den Start-Impuls für die ermittelte Richtung ($O_up oder $O_down). Dieser Impuls sollte nur einen Zyklus antaugen.
// 5. Stop-Impuls (Timer):
// a) Ein Monoflop wird durch den Start der Fahrt getriggert (z.B. durch ein OR von $Lgc_up und $Lgc_down). Seine Dauer ist die berechnete $Lgc_delay.
// b) Der Set-Eingang des Monoflops wird nur aktiviert, wenn KEINE Vollfahrt vorliegt.
// c) Das Ablaufen des Timers wird durch das Pattern "Sequenzer (Stabile Timer-Kaskade)" detektiert.
// d) Der resultierende Impuls löst direkt ein SendExplicit für den Stop-Impuls aus (erneuter Impuls auf denselben Kanal wie der Start-Impuls).
// 6. Desynchronisations-Prävention: Eine übergeordnete Logik-Zelle sollte periodisch (z.B. einmal täglich) eine Kalibrierungsfahrt zu einer Endposition (0 oder 100) auslösen, um den internen Zustand ($State_pos_cur) zu korrigieren.

// --- Impuls-gesteuerter Negator (Stateless Stromstoßschalter) ---
// Muster: Implementiert einen zustandslosen Stromstoßschalter, der bei jedem Impuls
// den eingelesenen Ist-Zustand negiert. Der Zustand wird extern gehalten.
Pattern: [ "Xor", [ "-$State_Feedback_In" ], "Command_Out" ]

// --- Initialisierungsphase für Periodische Zähler (Verzögerte Anzeige) ---
// **Muster:** Beschreibt das typische Verhalten von periodischen Zählerlogiken, bei denen die "aktuellen" Verbrauchswerte für eine Periode (Tag, Woche, Monat, Jahr) erst nach dem *ersten vollständigen Abschluss* dieser Periode (nach dem Speichern der Logik) korrekt angezeigt werden. Bis dahin können diese Werte `0` sein.
// **Prinzip:** Die Berechnung des "aktuellen Verbrauchs" (`Consumption_CurrD` etc.) basiert auf der Differenz zwischen dem aktuellen Zählerstand (`$Counter`) und dem Zählerstand am Periodenbeginn (`$End_D`). Da `$End_D` erst am Ende der ersten Periode gesetzt wird, ist die Differenz davor nicht aussagekräftig oder `0`.
// **Implikation:** Nutzer müssen sich dieser Initialisierungsphase bewusst sein. Für eine sofortige Anzeige des *laufenden* Verbrauchs ab Logikstart (unabhängig von Periodenende) ist eine separate Logik erforderlich, die den Zählerstand kontinuierlich akkumuliert (wie unser `Energieverbrauchszähler` mit dynamischem Intervall).
// **KORREKTES PATTERN (Beispiel):**
/*
// Level-Block (Ausschnitt)
// ["$Counter","float",0.0],
// ["$End_D","float",0.0], // Zählerstand am Tagesbeginn
// ["$Consumption_CurrD","float",0.0], // Aktueller Verbrauch des Tages
// ["$Formula","string","X2>0?X1-X2:0"], // Perioden-richtige Differenz

// Module-Block (Ausschnitt)
// Berechnung des aktuellen Verbrauches in der Periodenende
["CalcFormula",["$Counter","$End_D"], "$Consumption_CurrD", "$Formula"],
// ... (Logik zum Setzen von $End_D am Tagesende) ...
*/

// --- Initialisierungs-Sequenz (First Run / On-Demand Pattern) ---
// Muster: Führt eine Reihe von Aktionen aus, um einen definierten Systemzustand herzustellen. Der Trigger
// für diese Sequenz ist entscheidend und muss je nach Anforderung selbst gebaut werden (Regel 1.7).
// Mögliche Trigger:
// - Nach TWS-Neustart: Verwendung des 'First Run'-Patterns mit einem selbsthaltenden Latch.
// - Nach KNX Bus-Reset: Verwendung eines externen Triggers (z.B. von einem Aktor, der einen 'Senden bei Reset'-Befehl
// unterstützt), der einen Impuls an die Initialisierungs-Logik sendet.
// - Manuell auf Anforderung: Ein dedizierter Taster in der Visu.
// Die Sequenz selbst nutzt typischerweise SendExplicit-Modulbausteine, um die gewünschten Werte auf den Bus zu senden.
// Ein kritischer Anwendungsfall ist die Initialisierung des "alten Wertes" für den Differenziator (z.B. $State_Wert_Alt) mit dem aktuellen Ist-Wert, um eine Fehlauslösung beim ersten Start zu verhindern.

// --- Init-Senden nach Reboot (Zustandswiederherstellung) ---
// **Muster:** Nach einem Neustart des Logik-Subsystems oder des Timberwolf Servers werden einmalig die letzten persistenten Werte wichtiger Zustandsvariablen aktiv auf den Bus gesendet.
// **Architektur:** Zwei Logik-Zellen: "Reboot-Trigger" (Baustein 1) und "Persistenter Wertesender" (Baustein 2).
// **Details:** Baustein 1 sendet einmalig einen Trigger (keine Persistenz). Baustein 2 speichert Werte persistent (`Persistenz MUSS aktiviert sein`), reagiert auf den Trigger von Baustein 1 und sendet die gespeicherten Werte. Eingänge in Baustein 2 mit `"u"`, Ausgänge mit `"a"`.
// **Verweis auf `Regel 1.23`:** Dieses Pattern ist essenziell, um den durch `Regel 1.23` beschriebenen Verlust von *internen* persistenten Werten bei *Logik-Konfigurationsänderungen* zu umgehen.

// --- Input-Kompensation für Regelkreise (Bereinigter Istwert) ---
// **Muster:** Bereinigt den primären Mess-Input eines Regelkreises um die eigene, zuletzt ausgegebene Aktorleistung, um einen "ungestörten" oder "ursprünglichen" Systemzustand für die Reglerberechnung zu erhalten.
// **Prinzip:** `Bereinigter_Istwert = Gemessener_Istwert - Letzte_Eigene_Aktorleistung`. Dieser bereinigte Wert wird dann als Input für die Reglerlogik verwendet. Die `Letzte_Eigene_Aktorleistung` MUSS dabei aus der zuvor *vom Regler ausgegebenen* Leistung stammen (z.B. `$State_LastSollLeistungLaden` oder `$State_LastSollLeistungEntladen`) und nicht von der tatsächlichen Ist-Leistung des Aktors, da diese Verzögerungen unterliegen kann.
// **KORREKTES PATTERN (Beispiel für Ladeleistung):**
// Level-Block (Ausschnitt)
// ["$P_Netz_Ist_W", "float", 0.0],          // Gemessener Netzbezug
// ["$State_LastSollLeistungBattery_Laden", "float", 0.0],  // Zuletzt gesendete Ladeleistung
// ["$State_LastSollLeistungBattery_Entladen", "float", 0.0], // Zuletzt gesendete Entladeleistung
// ["$Lgc_Netz_Ist_Bereinigt_W", "float", 0.0], // Der bereinigte Istwert
// ["$Konst_One_Float", "float", 1.0],
// ["$Konst_Minus_One_Float", "float", -1.0], // Für Negation

// Module-Block (Ausschnitt)
// Bereinige den Netz-Ist-Wert um die zuletzt ausgegebene Batterieleistung
// (Netz_Ist_Bereinigt = (1 * P_Netz_Ist_W) + (-1 * State_LastSollLeistungBattery_Entladen) + (1 * State_LastSollLeistungBattery_Laden))
/*
["Polynomial", "$State_LastSollLeistungBattery_Entladen", "$Lgc_Entladen_Negiert", ["$Konst_0_Float", "$Konst_Minus_One_Float"] ],
["Polynomial", "$P_Netz_Ist_W", "$Lgc_Netz_Plus_Laden", ["$State_LastSollLeistungBattery_Laden", "$Konst_One_Float"] ],
["Polynomial", "$Lgc_Netz_Plus_Laden", "$Lgc_Netz_Ist_Bereinigt_W", ["$Lgc_Entladen_Negiert", "$Konst_One_Float"] ]
*/
// Verwende $Lgc_Netz_Ist_Bereinigt_W als Input für weitere Reglerberechnungen
// Hinweis: Die Kompensation muss die Vorzeichenkonvention des Netzbezugs und der Batterieleistung korrekt berücksichtigen.

// --- Isolations-Diagnose (Das "Herz-Transplantations"-Pattern) ---
// Muster: Eine systematische Methode zur Fehlersuche in komplexen Logiken,
// die sich anlegen, aber nicht speichern lassen.
// Prinzip: Der Fehler liegt mit hoher Wahrscheinlichkeit in einem neu hinzugefügten
// oder geänderten Logik-Block, der einen Zirkelbezug erzeugt hat. Anstatt
// im Gesamt-Code zu suchen, wird der verdächtige "Organ" (z.B. die Grundlüftungs-Logik)
// isoliert.
// Workflow:
// 1. BASIS SICHERN: Speichere die letzte, nachweislich funktionierende Version der Logik-Zelle ab.
// 2. ISOLIEREN: Erstelle eine neue, minimale Custom Logic. Kopiere NUR den
// verdächtigen Modul-Block hinein und deklariere NUR die absolut notwendigen
// Variablen im Level-, Input- und Output-Block, um ihn lauffähig zu machen.
// 3. TESTEN: Versuche, diese Minimal-Logik zu speichern.
// - a) Schlägt sie fehl, ist der Fehler im isolierten Block gefunden und kann dort korrigiert werden.
// - b) Funktioniert sie, lag der Fehler in der fehlerhaften Integration des Blocks
// in die Gesamtlogik (z.B. durch eine neue, unerwartete Rückkopplung).
// 4. INTEGRIEREN: Füge den korrigierten und isoliert getesteten Block wieder
// in die Gesamtlogik ein.

// --- Kanonischer Inverter (Multiplexer-Negation) ---
// Muster: Implementiert eine zustandslose Negation (NOT-Gatter), da kein nativer
// "Inverter"-Modulbaustein existiert.
// Prinzip: Nutzt das "Bool-zu-Wert Selektor"-Pattern, um bei Input=false den Wert
// true auszugeben und umgekehrt. Dies ist der einzig validierte Weg zur Invertierung.
// Voraussetzung: Variablen $Konst_True (bool, true) und $Konst_False (bool, false)
// müssen deklariert sein.
Pattern: [ "Multiplexer", [ "$Konst_True", "$Konst_False" ], "Output_Bool", "$Input_Bool" ]

// --- Komplexe Formel-Zerlegung (Heizkurven-Pattern) ---
// Muster: Die kanonische Methode zur Abbildung einer komplexen mathematischen Formel (wie einer Heizkurve),
// die über simple Addition/Multiplikation hinausgeht.
// Prinzip: Anstatt zu versuchen, die gesamte Formel in einen einzigen [CalcFormula]-Modulbaustein zu zwingen,
// wird die Formel in ihre logischen Einzelteile zerlegt. Jeder Teil wird durch einen stabilen, einfachen
// [Polynomial]-Modulbaustein berechnet. Die Zwischenergebnisse werden in `Lgc_...`-Variablen gespeichert
// und in den nachfolgenden Schritten wiederverwendet. Am Ende wird das Endergebnis mit einem [Limiter]
// auf einen gültigen Bereich begrenzt.
// Praxis-Beispiel (Heizkurve V3): Der Autor zerlegt eine komplexe Formel in 5 Berechnungen und eine
// finale Begrenzung, die jeweils durch einen eigenen Modulbaustein repräsentiert werden. Dies führt zu einer
// extrem robusten, nachvollziehbaren und wartbaren Logik.
Pattern:
// 1. [ "Polynomial", ... "$T1", ... ] // Berechne ersten Teil der Formel in T1
// 2. [ "Polynomial", ... "$T2", ... ] // Berechne zweiten Teil der Formel in T2
// 3. [ "Polynomial", "$T1", "$T_final", ["$T2", ...] ] // Kombiniere Zwischenergebnisse
// 4. [ "Limiter", "$T_final", "$O_Output", "$Lgc_Limiter_OK", ["$Min", "$Max"] ] // Begrenze das Endergebnis

// --- Kontrolliertes Inhibit/Deaktivieren (Fail-Safe Shutdown) ---
// **Muster:** Sicheres Deaktivieren oder Überführen einer Logik in einen sicheren Zustand (ohne `Break`).
// **Prinzip:** `Statemachine`-Übergänge (`Finishing`-Zustand mit Timeout) und/oder `Multiplexer` zum Erzwingen sicherer Ausgangswerte.

// --- Lastabwurf mit Hysterese & Einschaltverzögerung ---
// Muster: Kombiniert Modulbausteine zu einer robusten Laststeuerung.

// --- Manuelle Reset-Funktion für Zähler ---
// **Muster:** Manuelles Zurücksetzen von Zählern (z.B. über Visu).
// **Prinzip:** Boolescher `Input` (z.B. `$P_Manual_Reset`) mit Trigger `"c"` auf `Multiplexer` anwenden, um Zähler-Variable auf `$Konst0_Float` zu setzen. `Multiplexer` nach Akkumulationslogik, vor `Latch` für `$XLast`. `Input` zurücksetzen (`Automatischer Trigger-Reset`).

// **HÄRTUNG (PATTERN: Mathematischer Selektor / Polynomial Indexing) [NEU V7.02.04]:**
// Problem: Wie wählt man basierend auf mehreren exklusiven Booleschen Flags (z.B. Is_Szene1, Is_Szene2...) einen Wert aus einer Liste, ohne eine unlesbare Kette von verschachtelten Multiplexern zu bauen?
// Lösung: Man berechnet den Index mathematisch mit dem Polynomial-Modul.
// Formel: Index = 0 + (1 * Is_S2) + (2 * Is_S3) + (3 * Is_S4) ...
// Vorteil: Ein einziger Integer-Wert steuert einen flachen Multiplexer. Das ist extrem effizient und wartbar.
// Voraussetzung: Die Booleschen Flags müssen sich gegenseitig ausschließen (Mutually Exclusive).

// --- Minimum Update/Send Interval nach Änderung ---
// **Muster:** Sorgt dafür, dass ein Ausgangswert nach einer Änderung nur dann aktualisiert/gesendet wird, wenn seit der letzten Aktualisierung/Sendung eine bestimmte Mindestzeit verstrichen ist. Dies dient dazu, "nervöse" Ausgänge bei schnellen, geringfügigen Änderungen zu beruhigen.
// **Prinzip:** Ein `Monoflop` wird jedes Mal getriggert, wenn sich ein Wert ändert (z.B. durch das `Single-Pulse-on-Update` Detektor-Pattern oder Flankenerkennung). Der Ausgang des `Monoflop`s (`Q`) wird als Selektor für einen `Multiplexer` verwendet. Solange der `Monoflop` läuft (`Q` ist `true`), wird die Aktualisierung des Outputs unterdrückt, oder es wird der letzte Wert beibehalten. Erst wenn der `Monoflop` abgelaufen ist (`Q` ist `false`), wird der neue Wert durchgelassen und der `Monoflop` kann erneut getriggert werden.

// --- Modulare Systemarchitektur (Drei-Ebenen-Modell) ---
// Muster: Zerlegt eine komplexe Steuerungsaufgabe (z.B. eine Bewässerungsanlage) in logische Schichten mit klar getrennten Verantwortlichkeiten. Dies ist fundamental für die Schaffung wartbarer, skalierbarer und robuster Systeme.
// Prinzip:
// 1. Ebene: SystemControl (Hardware-Abstraktion & Sicherheit): Dieser Baustein ist die unterste Ebene. Er verwaltet geteilte, kritische Ressourcen (z.B. Haupt-Wasserpumpe, Netzteile) und überwacht systemweite Zustände (z.B. Fehler). Er abstrahiert die Komplexität der Hardware und stellt den übergeordneten Logik-Zellen einen einfachen, globalen Freigabe-Zustand (z.B. O_System_OK) zur Verfügung.
// 2. Ebene: CircuitControl (Einzellogik): Dies ist die Logik-Zelle für eine einzelne, gekapselte Funktion (z.B. einen Bewässerungskreis). Es gibt typischerweise mehrere Instanzen dieses Bausteins. Jede Instanz fordert bei Bedarf Ressourcen von der SystemControl-Ebene an (P_System_Request) und wartet auf deren Freigabe (P_System_OK_Feedback), bevor sie ihre eigentliche Aufgabe (z.B. ein Ventil für eine bestimmte Zeit öffnen) ausführt.
// 3. Ebene: Sequencer (Koordinator): Dieser Baustein ist die "Dirigent" des Gesamtsystems. Er startet, stoppt und orchestriert den Ablauf der einzelnen CircuitControl-Instanzen. Er nutzt typischerweise das 'Index-basierte Demultiplexer'-Pattern, um die einzelnen Kreise nacheinander zu triggern und wartet auf deren "Fertig"-Meldung, um zum nächsten Schritt überzugehen.
// Vorteil: Extreme Entkopplung. Die Logik eines einzelnen Bewässerungskreises muss nichts über die Pumpe oder andere Kreise wissen. Der Sequenzer muss nichts über die interne Funktionsweise eines Kreises wissen. Dies ermöglicht einfaches Testen, Wiederverwendung und flexible Erweiterungen.

// --- Multiplikation (via Polynomial-Pattern) ---
// Muster: Emuliert die Multiplikation A * B, da kein nativer Modulbaustein existiert. Dies ist ein
// fundamentales Pattern, das die kreative Nutzung vorhandener Modulbausteine demonstriert.
// Prinzip: Es nutzt die Polynom-Formel Y = A0 + A1*X. Mit dem konstanten Koeffizienten A0=0
// und dem zweiten Faktor als Koeffizient A1 wird die Gleichung zu Y = FaktorB * FaktorA.
Pattern: [ "Polynomial", "FaktorA", "Produkt", [ 0, "$FaktorB" ] ]

// --- Optionale-Parameter-Verkettung ---
// Muster: Dies ist das offizielle, von den Entwicklern validierte Muster, um eine
// Kommando-Zeichenkette aus einem obligatorischen Startteil und mehreren
// optionalen Parameterteilen zusammenzusetzen. Es ist die kanonische Lösung
// für das Problem: "Wie füge ich 'param=wert' nur dann an, wenn 'wert' nicht leer ist?".
//
// Problemstellung: Eine einfache Verkettung würde bei leeren Eingängen zu
// fehlerhaften Befehlen wie "target=ftp|url=|tls=no" führen. Dieses Muster
// stellt sicher, dass nur nicht-leere Parameter an den finalen String angehängt werden.
//
// Prinzip: Die Logik wird in wiederholbare Blöcke für jeden optionalen Parameter aufgeteilt.
// Ein Haupt-String ($O_Command) wird schrittweise aufgebaut.
//
// Kanonischer Workflow für jeden optionalen Parameter (z.B. $P_Input_URL):
// 1. PUFFER ZURÜCKSETZEN: Ein temporärer Puffer ($Lgc_Buffer_Out) wird zu Beginn
//    jedes Blocks mit einem leeren String überschrieben. Dies ist ein wichtiger
//    Schritt der "defensiven Programmierung", um einen sauberen Zustand zu garantieren.
//    [ "Latch", "$Konst_Empty_String", "$Lgc_Buffer_Out", "$Konst_True", 0 ]
//
// 2. EINGANG PRÜFEN: Der Eingabeparameter ($P_Input_URL) wird mit dem [Regex]-Modul
//    darauf geprüft, ob er leer ist.
//    - Regulärer Ausdruck: "^\\s*$" (prüft auf "String ist leer oder enthält nur Leerzeichen").
//    - Ergebnis: Der Regex-Ausgang "Match_Bool" wird `true`, wenn der Input leer ist.
//    - Steuer-Flag: Eine boolesche Variable ($Lgc_Use_Param) wird an den negierten
//      Regex-Ausgang gekoppelt. Sie ist somit `true`, wenn der Input NICHT leer ist.
//    [ "Regex", "$P_Input_URL", "$Konst_Regex_Empty", "-$Lgc_Use_Param", 0,0,0,0,0,0 ]
//
// 3. FORMATIEREN & KONDITIONALISIEREN (Das "Gatekeeper"-Pattern):
//    a) Mit [Printf] wird der Parameter-String (z.B. "|url=mein.server") IMMER
//       formatiert und in einem ersten Puffer ($Lgc_Buffer_In) abgelegt.
//    b) Mit einem pegelgesteuerten [Latch, Mode 0] wird der formatierte String aus
//       $Lgc_Buffer_In in den finalen Block-Puffer ($Lgc_Buffer_Out) übernommen,
//       aber NUR, wenn das Steuer-Flag $Lgc_Use_Param `true` ist.
//    [ "Printf", "$P_Input_URL", "$Konst_Format_URL", "$Lgc_Buffer_In" ]
//    [ "Latch", "$Lgc_Buffer_In", "$Lgc_Buffer_Out", "$Lgc_Use_Param", 0 ]
//
// 4. ANHÄNGEN: Der Inhalt von $Lgc_Buffer_Out (der entweder den formatierten
//    Parameter oder einen leeren String enthält) wird an den Haupt-Befehl angehängt.
//    [ "Concat", [ "$O_Command", "$Lgc_Buffer_Out" ], "$O_Command" ]
//
// Dieser vierteilige Block wird für jeden weiteren optionalen Parameter wiederholt.

// --- P-Regler-Implementierung mit PID_awu ---
// **Muster:** Implementierung einer proportionalen Regelungslogik, um einen Istwert auf einen Sollwert auszusteuern, inklusive Anti-Windup und Begrenzung.
// **Prinzip:** Nutzt das `PID_awu`-Modul, wobei die `Ki_float` und `Kd_float` Parameter auf 0.0 gesetzt werden können, um eine reine P-Regelung zu erzielen. `Ramp` kann nachgeschaltet werden, um die Änderungsgeschwindigkeit des `PID_awu`-Outputs zu begrenzen.

// --- Perioden-richtige Differenz (CalcFormula) ---
// **Muster:** Berechnung des Verbrauchs innerhalb einer Periode, vermeidet negative Ergebnisse vor erstem Periodenende.
// **Prinzip:** `CalcFormula` mit `X2>0?X1-X2:0`.
// **KORREKTES PATTERN:** `["CalcFormula",["$Counter","$End_Period"], "$Consumption_CurrPeriod", "X2>0?X1-X2:0"]`

// --- Periodischer Sender (Heartbeat-Pattern) ---
// Muster: Sendet einen Wert sowohl bei einer Wertänderung (Trigger 'a' am Eingang) als
// auch in einem regelmäßigen Intervall. Dies ist fundamental für Systeme, die periodische
// Aktualisierungen benötigen (z.B. Visualisierungen, Überwachung).
// Prinzip: Die Kombination der Trigger ist entscheidend. Ein Clocksignal löst periodisch
// einen Logik-Llauf aus. Eine Änderung am Eingangswert löst ebenfalls einen Lauf aus.
// Der Ausgang mit Trigger 't' (on Trigger) sendet seinen Wert bei JEDEM dieser Läufe.
// Das "Variablen-Kopie"-Pattern wird genutzt, um den Eingangswert stabil auf den Ausgang zu kopieren.
// **HÄRTUNG: ZWECKKLARSTELLUNG**
// Dieses Pattern dient primär für das **periodische Senden des aktuellen Werts (auch wenn er unverändert ist)** zur Visualisierung oder Überwachung (mittels `Clocksignal` und Output-Trigger `"t"`).
Pattern:
// -- Level-Block --
// [ "$P_Value_In", "string,64", "" ],
// [ "$P_Interval_s", "float", 300.0 ],
// [ "$O_Value_Out", "string,64", "" ],
// [ "$Lgc_Clock_Impuls", "bool", false ],
// [ "$Konst_True", "bool", true ]
// -- Module-Block --
// [ "Clocksignal", "$Konst_True", "$Lgc_Clock_Impuls", "$P_Interval_s" ],
// [ "Multiplexer", [ "$P_Value_In", "$P_Value_In" ], "$O_Value_Out", "$Konst_True" ] // Workaround für Variablen-Kopie
// -- Input-Block --
// [ "Wert", ..., "$P_Value_In", "a" ],
// [ "Intervall", ..., "$P_Interval_s", "c" ]
// -- Output-Block --
// [ "Ausgang", ..., "$O_Value_Out", "t" ]

// --- Periodischer Zähler mit Reset & Akkumulation (Betriebsstundenzähler-Pattern) ---
// Muster: Implementiert einen robusten Betriebsstundenzähler mit getrennten Zählern für
// Tag, Monat und Jahr. Dieses komplexe Pattern kombiniert mehrere Kern-Modulbausteine.
// Prinzip:
// 1. Basiszähler: Ein [HobbsMeter] zählt die Betriebsstunden des Geräts in eine
//    Tageszähler-Variable ($Betriebsdauer_Tag). Dieser Zähler wird durch ein
//    kombiniertes Reset-Signal zurückgesetzt.
// 2. Zeitliche Trigger: Mehrere [Cron]-Module erzeugen einmalige Impulse ($ResetTag, $ResetMonat, ...)
//    kurz nach Mitternacht, um die verschiedenen Reset-Zyklen zu steuern.
// 3. Reset-Verarbeitung:
//    a) Die Reset-Impulse werden mit [Or] zu einem Gesamt-Reset-Signal ($Reset) kombiniert.
//    b) Das $Reset-Signal löst das Zurücksetzen des [HobbsMeter] aus.
//    c) Das $Reset-Signal wird genutzt, um die finalen Zählerstände in "Last"-Variablen
//       zu kopieren, bevor sie zurückgesetzt werden (z.B. [Multiplexer, ..., "$Betriebsdauer_TagLast", "$Reset"]).
// 4. Akkumulation: Vor dem Reset wird der Wert des Tageszählers auf die Monats- und
//    Jahreszähler aufaddiert. Dies geschieht mit [Polynomial].
//    [ "Polynomial", "$Betriebsdauer_Monat", "$Betriebsdauer_Monat_Neu", [ "$Betriebsdauer_Tag", 1 ] ]
// 5. Selbst-Reset der Trigger: Die von Cron gesetzten Reset-Impulse werden mit dem
//    "Automatischer Impuls-Reset"-Pattern sofort wieder zurückgesetzt, um für den nächsten Zyklus bereit zu sein.

// --- Plausibilisierung von Energieflüssen (Anti-Timing-Artefakte) ---
// **Muster:** Sicherstellen, dass berechnete Energie- oder Leistungsflüsse niemals physikalisch unmögliche oder logisch widersprüchliche Werte annehmen (z.B. negativer Eigenverbrauch, Eigenverbrauch > Gesamtleistung) durch asynchrone Eingangsdaten.
// **Prinzip:** `Limiter`-Modul proaktiv an Ausgängen von Berechnungen einsetzen, um Werte auf plausible Bereiche zu begrenzen (z.B. Minimum 0).
// **KORREKTES PATTERN:** `["Limiter","$I_Leistung_PV","$Leistung_Eigenverbrauch","$Unused_bool",["$Konst0", "$Leistung_Haus"]]`

// --- Prädiktive PV-Ertragsabschätzung (Sonnenstands-basiert) ---
// **Muster:** Nutzt den aktuellen Sonnenstand (Elevation und Azimut) als Indikator, um den *potenziellen* PV-Ertrag der nächsten Stunden abzuschätzen und die Energiemanagement-Strategie vorausschauend anzupassen.
// **Prinzip:** Das `Astro`-Modul liefert Elevation und Azimut. Diese Werte können als Eingänge für eine Logik dienen, die entweder:
// *   Eine einfache `Interpolation`-Tabelle verwendet, die Sonnenstandsbereiche bestimmten "Ertragsprofilen" zuordnet.
// *   Oder komplexere `CalcFormula`- oder `Polynomial`-Berechnungen durchführt, die den Sonnenstand mit historischen Ertragsdaten (wenn über ein zukünftiges Modul verfügbar) korrelieren.
// **Implikation:** Dies ermöglicht eine vorausschauende Steuerung (z.B. "erwarte in 2 Stunden hohen PV-Überschuss, beginne jetzt schon mit der Pufferladung, um bereit zu sein").

// --- PWM-Generator ---
// Muster: Erzeugt ein PWM-Signal mittels Clocksignal, Polynomial und Monoflop.

// --- Priorisierte Zustands-Selektion (Kaskadierender Multiplexer) ---
// Muster: Um einen finalen Zustand (z.B. einen Status-Text) aus mehreren Bedingungen mit unterschiedlicher Priorität zu bestimmen, wird eine Kette von Multiplexer-Bausteinen verwendet. Der erste Multiplexer behandelt die niedrigste Priorität (Default-Zustand). Jeder nachfolgende Multiplexer nimmt das Ergebnis des vorherigen als "False"-Eingang und überschreibt es nur dann mit einem höher priorisierten Zustand, wenn seine eigene Bedingung (Selector) true ist.

// --- Robuste Division (Vermeidung Division durch Null) ---
// **Muster:** Sicherstellt, dass der Nenner in einer Divisionsoperation immer einen gültigen Wert (`>0`) hat, um Laufzeitfehler zu vermeiden.
// **Prinzip:** Ein `Limiter`-Modul wird verwendet, um den Nenner auf einen Minimalwert (`>0`, z.B. `0.001` oder `1.0`) zu begrenzen.
// **KORREKTES PATTERN:**
/*
// Level-Block (Ausschnitt)
// ["$Input_Nenner", "float", 0.0],
// ["$Input_Zaehler", "float", 0.0],
// ["$Lgc_Nenner_Gelimited", "float", 1.0],
// ["$Output_Result", "float", 0.0],
// ["$Konst_MinNenner", "float", 0.001], // Kleine Zahl > 0

// Module-Block (Ausschnitt)
// Begrenze den Nenner auf einen Wert > 0
["Limiter", "$Input_Nenner", "$Lgc_Nenner_Gelimited", 0, ["$Konst_MinNenner", "$Input_Nenner"]],
// Führe die Division mit dem begrenzten Nenner durch
["Ratio", "$Input_Zaehler", "$Output_Result", "$Lgc_Nenner_Gelimited"]
*/

// --- Robuster Kleiner-Gleich Vergleich (<=) ---
// **Muster:** Implementiert einen logisch korrekten "Kleiner gleich" (`<=`) Vergleich zwischen zwei numerischen Werten, der die Spezifika des `Comparator`-Moduls berücksichtigt.
// **Prinzip:** Der `Comparator` kann direkt nur `>` prüfen. Ein `<=` Vergleich (`A <= B`) kann durch die Prüfung der inversen Bedingung `A > B` und anschließende Negation des Ergebnisses realisiert werden. Alternativ kann eine Kombination aus `Comparator` (für `<`) und `Limiter` (für `==`) verwendet werden.
// **KORREKTES PATTERN (für `A <= B`):**
/*
// Level-Block (Ausschnitt)
// ["$A_Value", "float", 0.0],
// ["$B_Value", "float", 0.0],
// ["$Lgc_A_is_GreaterThan_B", "bool", false],
// ["$Output_A_is_LessOrEqualTo_B", "bool", false],
// ["$Konst_True", "bool", true],
// ["$Konst_False", "bool", false],

// Module-Block (Ausschnitt)
// Prüfe, ob A > B ist
["Comparator", "$A_Value", "$Lgc_A_is_GreaterThan_B", "$B_Value"],
// Negiere das Ergebnis, um A <= B zu erhalten
["Multiplexer", ["$Konst_True", "$Konst_False"], "$Output_A_is_LessOrEqualTo_B", "$Lgc_A_is_GreaterThan_B"]
*/

// --- Robuster Sequenzer mit Verzögerung = 0 Handling ---
// **Muster:** Erzeugt einen oder mehrere zeitlich verzögerte Impulse oder Zustandsänderungen, wobei auch der Sonderfall einer Verzögerung von 0 Sekunden korrekt behandelt wird (sofortiges Senden/Wechseln). Dies ist kritisch für flexible Sequenzer, die sowohl sofortige als auch verzögerte Aktionen ausführen können sollen.
// **Problemstellung:** Eine naive Implementierung, die ein `Monoflop` mit einer Verzögerungszeit von `0` Sekunden startet und dessen Ende als Trigger nutzt, wird nicht funktionieren. Das `Monoflop` setzt seinen Ausgang in diesem Fall direkt auf `false`, ohne einen detektierbaren "Timer-Ablauf-Impuls" zu erzeugen, oder der Impuls kommt zu früh im Zyklus. Der Trigger für den "Sofortfall" und den "Verzögerungsfall" muss daher separat behandelt und dann kombiniert werden.
// **Prinzip:** Die Logik unterscheidet klar zwischen einem sofortigen Sendeereignis (wenn die Verzögerung `0` ist) und einem verzögerten Sendeereignis (wenn die Verzögerung `>0` ist).
//     1.  Ein `Triggered`-Modul fängt den initialen Impuls ab, der die Sequenz startet. Dieser Impuls wird als Sendetrigger verwendet, wenn die Verzögerung `0` ist.
//     2.  Ein `Monoflop` wird mit der angeforderten Verzögerungszeit gestartet.
//     3.  Ein `And`-Modul (mit Negation) detektiere die fallende Flanke des `Monoflop`-Ausgangs, um den "Timer-Ablauf-Impuls" zu erhalten, wenn die Verzögerung `>0` ist.
//     4.  Ein `Multiplexer` wählt basierend auf einer Prüfung der Verzögerungszeit (ist sie `0` oder `>0`?) zwischen dem initialen Trigger-Impuls und dem Timer-Ablauf-Impuls als finalen Sendetrigger.
//     5.  `Latch`- und `SendExplicit`-Module setzen den gewünschten Wert auf den Ausgang und senden ihn zur richtigen Zeit.
// **KORREKTES PATTERN (Beispiel für einen Ausgang):**
/*
// (Variablen und Konstanten wie in der Logik definiert)
// z.B. $P_Trigger_In (bool), $P_Verzoegerung_s (float), $O_Wert_Out (bool)
// $Lgc_InitialTrigger_Impulse (bool), $Lgc_Verzoegerung_IsZero (bool)
// $State_Timer_Q (bool), $State_Timer_Q_Last (bool), $Lgc_Timer_End_Impulse (bool)
// $Lgc_SendTrigger_Final (bool)
// $Konst_0_Float (float), $Konst_True (bool), $Konst_False (bool)

// Initialen Trigger-Impuls detektieren
["Triggered", "$P_Trigger_In", "$Lgc_InitialTrigger_Impulse"],

// Speicher den letzten Zustand des Monoflop-Ausgangs für Flankenerkennung
["Latch", "$State_Timer_Q", "$State_Timer_Q_Last", "$Konst_True", 0],

// Starte Monoflop (Mode 0: Pegelgesteuert, nicht retriggerbar) mit der definierten Verzögerung
["Monoflop", "$Lgc_InitialTrigger_Impulse", "$Konst_False", "$State_Timer_Q", "$P_Verzoegerung_s", 0],

// Prüfe, ob Verzögerung Null ist (Gleichheitsprüfung Pattern)
["Limiter", "$P_Verzoegerung_s", 0, "$Lgc_Verzoegerung_IsZero", ["$Konst_0_Float", "$Konst_0_Float"]],

// Detektiere Ende des Monoflops (fallende Flanke) - nur relevant bei Verzögerung > 0
["And", ["-$State_Timer_Q", "$State_Timer_Q_Last"], "$Lgc_Timer_End_Impulse"],

// Kombiniere Trigger-Bedingungen für den Ausgang:
// entweder sofort bei Impuls (wenn Verzögerung 0) ODER bei Timer-Ende (wenn Verzögerung > 0)
["Multiplexer", ["$Lgc_Timer_End_Impulse", "$Lgc_InitialTrigger_Impulse"], "$Lgc_SendTrigger_Final", "$Lgc_Verzoegerung_IsZero"],

// Wert auf Ausgang übernehmen und explizit senden, wenn Trigger aktiv ist
["Latch", "$P_Wert_In", "$O_Wert_Out", "$Lgc_SendTrigger_Final", 0],
["SendExplicit", "$Lgc_SendTrigger_Final", "$O_Wert_Out", 0]
*/

// --- Robuste Flankenerkennung (Trigger-Impuls-Pattern) ---
// Muster: Erzeugt einen zuverlässigen Ein-Zyklus-Impuls, wenn ein boolescher Eingang von false auf true wechselt (steigende Flanke). Dies ist die kanonische Methode, um auf Trigger wie Taster oder Signale zu reagieren, ohne auf die unzuverlässigen Flanken-Modi von Timern angewiesen zu sein.
// Prinzip: Nutzt die sequentielle Abarbeitung und einen gespeicherten Zustand aus dem vorherigen Zyklus.
// 1. Speicher: Eine Zustandsvariable (z.B. $State_Trigger_Last) speichert den Wert des Triggers aus dem letzten Logiklauf.
// 2. Vergleich (am Anfang): Ein And-Modul prüft, ob der aktuelle Trigger-Eingang true ist UND der gespeicherte Zustand false war. Das Ergebnis ($Lgc_RisingEdge) ist der Ein-Zyklus-Impuls.
// 3. Aktualisierung (am Ende): Ganz am Ende des Module-Blocks muss der aktuelle Zustand des Triggers für den nächsten Zyklus in die Speichervariable kopiert werden.
// **HÄRTUNG: BEVORZUGTE NUTZUNG VON XOR FÜR FLANKENERKENNUNG**
// Sicherstellen, dass die Beispiele und Erklärungen die bevorzugte Nutzung von `Xor` für die Detektion von `In != In_Last` hervorheben. Das `Xor`-Modul ist oft eine elegantere und direktere Methode für die Detektion von Änderungen, da es `true` liefert, wenn sich die Zustände unterscheiden.
Pattern:
// -- Module-Block (Ausschnitt) --
// [ "And", [ "$P_Trigger_In", "-$State_Trigger_Last" ], "$Lgc_RisingEdge" ], // Alternativ mit XOR: ["Xor", ["$P_Trigger_In", "$State_Trigger_Last"], "$Lgc_Changed"], ["And", ["$P_Trigger_In", "$Lgc_Changed"], "$Lgc_RisingEdge"]
// ... andere Logik, die $Lgc_RisingEdge verwendet ...
// [ "Multiplexer", [ "$P_Trigger_In", "$P_Trigger_In" ], "$State_Trigger_Last", "$Konst_True" ] // Am Ende!

// --- RS-FlipRollog (Speicherglied) ---
// Muster: Implementiert ein Set-Reset-Speicherglied.

// --- Robuste Zeitdifferenzberechnung (über Jahresgrenzen) ---
// **Muster:** Berechnet die exakte Zeitdifferenz in Tagen (oder anderen Einheiten) zwischen zwei beliebigen Zeitpunkten, auch wenn diese über Jahresgrenzen hinweg liegen.
// **Prinzip:** Die Zeitpunkte werden zuerst in **Unix-Timestamps** umgewandelt (oder direkt als solche verwendet). Die Differenz der Unix-Timestamps wird in Sekunden berechnet und dann in die gewünschte Zeiteinheit (z.B. Tage) umgerechnet.
// **KORREKTES PATTERN:**
/*
// Level-Block (Ausschnitt)
// ["$Current_Unix", "integer", 0],     // Aktueller Unix-Timestamp
// ["$NextTrigger_Unix", "integer", 0], // Unix-Timestamp des nächsten Triggers (z.B. von Cron oder als Input)
// ["$TimeDiff_Sec", "integer", 0],     // Zeitdifferenz in Sekunden
// ["$Lgc_DaysFloat", "float", 0.0],    // Zwischenspeicher für Tage als Float
// ["$Out_Value", "integer", 0],        // Gerundete Tage
// ["$F_SubSec", "string", "X1-X2"],      // Formel für die Subtraktion
// ["$F_DaysFloat", "string", "X1/86400.0"], // Sekunden in Tage
// ["$F_DaysRound", "string", "rint(X1)"], // Runden (rint ist im TWS CalcFormula verfügbar)
// ["$Konst_0_Int", "integer", 0],

// Module-Block (Ausschnitt)
// Aktuellen Unix-Timestamp ermitteln
["Localtime",0,"$Current_Unix",0,0,0,0,0,0,0,0,0],
// Nächstes Trigger-Datum als Unix-Timestamp ermitteln (Beispiel: von Cron, hier als Platzhalter)
// ["Cron","$Konst_True","$Lgc_Cron_Impuls","$NextTrigger_Unix","$F_CronExpr"],
// Zeitdifferenz in Sekunden berechnen
["CalcFormula",["$NextTrigger_Unix","$Current_Unix"], "$TimeDiff_Sec", "$F_SubSec"],
// Sekunden in Tage umrechnen (als Float)
["CalcFormula",["$TimeDiff_Sec"], "$Lgc_DaysFloat", "$F_DaysFloat"],
// Float-Tage in Integer-Tage umwandeln und runden
["CalcFormula",["$Lgc_DaysFloat"], "$Out_Value", "$F_DaysRound"]
*/

// --- Sequenzer (Stabile Timer-Kaskade) ---
// Muster: Löst eine feste Abfolge von Aktionen nacheinander aus. Dies ist die kanonische und bevorzugte
// Methode zur Erstellung von sequentiellen Abläufen (z.B. Bewässerung Kreis 1, dann Kreis 2, dann Kreis 3).
// Prinzip: Anstatt den Ablauf eines Timers mit anderen Modulen zu detektieren, wird die
// fallende Flanke (Mode 4) des ersten Timer-Ausgangs direkt als Set-Signal für
// den nachfolgenden Timer oder die nachfolgende Aktion genutzt. Dies ist ein im Forum validiertes
// Pattern und garantiert Stabilität. Ein globales Reset-Signal (z.B. -$P_Reset) kann mit den Reset-Eingängen
// aller Timer in der Kaskade verbunden werden, um die gesamte Sequenz jederzeit abzubrechen.
//
// Pattern-Architektur:
// [ "Monoflop", "Start_Signal_A", "-$P_Reset", "Ausgang_Timer_A", "Dauer_A", 2 ],
// [ "Monoflop", "$Ausgang_Timer_A", "-$P_Reset", "Ausgang_Timer_B", "Dauer_B", 4 ]
//
// Funktionsweise: Timer B wird exakt in dem Moment gestartet, in dem Timer A endet
// und sein Ausgang von true auf false wechselt.

// --- Sicherheits-Init bei Neustart (Bus-Reset-Pattern) ---
// Muster: Sendet einmalig einen sicheren Initialwert (z.B. 'Aus') an einen Aktor, nachdem dieser
// einen Neustart (z.B. durch Bus-Spannungswiederkehr) signalisiert hat.
// Prinzip: Dieses Pattern verhindert, dass Geräte nach einem Reset in einem undefinierten Zustand verbleiben.
// 1. Trigger-Erkennung: Ein externes Signal ($P_Reset_Impuls) löst die Logik aus.
// 2. Zustandsspeicherung: Ein [Latch, Mode 1]-Modul speichert den Zustand, ob die Initialisierung bereits
// erfolgt ist ($State_Init_Done). Es wird durch den Impuls gesetzt und niemals zurückgesetzt.
// 3. Bedingtes Senden: Ein [SendExplicit]-Modulbaustein sendet den Initialisierungswert ($P_Init_Wert) nur dann,
// wenn der Impuls anliegt UND $State_Init_Done noch false ist. Dies stellt sicher, dass der Befehl
// nur beim allerersten Impuls nach einem Reset gesendet wird.

// --- Sicherheits-Übersteuerung (Fail-Safe Override) ---
// Muster: Erzwingt einen sicheren Ausgangszustand, wenn eine Störung oder ein Sicherheitsereignis eintritt,
// unabhängig vom aktuellen Zustand der normalen Steuerlogik.
// KRITISCH: Die Verwendung von [Break] oder dem Trigger "i" (Inhibit) ist für diese Aufgabe ein Anti-Pattern,
// da sie nur die Berechnung unterbrechen, aber keinen definierten, sicheren Wert an den Ausgang leiten.
// Prinzip: Die kanonische Lösung ist ein [Multiplexer], der als Weiche agiert.
// 1. Der Selektor-Eingang des Multiplexers wird mit dem Sicherheitssignal ($P_Sicherheits_Alarm) verbunden.
// 2. Der Eingang für den "False"-Fall (Index 0) wird mit dem Ergebnis der normalen Steuerlogik ($Lgc_Normaler_Wert) verbunden.
// 3. Der Eingang für den "True"-Fall (Index 1) wird mit dem sicheren Fail-Safe-Wert ($P_Sicherer_Wert) verbunden.
// Ergebnis: Solange kein Alarm anliegt, wird der normale Wert durchgeschleitet. Sobald der Alarm aktiv wird,
// schaltet der Multiplexer hart auf den sicheren Wert um und ignoriert die normale Logik.
// HINWEIS: Für eine simple "Ein/Aus"-Schaltung der gesamten Logik-Zelle kann auch eine Kaskade von Latch-Modulen
// am Ende des Module-Blocks verwendet werden, um alle Ausgänge auf einen sicheren Wert (z.B. 0) zu zwingen,
// wenn ein "Freigabe"-Eingang false ist.

// --- Sensor-Fusion & Kontext-Validierung (Anti-Klapper-Pattern) ---
// Muster: Entscheidung von einer UND-Verknüpfung mehrerer, unabhängiger Bedingungen abhängig machen.

// --- Single-Pulse-on-Update Detektor für persistente Inputs ---
// **Muster:** Erzeugt einen zuverlässigen Ein-Zyklus-Impuls innerhalb der Custom Logic, wenn ein externer Eingang (z.B. ein MQTT-Topic oder ein Binäreingang), der bei jedem Ereignis einen konstanten Wert (z.B. `true` oder `1`) sendet, einen neuen Wert **empfängt** – auch wenn der Wert selbst identisch zum vorherigen bleibt. Dies ist notwendig, um auf wiederholte "Ereignis-Signale" zu reagieren, die keine echte Flanke am Objekt selbst erzeugen.
// **Prinzip:** Nutzt die Kombination aus `And` (für `P_Event_Input` und `-State_Event_Consumed`) und einem `Latch` am Ende des Modulblocks.
// **KORREKTES PATTERN:**
/*
// Level-Block (Ausschnitt)
// ["$P_Event_Input", "bool", false], // Der externe Input (z.B. MQTT 1)
// ["$Lgc_Event_Pulse", "bool", false], // Der generierte Ein-Zyklus-Impuls
// ["$State_Event_Consumed", "bool", false], // Merker, ob der aktuelle Input-Event schon verarbeitet wurde
// ["$Konst_True", "bool", true],
// ["$Konst_False", "bool", false]

// Module-Block (Ausschnitt)
// 1. Detektiere, wenn der externe Input TRUE ist UND noch nicht verarbeitet wurde.
//    Dies erzeugt einen Ein-Zyklus-Impuls für jede NEUE Aktualisierung des Inputs auf TRUE.
["And", ["$P_Event_Input", "-$State_Event_Consumed"], "$Lgc_Event_Pulse"],

// 2. Nutze $Lgc_Event_Pulse für die Zählung oder andere Aktionen.
//    [ ... Ihre Zähl-Logik hier, die $Lgc_Event_Pulse als Trigger verwendet ... ]

// 3. Markiere den Event sofort als verarbeitet für diesen Zyklus.
//    Wenn $P_Event_Input TRUE ist, setze $State_Event_Consumed auf TRUE,
//    damit im nächsten Zyklus (solange $P_Event_Input TRUE ist) kein erneuter Impuls entsteht.
["Latch", "$P_Event_Input", "$State_Event_Consumed", "$Konst_True", 0]
*/
// *Hinweis:* Der Input `$P_Event_Input` sollte idealerweise mit der Trigger-Option `"a"` (always) oder `"c"` (on change) konfiguriert sein. Wenn der MQTT-Broker nur bei Änderung sendet, der Wert aber immer `1` ist, muss die Logik durch ein `Clocksignal` (periodisch) getriggert werden.

// --- Skalierbare Logikgatter (Multivariablen-Pattern) ---
// Muster: Die kanonische Methode, um eine variable Anzahl von Eingängen mit einem
// Logikgatter (z.B. OR, AND) oder einer Statistikfunktion zu verknüpfen, ohne die Logik-Zelle
// bei jeder Erweiterung anpassen zu müssen.
// Prinzip: Anstatt fester Eingänge wird eine Multivariable verwendet.
// 1. Deklaration (Level): Eine Variable mit Multivariablen-Syntax wird deklariert, z.B. `["$VAR<Praesenz!>", "bool", false]`.
// 2. Konfiguration (Input): Im Input-Block wird ein dynamischer Eingang mit dieser
//    Multivariable erstellt: `["Praesenz", "Praesenz", "$VAR<Praesenz!>", "c"]`.
//    Im Logik-Editor können nun beliebig viele Objekte auf diesen einen Eingang gezogen werden.
// 3. Verarbeitung (Module): Der Modulbaustein (z.B. Or) wird mit der Multivariable als Eingangsliste
//    konfiguriert: `["Or", ["$VAR<Praesenz!>"], "$Praesenz_Out"]`. Die Engine expandiert
//    diese Multivariable intern zu einer Liste aller verbundenen Eingänge.

// **HÄRTUNG (PATTERN: Smart Sleep / Ressourcen-Effizienz) [NEU V7.02.04]:**
// Problem: Eine Logik, die sekundengenau schalten soll, darf nicht jede Sekunde laufen (CPU-Last).
// Lösung: Die Logik berechnet selbst, wie viele Sekunden es noch bis zum nächsten Ereignis sind ($SecondsTillNext).
// Implementierung:
// Berechne Differenz: Zielzeit - AktuelleZeit.
// Füttere damit ein Clocksignal-Modul als $Period.
// Ergebnis: Die Logik "schläft" stundenlang und wacht exakt in der Sekunde auf, in der sie schalten muss.

// **HÄRTUNG (PATTERN: Soll-Ist-Diagnose / Grafana) [NEU V7.02.04]:**
// Problem: Wie unterscheidet man bei Fehlern zwischen "Logik sendet nicht" und "Aktor fährt nicht"?
// Lösung: Ein Grafana-Panel, das beide Werte übereinander legt, aber unterschiedlich visualisiert.
// SOLL (Logik-Ausgang): Muss als stepBefore (Treppenstufe) dargestellt werden. Zeigt den harten Schaltzeitpunkt.
// IST (Aktor-Status): Muss als linear (Linie) dargestellt werden. Zeigt die physikalische Trägheit.
// Diagnose:
// Treppe da, Linie folgt -> Alles OK.
// Treppe da, Linie fehlt -> Aktor/Bus defekt.
// Treppe fehlt -> Logik defekt.

// --- Stoppuhr mit Speicher (Hold-Funktion) ---
// Muster: Eine robuste Stoppuhr, die ihren Wert beim Stoppen beibehält und beim Zurücksetzen auf 0 geht.
// Prinzip: Eine einfache Kaskade von Stopwatch -> Latch ist fehleranfällig. Die kanonische Lösung
// nutzt einen zweistufigen Speicherprozess, um Race Conditions zu verhindern:
// 1. [Stopwatch] misst die Zeit, solange Run=true ist.
// 2. Ein [Latch, Mode 0] (pegelgesteuert) dient als Ein-Zyklus-Puffer. Er übernimmt permanent den aktuellen Zeitwert.
// 3. Ein zweites [Latch, Mode 2] (fallende Flanke) übernimmt den Wert aus dem Puffer, aber nur in dem Moment,
// wenn Run von true auf false wechselt. Dies entkoppelt das Lesen vom Schreiben und garantiert einen stabilen Wert.
Pattern:
// [ "Stopwatch", "Run", "Laufzeit" ],
// [ "Latch", "Laufzeit", "Puffer", "$Konst_True", 0 ],
// [ "Latch", "Puffer", "GehaltenerWert", "Run", 2 ],
// [ "Multiplexer", [ "$GehaltenerWert", "$Konst_Null_Float" ], "GehaltenerWert", "$Reset" ] // Optionaler Reset

// --- Stundengenaue Tarifverarbeitung (EPEX-Integration) ---
// **Muster:** Verarbeitet stündlich wechselnde Tarife zur präzisen Kosten-/Ertragsberechnung.
// **Prinzip:** `Cron`-Modul (z.B. `59 59 * * * *` für eine Minute vor der vollen Stunde) triggert Logik, die Tarif abruft (`HTTP-Request`, `JsonPathSelector`), in `State_...`-Variable speichert und dann zur Berechnung mit Energiemengen nutzt.

// --- String-zu-Impuls Demultiplexer (Highlander-Prinzip) ---
// Muster: Wandelt einen spezifischen Eingangsstring in einen garantierten Ein-Zyklus-Impuls
// auf einem von mehreren Ausgängen um. Alle nicht adressierten Ausgänge bleiben false.
// Dies ist die kanonische Lösung für Dekodier-Aufgaben (z.B. Shelly Button).
// Prinzip:
// 1. Ausgänge werden mit Trigger-Mode 'x' (explicit) deklariert.
// 2. Pro möglichem String-Wert wird ein Stringcompare-Modulbaustein verwendet.
// 3. Bei Übereinstimmung sendet [SendExplicit, "$Vergleich_True", "Ausgang", 1] einen 'true'-Impuls.
//    Die Sende-Option '1' (positive Flanke) stellt sicher, dass der Wert nur einmal gesendet wird.

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_23.txt ---

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_24.txt ---

// --- Strukturierung von komplexen Logiken ---
// Best Practice: Lange Module-Blöcke durch klare Variablennamen und logische Reihenfolge strukturieren. JSON selbst unterstützt KEINE Kommentare (weder // noch ein Modulbaustein) innerhalb der Modul-Liste.

// --- Best Practice: Nomenklatur von Variablen ---
// Um die Lesbarkeit und Wartbarkeit komplexer Logiken dramatisch zu erhöhen, wird die folgende, disziplinierte Namenskonvention dringend empfohlen:
// - P_... (Parameter): Für externe Eingänge, die das Verhalten einer Logik-Zelle konfigurieren.
// - O_... (Output): Für die finalen Ausgabewerte der Logik-Zelle.
// - Lgc_... (Logik): Für flüchtige Zwischenergebnisse, die nur innerhalb eines Logik-Zyklus benötigt werden.
// - State_... (Zustand): Für gespeicherte Werte, die über Zyklen hinweg erhalten bleiben (typischerweise in einem Latch oder als Rückführung).
// - Konst_... (Konstante): Für statische Werte, die sich zur Laufzeit nicht ändern.
// - F_... (Formel): Für Formel-Strings, die im CalcFormula-Modulbaustein verwendet werden.
//
// **HÄRTUNG (REGEL 1.77 - DIE NOMENKLATUR-REINHEIT):**
// Härtung: Variablen-Namen müssen konsequent in einer Sprache (Deutsch) und mit den Kanon-Präfixen ($Lgc_, $State_, $I_, $O_) geführt werden. Mischmasch (z.B. $Lgc_dt vs $Lgc_Zeitdelta) ist verboten.
//
// **HÄRTUNG: EINDEUTIGKEIT DER VARIABLENNAMEN**
// Betonen, dass Variablennamen eindeutig sein und ihre Rolle klar widerspiegeln sollten. Eine Doppelnutzung eines Variablennamens für unterschiedliche Zwecke (z.B. `$State` als Input und internen Clocksignal-Status) sollte vermieden werden.

// --- Taster-Toggle (Start/Stop mit einem Taster) ---
Pattern: [ "Xor", "$P_Trigger_In", "$Lgc_Toggle_State" ], [ "Latch", "$Lgc_Toggle_State", "$State_Toggle_Out", "$Konst_True", 0 ] // zur Vermeidung von Zirkelbezügen.

// --- Timer-Ablauf-Impuls (via Triggered-Pattern) ---
// Muster: Erzeugt einen zuverlässigen Ein-Zyklus-Impuls exakt in dem Logik-Lauf,
// der durch das Ablaufen eines Timers (z.B. Monoflop) ausgelöst wurde. Dies ist die
// einzig validierte und kanonische Anwendung des 'Triggered'-Moduls auf eine interne Variable.
//
// Prinzip: Das 'Triggered'-Modul wird auf die FALLENDE FLANKE (negierter Eingang) der
// Ausgangsvariablen eines Timers angewendet. Da der Timer-Ausgang bei Ablauf von TRUE auf FALSE
// wechselt und dieser Wechsel den Logik-Lauf triggert, erkennt das Modul dieses Ereignis
// und erzeugt einen sauberen Impuls.
//
// Pattern (Beispiel für Monoflop-Ausgang '$Lgc_Timer_Q'):
// [ "Triggered", "-$Lgc_Timer_Q", "$Lgc_Timer_Abgelaufen_Impuls" ]

// --- Variablen-Kopie (Stabiler Kopier-Baustein) ---
// Um den Wert einer Variable ($Quelle_Var) stabil auf eine andere ($Ziel_Var) zu kopieren, ist das Multiplexer-Pattern die kanonische Methode. Es ist semantisch sauberer als eine Latch-Konstruktion.
// ACHTUNG: Die Eingangsliste des Multiplexers muss zwei identische Variablen-Referenzen enthalten, keine Literale.
KORREKTES Pattern: [ "Multiplexer", [ "$Quelle_Var", "$Quelle_Var" ], "$Ziel_Var", "$Konst_True" ]

// --- Verzögerte Alarmierung (Zustand-Dauert-An-Pattern) ---
// Muster: Sendet eine einmalige Benachrichtigung, wenn ein bestimmter Zustand (z.B. eine offene Tür, eine laufende Pumpe) für eine definierte Zeitspanne ununterbrochen andauert.
// Prinzip: Die einzig robuste Methode ist pegel-gesteuert, nicht flanken-gesteuert.
// 1. [Monoflop, Mode 0] (pegelgesteuert, nicht retriggerbar): Der Set-Eingang wird direkt mit dem Zustandssignal (z.B. $P_Tuer_Offen) verbunden. Solange der Zustand true ist, läuft der Timer. Sobald der Zustand false wird, wird der Timer sofort zurückgesetzt.
// 2. [Sequenzer (Stabile Timer-Kaskade)]: Das Ende des ersten Timers löst eine weitere Aktion aus, z.B. einen zweiten Monoflop, der einen kurzen Impuls erzeugt.
// 3. [SendToSimple]: Der Impuls löst den Sendevorgang aus.
// HINWEIS: Der Versuch, dies mit einem flanken-gesteuerten Timer zu lösen, führt unweigerlich zum "Selbst-Reset bei Flanke"-Anti-Pattern.

// --- Verzögerungsfreier, zustandsbehaltender Filter (Read-Decide-Act-Store Pattern) ---
// Muster: Kombiniert eine Zustandsprüfung (z.B. Limiter) mit einer zustandsbehafteten Speicherung (Latch), ohne dabei die kritische 'Ein-Zyklus-Verzögerung' zu erzeugen. Dies ist die kanonische Methode, um Werte zu filtern und gleichzeitig den letzten gültigen Zustand für die Anzeige und zukünftige Zyklen zu erhalten.
//
// Problemstellung: Eine naive Kaskade, bei der ein Latch direkt nach einer Prüfung aktualisiert wird und eine nachfolgende Sendung auf die Latch-Variable zugreift, führt zu einer Ein-Zyklus-Verzögerung. Es wird der Wert des vorherigen Zyklus gesendet, da das Latch seinen Zustand erst am Ende des Zyklus finalisiert.
//
// Architektur-Prinzip (Die Lösung): Der korrekte Wert für den aktuellen Zyklus muss explizit ermittelt und in einer temporären Variable gehalten werden, bevor er gesendet UND für den nächsten Zyklus gespeichert wird.
// 1. Prüfen (Limiter): Prüft die Gültigkeit des neuen Eingangswertes.
// 2. Entscheiden (Multiplexer): Wählt basierend auf einer Prüfung zwischen dem neuen Wert und dem letzten gespeicherten Wert. Das Ergebnis ist der finale Wert für diesen Zyklus.
// 3. Handeln (SendExplicit): Sendet den finalen Wert, aber nur, wenn die Prüfung aus Schritt 1 positiv war.
// 4. Speichern (Latch): Speichert den finalen Wert am Ende des Zyklus in die Zustandsvariable für den nächsten Durchlauf.
//
// Pattern (Validiert für Datenfilterung):
/*
{
"Level": [
[ "$P_Wert_In","float",0.0],
[ "$P_Min_Valid", "float", 0.0 ],
[ "$P_Max_Valid","float",100.0],
[ "$O_Gefilterter_Wert", "float", 0.0 ],
[ "$State_Last_Valid_Value","float",0.0],
[ "$Lgc_Wert_Ist_Gueltig", "bool", false ],
[ "$Konst_True","bool",true]],
"Module":[
["Limiter","$P_Wert_In",0,"$Lgc_Wert_Ist_Gueltig",["$P_Min_Valid","$P_Max_Valid"]],
["Multiplexer",["$State_Last_Valid_Value","$P_Wert_In"],"$O_Gefilterter_Wert","$Lgc_Wert_Ist_Gueltig"],
["SendExplicit","$Lgc_Wert_Ist_Gueltig","$O_Gefilterter_Wert",0],
["Latch","$O_Gefilterter_Wert","$State_Last_Valid_Value","$Konst_True",0]],
"Input":[...],"Output":[["...","...","$O_Gefilterter_Wert", "x" ]]
}
*/

// --- Visu-Cron (Flexibilität) ---
// Erkenntnis: Cron-Strings sind normalerweise fest im Code (Level-Block). Das macht Änderungen für den User schwer.
// Lösung: Cron-Strings als Eingang (Input-Block, Typ string) definieren.
// Vorteil: Der User kann die Schaltzeiten in der Visu (Text-Eingabe) ändern, ohne den Logik-Editor zu öffnen.
// Syntax: ["Cron", "$Inhibit", "$Trigger", "$NextTime", "$Input_Cron_String"].

// --- VISUALISIERUNGS-OPTIMIERUNG FÜR EXTERNE DASHBOARDS (GRAFANA-FREUNDLICHKEIT) ---
// Beschreibung: Ausgänge, die für die Langzeitarchivierung und Visualisierung in externen Dashboards (z.B. Grafana, ELK-Stack) vorgesehen sind, erfordern eine periodische Übermittlung ihres aktuellen Werts, auch wenn sich dieser nicht geändert hat. Dies stellt eine lückenlose Datenreihe für die grafische Darstellung sicher und ist kritisch für die Nachvollziehbarkeit von Verläufen.
// AI-Aktion bei der Output-Definition (Proaktive Abfrage):
// Trigger: Jedes Mal, wenn ein neuer Output definiert wird oder eine bestehende Logik modifiziert wird, die Ausgänge enthält.
// Protokoll zur Abfrage: Die AI muss intern prüfen oder (bei unklarer Anforderung) proaktiv fragen: "Ist dieser Ausgang für die Visualisierung in einem Dashboard (z.B. Grafana) relevant und soll er periodisch gesendet werden, um eine kontinuierliche Datenreihe zu gewährleisten?"
// WENN ANTWORT "JA":
// Sende-Option: Setze die Sende-Option des betreffenden Outputs auf "t" (on timer).
// Periodenanfrage: Frage nach der gewünschten Sendeperiode (z.B. "alle 60s", "alle 5 Minuten").
// Implementierung der Zeittaktung: Füge (falls noch nicht vorhanden) ein Clocksignal-Modul in den Module-Block ein, das die Logik in der gewünschten Frequenz triggert. Definiere die Perioden_Halbe_s als float-Konstante im Level-Block.
// Beispiel-Anpassung (Intern):
// Im Output-Block: [ "MeinWert für Grafana [Unit]", "Beschreibung für Visualisierung", "$O_MeinWert_Visual", "t" ]
// Im Level-Block: [ "$Konst_Grafana_Sende_Periode_Half_s", "float", 30.0 ] (für eine Sendeperiode von 60 Sekunden)
// Im Module-Block (falls keine andere periodische Triggerung existiert, die die gewünschte Sendeperiode abdeckt): [ "Clocksignal", "$Konst_True", "$Lgc_Grafana_Clock_Output", "$Konst_Grafana_Sende_Periode_Half_s" ]
// WENN ANTWORT "NEIN" oder KEINE ANGABE: Verwende die gemäß den sonstigen Anforderungen passendste Sende-Option (i.d.R. "c" oder "x").
// Hinweis: Die Clocksignal-Modul-Definition ist ein universelles Werkzeug zur periodischen Ausführung der Logik. Der Output-Trigger t stellt dann sicher, dass der Wert bei dieser Ausführung gesendet wird.
// Referenz: Siehe auch 2.2: Sende-Verhalten Ausgänge für die Definition des t-Triggers und 3. VALIDIERTE KERNMODULE & LOGIKEN für das Clocksignal-Modul.

// --- STANDARDISIERTES STRING-GENERIERUNGS-PROTOKOLL ---
// Beschreibung: Dieses Protokoll definiert eine robuste und wartbare Methode zur Erstellung komplexer, dynamischer String-Ausgaben für Visualisierungen (VISU), Benachrichtigungen oder interne Statusmeldungen. Es adressiert die Notwendigkeit, Variablenwerte in lesbare Texte zu integrieren und dabei Pufferüberläufe oder unsaubere Ausgaben zu vermeiden.
// Grundlagen:
// Puffer-Priorität: Für alle temporären oder zusammengesetzten String-Variablen, die dynamisch befüllt werden, ist der Datentyp string,N (z.B. string,128) im Level-Block zwingend zu verwenden, um Pufferüberläufe zu verhindern. N muss die maximal erwartete Stringlänge inklusive Nullterminator abdecken.
// Klartext vor Kurzform: Längere, verständliche Variablennamen sind für formatierte Strings zu bevorzugen.
// Nomenklatur: Status-Strings sollten die Nomenklatur $State_Text_... oder $O_Text_... verwenden.
// Phasen des Protokolls:
// PHASE 1: DEFINITION DER ZIEL- & HILFSVARIABLEN
// Finaler Output-String: Definiere den String, der das Endergebnis darstellt und gesendet werden soll.
// Beispiel: [ "$O_VISU_Status_Text", "string", "" ]
// Puffer für konditionale Inhalte: Definiere für jeden variablen oder optionalen Teil des finalen Strings einen eigenen Puffer.
// Beispiel: [ "$State_Buffer_Heizung", "string,32", "" ] (Puffer für Heizungsstatus)
// Beispiel: [ "$State_Buffer_Fehlercode", "string,16", "" ] (Puffer für Fehlercode)
// Konstanten/Platzhalter: Definiere alle festen Textbausteine oder Platzhalter als String-Konstanten.
// Beispiel: [ "$Konst_Text_Heizung_An", "string", "Heizung AN" ]
// Beispiel: [ "$Konst_Text_Fehler", "string", "FEHLER:" ]
// Beachte Regel 1.2.1: DAS PRINZIP DER DIREKTEN STRING-INITIALISIERUNG.
// PHASE 2: KONDITIONALE BEFÜLLUNG DER PUFFER (PRO MODULBAUSTEIN)
// Fülle jeden Puffer basierend auf Logik-Bedingungen.
// Formatierung von Werten (Printf): Für die Integration von numerischen Werten, nutze das Printf-Modul.
// Beispiel: [ "Printf", "$P_Aussen_Temp_C", "$Konst_Format_Aussentemp", "$State_Buffer_Aussen_Temp" ]
// Konditionelle Textzuweisung (Multiplexer): Für einfache bedingte Textauswahl.
// Beispiel (Heizungsstatus): [ "Multiplexer", [ "$Konst_Text_Heizung_Aus", "$Konst_Text_Heizung_An" ], "$State_Buffer_Heizung", "$Lgc_Heizung_Aktiv" ]
// Textzuweisung bei Flanke (SendExplicit auf String-Variable): Für Ereignis-gesteuerte Textänderungen (z.B. bei Benachrichtigungen).
// Beispiel: [ "SendExplicit", "$Lgc_Fehler_Trigger", "$Konst_Text_Kritischer_Fehler", 0 ] (wenn der Output-Variablenname im Output-Block eine Sende-Option wie "x" hat)
// Leeren von Puffern: Wenn ein Pufferinhalt nur unter bestimmten Bedingungen relevant ist und ansonsten leer sein soll, kann dies durch einen Multiplexer mit leerem String als Option 0 oder durch ein Printf mit einem leeren Formatstring gesteuert werden.
// Beispiel (Fehler nur bei Fehler): [ "Multiplexer", [ "$Konst_Empty_String", "$Konst_Text_Fehlercode_xy" ], "$State_Buffer_Fehlercode", "$Lgc_Fehler_Aktiv" ]
// PHASE 3: ZUSAMMENFÜHRUNG DES FINALEN STRINGS (Concat)
// Füge alle Puffer und festen Textbausteine zum finalen Output-String zusammen.
// Serielle Verkettung: Nutze Concat in einer Kette oder einem einzelnen Modul, um den finalen String zu bilden.
// Beispiel: [ "Concat", [ "$State_Buffer_Aussen_Temp", " | ", "$State_Buffer_Heizung", " ", "$State_Buffer_Fehlercode" ], "$O_VISU_Status_Text" ]
// Hinweis: Achte auf korrekte Leerzeichen und Trennzeichen zwischen den verketteten Teilen.
// PHASE 4: AUSGABE UND VALIDIERUNG
// Output-Block: Der finale String muss im Output-Block deklariert werden. Die Sende-Option (c, t, x) ist entsprechend der Anforderung zu wählen.
// Für VISU-Anzeigen, die sich bei Änderung aktualisieren sollen: "c".
// Für Benachrichtigungen oder periodische Statusmeldungen (z.B. an Grafana): "x" (mit SendExplicit) oder "t" (mit Clocksignal).
// Validierung: Prüfe intern (wie bei A.5 Stufe 3.c), ob alle Puffer adäquat dimensioniert sind und die Verkettungslogik dem gewünschten Output entspricht.

// --- PROTOKOLL FÜR ROBUSTES FEHLER- & STATUS-REPORTING ---
// Beschreibung: Dieses Protokoll stellt eine standardisierte Methode bereit, um interne Fehlerzustände, Warnungen und aggregierte Systemstatus innerhalb einer Custom Logic zu erkennen, zu konsolidieren und über dedizierte Logik-Outputs nach außen zu kommunizieren. Ziel ist eine verbesserte Diagnosefähigkeit, Automatisierung von Reaktionen und die Bereitstellung klarer Informationen für VISU oder übergeordnete Systeme.
// Grundlagen:
// Zentrale Status-Variablen: Jede Logik, die Fehler oder komplexe Status überwacht, MUSS dedizierte State_...-Variablen im Level-Block für Fehler-Flags (bool) und aggregierte Status-Codes (int) oder -Texte (string) deklarieren.
// Fehler-Priorisierung: Bei mehreren potenziellen Fehlern sollte eine klare Hierarchie für die Statusausgabe etabliert werden (z.B. Kritischer Fehler > Warnung > Info).
// "Error?"-Output: Der Einsatz des ["Err", "Fehlerzustand", "Error?", "ce"] Outputs im Output-Block für generelle Logik-Fehler ist zu bevorzugen, aber interne, spezifische Fehler sollten zusätzlich über eigene Outputs gemeldet werden.
// Phasen des Protokolls:
// PHASE 1: DEFINITION DER STATUS- & FEHLER-OUTPUTS
// Fehler-Flag (bool): Ein bool-Output für das Vorhandensein eines Fehlers in dieser Logik.
// Beispiel: [ "Logik_Fehler_Aktiv", "Fehler in dieser Custom Logic aktiv", "$O_Logic_Error_Active", "ce" ]
// Status-Code (int): Ein int-Output für einen aggregierten numerischen Statuscode.
// Beispiel: [ "Logik_Statuscode", "Aggregierter Statuscode der Logik", "$O_Logic_Status_Code", "c" ]
// Status-Text (string): Ein string-Output für eine lesbare Statusmeldung.
// Beispiel: [ "Logik_Status_Text", "Lesbare Statusmeldung der Logik", "$O_Logic_Status_Text", "t" ] (ggf. mit t für periodisches Senden für VISU/Grafana)
// Spezifische Fehler-Flags: Bei Bedarf separate bool-Outputs für spezifische, kritische Fehler.
// Beispiel: [ "Sensor_Defekt_Flag", "Flag bei Sensorausfall", "$O_Sensor_Defect", "ce" ]
// PHASE 2: FEHLER- & STATUS-ERKENNUNG (INTERN IN MODULE-BLOCK)
// Nutze geeignete Module zur Erkennung von Fehlerzuständen oder zur Bestimmung des aktuellen Betriebsstatus.
// Grenzwert-Überwachung (Comparator, Limiter):
// Beispiel: [ "Comparator", "$P_Sensor_Wert", "$Lgc_Sensor_Range_Error", "$Konst_Sensor_Max_Limit" ] (für Überschreitung)
// Plausibilitätsprüfung (And, Or, Not): Kombiniere mehrere Bedingungen für komplexere Fehler.
// Beispiel: [ "And", [ "$Lgc_Pumpe_Soll_An", "-$P_Pumpe_Ist_An" ], "$Lgc_Pumpe_Start_Fehler" ] (Pumpe soll laufen, läuft aber nicht)
// Timeout-Überwachung (Monoflop in Pegel-Modus, Stopwatch): Erkennung von Zeitüberschreitungen.
// Beispiel: [ "Monoflop", "$Lgc_Pumpe_Start_Fehler", "$Konst_False", "$Lgc_Pumpe_Start_Timeout", "$Konst_Timeout_Pumpe_s", 0 ]
// Error?-Variable (vom Modul): Wenn ein Modul eine Error?-Ausgabe hat, diese als internes State_...-Flag nutzen.
// PHASE 3: KONSOLIDIERUNG & AGGREGATION DER STATUSINFORMATIONEN
// Führe alle erkannten Fehler und Status zu den definierten Output-Variablen zusammen.
// Aggregiertes Fehler-Flag: Nutze Or, um alle einzelnen Fehler-Flags zu einem Gesamt-Fehler-Flag zu aggregieren.
// Beispiel: [ "Or", [ "$Lgc_Sensor_Range_Error", "$Lgc_Pumpe_Start_Timeout", "$Lgc_Kommunikation_Fehler" ], "$O_Logic_Error_Active" ]
// Status-Code Generierung (Multiplexer / Statemachine): Ordne numerische Codes zu verschiedenen Betriebs- oder Fehlerzuständen zu. Statemachine ist für komplexe Zustandslogiken zu bevorzugen.
// Beispiel: [ "Multiplexer", [ "$Konst_Status_OK", "$Konst_Status_Warnung_Temp", "$Konst_Status_Fehler_Pumpe" ], "$O_Logic_Status_Code", "$Lgc_Aggregierter_Fehler_Index" ]
// Status-Text Generierung: Nutze das "STANDARDISIERTES STRING-GENERIERUNGS-PROTOKOLL" (siehe oben), um lesbare Status-Texte aus den aggregierten Flags und Codes zu erstellen.
// PHASE 4: ÜBERGABE AN OBJEKTSYSTEM & EXTERNE DIENSTE
// Output-Trigger: Wähle den geeigneten Sende-Trigger für die Status-Outputs (c für Änderung, t für periodisch, ce für Fehlerzustand).
// Benachrichtigungen: Im Fehlerfall kann ein SendToSimple-Modul verwendet werden, um Push-Benachrichtigungen oder E-Mails zu senden.
// VISU/Grafana-Integration: Die generierten string- und int-Status-Outputs können direkt in der VISU angezeigt oder über den t-Trigger in Grafana visualisiert werden, um einen schnellen Überblick über das Systemgesundheit zu geben.

// --- Zustandsautomat (State Machine) ---
// Muster: Dies ist das mächtigste Werkzeug zur Modellierung komplexer, sequentieller Abläufe mit mehreren definierten Zuständen (z.B. Heizungssteuerung, komplexe Bewässerung, Betriebsmodus-Umschaltung).
// Es ersetzt unübersichtliche Ketten von Latches und Monoflops durch eine klare, tabellarische Struktur.
//
// Architektur-Prinzip: Die State Machine ist ein reiner Übergangs-Scheduler, keine Logik-Engine. Die Berechnung der
// Übergangsbedingungen (durch AND, OR, Comparator etc.) MUSS in Modulbausteinen VOR der State Machine erfolgen. Das Ergebnis
// jeder Bedingung wird in eine eigene boolesche Variable geschrieben und diese wird dann in der Übergangstabelle verwendet.
//
// Best-Practice-Workflow:
// 1. Konzept (Skizze): Zeichnen Sie alle Zustände als Kreise und alle möglichen Übergänge als Pfeile auf.
// 2. Zustände definieren: Weisen Sie jedem Zustand eine eindeutige Ganzzahl zu (z.B. 0=AUS, 1=STANDBY, 2=HEIZEN).
// 3. Übergangsbedingungen berechnen: Erstellen Sie für jeden Pfeil in der Skizze eine boolesche Variable, die true wird, wenn der Übergang stattfinden soll.
// 4. Übergangstabelle erstellen: Füllen Sie die Tabelle der State Machine von oben nach unten. Die Reihenfolge ist entscheidend, da die erste passende Regel gewinnt. Kritische Übergänge (z.B. NOT-AUS, RESET) gehören an den Anfang.
//
// Das Kerngeheimnis - Der Timeout-Mechanismus:
// Der 4. Parameter ($Timeout_Float) einer Übergangsregel ist keine globale Zeitüberschreitung, sondern ein hochflexibles Werkzeug.
// - Timer STARTEN: Ein Wert > 0 in diesem Parameter startet einen internen Timer, wenn der Übergang in den Ziel-Zustand (To_State) durch genau diese Regel erfolgt.
// - Timer ABFRAGEN: Eine 0 im 1. Parameter ($Bedingung) ist eine spezielle Anweisung. Sie bedeutet: "Prüfe, ob der für den aktuellen Zustand (From_State) laufende interne Timer abgelaufen ist". Der Timer-Ablauf selbst triggert nur einen neuen Logik-Lauf; diese Regel wertet das Ergebnis dieses Laufs aus.
// - Flexibilität: Da der Timer beim Übergang gesetzt wird, kann ein Zustand unterschiedliche Laufzeiten haben, je nachdem, von wo er aufgerufen wurde (z.B. kann der Zustand "HEIZEN" für 600s laufen, wenn er von "FROSTSCHUTZ" kommt, aber für 3600s, wenn er von "MANUELL EIN" kommt).
// - Implizites Löschen: Wird ein Zustand durch eine andere (nicht-zeitbasierte) Regel verlassen, bevor sein Timer abläuft, wird der Timer automatisch gelöscht. KRITISCHE ERKENNTNIS (basierend auf der Analyse des "Piranha-Problems" im Forum): Dies gilt auch dann, wenn der Übergang in den EIGENEN Zustand stattfindet (z.B. eine Regel von Zustand 1 nach Zustand 1). Ein solcher Übergang löscht ebenfalls einen zuvor in diesem Zustand gestarteten Timer.

// --- Zustandswechsel-Detektor (A -> B Pattern) ---
// Muster: Erkennt einen spezifischen Zustandswechsel (z.B. von Wert A nach Wert B) und erzeugt einen Ein-Zyklus-Impuls. Dies ist die kanonische Methode zur Flankenerkennung für beliebige Datentypen.
// Prinzip: Nutzt die strikt sequentielle Abarbeitung des Module-Blocks.
// 1. Vergleichen (am Anfang): Zwei Limiter- oder Comparator-Module prüfen, ob der $AktuelleWert dem Zielwert 'B' entspricht UND ob die Variable $LetzterWert dem Quellwert 'A' entspricht.
// 2. Verknüpfen: Ein And-Modul verknüpft die Ergebnisse. Der Ausgang ist nur true, wenn BEIDE Bedingungen erfüllt sind.
// 3. Speichern (am Ende): Ein Latch-Modulbaustein am Ende des Module-Blocks kopiert den $AktuellenWert in die Variable $LetzterWert und bereitet die Logik so für den nächsten Zyklus vor.
// Dieses 'Vergleiche, dann Speichere'-Prinzip ist fundamental für alle zustandsbehafteten Vergleiche.

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_24.txt ---

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_25.txt ---

// ==========================================================================
// 5. ARCHITEKTONISCHE MUSTER FÜR KOMPLEXE LOGIKEN
// ==========================================================================

// --- 5.1: DAS "INPUT -> GEHIRN -> MUSKELN"-PRINZIP FÜR MODULARE LOGIKEN (Robert Minis Muster) ---

// a) Beschreibung: Für mittelgroße bis komplexe Custom Logic-Implementierungen wird eine klare Trennung der Verantwortlichkeiten in drei logische Sektionen oder, bei sehr hoher Komplexität, in drei separate, miteinander verbundene Custom Logic-Zellen empfohlen. Dieses Muster fördert die Übersichtlichkeit, Wartbarkeit, Testbarkeit und die Erweiterbarkeit erheblich.

// b) Die drei Sektionen/Logiken:

//     INPUT (Datenerfassung & Vorverarbeitung):
//         Aufgabe: Alle relevanten Rohdaten von externen und internen Quellen (Sensoren, MQTT, GUI-Inputs) sammeln, validieren, filtern, skalieren und bei Bedarf aggregieren (z.B. Phasendaten zu Gesamtleistung). Dies umfasst auch die Berechnung von Netto-Energiewerten für Batteriespeicher, indem Umwandlungsverluste berücksichtigt werden.
//         Ergebnis: Bereitstellung von "bereinigten" und für die Entscheidungslogik sofort nutzbaren internen Variablen (z.B. $Lgc_Gesamtleistung_Bereinigt_W).
//         Vorteil: Entkoppelt die Rohdatenverarbeitung von der eigentlichen Steuerung. Änderungen an Sensoriken betreffen primär diese Schicht.

//     GEHIRN (Entscheidungs- und Zustandslogik):
//         Aufgabe: Basierend auf den bereinigten Inputs, internen Zustandsvariablen und den Geschäftsregeln (z.B. Hysterese, Zeitpläne, Prioritäten, Sicherheitsbedingungen) alle relevanten Steuerungsentscheidungen treffen. Hier sitzt die "Intelligenz" der Logik (z.B. Statemachine).
//         Ergebnis: Setzt "Soll-Zustände" oder "Soll-Werte" für die Aktoren in internen Variablen (z.B. $Lgc_Soll_Heizstab_Ein, $Lgc_Soll_WP_Leistung_W). Diese Variablen stellen die "Absicht" des Systems dar, bevor sie an die Aktoren gehen.
//         Vorteil: Die Kernlogik ist klar definiert. Änderungen an den Steuerstrategien betreffen primär diese Schicht.

//     MUSKELN (Aktorik & Output-Umsetzung):
//         Aufgabe: Die "Soll-Zustände" oder "Soll-Werte" vom "Gehirn" entgegennehmen und in konkrete, systemspezifische Befehle für die physischen Aktoren übersetzen und diese über die Timberwolf-Ausgänge (Output-Block) senden (z.B. SendExplicit für MQTT-Befehle, direkte Zuweisung an verknüpfte Objekte). Hier werden auch die Sende-Optionen (a, c, t, x) festgelegt. Diese Ebene muss oft auch aktor-spezifische Schutz- und Zwangslaufzeiten implementieren, die die reinen "Soll-Werte" des "Gehirns" übersteuern können.
//         Ergebnis: Physikalische Aktionen am System.
//         Vorteil: Entkoppelt die Entscheidungen von der Aktoransteuerung. Änderungen an Aktortypen oder Kommunikationsprotokollen betreffen primär diese Schicht.

// c) Verbindung der Logiken:
//     Die internen Variablen zwischen den Sektionen dienen als klare Schnittstellen. Bei der Aufteilung in separate Custom Logic-Zellen werden diese Schnittstellen als Input/Output zwischen den Zellen definiert.

// d) Kanon-Referenz: Erweitert 1.19 (Architektonisches Komplexitäts-Protokoll) um eine konkrete Strukturierungsmethode, unterstützt 1.20 (Prinzip der Nachvollziehbarkeit) und 1.21 (Dogma der Schnittstellen-Stabilität).

// **HÄRTUNG: VERFEINERUNG DER MODULAREN ARCHITEKTUR FÜR KOMPLEXE SYSTEME**
// Explizite Empfehlung für **vertikale (Schichten)** und **horizontale (Instanzen)** Modularität als **"SystemControl / CircuitControl / Sequencer"-Modell**.
// *   **SystemControl:** Verwaltet globale Systemzustände (Pumpe, Netzteile), Verzögerungen (Vorlauf, Nachlauf) und systemweite Fehlerbedingungen.
// *   **CircuitControl:** Kümmert sich um die Steuerung eines *einzelnen* Aktors/Kreises, stellt Systemanforderungen und wartet auf Freigaben.
// *   **Sequencer:** Orchestriert den Ablauf mehrerer `CircuitControl`-Instanzen, verwaltet die Reihenfolge, überspringt gesperrte Kreise und verteilt globale Parameter.
// **Grenzen der Modularisierung:** Eine monolithische Logik > 2 Bildschirmseiten oder mit einer hohen Anzahl von Modulen/Variablen (> ca. 50-70) ist ein starker Indikator für eine notwendige Modularisierung.

// ==========================================================================
// 6. ANTI-PATTERNS (HÄUFIGE FEHLERQUELLEN)
// ==========================================================================

// --- ANTI-PATTERN 1: MISSBRAUCH VON DATENTYPEN ---
// Die Verwendung des Datentyps integer für die Darstellung von booleschen Zuständen (wahr/falsch) ist ein kritisches Anti-Pattern. Regel: Für logische Zustände immer den Datentyp bool verwenden.

// --- ANTI-PATTERN 2: DAS GEDÄCHTNISLOSE IF (LATCH ALS ZUWEISUNG) ---
// Der Versuch, einen Latch-Modulbaustein als zustandslose "Wenn-Dann"-Zuweisung zu missbrauchen, ist ein fundamentales Missverständnis seiner Funktion. Regel: Verwende ein Latch nur, wenn du explizit einen Zustand speichern willst.

// --- ANTI-PATTERN 3: ZIRKELBEZÜGE OHNE KLARE AUFLÖSUNG ---
// Eine direkte oder indirekte Rückkopplung eines Ausgangs auf einen seiner eigenen Eingänge innerhalb derselben Logik-Zelle kann zum Blockieren der Logik-Engine führen. Regel: Rückkopplungen MÜSSEN durch stateful Modulbausteine wie Latch oder Monoflop kontrolliert werden.

// --- ANTI-PATTERN 4: DER "DURCHLAUFERHITZER" ---
// Die Verwendung einer Logik-Zelle zur reinen Distribution eines dynamischen Eingangswertes auf einen Ausgang ohne jegliche Transformation ist ein Anti-Pattern. Regel: Verbinde Quellen und Ziele direkt miteinander.

// --- ANTI-PATTERN 5: ÜBERKOMPLEXE TRIGGER-BEDINGUNGEN ---
// Der Versuch, einen Modulbaustein mit einer komplex zusammengesetzten Bedingung als Flanken-Trigger zu steuern, ist fehleranfällig. Regel: Die Trigger-Bedingung für eine Aktion sollte so einfach und direkt wie möglich sein.

// --- ANTI-PATTERN 6: DER TRÜGERISCHE INVERTER ---
// Die Verwendung eines einfachen Inverters (z.B. [ "And", [ "-A" ], "B" ]) für die Aufgabe einer gegenseitigen Verriegelung ist ein kritisches Anti-Pattern. Regel: Für einen echten gegenseitigen Ausschluss MUSS das "Interlock"-Pattern verwendet werden.

// --- ANTI-PATTERN 7: DER SELBST-RESET BEI FLANKE ---
// Das Verbinden derselben Signalquelle mit einem flanken-gesteuerten Set-Eingang und einem davon abgeleiteten, negierten Reset-Eingang (z.B. eines Monoflop-Moduls) ist ein kritisches Anti-Pattern. Regel: Für die Detektion eines andauernden Zustands MUSS eine pegel-gesteuerte Logik verwendet werden.

// --- ANTI-PATTERN 8: DIE ALTERNIERENDE STOPPUHR ---
// Die Verwendung einer einzigen Stoppuhr, um die gesamte Einschaltdauer zu messen, ist ein Anti-Pattern. Regel: Um die Gesamtdauer einer Aktivität zu messen, MUSS das HobbsMeter-Modul verwendet werden.

// --- ANTI-PATTERN 9: DER MANUELLE IMPULSGENERATOR ---
// Der Versuch, einen Ein-Zyklus-Impuls manuell durch die Sequenz [SendExplicit, ..., "true", ...], gefolgt von [SendExplicit, ..., "false", ...] zu erzeugen, ist unzuverlässig. Regel: Nutze das Pattern "Sequenzer (Stabile Timer-Kaskade)".

// --- ANTI-PATTERN 10: DIE SYNCHRONISIERTE HERDE ---
// Das Konfigurieren mehrerer, unabhängiger Logik-Zellen mit identischen Zeitparametern ist ein kritisches Anti-Pattern (Lastspitzen). Kanonische Lösung: Zyklische Trigger MÜSSEN bewusst "entzerrt" werden (z.B. 299s statt 300s).
// **HÄRTUNG: GLEICHZEITIGE CRON-TIMER IN SELBEN LOGIK-ZELLE**
// Beschreibung: Gleichzeitige `Cron`-Timer in derselben Logik-Zelle führen zu ineffizienten Mehrfachberechnungen. Kanonische Lösung: `Cron`-Trigger bewusst um Sekunden/Minuten versetzen.

// --- ANTI-PATTERN 11: DIE GETEILTE CRON-DEFINITION ---
// Die Wiederverwendung derselben String-Variable für die Zeitdefinition in mehreren Cron-Modulbausteinen kann zum Absturz führen. Kanonische Lösung: Jeder Cron-Modulbaustein MUSS eine eigene, dedizierte String-Variable erhalten.

// --- ANTI-PATTERN 12: DAS GETEILTE TIMER-ZIEL ---
// Das Schreiben der Ausgänge mehrerer Monoflop-Module in dieselbe Zielvariable ist ein kritisches Anti-Pattern. Kanonische Lösung: Jeder Timer MUSS eine eigene, dedizierte Ausgangsvariable haben.

// --- ANTI-PATTERN 13: DER MANUELLE ZUSTANDSAUTOMAT ---
// Der Versuch, eine komplexe Zustandslogik mit einer Kette von Latch-, Limiter- und Multiplexer-Modulbausteinen nachzubauen, ist ein Anti-Pattern. Regel: Nutze die Statemachine.

// --- ANTI-PATTERN 14: DER MISSBRAUCHTE FLANKENDETEKTOR ---
// Der Versuch, das "Triggered"-Modul zur allgemeinen Erkennung des Zustandswechsels einer internen Variable zu verwenden, ist ein kritisches Anti-Pattern. Regel: Für die Reaktion auf das Ende eines Timers MUSS das Pattern "Timer-Ablauf-Impuls (via Triggered-Pattern)" verwendet werden.

// --- ANTI-PATTERN 15: DAS GESCHLOSSENE REGELKREIS-PARADOXON ---
// Der Versuch, einen vollständigen, geschlossenen Regelkreis innerhalb einer einzigen Custom Logic-Zelle abzubilden, ist ein kritisches Anti-Pattern. Regel: MUSS zwingend das Pattern "Architektonische Entkopplung (Das Zwei-Zellen-Prinzip)" verwendet werden.

// --- ANTI-PATTERN 16: TRÜGERISCHE KOMPAKT-FORMEL ---
// Der Versuch, komplexe mathematische Formeln in einem einzigen [CalcFormula]-Modulbaustein abzubilden. Regel: Zerlege komplexe Formeln in eine Kette von einfachen [Polynomial]-Modulbausteinen.

// --- ANTI-PATTERN 17: IMPULS-FALLE ---
// Das Design von Logiken, die auf flüchtige Impulse reagieren, die nicht durch explizite Flankendetektions-Pattern abgesichert sind. Regel: Reagiere IMMER auf STABILE ZUSTÄNDE oder verwende kanonische Flankendetektions-Pattern.

// --- ANTI-PATTERN 18: DIE "GORDISCHER KNOTEN"-FALLACY ---
// Der Versuch, mehrere, voneinander abhängige logische Entscheidungen in einem einzigen, komplexen Modul zu verschachteln. Regel: Zerlege komplexe Entscheidungen in eine Reihe von einfachen, sequentiellen Modulbausteinen.

// --- ANTI-PATTERN 19: "SCHWEIZER TASCHENMESSER" ---
// Der Versuch, ein einziges Modul mit zu vielen, voneinander unabhängigen Funktionen zu überfrachten. Regel: Halte Logiken auf eine klare, atomare Funktion beschränkt.

// --- ANTI-PATTERN 20: AKADEMISCHES TRUGBILD ---
// Der Versuch, komplexe physikalische Modelle direkt in Custom Logic abzubilden. Regel: Modelliere das VERHALTEN, nicht die PHYSIK. Nutze Heuristiken und Schwellenwerte.

// --- ANTI-PATTERN 21: DYNAMISCHE FORMEL ---
// Die Verwendung einer String-Variable als `$Formel_String` im [CalcFormula]-Modulbaustein, deren Inhalt zur Laufzeit dynamisch geändert wird. Regel: Der `$Formel_String` MUSS eine Konstante im Level-Block sein.

// --- ANTI-PATTERN 22: SYNTAX-HALLUZINATION ---
// Die Erfindung von nicht dokumentierten Modulen oder Parametern. Regel: Wenn ein Werkzeug nicht explizit in diesem Kanon aufgeführt ist, existiert es nicht (Regel 1.17).

// --- ANTI-PATTERN 23: UNDOKUMENTIERTE POLYNOMIAL-SYNTAX FÜR IMPULSZÄHLUNG ---
// Der Versuch, Zustandswechsel von Booleans durch nicht-standardisierte Parameterübergaben an das `Polynomial`-Modul zu detektieren. Regel: Zur Detektion von Zustandswechseln MUSS das `Xor`-Modul oder das `Robuste Flankenerkennung`-Pattern verwendet werden.

// --- ANTI-PATTERN 24: ZÄHLER-AKKUMULATION OHNE LAST-REFERENZ ---
// Der Versuch, einen akkumulierenden Zähler ohne separate `$State_Last_...`-Variable zu implementieren. Regel: Immer `Zustandswechsel-Detektor` oder `Robuste Flankenerkennung` in Kombination mit `$State_Last_...`-Variablen nutzen.

// --- ANTI-PATTERN 25: UNKONTROLLIERTER INHIBIT (Break-Missbrauch) ---
// Das `Break`-Modul für Inhibit führt zu undefinierten Zuständen am Ausgang. Regel: Für ein sicheres Deaktivieren MUSS das `Kontrolliertes Inhibit/Deaktivieren`-Pattern genutzt werden.

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_25.txt ---

