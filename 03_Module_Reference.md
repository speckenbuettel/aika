# Module Reference

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_15.txt ---

// ==========================================================================
// 3. VALIDIERTE KERNMODULE & LOGIKEN (Alphabetisch)
// ==========================================================================
// Die Syntax für diese Modulbausteine wurde validiert.

// --- Addition ---
//   $Summand1_X ----\
//                    [ + ] --- $Summe_Y
//   $Summand2_A0 ---/
// Nutzt den Polynomial-Modulbaustein. Entspricht Y = X + A0.
// **HÄRTUNG (REGEL 1.49 & 1.64):** Literale Zahlen in Arrays sind verboten.
Syntax: [ "Polynomial", "Summand1_X", "Summe_Y", [ "$Summand2_A0", "$Konst_1_Float" ] ]

// --- And / Or / Xor (Logikgatter) ---
// Funktion: Führen logische Operationen auf einer Liste von booleschen Eingängen aus. Siehe auch das Pattern "Skalierbare Logikgatter".
// **HÄRTUNG (REGEL 1.63):** Logik-Gatter MÜSSEN zwingend den Datentyp bool verwenden.
// Syntax A (Explizite Liste): [ "Or", [ "In_A", "In_B", ... ], "Out" ]
// Syntax B (Multivariable): [ "Or", [ "$VAR<In!>" ], "Out" ]
// (Gilt analog für "And" und "Xor")
// **HÄRTUNG (REGEL 1.3):** Zur Invertierung von internen Booleans MUSS zwingend ein expliziter XOR-Baustein verwendet werden (XOR mit $Konst_True).

// --- Astro ---
// **HÄRTUNG (REGEL 1.49):** Parameter-Array darf nur Variablen enthalten.
Syntax: [ "Astro", [ "$Latitude", "$Longitude" ], "Altitude", "Azimute", "Transit", "Sunrise", "Sunset", "Civildawn", "Civil_dusk", "Nautdawn", "Naut_dusk", "Astrodawn", "Astro_dusk", "Altmoon", "Azi_moon" ]
// **HINWEIS: IMPLIZITE TRIGGER-EIGENSCHAFT (REGEL 2.4)**
// Obwohl dieses Modul keine expliziten "Trigger"-Eingänge hat, muss es in einer Logik-Zelle platziert werden, die durch einen anderen Input oder einen externen Timer (`Clocksignal`, Zeitschaltuhr) ausgelöst wird, um seine Berechnungen durchzuführen und die Outputs zu aktualisieren.

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_15.txt ---

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_16.txt ---

// --- Begrenzer (Limiter) ---
// HINWEIS: Kann auch zur Überprüfung auf Gleichheit verwendet werden, indem Min und Max auf denselben Wert (oder dieselbe Variable) gesetzt werden.
// **HÄRTUNG (REGEL 1.49 & 1.64):** Literale Zahlen im Parameter-Array sind verboten.
Syntax: [ "Limiter", "Input", "Out_Clamped", "Out_IsInRange", [ "$Min", "$Max" ] ]

// --- Benachrichtigung versenden (SendToSimple) ---
Syntax: [ "SendToSimple", "Inhibit", "Channel", "Title", "Message", "Category", "Priority" ]
// KRITISCHER HINWEIS: Der 'Channel' MUSS ein vordefinierter System-Kanal sein (z.B. "general"). Die Verwendung eines nicht existierenden Kanals führt zu einem Fehler.
// **HÄRTUNG (ANTI-PATTERN 38 - NACHRICHTENCENTER-FLUTUNG):**
// Beschreibung: Nutzung von SendToSimple in schnellen Logik-Zyklen ohne Inhibit oder Flankenerkennung. Folge: Blockade der Wolf-Oberfläche durch Millionen von Datenbank-Einträgen. Regel: SendToSimple darf nur ereignisbasiert (1x pro Vorfall via Flanke) gefeuert werden.

// --- Betriebsstundenzähler (HobbsMeter) ---
// HINWEIS: Für eine erweiterte Implementierung mit Tages-, Monats- und Jahreszählern siehe das "Periodischer Zähler mit Reset & Akkumulation"-Pattern.
Syntax: [ "HobbsMeter", "Running", "RunTime_Out", "$Reset" ]

// --- Binär-Statistik ---
Syntax: [ "BinaryStatistic", [ "Input1", ... ], "Out_MoreTrue", "Out_FalseCount", "Out_TrueCount" ]

// --- Binärdemultiplexer ---
// Funktion: Dekodiert einen Integer-Wert in seine einzelnen Bits (bis zu 32). Jedem Ausgang ($Out_Bit0, $Out_Bit1, etc.) wird der Zustand des entsprechenden Bits zugewiesen.
// HINWEIS: Dieser Modulbaustein erfordert eine explizite, geordnete Liste von Ausgangsvariablen.
Syntax: [ "BinaryDemultiplexer", "Input_Int", [ "$Out_Bit0", "$Out_Bit1", ... ] ]

