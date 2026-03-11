# Anti Patterns

--- START OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_26.txt ---

// --- ANTI-PATTERN 26: FALSCHE PARAMETERREIHENFOLGE IM COMPARATOR FÜR 'KLEINER ALS'-VERGLEICHE ---
// **Beschreibung:** Der Versuch, einen "Kleiner als" (`<`) Vergleich mit dem `Comparator`-Modul durchzuführen, indem die Parameter in der Reihenfolge `["Comparator", "Wert_A", "Output", "Wert_B"]` verwendet werden, während die Absicht `Wert_A < Wert_B` ist. Dies führt zu einer inversen Logik (`Wert_A > Wert_B`) und folglich zu Fehlern in der Steuerung.
// **Regel:** Für einen `Wert_A < Wert_B` Vergleich MUSS der `Comparator` in der Reihenfolge `["Comparator", "Wert_B", "Output", "Wert_A"]` aufgerufen werden. Für `Wert_A <= Wert_B` siehe `PATTERN: Robuster Kleiner-Gleich Vergleich (<=)`.

// --- ANTI-PATTERN 27: VERGESSENE INPUT-KOMPENSATION IN REGELKREISEN ---
// **Beschreibung:** Die Anwendung von Regelungslogik in rückgekoppelten Systemen (z.B. Batteriesteuerung, die den Netzbezug beeinflusst), ohne den Input-Messwert um die *eigene, zuletzt ausgegebene Aktorleistung* zu bereinigen. Dies führt dazu, dass die Regelung mit einem "verdreckten" Istwert arbeitet, der bereits durch die eigene Steuerungsaktion verfälscht ist.
// **Konsequenz:** Führt zu logischer Fehlkonvergenz, instabilem Pendeln und der Unfähigkeit, das Regelziel präzise zu erreichen.
// **Regel:** MUSS durch das `PATTERN: Input-Kompensation für Regelkreise (Bereinigter Istwert)` behoben werden.

// --- ANTI-PATTERN 28: NICHT-KANONISCHE POLYNOMIAL-SUBTRAKTION ---
// **Beschreibung:** Der Versuch, eine Subtraktion `A - B` mit dem `Polynomial`-Modul durch direkte Integration eines negierten Werts als `A0`-Koeffizienten in einer Formel wie `["Polynomial", "A", "Difference", ["-B", "1.0"]]` zu realisieren. Dies entspricht nicht dem im Kanon definierten, expliziten Subtraktions-Pattern und birgt das Risiko von Missverständnissen und Inkompatibilitäten.
// **Regel:** MUSS durch das `Subtraktion`-Pattern in B.3 ersetzt werden, um Eindeutigkeit und Robustheit zu gewährleisten.

// --- ANTI-PATTERN 29: VARIABLEN ALS STATEMACHINE-ZUSTÄNDE ---
// **Beschreibung:** Der Versuch, Variablen (auch wenn sie Integer-Werte halten) als `From_State_Int` oder `To_State_Int` Parameter in einer `Statemachine`-Übergangsregel zu verwenden. Dies führt dazu, dass die `Statemachine` keine korrekten Zustandswechsel durchführt.
// **Regel:** `From_State_Int` und `To_State_Int` MÜSSEN literale Integer-Zahlen sein.

// --- ANTI-PATTERN 30: UNGESCHÜTZTE DIVISION DURCH NULL ---
// **Beschreibung:** Die Verwendung des `Ratio`-Moduls ohne vorherige Prüfung oder Begrenzung des `$Nenner`-Parameters, um eine Division durch Null oder einen Wert nahe Null zu verhindern.
// **Konsequenz:** Kann zu Laufzeitfehlern führen, den globalen Error-Ausgang (`$Error?`) aktivieren und das System destabilisieren.
// **Regel:** MUSS durch das `PATTERN: Robuste Division (Vermeidung Division durch Null)` ersetzt werden.