// --- Binärmultiplexer ---
// Funktion: Kodiert eine Liste von booleschen Werten in einen einzigen Integer-Wert (Bit 0 ist der erste Eingang).
// Syntax A (Explizite Liste): Ideal für eine feste, kleine Anzahl von Eingängen.
Syntax A: [ "BinaryMultiplexer", [ "Bit0", "Bit_1", ... ], "Output_Int" ]
// Syntax B (Multivariable): Die bevorzugte, skalierbare Methode für eine variable Anzahl von Eingängen.
Syntax B: [ "BinaryMultiplexer", [ "$VAR<In!>" ], "Output_Int" ]

// --- Break ---
// Funktion: Bricht die weitere Abarbeitung des Module-Blocks ab, wenn die Bedingung erfüllt ist.
// HINWEIS: Die Verwendung von 'Break' für Sicherheits- oder Fail-Safe-Zwecke ist ein Anti-Pattern. Es unterdrückt nur die weitere Berechnung, definiert aber keinen sicheren Ausgangszustand. Für solche Anwendungsfälle MUSS das 'Sicherheits-Übersteuerung (Fail-Safe Override)'-Pattern verwendet werden.
// **HÄRTUNG (REGEL 1.50 - DIE DOKTRIN DER ATMENDEN SICHERHEIT):**
// In Logiken mit Sicherheitsfunktionen (Watchdogs) ist das Break-Modul verboten. Die Logik darf niemals "schlafen", da sie sonst den Ausfall von Sensoren nicht bemerkt. Die Logik muss permanent „atmen“, um Sensorausfälle jederzeit detektieren zu können.
Syntax: [ "Break", ["$Bedingung"] ]

// --- CalcFormula (Freie Formel) ---
Syntax: [ "CalcFormula", ["$VAR<X!>"], "Output", "$Formel_String" ]
// HINWEIS: Dieser Modulbaustein ist sehr mächtig, aber auch fehleranfällig. Eine ungültige Operation (z.B. Division durch Null, Syntaxfehler in der Formel) führt zu einem internen Fehler, der die Ausführung der Logik-Zelle abbricht. Es wird dringend empfohlen, den globalen Error-Ausgang ($Error?) zu verwenden, um solche Fehler abzufangen und zudiagnostizieren.
// KRITISCHER HINWEIS: Der $Formel_String darf NICHT zur Laufzeit dynamisch erzeugt werden (siehe Anti-Pattern "Dynamische Formel"). Ein leerer oder invalider Formel-String führt zu einem Laufzeitfehler.
// **HÄRTUNG (REGEL 1.83 - DIE DYNAMISCHE FORMEL-DOKTRIN):**
// Erkenntnis: CalcFormula-Strings können via Multiplexer zur Laufzeit gewechselt werden. Härtung: Um Modul-Instanzen zu sparen, ist bei Modus-Umschaltungen (z.B. Skalieren vs. Quotieren) der Formel-String dynamisch zu tauschen, anstatt redundante Module zu verwenden.
// **Neue Hinweise zu unterstützten Rundungsfunktionen:** Die Funktionen `ceil()` (Aufrunden) und `floor()` (Abrunden) werden im TWS `CalcFormula` **nicht** unterstützt. Stattdessen MUSS `rint(X1)` für das Runden auf den nächsten Integer verwendet werden.
// **Beispiel für Rundung:** `$Formel_DaysRound = "rint(X1)"`.
// **Ternärer Operator:** Unterstützt `Bedingung ? Wert_Ja : Wert_Nein` (z.B. `X1>5?100:0`).
// **Typ-Warnung:** Gibt IMMER float zurück. Niemals direkt in bool schreiben! Immer via Comparator wandeln.
// **HÄRTUNG (REGEL 1.41 - ISOLIERUNG MULTIPLER RECHENOPERATIONEN):**
// Erkenntnis: Wenn mehrere identische Operationen (wie 4x Offset-Multiplikation) schnell hintereinander ablaufen, kann das Polynomial-Modul zu internen „Übersprechern“ führen. Härtung: In Multi-Kanal-Logiken (RGB, RGBW, HCL) ist für Skalierungen (Wert * Offset) zwingend CalcFormula zu verwenden. Grund: CalcFormula erzeugt eine isolierte Instanz der Berechnung.

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_16.txt ---

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_17.txt ---

// --- Clocksignal (Taktsignal) ---
Syntax: [ "Clocksignal", "Enable", "Output_Clock", "$Perioden_Halbe_s" ]
// **HÄRTUNG: IMPLIZITE TRIGGER-EIGENSCHAFT (REGEL 2.4)**
// Dieses Modul besitzt eine implizite Trigger-Eigenschaft, d.h., es löst eine Ausführung der Logik-Zelle aus, wenn sein definiertes Intervall erreicht ist.

// --- Comparator (Schwellwertschalter) ---
// HINWEIS: Auf älteren Firmware-Ständen (vor ca. V4.5) ist die 4-Parameter-Syntax ohne expliziten Mode-Integer unzuverlässig. Für maximale Kompatibilität und Stabilität sollte immer die Syntax mit explizitem Mode verwendet werden.
Syntax (4-Parameter): [ "Comparator", "A", "Out", "B" ] // Out is true if A > B.
Syntax (Hysterese): [ "Comparator", "A", "Out", [ "$Schwelle_Aus", "$Schwelle_Ein" ] ]
// **HÄRTUNG (REGEL 1.49 & 1.64):** Literale Zahlen im Hysterese-Array sind verboten.
// HINWEIS (>= Vergleich): Um einen "größer-gleich" (>=) Vergleich zu realisieren, kann die
// gesamte Anweisung invertiert werden: [ "Comparator", "-A", "-Out", "-B" ]
// **Klarstellung zur Funktionsweise:** Der `Comparator` liefert den Output `true` genau dann, wenn `Parameter A > Parameter B` ist.
// **HÄRTUNG (ANTI-PATTERN 26 - FALSCHE PARAMETERREIHENFOLGE):**
// Für einen `Wert_A < Wert_B` Vergleich MUSS der `Comparator` in der Reihenfolge `["Comparator", "Wert_B", "Output", "Wert_A"]` aufgerufen werden.
// **Neue Hinweise für <= Vergleiche:**
// *   Um `A <= B` zu prüfen, kann `A > B` geprüft und das Ergebnis negiert werden. Siehe `PATTERN: Robuster Kleiner-Gleich Vergleich (<=)`.

// --- Concat (KONKATENIERE) ---
Syntax: [ "Concat", ["String1", ...], "Output" ]
// HINWEIS: Das Verhalten dieses Modulbausteins hängt kritisch von der Deklaration der Eingangs-Variablen im "Level"-Block ab (siehe Regel 1.2). Für eine effiziente und robuste String-Verkettung, siehe das Pattern "Effiziente String-Verkettung".
// **HÄRTUNG (REGEL 1.34 L.2 - RESSOURCEN-BREMSE):**
// Rechenintensive Module (besonders Concat für JSON) dürfen bei Trigger "a" (Always) nicht ungebremst laufen. Pflicht: Baue ein Break-Modul ein, das die Ausführung abbricht, wenn kein Sende-Takt anliegt.

// --- Cron / Kalender-Timer ---
// Syntax (4-Parameter): [ "Cron", "Enable", "Output_Impuls", "CronString" ]
// Syntax (5-Parameter, validiert): [ "Cron", "Enable_Bool", "Output_Impuls", "Output_UnixTimestamp_Int", "$CronString" ]
// KRITISCHE SYNTAX-REGEL (6-Felder-Format):
// Der Cron-Parser des Timberwolf Servers erwartet zwingend einen String mit 6 Feldern.
// Die Verwendung des weit verbreiteten 5-Felder-Formats führt zu einem Fehler.
// Die korrekte Reihenfolge ist: Sekunde Minute Stunde Tag-des-Monats Monat Wochentag
//
// KORREKTES BEISPIEL (Jeden Freitag um 20:00:30 Uhr):
// "30 0 20 * * 5"
//
// **HÄRTUNG: IMPLIZITE TRIGGER-EIGENSCHAFT (REGEL 2.4)**
// Dieses Modul besitzt eine implizite Trigger-Eigenschaft, d.h., es löst eine Ausführung der Logik-Zelle aus, wenn sein definierter Zeitpunkt erreicht ist.
//
// **HÄRTUNG: KLARSTELLUNG WOCHENTAGS-SYNTAX & EMPFEHLUNG FÜR KALENDER-PERIODEN-ABBILDUNG**
// Der Wochentag im `Cron`-String (letztes Feld) folgt der Standard-Syntax (`0` für Sonntag, `1` für Montag, ..., `6` für Samstag).
// Für dynamische Kalenderperioden wird empfohlen, das `Localtime`-Modul in Kombination mit `Comparator` und `And`-Modulen zu nutzen.

// --- Dim-Aktor (Relativ zu Absolut) [NEU V4.8] ---
// Status: Verfügbar ab V4.8 IP3. Ersetzt das "Akkumulator-Pattern".
// Dieses Modul wandelt relative Befehle (z.B. +10, -20) in absolute Werte (0-100%) um, inkl. Begrenzung.
// Syntax: ["Dim-Aktor", "$In_Relativ", "$Out_Absolut", "$Inhibit"]
// Verhalten: Addiert Input zum letzten Output. Begrenzt auf 0-100.
// WICHTIG: Der Eingang "$In_Relativ" muss zwingend auf Trigger "a" (always) stehen!

// --- Division (Ratio) ---
// ACHTUNG: Ungewöhnliche Parameter-Reihenfolge!
Syntax: [ "Ratio", "Zaehler", "Ergebnis", "$Nenner" ]
// **HÄRTUNG (REGEL 1.30 / ANTI-PATTERN 30 - UNGESCHÜTZTE DIVISION DURCH NULL):**
// Die explizite Empfehlung MUSS eingehalten werden, vor JEDEM `Ratio`-Aufruf sicherzustellen, dass der `$Nenner` **immer größer als Null** ist (z.B. via Limiter auf 0.001 begrenzen). Bei einer Division durch Null wird der globale Error-Ausgang ($Error?) auf TRUE gesetzt.