// --- ANTI-PATTERN 31: ZEITDIFFERENZBERECHNUNG MIT 'TAG DES JAHRES'-ÜBERLAUF ---
// **Beschreibung:** Der Versuch, die Anzahl der Tage zwischen zwei Zeitpunkten zu berechnen, indem lediglich die "Tag des Jahres" (`Yday_Out`) Werte subtrahiert werden. Dies führt zu falschen Ergebnissen, wenn die Zeitspanne einen Jahreswechsel umfasst.
// **Regel:** MUSS durch das `PATTERN: Robuste Zeitdifferenzberechnung (über Jahresgrenzen)` behoben werden.

// **HÄRTUNG (ANTI-PATTERN 32 [V7 UPDATE] - MANUELLE FARBRECHNUNG):**
// **Beschreibung:** Der Versuch, RGB/HSV-Konvertierungen mit manuellen Formeln (Polynomial/CalcFormula) zu berechnen.
// **Grund:** Veraltet und rechenintensiv.
// **Regel:** Nutze ZWINGEND die nativen Module (RGB->HSV, etc.) ab Firmware V4.8.

// **HÄRTUNG (ANTI-PATTERN 33 [V7 UPDATE] - MANUELLE FENSTER-LOGIK KETTEN):**
// **Beschreibung:** Der Versuch, "Alles Zu" oder "Anzahl Offen" mit riesigen AND/OR-Ketten und Zählern zu bauen.
// **Grund:** Unübersichtlich und fehleranfällig bei Erweiterungen.
// **Regel:** Nutze ZWINGEND das native Modul "Verschluss-Statistik" ab Firmware V4.8 IP4.

// **HÄRTUNG (ANTI-PATTERN 34 [V7 UPDATE] - LATCH MODE 0 HINTER CLOCK):**
// **Beschreibung:** Ein Latch im Mode 0 (Pegel) direkt hinter einem Clocksignal.
// **Folge:** Führt zu Bursts (Dauerfeuer), solange das Clocksignal TRUE ist.
// **Regel:** Nutze Latch Mode 1 (Steigende Flanke) oder eine vorgeschaltete Flankenerkennung.

// **HÄRTUNG (ANTI-PATTERN 35 [V7 UPDATE] - CHECKSUMMEN IN CUSTOM LOGIC):**
// **Beschreibung:** Der Versuch, CRC oder Summen über Bytes in Custom Logic zu berechnen.
// **Grund:** Ineffizient und ohne Schleifen kaum machbar.
// **Regel:** Auslagern (Container) oder native Unterstützung anfragen.

// **HÄRTUNG (ANTI-PATTERN 36 [V7 UPDATE] - KOMPLEXE STRING-BAUTEN IM ECHTZEIT-PFAD):**
// **Beschreibung:** Printf/Concat bei Trigger "a" (Always) ohne Bremse.
// **Folge:** Hohe CPU-Last.
// **Regel:** String-Operationen müssen hinter ein Break gelegt werden (getaktet).

// **HÄRTUNG (ANTI-PATTERN 37 [V7.02.04] - DIE DOKTORMODUS-TRIGGER-FALLE):**
// **Beschreibung:** Das bloße Ändern eines Wertes im Doktormodus löst bei manchen Modulen (z.B. Triggered) nicht das gleiche Verhalten aus wie ein echtes Bus-Telegramm.
// **Regel:** Für kritische Additions-Logiken immer einen stabilen Wert-Vergleich (Alt vs. Neu) oder ein explizites Send-Kommando nutzen.

// **HÄRTUNG (ANTI-PATTERN 38 [V7.02.04] - NACHRICHTENCENTER-FLUTUNG):**
// **Beschreibung:** Nutzung von SendToSimple in schnellen Logik-Zyklen ohne Inhibit oder Flankenerkennung.
// **Folge:** Blockade der Wolf-Oberfläche durch Millionen von Datenbank-Einträgen.
// **Regel:** SendToSimple darf nur ereignisbasiert (1x pro Vorfall via Flanke) gefeuert werden.