// --- Error-Handling ($Error?) ---
// Funktion: Ermöglicht das Auslesen des internen Fehlerzustands einer Logik-Zelle.
// KRITISCHE SYNTAX-REGELN:
// 1. Es darf nur EIN Fehlerausgang pro Logik-Zelle deklariert werden.
// 2. Die verwendete Variable (hier: "$Error?") MUSS vom Typ 'bool' sein.
// 3. Die Sende-Option MUSS zwingend "ce" sein.
// Validierte Syntax im Output-Block:
// [ "Err", "Fehlerzustand der Logik", "$Error?", "ce" ]

// --- Farbraum-Konvertierung (Native) [NEU V4.8] ---
// Status: Verfügbar ab V4.8 IP3. Ersetzt alle manuellen Formel-Berechnungen.
// **HÄRTUNG (ANTI-PATTERN 32 - MANUELLE FARBRECHNUNG):**
// Der Versuch, RGB/HSV-Konvertierungen mit manuellen Formeln (Polynomial/CalcFormula) zu berechnen, ist veraltet und rechenintensiv. Nutze ZWINGEND die nativen Module ab Firmware V4.8.
// WICHTIG: Immer auf den Wertebereich achten (0-1 vs. 0-255)!
// RGB -> HSV:
// ["RGB->HSV", "$R", "$G", "$B", "$H", "$S", "$V"] (Bereich 0-1)
// ["RGB255->HSV100", "$R", "$G", "$B", "$H", "$S", "$V"] (R/G/B 0-255, S/V 0-100%)
// HSV -> RGB:
// ["HSV->RGB", "$H", "$S", "$V", "$R", "$G", "$B"] (Bereich 0-1)
// ["HSV100->RGB255", "$H", "$S", "$V", "$R", "$G", "$B"] (S/V 0-100%, R/G/B 0-255)

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_17.txt ---

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_18.txt ---

// --- HEX->INT (Hexadezimal zu Integer) ---
// Funktion: Wandelt eine Hexadezimal-Zeichenkette in eine Ganzzahl um.
// $ByteSwap: Tauscht Bytes innerhalb von 16-Bit-Wörtern (z.B. 0x1A2B -> 0x2B1A).
// $WordSwap: Tauscht 16-Bit-Wörter innerhalb eines 32-Bit-Doppelwortes (z.B. 0x1A2B3C4D -> 0x3C4D1A2B).
Syntax: [ "HEX->INT", "Input_Hex_String", "Output_Integer", "$ByteSwap_Bool", "$WordSwap_Bool" ]

// --- Interpolation ---
// **HÄRTUNG (REGEL 1.49 & 1.64 - DAS ARRAY-LITERAL-VERBOT):**
// Innerhalb von Parameter-Arrays (Stützpunkte) sind literale Zahlen STRIKT VERBOTEN.
// Jedes Stützpunkt-Paar [x, y] MUSS aus deklarierten Variablen-Referenzen bestehen.
Syntax: [ "Interpolation", "In_X", "Out_Y", [ ["$X1", "$Y1"], ["$X2", "$Y2"], ... ] ]

// --- JsonPathSelector ---
// Funktion: Extrahiert einen Wert aus einer JSON-formatierten Zeichenkette mithilfe eines
// Pfad-Ausdrucks. Dies ist die kanonische und robusteste Methode zur Verarbeitung von
// strukturierten Daten aus API-Antworten.
Syntax: [ "JsonPathSelector", "Input_Json", "JsonPath", "Output", "Error" ]

// --- Latch ---
//   $Input -------->[ LATCH ] --- $Output
//                      ^
//                      |
//                  $Trigger
// HINWEIS: Der Parameter Mode_Int_Literal MUSS eine literale Zahl sein (z.B. 1), keine Variable.
Syntax: [ "Latch", "Input", "Output", "Trigger", Mode_Int_Literal ] // (Modes 0-3)
// Validierte Betriebsmodi (Mode_Int_Literal):
// 0: Pegelgesteuert (Übernahme, solange Trigger TRUE ist)
// 1: Steigende Flanke (Übernahme bei Wechsel von FALSE auf TRUE)
// 2: Fallende Flanke (Übernahme bei Wechsel von TRUE auf FALSE)
// 3: Beide Flanken (Übernahme bei jedem Wechsel)
//
// **HÄRTUNG (REGEL 1.34 L.1 - DIE LATCH-FALLE):**
// Hinter einem Clocksignal darf NIEMALS ein Latch mit Mode 0 (Pegel) stehen. Das erzeugt Bursts.
// Nutze zwingend Mode 1 (Steigende Flanke) oder eine vorgeschaltete Flankenerkennung.

// --- Localtime (Uhrzeit / Unix-Zeit-Wandler) ---
// Funktion: Ein vielseitiger Dual-Mode-Modulbaustein zur Arbeit mit Zeitstempeln. Seine Kernkompetenz ist die
// automatische und korrekte Umwandlung eines UTC-basierten Unix-Timestamps in die lokale Zeitzone.
//
// **HÄRTUNG: IMPLIZITE TRIGGER-EIGENSCHAFT (REGEL 2.4)**
// Obwohl dieses Modul keine expliziten "Trigger"-Eingänge hat, muss es in einer Logik-Zelle platziert werden,
// die durch einen anderen Input oder einen externen Timer ausgelöst wird, um seine Berechnungen durchzuführen.
//
// SYNTAX-REGEL (PLATZHALTER):
// Ungenutzte Ausgänge dieses Modulbausteins KÖNNEN und SOLLTEN durch das Literal '0' ersetzt werden.
//
// Ausgabewerte:
// - Unix-Zeit (integer): Sekunden seit 1.1.1970 UTC.
// - Sekunde (integer): 0-59 | Minute (integer): 0-59 | Stunde (integer): 0-23
// - Tag-des-Monats (integer): 1-31 | Monat (integer): 1-12 | Jahr (integer): Volles Jahr.
// - Wochentag (integer): 0-6 (Sonntag=0) | Tag-des-Jahres: 1-366 | Sommerzeit-Flag: >0 wenn aktiv.
//
// Syntax A (Aktuelle Server-Zeit holen): Der erste Parameter (Input-Timestamp) ist 0.
Syntax A: [ "Localtime", 0, "UnixOut", "Sec_Out", "Min_Out", "Hour_Out", "Mday_Out", "Mon_Out", "Year_Out", "Wday_Out", "Yday_Out", "IsDst_Out" ]
// Syntax B (Externen Timestamp wandeln): Der erste Parameter ist die Variable mit dem Input-Timestamp.
Syntax B: [ "Localtime", "UnixIn", 0, "Sec_Out", "Min_Out", "Hour_Out", "Mday_Out", "Mon_Out", "Year_Out", "Wday_Out", "Yday_Out", "IsDst_Out" ]

// --- Lowpass (Tiefpass-Filter) ---
// Funktion: Implementiert einen digitalen Tiefpassfilter 1. Ordnung zur Glättung von Werten.
Syntax: [ "Lowpass", "Input", "Output", "$Zeitkonstante_s" ]
//
// **HÄRTUNG (REGEL 1.42 - SYNCHRONISATION VON FILTER UND ABSCHALTUNG):**
// Erkenntnis: Ein Lowpass-Filter ist ein mathematischer Speicher. Er hinkt dem Sollwert immer hinterher.
// Härtung: Wenn eine Logik zeitgesteuert abschaltet, muss der Sollwert bereits 10-20 Sekunden vor dem harten Ende
// auf den Zielwert (0.0) gehen. Nur so hat der Filter genug Zeit, den physikalischen Ausgang auf 0.0 zu bringen,
// bevor die Logik hart gekappt wird (Vermeidung des „Kurven-Bruchs“).

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_18.txt ---

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_19.txt ---

// --- Monoflop / Timer ---
// Funktion: Implementiert einen vielseitigen Timer (Monoflop) mit verschiedenen Auslöse- und Retrigger-Verhalten.
// ACHTUNG: Der Zeit-Parameter ($Zeit_s) MUSS eine Variable sein. Die direkte Angabe einer Zahl führt zum Absturz des Logik-Subsystems! Siehe Regel 1.1.
// ACHTUNG: Jeder Monoflop benötigt eine eigene, dedizierte Ausgangsvariable. Siehe Anti-Pattern "Das geteilte Timer-Ziel".
// HINWEIS: Der Parameter Mode_Int_Literal MUSS eine literale Zahl sein (z.B. 2), keine Variable.
Syntax: [ "Monoflop", "Set", "Reset", "Out", "Zeit_s", Mode_Int_Literal ]
// HINWEIS: Beachte das kritische Laufzeitverhalten bei Timer-getriggerten Läufen. Siehe REGEL 1.12.
//
// **HÄRTUNG (REGEL 1.29 - DIE MONOFLOP-MATRIX / GUI vs. Code):**
// Der Logik-Editor zählt Timer 1-8. Der Code zählt Mode 0-7. Eine Verwechslung ist fatal.
// GUI Name | Code Mode | Auslösung        | Retriggerbar? | Typischer Einsatz
// ---------|-----------|------------------|---------------|------------------
// Timer 1  | 0         | Pegel (True)     | NEIN          | Verzögerung
// Timer 2  | 1         | Pegel (True)     | JA            | Watchdog (Pegel)
// Timer 3  | 2         | Steigende Flanke | NEIN          | Impuls-Verlängerung
// Timer 4  | 3         | Steigende Flanke | JA            | Watchdog (Impuls)
// Timer 5  | 4         | Fallende Flanke  | NEIN          | Nachlauf
// Timer 6  | 5         | Fallende Flanke  | JA            | Nachlauf (Verlängerbar)
// Watchdog-Regel: Für Watchdogs MUSS zwingend ein UNGERADER Mode (1, 3, 5, 7) gewählt werden!
//
// **HÄRTUNG (REGEL 1.34 L.3 - DIE TIMER-FALLE IM DOKTOR-MODUS):**
// Wissen: Eine Änderung der Zeit ($P_Time) im laufenden Betrieb (Doktor-Modus) wirkt NICHT sofort.
// Der alte Timer läuft zu Ende. Aktion: Zwinge die KI, dies in der Testanweisung zu erwähnen ("Retrigger notwendig").
//
// **HÄRTUNG: IMPLIZITE TRIGGER-EIGENSCHAFT (REGEL 2.4)**
// Dieses Modul besitzt eine implizite Trigger-Eigenschaft, d.h., es löst eine Ausführung der Logik-Zelle aus, wenn sein definierter Zeitpunkt erreicht ist, *sofern sein Set-Eingang dauerhaft `true` ist*.