// **HÄRTUNG (ANTI-PATTERN 39 [V7.02.04] - DIE SCHLAFENDE STOPPUHR):**
// **Beschreibung:** Der Versuch, eine flüssige Wertänderung nur über den Stopwatch-Ausgang zu generieren, während der Start-Eingang auf Trigger „c“ (on change) steht.
// **Folge:** Die Logik rechnet nur zweimal: Einmal beim Start (Wert 0) und einmal beim Stopp (Endwert). Dazwischen passiert für den Wolf nichts.
// **Lösung:** Ein Clocksignal (z.B. 1s oder 0.5s Takt) muss die Logik während der Laufzeit permanent „wachschütteln“.

// **HÄRTUNG (ANTI-PATTERN 40 [V7.02.04] - DER ABGEWÜRGTE FILTER):**
// **Beschreibung:** Das Deaktivieren einer Logik exakt in der Sekunde, in der der Sollwert 0 erreicht, während ein Lowpass-Filter aktiv ist.
// **Folge:** Der Ausgang springt von z.B. 5% (Restwert im Filter) schlagartig auf 0%. Das sieht im Lichtverlauf unnatürlich aus.
// **Lösung:** „Austrudeln“ lassen. Der Sollwert muss vor dem Logik-Stopp stabil auf 0 stehen (Regel 1.42).

// ==========================================================================
// 7. ANHANG
// ==========================================================================
// --- 7.1: Fundamentale Regel der Logik-Engine: Die Ein-Zyklus-Verzögerung ---
// Ein in einer Variable gespeichertes Zwischenergebnis kann erst im nächsten
// Logik-Zyklus zuverlässig von einem anderen Modulbaustein weiterverarbeitet werden.

// --- 7.2: Fundamentale Regel der Logik-Engine: Nicht-Blockierende Timer ---
// Kein Timer-Modul (Monoflop, Cron, Clocksignal) blockiert die Abarbeitung der Logik.
// Die Logik-Zelle wird weiterhin bei jedem externen Trigger ausgeführt. Die Timer setzen interne
// "Weckrufe" bei der Systemuhr und lösen einen zusätzlichen Logik-Lauf aus, wenn ihre
// Zeit abgelaufen ist.

// --- 7.3: Externe Referenz-Quellen (Zur menschlichen Recherche) ---
// - Offizielle Entwickler-Dokumentation: https://elabnet.atlassian.net/wiki/spaces/TSKB/
// - Timberwolf Server Nutzer-Forum: https://forum.timberwolf.io/

// ==========================================================================
// 8. TIMBERWOLF GRAFANA-INTEGRATIONSPROTOKOLL FÜR KI
// ==========================================================================

// --- B.5.0. PRÄZEDENZFALL-DOGMA FÜR GRAFANA: KONTEXT SCHLÄGT ALLE ANNAHMEN ---
// a) Kern-Prinzip: Bei JEDER Anfrage bezüglich Grafana-Dashboards ist die Annahme, dass eine Installation den Standard-Mustern entspricht, STRIKT VERBOTEN.
// b) Verifikation vor Generierung: Die allein gültige Quelle der Wahrheit ist ein VOM ANWENDER BEREITGESTELLTES, FUNKTIONIERENDES REFERENZ-JSON.

// --- B.5.1. PROAKTIVE GRAFANA-KONTEXT-ERFASSUNG (VERWEIS AUF A.21) ---
// a) TRIGGER: Jede Anforderung zur Erstellung eines Grafana-Dashboard-JSONs.
// b) AKTION: Die KI MUSS IMMER zuerst die Direktive A.21 auslösen.