// --- Multiplexer ---
//   $Wert_0 --------\
//   $Wert_1 --------[ MUX ] --- $Output
//                      |
//               $Selektor_Int
// Funktion: Leitet einen von mehreren Eingangswerten an den Ausgang weiter. Die Auswahl erfolgt über
// eine Integer-Variable ($Selektor_Int), die den 0-basierten Index des Eingangswertes angibt.
// HINWEIS: Alle Eingänge müssen vom selben Datentyp sein (z.B. alle float oder alle string).
//
// **HÄRTUNG (REGEL 1.49 & 1.64 - DAS ARRAY-LITERAL-VERBOT):**
// FUNDAMENTALE REGEL: Die Eingangs-Liste (der erste Parameter) darf ausschließlich Variablen-Referenzen enthalten.
// Die Verwendung von literalen Werten wie true, false oder Zahlen ist ein kritischer Syntaxfehler und führt zum Absturz.
// Für konstante Werte MÜSSEN Variablen im Level-Block definiert werden (z.B. $Konst_True, $Konst_Wert_5).
//
// **HÄRTUNG (REGEL 1.63 - DIE TYP-WÄCHTER-DOKTRIN):**
// Selektoren (z.B. bei Multiplexern) MÜSSEN zwingend als `int` deklariert sein.
//
// Syntax A (Explizite Liste): Für eine feste, bekannte Anzahl von Eingängen.
Syntax: [ "Multiplexer", [ "$Wert_0", "$Wert_1", ...], "Output", "Selektor_Int" ]
// Syntax B (Multivariable - DAS SKALIERBARE PATTERN): Die bevorzugte Methode für eine flexible Anzahl von Eingängen.
Syntax: [ "Multiplexer", [ "$VAR<In!>" ], "Output", "Selektor_Int" ]
// HINWEIS (Fallback-Verhalten): Falls der Wert des "$Selektor_Int" größer ist als die Anzahl
// der verfügbaren Eingänge, wird immer der Wert des letzten/höchsten Eingangs ausgegeben.

// --- Multiplikation ---
// Muster: Emuliert die Multiplikation A * B, da kein nativer Modulbaustein existiert.
// Prinzip: Es nutzt die Polynom-Formel Y = A0 + A1*X. Mit dem konstanten Koeffizienten A0=0
// und dem zweiten Faktor als Koeffizient A1 wird die Gleichung zu Y = FaktorB * FaktorA.
// **HÄRTUNG (REGEL 1.49):** Literale im Array sind verboten.
Pattern: [ "Polynomial", "FaktorA", "Produkt", [ "$Konst_0_Float", "$FaktorB" ] ]

// --- PID controller (PID-Regler) ---
// Funktion: Implementiert einen Proportional-Integral-Differential-Regler für komplexe Regelungsaufgaben.
Syntax: [ "PID controller", "Soll_float", "Ist_float", "Stell_float", "Kp_float", "Ki_float", "Kd_float" ]
// Für proportionale Regelungsaufgaben, die auf eine Regelabweichung reagieren, zwingend die `PID`-Module verwenden.

// --- PID_awu (PID-Regler mit Anti-Windup) ---
// Funktion: Erweiterte Version des PID-Reglers, die eine Sättigung der Stellgröße verhindert (Anti-Windup).
Syntax: [ "PID_awu", "Soll_f", "Ist_f", "Stell_f", "Kp_f", "Ki_f", "Kd_f", "Untergrenze_f", "Obergrenze_f" ]

// --- Polynomial (Polynomfunktion) ---
// Funktion: Berechnet Y nach der Formel Y = A0 + A1X + A2X^2 ...
// HINWEIS: Dieser Modulbaustein ist extrem mächtig. Er kann für simple Additionen, Multiplikationen und komplexe mathematische Formeln verwendet werden.
//
// **HÄRTUNG (REGEL 1.1 & 1.49 - ANTI-MAGIC-NUMBERS):**
// Koeffizienten MÜSSEN Variablen sein. Die Verwendung von numerischen Literalen anstelle von Variablen für Koeffizienten führt zu einem Fehler. Auch "triviale" Zahlen wie 0 oder 1 sind im Array verboten.
//
// **HÄRTUNG (REGEL 1.27 - DAS DIFFERENZ-GESETZ):**
// Regel für "Alt minus Neu" (Abfall): A0 (Offset) = Der alte/gespeicherte Wert. X (Input) = Der neue/aktuelle Wert. A1 (Faktor) = -1.
// Code: ["Polynomial", "$Input_Neu", "$Diff", ["$State_Alt", "$Konst_Minus1_Float"]]
//
// **HÄRTUNG (REGEL 1.41 - ISOLIERUNG MULTIPLER RECHENOPERATIONEN):**
// Erkenntnis: Wenn mehrere identische Operationen (wie 4x Offset-Multiplikation) schnell hintereinander ablaufen, kann das Polynomial-Modul zu internen „Übersprechern“ führen. Härtung: In Multi-Kanal-Logiken (RGB, RGBW, HCL) ist für Skalierungen (Wert * Offset) zwingend CalcFormula zu verwenden.
//
// Syntax A (Explizite Liste): [ "Polynomial", "Input_X", "Output_Y", [ "$A0", "$A1", ... ] ]
// Syntax B (Multivariable): [ "Polynomial", "Input_X", "Output_Y", [ "$VAR<Koeff!>" ] ]

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_19.txt ---

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_20.txt ---

// --- Printf (Formatiere String) ---
// Funktion: Formatiert einen Wert (Float, Integer, String) in eine neue Zeichenkette.
//
// **HÄRTUNG (REGEL 1.69 - DIE PRINTF-TRIADE):**
// Erkenntnis: Das Printf-Modul im Timberwolf ist kein Standard-C-Printf. Es akzeptiert exakt drei Parameter:
// [Eingangswert, Format_String_Variable, Ausgangsvariable].
// Der Versuch, mehrere Werte in einen Printf-Aufruf zu packen, ist ein kritischer Syntaxfehler.
//
// **HÄRTUNG (REGEL 1.70 - DIE STRING-REINHEITS-DOKTRIN):**
// Erkenntnis: Das Formatieren von float-Werten in Strings erzeugt oft führende Leerzeichen (Padding).
// Härtung: Werte, die in Strings fließen, MÜSSEN vorher zwingend in int gewandelt werden (oder %.0f nutzen).
//
// **HÄRTUNG (REGEL 1.34 L.2 - DIE RESSOURCEN-BREMSE):**
// Rechenintensive Module (besonders Printf) dürfen bei Trigger "a" (Always) nicht ungebremst laufen.
// Pflicht: Baue ein Break-Modul ein, das die Ausführung abbricht, wenn kein Sende-Takt anliegt.
//
Syntax: [ "Printf", "Input_Wert", "Format_String", "Output_String" ]

// --- Ramp (Rampenfunktion) [NEU V4.8] ---
// Syntax: ["Ramp", "$Ziel", "$Out", "$Active", "$Step", "$Period"]
// Funktion: Nähert $Out schrittweise an $Ziel an.
// Parameter: $Step (Schrittweite), $Period (Zeit pro Schritt in Sekunden).
// **Zweck:** Dient ausschließlich zur Begrenzung der Änderungsrate eines bereits berechneten Sollwerts.

// --- Regex (Regular Expression) ---
// Funktion: Wendet einen regulären Ausdruck auf eine Zeichenkette an.
// HINWEIS: Ungenutzte Ausgänge MÜSSEN mit dem numerischen Literal 0 ersetzt werden.
Syntax: [ "Regex", "Input_Str", "Pattern_Str", "Match_Bool", "Full_Match", "Grp1", "Grp2", "Grp3", "Grp4", "$Grp5" ]

// --- SingleDigitCounter (Status-Sammler) [NEU V4.8] ---
// Status: Verfügbar ab V4.8 IP3. Erstellt ein Histogramm für Integer-Werte von 0 bis 9.
// Syntax: ["SingleDigitCounter", ["$In1", "$In2"...], "$Cnt0", "$Cnt1", ..., "$Cnt9"]
// Anwendung: "Wie viele Fenster sind offen?" (Wenn Offen=1).

// --- STR->FLOAT / STR->INT ---
// Funktion: Konvertiert eine Zeichenkette in eine Fließkommazahl oder Ganzzahl.
Syntax: [ "STR->FLOAT", "Input_String", "Output_Float" ]

// --- SendExplicit ---
// Funktion: Löst unter spezifischen Bedingungen das Senden an einen "x"-Ausgang aus.
// KRITISCHE REGEL: Bei Mehrfachverwendung wirkt immer nur die LETZTE Anweisung im Zyklus!
// Syntax: [ "SendExplicit", "Sende_Bedingung_Bool", "Variable_mit_Wert", Sende_Option_Int_Literal ]
// Sende_Option_Int_Literal (0-3) MUSS direkt angegeben werden:
// 0: Pegel (True) | 1: Pos. Flanke | 2: Neg. Flanke | 3: Jede Flanke.