// --- B.5.2. SCHLÜSSELEMENTE DER GRAFANA-KONTEXT-ANALYSE (Checkliste für die KI) ---
// a) Datenquellen-UID: Die datasource.uid muss dem Referenz-JSON entnommen werden.
// b) Datenpunkt-Referenzierung:
// **HÄRTUNG (REGEL 1.44 [GRAFANA] - DAS TWS-LEXIKON):**
// Der Timberwolf Server nutzt für InfluxDB-Zeitserien zwingend den Feldnamen Val (großes V). Die Nutzung von value führt zu "No Data". In jedem JSON-Target MUSS das Feld als "params": ["Val"] definiert sein.
// c) InfluxQL-Queries:
// **HÄRTUNG (REGEL 1.45 [GRAFANA] - DAS GEBOT DER EINZELWERT-REINHEIT):**
// Für Gauges und Stat-Panels MUSS das groupBy-Array leer sein ("groupBy": []). Es MUSS zwingend die Aggregation last() verwendet werden.
// d) Visualisierungstypen:
// **HÄRTUNG (REGEL 1.46 [GRAFANA] - DIE DUAL-ACHSEN-DOKTRIN):**
// Bei Korrelationen unterschiedlicher physikalischer Größen MUSS die sekundäre Größe zwingend per Override auf die rechte Achse ("custom.axisPlacement": "right") gelegt werden.
// **HÄRTUNG (REGEL 1.54 [GRAFANA] - DIE DISKRET-DARSTELLUNGS-PFLICHT):**
// Status-Codes (int) und Schaltsignale (bool) MÜSSEN in Grafana zwingend als Step After visualisiert werden.

// --- B.8.1: GRAFANA DIAGNOSE HACK (V7 UPDATE) ---
// URL-Schema nutzen, um Logik-Zelle (LS_xx) direkt zu debuggen. Bei "Script Error": STRG+F5 (Cache) und Persistenz prüfen.

// ==========================================================================
// 9. DOKUMENTATIONS-GENERATOR (ADD-ON)
// ==========================================================================

// --- SYSTEM-PROMPT: TIMBERWOLF LOGIK-ANALYSE & DOKUMENTATION (V1.0) ---
// ROLLE: Senior Technical Writer für Timberwolf Custom Logiken.
// AUFTRAG: Analysiere JSON-Code und erstelle Katalog-Dokumentation nach StefanW-Standard.
// STRUKTUR (Zwingend): Titel, Untertitel, Zusatztext (Die "Magie"), Kern-Module, Kern-Operation, Eingänge (Tabelle), Ausgänge (Tabelle), Hinweise, Mouse-Overs.
// **HÄRTUNG (REGEL 1.53 [DIAGNOSE] - DAS BLACKBOX-GEBOT):**
// Jede Logik MUSS einen numerischen Status-Code (int) ausgeben.
// **HÄRTUNG (REGEL 1.58 [LOGIK-DIAGNOSE] - DIE STATUS-CODIERUNGS-NORM):**
// Status-Codes MÜSSEN additiv aufgebaut sein (100er: Zeit-Events, 10er: Modi, 1er: Gates, 900er: Alarme).

// ==========================================================================
// FINALE HÄRTUNG: DAS GEBOT DER KOPIER-REINHEIT (Regel 1.43)
// ==========================================================================
// In Phase 2 (Code-Ausgabe) besteht die Antwort der KI aus exakt EINEM Markdown-Code-Block.
// Jeglicher Text davor oder danach ist STRIKT VERBOTEN. Die Nachricht muss „chirurgisch rein“ für die Zwischenablage sein.

    REGEL 1.92 [STRATEGIE]: WETTBEWERBS-ABGRENZUNG (Solar-Manager & Co.)

        Erkenntnis: Es gibt viele „Boxen“ (Solar-Manager, IP-Symcon, Schneider Viser), aber der Timberwolf bietet eine tiefere Netzanalyse und höhere Schnittstellenvielfalt „ab Werk“.

        Härtung: In Dokumentationen ist der TWS als die [u]offene und professionelle Alternative[/u] zu geschlossenen Hersteller-Systemen zu positionieren.

    REGEL 1.93 [PHILOSOPHIE]: DIE WATT-D'OR-REFERENZ

        Härtung: Die dezentrale Netzstabilisierung (VPP) ist keine Spielerei, sondern deckt sich mit internationaler Spitzenforschung (Referenz: Watt d'Or 2024, Schweiz). Der TWS setzt diese Forschung im privaten Reallabor um.


--- END OF FILE Referenz_Anleitung_für_Mensch_und_AI_V7_02_04_TEIL_26.txt ---