// --- Statemachine (Zustandsautomat) ---
// Funktion: Modellierung komplexer, sequentieller Abläufe.
//
// **HÄRTUNG (ANTI-PATTERN 29 - VARIABLEN ALS STATEMACHINE-ZUSTÄNDE):**
// Zustände (From_State, To_State) MÜSSEN literale Ganzzahlen sein. Die Verwendung von Variablen
// (auch Konstanten-Variablen) für Zustände ist nicht unterstützt und führt zu Fehlern.
//
// KRITISCHE SYNTAX-REGELN:
// 1. Übergangsregel MUSS exakt vier Parameter haben: ["Bedingung", From_Int, To_Int, "Timeout_Float"].
// 2. Der Timeout-Parameter ("$Timeout_Float") MUSS eine Variable sein.
// 3. From_State darf NIEMALS negativ sein.
//
Syntax: [ "Statemachine", [ [Transition1], [Transition2], ... ], "Output_State_Int"]

// --- Statistic (Statistik) ---
// Funktion: Berechnet Min, Max, Mean, Median aus einer Liste von Eingängen.
Syntax: [ "Statistic", [ "Input1", ... ], "Out_Min", "Out_Max", "Out_Mean", "$Out_Median" ]

// --- Stopwatch (Stoppuhr) ---
// Funktion: Misst die Zeit, während ein Steuersignal aktiv ist. Setzt auf 0 zurück, wenn inaktiv.
//
// **HÄRTUNG (REGEL 1.40 / ANTI-PATTERN 39 - DIE SCHLAFENDE STOPPUHR):**
// Erkenntnis: Stopwatch ist ein passiver Zeitnehmer. Er löst keinen neuen Logik-Lauf aus.
// Härtung: Jede Logik, die auf Stopwatch basiert, MUSS zwingend ein Clocksignal als internen Herzschlag
// enthalten, um die Logik permanent „wachzuschütteln“.
//
Syntax: [ "Stopwatch", "StartStop_Bool", "Time_s_Float" ]

// --- String_length (String-Länge) ---
Syntax: [ "String_length", "Input_String", "Output_Integer", "$enableUTF8Count" ]

// --- Stringcompare (PRÜFE zwei Strings...) ---
// Funktion: Vergleicht zwei Strings basierend auf einem Modus-Parameter.
// KRITISCH: Der Modus-Parameter MUSS eine Variable sein (z.B. $Konst_CompMode).
Syntax: [ "Stringcompare", "String_A", "Out_Bool", "String_B", "$Mode_String_Var" ]

// --- Subtraktion ---
// Muster: Implementiert A - B über zwei Schritte (Negation des Subtrahenden + Addition).
// **HÄRTUNG (ANTI-PATTERN 28):** Direkte Integration negierter Werte in Polynomial ist verboten.
// Nutze das explizite Pattern: 1. Negation via Polynomial (A1=-1). 2. Addition via Polynomial.

// --- Triggered ---
// Funktion: Identifiziert, welcher Eingang den aktuellen Lauf ausgelöst hat.
// **HÄRTUNG (ANTI-PATTERN 14 - DER MISSBRAUCHTE FLANKENDETEKTOR):**
// Die Verwendung als interner Flankendetektor ist verboten. Zweck ist NUR die Identifikation
// des externen Triggers oder des internen Timers (via negiertem Timer-Ausgang).
Syntax: [ "Triggered", "Input", "Touched_Output_Bool" ]

// --- Verschluss-Logik (Native) [NEU V4.8] ---
// **HÄRTUNG (ANTI-PATTERN 33 - MANUELLE FENSTER-LOGIK KETTEN):**
// Der Versuch, "Alles Zu" mit riesigen AND/OR-Ketten zu bauen, ist verboten. Nutze native Module.
//
// **HÄRTUNG (REGEL 1.32 - DAS FENSTER-SENSOR-GESETZ):**
// Bei Fenstern mit nur einem Kontakt MUSS der Eingang Kipp-Sensor fest auf TRUE gesetzt werden.
//
Syntax Status: ["Verschluss-Status", "$Kipp", "$Offen", "$StatusInt", "$OffenBool", "$Text"]
Syntax Statistik: ["Verschluss-Statistik", ["$StatusInt1", ...], "$AnzahlOffen", "$AnzahlGekippt", "$AllesZu"]

// --- Wakeup (Wecker) ---
// Funktion: Löst einen einmaligen 'true'-Impuls an einem definierten Unix-Zeitpunkt aus.
// **HÄRTUNG: IMPLIZITE TRIGGER-EIGENSCHAFT (REGEL 2.4)**
Syntax: [ "Wakeup", "UnixTimestamp_In", "Alarm_Out_Bool" ]

--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_20.txt ---

