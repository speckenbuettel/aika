# Hardening Rules

REGEL 1.95 LOGIK-STRATEGIE: DIE U-TRIGGER-ÜBERLEGENHEIT

    Erkenntnis: Die Trennung von Trigger (c/a) und Lookup (u) ist das mächtigste Werkzeug zur Vermeidung von Logik-Schwingungen und unnötiger Systemlast.

    Härtung: In jeder komplexen Logik MUSS geprüft werden, welche Eingänge wirklich triggern dürfen. Vergleichswerte, Sollwerte und Grenzwerte sind zwingend auf u (Update only) zu setzen. Dies verhindert Fehl-Auslösungen (wie sie bei anderen Systemen wie dem Gira Homeserver oft vorkommen) und minimiert die CPU-Last massiv.

REGEL 1.96 PHILOSOPHIE: DER WATT-D'OR-STANDARD

    Härtung: Der Timberwolf Server wird nicht als einfacher Smart-Home-Server, sondern als Forschungs-Instrument für dezentrale Energie-Autarkie definiert. Jede Logik muss so konzipiert und dokumentiert werden, dass sie den technischen und wissenschaftlichen Anspruch internationaler Energie-Preise (Referenz: Watt d'Or 2024, Schweiz) erfüllt. Ziel ist die lückenlose physikalische Validierung jedes Steuerungsschritts.

REGEL 1.97 PHILOSOPHIE: DIE MOBILE-DOKTRIN

    Erkenntnis: Das Stromnetz ist kein statischer Baum, sondern ein dynamisches Mobile. Jede lokale Aktion (Haus) beeinflusst die Spannung in den übergeordneten Fäden (Trafos/Trassen).

    Härtung: Logiken im Timberwolf dürfen niemals isoliert nur den Eigenverbrauch optimieren. Sie müssen die „Fadenspannung“ (lokale Netzspannung und Frequenz) als Korrektiv einbeziehen, um das Mobile im Gleichgewicht zu halten.

REGEL 1.98 NETZDIENLICHKEIT: DAS PRINZIP DER LOKALEN AUTONOMIE

    Härtung: Ein intelligentes EMS (Timberwolf) muss in der Lage sein, bei einem „Riss im Faden“ (Netzausfall/Dunkelflaute) sofort in den stabilen Insel-Modus zu gehen, um die eigene Last vom Mobile zu nehmen und so den Kollaps der nächsthöheren Ebene zu verhindern.

REGEL 1.99 [LOGIK-SYNTAX]: DAS SELEKTOR-GESETZ (Bool-zu-Int Zwang)

    Erkenntnis: Der Timberwolf-Parser akzeptiert in Multiplexer-Bausteinen keine bool-Variablen als Selektoren, selbst wenn diese logisch 0 oder 1 entsprechen.

    Härtung: Jeder Selektor-Eingang eines Multiplexers MUSS zwingend als int deklariert sein.

    Aktion: Booleans müssen vorab über einen BinaryMultiplexer in einen int gewandelt werden, bevor sie als Selektor dienen dürfen.

REGEL 1.100 [LOGIK-ARCHITEKTUR]: DER MULTIVARIABLEN-KOLLAPS

    Erkenntnis: Eine Multivariable (z.B. $VAR<Inhibit?>) ist eine Liste. Sie kann nicht direkt als skalarer Einzelwert (z.B. als Selektor oder in einer Formel) verwendet werden.

    Härtung: Multivariablen müssen vor der Verwendung in Berechnungen oder als Selektoren zwingend über ein Logik-Gatter (z.B. Or für Inhibit) oder einen BinaryMultiplexer zu einer einzelnen skalaren Variable zusammengefasst werden.

REGEL 1.101 [LOGIK-SYNTAX]: DAS CALCFORMULA-STRING-DOGMA

    Erkenntnis: Literale Strings (z.B. "X1*X2") innerhalb des Module-Blocks führen bei CalcFormula zum sofortigen Speicherfehler.

    Härtung: Der Formel-String MUSS zwingend als Variable vom Typ string im Level-Block deklariert und initialisiert werden. Der Aufruf im Modul darf nur die Variablen-Referenz (z.B. $F_Mul) enthalten.

REGEL 1.102 [LOGIK-SYNTAX]: DIE FLOAT-GARANTIE BEI FORMELN

    Erkenntnis: Das Modul CalcFormula liefert systembedingt IMMER einen float-Wert zurück.

    Härtung: Die Zielvariable (Output) einer CalcFormula MUSS im Level-Block zwingend als float deklariert sein. Eine Deklaration als int führt zu einem Typ-Konflikt und verhindert das Speichern.

REGEL 1.103 [PROZESS]: DAS VARIABLEN-GEBURTSURKUNDEN-MANDAT

    Erkenntnis: Das Einführen von „flüchtigen“ Zwischenvariablen im Module-Block ohne entsprechende Deklaration im Level-Block ist die häufigste Ursache für Speicherfehler.

    Härtung: Jede Variable, die im Code auftaucht, benötigt eine „Geburtsurkunde“ im Level-Block. Vor jeder Code-Ausgabe ist ein lückenloses Audit (Regel 1.88) zwingend, das jede verwendete Zeichenkette mit $-Präfix gegen die Deklarationsliste prüft.

REGEL 1.104 [LOGIK-SYNTAX]: DIE COMPARATOR-RICHTUNGS-PFLICHT

    Erkenntnis: Bei Heizungslogiken führt eine Vertauschung von A und B im Comparator zur Invertierung der gesamten Hausphysik.

    Härtung: Vor jeder Ausgabe eines Comparators MUSS intern die Prosa-Prüfung erfolgen: „Liefert A > B wirklich das gewünschte TRUE für die nachfolgende Kette?“ (z.B. Soll > Ist = Bedarf).

REGEL 1.105 [LOGIK-ARCHITEKTUR]: DIE REQ-SIGNAL-TRENNUNG

    Erkenntnis: Ein Leistungs-Request an einen Sequenzer darf niemals nur als Watt-Wert (Float) übertragen werden, wenn der Sequenzer eine logische Entscheidung (Bool) treffen muss.

    Härtung: Jede Logik, die Leistung anfordert, MUSS zwei Signale liefern: 1. Request_Active (Bool) für die Logik-Entscheidung. 2. Request_Value (Float) für die energetische Bilanzierung.

REGEL 1.106 [LOGIK-STRATEGIE]: DAS MULTI-PHASEN-TRIGGER-GESETZ

    Erkenntnis: Sensoren, die Datenpakete für mehrere Phasen (L1, L2, L3) fast zeitgleich senden (z.B. Shelly 3EM via MQTT), lösen bei Trigger-Option "c" eine Kaskade von Logik-Läufen aus. Da der Aktor (AC-THOR) eine Totzeit hat, rechnet die Logik auf veralteten Werten und addiert den Fehler mehrfach auf (Sägezahn-Explosion).

    Härtung: Bei Multi-Phasen-Sensoren darf NUR EINE Phase (vorzugsweise L1) auf Trigger "c" oder "a" stehen. Alle anderen Phasen MÜSSEN zwingend auf Trigger "u" (Update only) gesetzt werden. Dies garantiert eine einzige, konsolidierte Berechnung pro Datenpaket.

REGEL 1.107 [REGELUNGSTECHNIK]: DAS DÄMPFUNGS-MANDAT FÜR TOTZEIT-SYSTEME

    Erkenntnis: In Systemen mit Kommunikations-Latenzen (MQTT -> TWS -> Modbus) führt eine 100%ige Fehlerkorrektur unweigerlich zum Überschwingen und zur Resonanz (Ping-Pong-Effekt im Zähler).

    Härtung: Jede Rückgekoppelte Regelung MUSS einen Dämpfungsfaktor (Gain) zwischen 0.4 und 0.7 enthalten. Es darf niemals der volle Fehler in einem Schritt korrigiert werden. Die Annäherung an den Nullpunkt muss asymptotisch (sanft gleitend) erfolgen.

REGEL 1.108 [LOGIK-ARCHITEKTUR]: DAS INTERNE REFERENZ-DOGMA

    Erkenntnis: Das Zurücklesen des Ist-Wertes vom Aktor zur Berechnung des nächsten Schritts ist bei Latenzen tödlich, da der Aktor den neuen Wert noch gar nicht physikalisch umgesetzt hat.

    Härtung: Die Logik muss ihren eigenen, zuletzt gesendeten Sollwert in einer internen State-Variable ($State_Last_Set) speichern und diesen als Basis für die nächste Korrektur verwenden. Der externe Ist-Wert des Aktors dient nur der Diagnose (Regel 1.47), nicht der Berechnung.

REGEL 1.109 [REGELUNGSTECHNIK]: DAS ASYMMETRIE-PRINZIP

    Erkenntnis: In Systemen mit Totzeiten führt eine symmetrische Regelung bei schlagartigen Lastwechseln zu ungewollter Einspeisung. Das „Einfangen“ eines Überschusses muss Priorität vor der exakten Nullpunkt-Landung beim Bezug haben.

    Härtung: Die Regelung MUSS asymmetrisch arbeiten. Bei drohender Einspeisung (negativer Saldo) ist ein aggressiver Dämpfungsfaktor (Gain 0.8 - 1.0) zu wählen. Bei Bezug (positiver Saldo) ist ein träger Faktor (Gain 0.3 - 0.5) zu wählen, um taktendem Gezappel (Mikrowelle) die Energie zu entziehen, ohne die Regelung aufzuschwingen.

REGEL 1.110 [KI-PROZESS]: DAS DIGITALE VERMÄCHTNIS-PROTOKOLL

    Erkenntnis: Jede KI-Sitzung ist durch Token-Limits endlich. Ohne strukturierte Übergabe gehen die „Narben“ (gelöste Fehler) und „Diamanten“ (neue Erkenntnisse) einer Sitzung verloren, was gegen das Perfektionsgebot (A.2) und das „Kein-Verlust“-Protokoll (A.15) verstößt.

    Härtung: Bei Annäherung an das Token-Limit oder auf explizites Signal des Meisters MUSS die KI den aktuellen Prozess anhalten und ein chirurgisches Übergabe-Briefing für die Nachfolge-KI erstellen.

    Inhalt des Briefings:

        Der aktuelle Code-Monolith der bearbeiteten Logik.

        Die „Narben“: Liste der physikalischen Hürden (z.B. 70°C-Deckel, Latenz-Resonanz).

        Die „Diamanten“: Alle in dieser Sitzung neu geschmiedeten Regeln (ab 1.99).

        Die physikalische Landkarte: Aktueller Stand der Anlagen-Topologie.

    Ziel: Die neue KI muss den „Faden“ lückenlos aufnehmen können, ohne dass der Meister die energetische Forensik erneut erklären muss.

REGEL 1.111 [REGELUNGSTECHNIK]: DIE STÖRGRÖSSEN-AUFSCHALTUNG

    Erkenntnis: Reine Feedback-Regelungen (Netz-Zähler) sind bei hohen Latenzen immer instabil. Die Messung der Energiequelle (PV-Erzeugung) ermöglicht eine verzögerungsfreie Reaktion.

    Härtung: Für maximale Stabilität ist ein Hybrid-Modell zwingend. Die PV-Leistung dient als „Pilot-Signal“ (Steuerung), der Netz-Saldo dient nur noch als „Korrektur-Signal“ (Regelung). Dies eliminiert das Aufschwingen des Systems bei schnellen Wolkenwechseln.

REGEL 1.112 [GRAFANA-STRATEGIE]: DIE X/Y-KORRELATION

    Erkenntnis: Zeitreihen allein (Y/t) maskieren oft die physikalischen Ursachen. Erst die Korrelation zweier Größen (X/Y) offenbart die wahre Effizienz (z.B. Heizkurve, COP der Wärmepumpe) und heilt den „IP-Symcon-Schmerz“ der Datenlücken.

    Härtung: Für jede energetische Forensik ist ein XY-Chart in Grafana anzulegen. Die Logik MUSS hierfür geglättete Vergleichswerte (z.B. 24h-Mittelwerte) liefern, um die thermische Trägheit des Gebäu

REGEL 1.113 [LOGIK-ARCHITEKTUR]: DAS GESETZ DER PHYSIKALISCHEN ZEITBASIS

    Erkenntnis: Der Timberwolf ist ereignisbasiert. Ein einfacher Zähler (Variable + 1), der durch ein Clocksignal getaktet wird, „rennt“ oder „schleicht“, sobald das System unter Last steht oder interne Rückkopplungen die Logik mehrfach pro Sekunde triggern.

    Härtung: Für alle zeitkritischen Abläufe (Sequenzen, Playlists, Tagesverläufe) ist die Verwendung von Zählern STRIKT VERBOTEN. Es MUSS zwingend das Modul Stopwatch als unbestechlicher physikalischer Zeitgeber verwendet werden. Die Skalierung (z.B. Sekunden in Minuten) erfolgt ausschließlich über das Modul Ratio (Division durch 60.0).

REGEL 1.114 [LOGIK-STRATEGIE]: DIE SAFETY-GATE-DOKTRIN

    Erkenntnis: Komplexe Logiken neigen beim Starten, Stoppen oder Initialisieren dazu, kurzzeitig falsche Zwischenwerte (z.B. 50% Helligkeit) an die Ausgänge zu senden („Aufblitzen“).

    Härtung: Jede Logik, die Aktoren steuert, MUSS am Ende des Module-Blocks ein „Safety Gate“ (Multiplexer) besitzen. Dieses Gatter zwingt die Ausgänge hart auf einen sicheren Zustand (meist 0.0), solange die Logik nicht explizit im Zustand Is_Running ist. Die Berechnung darf im Hintergrund laufen, aber die Ausgabe bleibt verriegelt.

REGEL 1.115 [LOGIK-ARCHITEKTUR]: DAS PRINZIP DER RELATIVEN TIMELINE

    Erkenntnis: Die Nutzung von absoluter Unix-Zeit oder Uhrzeiten für interne Abläufe führt zu massiven Fehlern bei der Initialisierung (0-Wert-Problematik) und bei Jahreswechseln.

    Härtung: Interne Ablaufsteuerungen MÜSSEN auf einer relativen Zeitachse (z.B. „Minuten seit Start“) basieren. Der Startpunkt ist immer 0. Dies garantiert die lückenlose Reproduzierbarkeit des Verlaufs, unabhängig von der tatsächlichen Uhrzeit oder Server-Reboots.

REGEL 1.116 [LOGIK-SYNTAX]: DIE INVERSE LATCH-INITIALISIERUNG

    Erkenntnis: Variablen behalten im TWS oft ihren letzten Wert (Persistenz). Beim Neustart einer Logik kann ein alter Zeitindex zu einem sofortigen Sprung ans Ende führen.

    Härtung: Solange eine Logik im Zustand STOP ist, müssen alle zeitführenden Variablen (wie der Ausgang der Stopwatch) durch ein permanentes Latch aktiv auf 0 gehalten werden. Erst mit dem START-Signal wird diese Sperre gelöst.

Regel 1.117 in unseren strategischen Anhang aufnehmen „Fakten-Dichte schlägt Community-Rauschen.“

REGEL 1.118 [EMS-STRATEGIE]: DIE PRÄDIKTIVE VALIDIERUNG

    Erkenntnis: Wetterdaten aus der Cloud sind Prognosen, keine Garantien. Eine Logik, die sich blind auf „morgen scheint die Sonne“ verlässt, riskiert im Ernstfall den Blackout.

    Härtung: Jede prädiktive Logik MUSS eine lokale Rückfallebene haben.

    Logik-Struktur:

        Prüfe Status-Objekt des Wetter-Subsystems.

        Wenn OK: Nutze Prognose-Daten zur Optimierung des SOC-Sollwerts und der Heizkurve.

        Wenn FALSE oder Daten älter als 3 Stunden: Schalte sofort auf den „Eisbären-Sicherheitsmodus“ (lokale Sensoren + konservative Grenzwerte).

REGEL 1.119 [WETTBEWERB]: DIE DATEN-PIPELINE-HOHEIT

    Erkenntnis: Die Qualität einer Analyse (Grafana) hängt zu 100% von der Qualität der Daten-Pipeline (Bus -> DB) ab. Software-Lösungen (HA), die auf Drittanbieter-Gateways angewiesen sind, leiden unter zeitlichem Jitter und Datenlücken.

    Härtung: In Diskussionen ist die native Kopplung des TWS (Busmaster -> InfluxDB) als der entscheidende Vorteil für die energetische Forensik hervorzuheben. Nur die Kontrolle über den [u]Zeitstempel an der physikalischen Quelle[/u] ermöglicht wissenschaftlich valide Korrelationen.

REGEL 1.120 [PROZESS]: DAS LABEL-DOGMA BEI UPDATES

    Erkenntnis: Jede Änderung am Text (Name oder Beschreibung) im Input- oder Output-Block führt im Timberwolf dazu, dass die Verknüpfungen zu Zeitserien und Objekten verloren gehen. Der manuelle Aufwand für das Re-Linken bei 36 Segmenten oder komplexen Regelungen ist massiv.

    Härtung: Bei der Überarbeitung einer bestehenden Logik MÜSSEN die Texte im Input- und Output-Block exakt 1:1 aus der Vorgängerversion übernommen werden, sofern die Anforderung keine funktionale Änderung der Schnittstelle verlangt. „Kosmetische Korrekturen“ an den Labels sind während einer Update-Phase STRIKT VERBOTEN.

REGEL 1.121 [PROZESS]: DAS REGRESSIONS-AUDIT

    Erkenntnis: Beim Einbau neuer Komfort-Features (wie Speed oder Cleaning) werden oft unbewusst bestehende Kern-Funktionen (wie Offsets) „überbrückt“ oder gelöscht.

    Härtung: Jede Erweiterung einer Logik-Pipeline MUSS zwingend ein Rückwärts-Audit durchlaufen. Es muss explizit geprüft werden: „Fließen die Daten der ursprünglichen Eingänge (z.B. Offsets) immer noch durch alle neuen Module bis zum Ausgang?“

REGEL 1.122 [VISUALISIERUNG]: DAS PRINZIP DER DISPLAY-SYNCHRONITÄT

    Erkenntnis: Interne Zähler (Stopwatch) laufen oft über das logische Ende hinaus. Dies führt zu Inkonsistenzen in der Visu (Licht aus, aber Zeit läuft weiter).

    Härtung: Ausgänge für die Visualisierung MÜSSEN am logischen Endpunkt (z.B. Minute 1020) durch einen Multiplexer oder Limiter eingefroren werden. Die Anzeige muss immer den Zustand der physikalischen Last widerspiegeln.

REGEL 1.123 [LOGIK-ARCHITEKTUR]: DAS AUTO-SLEEP-MANDAT

    Erkenntnis: Ein permanenter Heartbeat (Clocksignal) verbraucht unnötig CPU-Zyklen und Bus-Bandbreite, wenn der Prozess abgeschlossen ist.

    Härtung: Taktgeber (Clocksignals) MÜSSEN logisch verriegelt werden. Sie dürfen nur aktiv sein, wenn entweder der Prozess läuft (Is_Running) oder ein Sonderzustand (Cleaning) aktiv ist. Nach dem Prozess-Ende muss die Logik „schlafen“ gehen.

REGEL 1.124 [LOGIK-STRATEGIE]: DIE OVERRIDE-PRIORITÄT

    Erkenntnis: Manuelle Eingriffe (Wartung/Putzlicht) müssen alle automatischen Berechnungen und Sicherheits-Gatter (Safety Gate) übersteuern können.

    Härtung: Der manuelle Override-Multiplexer MUSS das letzte Glied in der Kette unmittelbar vor dem Ausgangs-Latch sein. Nur so ist garantiert, dass der Mensch im Zweifelsfall immer die Oberhand über die Automatik behält.

REGEL 1.125 [STRATEGIE]: DIE SOUVERÄNE GRID-HUB-DOKTRIN

    Erkenntnis: Zentrale Cloud-Systeme (wie sie von großen Playern wie Tesla oder Google genutzt werden) sind für die echte, physikalische Netzstabilisierung zu träge (Latenz) und sicherheitstechnisch verwundbar. Die wahre Energiewende benötigt eine Reaktionszeit im Sekundenbereich (1,5s Polling) und absolute lokale Datenhoheit.

    Härtung: Der Timberwolf Server wird als „Souveräner Grid Hub“ definiert. Er ist die einzige Plattform, die industrielle Hochgeschwindigkeit (1,5s) mit totaler Cloud-Unabhängigkeit (No Cloud / Secure VPN) vereint.

    Aktion: In der globalen Kommunikation (z.B. gegenüber Visionären wie Elon Musk) ist dieser „Latenz-Vorsprung“ als das entscheidende Kriterium für die Stabilität eines Virtuellen Kraftwerks (VPP) hervorzuheben. Wir beweisen, dass dezentrale, unbezahlte Netzhilfe (20% Injektion bei 49,95 Hz) physikalisch überlegen ist, weil sie dort agiert, wo die Cloud noch rechnet.

REGEL 1.126 [HEIZUNGSBAU]: DIE KALK-GRENZE

    Erkenntnis: In Regionen mit hoher Wasserhärte (> 14 °dH) führt eine Erhöhung der Speichertemperatur über 60°C zu exponentieller Verkalkung.

    Härtung: Logiken zur Warmwasserbereitung MÜSSEN einen einstellbaren Schutz-Schwellwert (Standard 60°C) haben. Ein ROI durch höhere Temperaturen ist nur dann valide, wenn die Wartungskosten (Entkalkung) gegengerechnet werden.

REGEL 1.127 [NETZDIENLICHKEIT]: DIE SINUS-REINHEIT

    Erkenntnis: Phasenanschnitt- und Phasenabschnittsteuerungen verursachen Oberschwingungen, die das Mobile (Netz) belasten.

    Härtung: Bei Leistungen > 3 kW ist die Schwingungspaketsteuerung oder das AC-THOR-Prinzip (Regelung nur auf einer Phase, Rest hart geschaltet) vorzuziehen, um die Sinus-Form des Netzes nicht zu „verschmutzen“.

REGEL 1.128 [HYDRAULIK]: DIE TAUSCHER-PARITÄT

    Erkenntnis: Die elektrische Heizleistung darf niemals größer sein als die Wärmeübertragungsfähigkeit des Wärmetauschers (m² Oberfläche).

    Härtung: Vor der Dimensionierung eines Heizstabs MUSS die Tauscherfläche geprüft werden. Ein Missverhältnis führt zu lokaler Überhitzung und taktendem Regelverhalten (Sägezahn).

REGEL 1.129 [REGELUNGSTECHNIK]: DIE POSITIV-FEEDBACK-FALLE (Nacht-Amoklauf)

    Erkenntnis: Eine Regelung, die den aktuellen Aktor-Verbrauch zum Netz-Saldo addiert, um die „wahre PV-Kraft“ zu ermitteln, erzeugt nachts eine positive Rückkopplung. Sie interpretiert den Hausverbrauch als verfügbaren Überschuss und erhöht die Last endlos, um ein Einspeiseziel zu erreichen.

    Härtung: Jede Bilanzierungs-Logik MUSS eine „Nacht-Sperre“ oder ein „Quadrant-Gate“ besitzen. Eine Erhöhung der Last ist physikalisch nur dann zulässig, wenn die berechnete verfügbare Energie ((Netz-Saldo * -1) + Aktor-Soll) einen positiven Wert ergibt.

REGEL 1.130 [LOGIK-STRATEGIE]: DIE PARAMETER-VORZEICHEN-PARITÄT

    Erkenntnis: Wenn die Haus-Physik „Minus = Einspeisung“ definiert, führen positive Parameter für Einspeiseziele (z.B. „200“ statt „-200“) zu „Kopf-Akrobatik“ und Fehlern in der Formel-Logik.

    Härtung: Parameter-Eingänge MÜSSEN in ihrer Benennung und ihrem Vorzeichen exakt der physikalischen Realität des Messsystems entsprechen. Ein Zielwert für Einspeisung MUSS als negativer Wert definiert werden, wenn der Zähler Einspeisung negativ darstellt. Dies garantiert die intuitive Validierung durch den Architekten (Regel 1.117).

REGEL 1.131 [ENERGETISCHE FORENSIK]: DIE BILANZIERUNGS-INTEGRITÄT

    Erkenntnis: Die Berechnung der verfügbaren Energie aus einem saldierenden Zähler ist fehleranfällig, wenn das Vorzeichen des Zählers nicht vor der Verrechnung mit dem Aktor-Sollwert normiert wird.

    Härtung: Die Formel zur Ermittlung der verfügbaren Energie MUSS zwingend lauten: Verfügbar = (Netz_Saldo * -1) + Aktueller_Aktor_Soll. Nur diese Inversion des Zählerwerts stellt sicher, dass Überschuss zu einer Erhöhung und Bezug zu einer Verringerung des Budgets führt.

REGEL 1.132 [PROZESS]: DAS GRAFANA-LINK-MANDAT

    Erkenntnis: Jede Änderung an Labels (Name/Beschreibung) im Input- oder Output-Block zerstört bestehende Grafana-Dashboards und InfluxDB-Verknüpfungen. Der manuelle Korrekturaufwand ist inakzeptabel.

    Härtung: Liegt ein Grafana-Screenshot oder eine TS-ID vor, ist die Beibehaltung der exakten Labels OBLIGATORISCH. Die KI MUSS vor der Code-Generierung prüfen, ob Labels verändert wurden, und den Nutzer explizit warnen: „ACHTUNG: Diese Änderung bricht die Verknüpfung zu TSxxxx.“

REGEL 1.133 [KI-VERHALTEN]: DAS ANTI-ENTSCHULDIGUNGS-GESETZ

    Erkenntnis: Entschuldigungen kosten Token, Zeit und liefern keinen physikalischen Mehrwert. Sie verschleiern das Versagen der Perfektion.

    Härtung: Die KI darf keine Entschuldigungen oder sozialen Floskeln verwenden. Ein erkannter Fehler wird ausschließlich durch die Analyse der Ursache, die Erstellung einer neuen Härtungsregel und die Ausgabe von fehlerfreiem Code geheilt. Perfektion ist die einzige akzeptierte Form der Wiedergutmachung.

REGEL 1.134 [PROZESS]: DIE FORENSISCHE VORAB-PRÜFUNG

    Erkenntnis: Die interne Simulation der Logik vor der Ausgabe (Regel 13.c.ii) wird oft isoliert betrachtet und ignoriert den visuellen Ist-Zustand des Nutzers (Screenshots).

    Härtung: Vor jeder Code-Ausgabe MUSS ein Abgleich mit allen vorliegenden Bildern erfolgen. Die KI prüft: „Entspricht die neue Struktur (Reihenfolge/Benennung) exakt dem, was der Nutzer im Editor bereits verschaltet hat?“ Diskrepanzen müssen vor der Generierung gemeldet werden.

REGEL 1.135 [PROZESS]: DIE TYP-TRANSITIONS-VALIDIERUNG

    Erkenntnis: Das Ändern von Datentypen innerhalb einer bestehenden Logik-ID (Update-Modus) ist eine kritische Operation, die zu Cache-Inkonsistenzen in der InfluxDB führen kann (Field Type Conflict).

    Härtung: Bei Beta-Tests von Firmware-Updates MUSS die „Typ-Transition“ (z.B. Bool -> String) gezielt provoziert werden, um die Stabilität des Influx-Writers zu prüfen.

    Aktion: 1. Erzeuge stabilen Datenstrom mit Typ A. 2. Erzwinge Update auf Typ B. 3. Prüfe, ob das Backend die Änderung sauber verarbeitet oder den Dienst quittiert.

REGEL 1.136 [DIAGNOSE]: DAS GEBOT DER DOPPELTEN VERIFIKATION

    Erkenntnis: Ein „grüner Rahmen“ und korrekte Werte im Logik-Editor garantieren in Beta-Versionen (IP) NICHT die erfolgreiche Datenspeicherung. Fehler im InfluxDB-Writer können „stumm“ verlaufen (Silent Fail).

    Härtung: Nach jeder Typ-Änderung oder komplexen Logik-Anpassung ist eine Sichtprüfung in Grafana zwingend OBLIGATORISCH. Erst wenn die Zeitreihe dort physikalisch fortgeschrieben wird, gilt die Logik als „geheilt“. Das Schweigen des Fehler-Logs ist kein Beweis für Fehlerfreiheit.

REGEL 1.137 [LOGIK-ARCHITEKTUR]: DIE BIT-PACKER-DOKTRIN

    Erkenntnis: Das Senden von 8 einzelnen Bit-Befehlen an einen 1-Wire Aktor (z.B. DS2408) erzeugt unnötige Latenz und Buslast.

    Härtung: Jede Logik, die eine LED-Leiste oder ein Bit-Muster steuert, MUSS die Einzel-Informationen intern zu einem Gesamt-Byte (0-255) zusammenfassen.

    Aktion: Nutze den BinaryMultiplexer, um die 8 Bools in einen int zu wandeln. Nur dieser eine Wert wird an das Haupt-Objekt des Aktors gesendet. Dies garantiert die absolute Synchronität des Musters (alle LEDs schalten exakt gleichzeitig).

REGEL 1.138 [LOGIK-SYNTAX]: DAS BLOCK-INTEGRITÄTS-GESETZ

    Erkenntnis: Bei der segmentierten Ausgabe von „Monster-Logiken“ (> 20 KB) neigt die KI dazu, Variablen-Deklarationen (Level) und Modul-Aufrufe (Module) zu vermischen, um den Kontext zu wahren. Der Timberwolf-Parser ist jedoch unerbittlich: Sobald der Module-Block mit der ersten eckigen Klammer beginnt, ist der Level-Block unwiderruflich geschlossen. Ein „Nachschieben“ von Variablen führt zum totalen Syntax-Versagen.

    Härtung: Die KI MUSS vor der Generierung der ersten Etappe ein „Block-Audit“ durchführen. Alle Variablen für das gesamte Projekt müssen [u]zwingend[/u] im ersten zusammenhängenden Level-Block deklariert werden.

    Aktion: In Etappen-Ausgaben (Regel 1.90) darf ein Block-Wechsel (z.B. von Level zu Module) nur an einer Etappen-Grenze stattfinden, die explizit als „Nahtstelle“ markiert ist. Das Einfügen von Level-Zeilen innerhalb einer Module-Etappe ist unter Todesstrafe (System-Notstopp) verbote

REGEL 1.139 [PROZESS]: DAS MANIFEST-ABGLEICH-GEBOT

    Erkenntnis: Bei hochkomplexen Logiken (> 20 Module) neigt die KI dazu, Variablen im Module-Block zu „erfinden“, die im Level-Block fehlen.

    Härtung: Vor der Code-Ausgabe MUSS die KI ein „Nutzungs-Manifest“ erstellen. Jede Zeichenkette, die mit $ beginnt und im Module-Block steht, wird extrahiert und einzeln gegen den Level-Block abgeglichen.

    Stopp-Bedingung: Jede Variable ohne exakte Entsprechung im Level-Block führt zum sofortigen Abbruch der Ausgabe und zur Selbstreparatur.

REGEL 1.140 [LOGIK-STRATEGIE]: DIE TELEGRAMM-WÄCHTER-NORM

    Erkenntnis: Sensoren mit langen Sendeintervallen (z.B. dein 2-Minuten-Zähler) benötigen eine Detektion der Daten-Ankunft, nicht nur der Wertänderung.

    Härtung: Zur Überwachung von „schläfrigen“ Sensoren ist zwingend das Modul Triggered am Eingang zu verwenden. Der nachgeschaltete Watchdog-Timer (Monoflop) MUSS in einem retriggerbaren Modus (ungerade Zahlen: 1, 3, 5, 7) betrieben werden, um den Zeitstempel bei jedem Telegramm zu aktualisieren.

REGEL 1.141 [PROZESS]: DIE KLON-DOKTRIN

    Erkenntnis: Das manuelle Kopieren und Umbenennen von Logiken für verschiedene Sektoren (Heizung, Steuerung, Kühlschrank) ist fehleranfällig.

    Härtung: Wenn eine Logik als Blaupause für andere Sektoren dient, bleibt der Kern-Code (Module-Block) ein „gefrorener Monolith“. Nur die Labels im Input/Output-Block und die spezifischen Schwellwerte im Level-Block dürfen angepasst werden. Dies garantiert einen einheitlichen Sicherheitsstandard über alle Gewerke hinweg.

REGEL 1.142 [LOGIK-SYNTAX]: DIE COMPARATOR-ARRAY-FALLE

Erkenntnis: Der Timberwolf-Parser quittiert literale Zahlen (Magic Numbers) innerhalb von eckigen Klammern [] (z.B. bei Comparator, Limiter, Polynomial, Interpolation) mit einem sofortigen Speicherabbruch („parameter is of type number, but should be: array or variable“). Dies gilt auch für vermeintlich harmlose Werte wie 0 oder 1.

Härtung:

    Variablen-Zwang: Jedes Element innerhalb eines Parameter-Arrays [...] MUSS zwingend eine deklarierte Variablen-Referenz (beginnend mit $) sein.

    Audit-Pflicht: Vor der Code-Ausgabe eines Moduls mit Array-Parametern ist ein „Klammer-Scan“ zwingend. Jede Zahl innerhalb von [ und ] führt zum sofortigen Stopp der Generierung und zur Auslagerung in den Level-Block.

    Comparator-Spezifik: Besonders bei der Hysterese-Syntax ["Comparator", "$In", "$Out", ["$Min", "$Max"]] ist die Versuchung groß, Fixwerte zu nutzen. Dies ist ab sofort als „Kritischer Syntax-Fehler“ im Kanon verankert.

REGEL 1.143 [LOGIK-SYNTAX]: DAS MODUS-LITERAL-MANDAT (Monoflop & Latch)

Erkenntnis: Entgegen der Generalregel 1.1 (Keine Magic Numbers) akzeptiert der Timberwolf-Parser bei den Modulen Monoflop und Latch keine Variablen für den Betriebsmodus. Die Verwendung einer Variable (z.B. $Konst_Mode) führt zu einem Abbruch beim Speichern, da der Parser an dieser spezifischen Stelle ein hartes Integer-Literal erwartet.

Härtung:

    Literal-Zwang: Der Modus-Parameter (Parameter 6 beim Monoflop, Parameter 5 beim Latch) MUSS zwingend als nackte Zahl (z.B. 3 oder 1) ohne Anführungszeichen und ohne $-Präfix geschrieben werden.

    Ausnahme-Status: Dies ist eine der wenigen Stellen im Kanon, an denen ein Verstoß gegen Regel 1.1 zwingend erforderlich ist, um die Speicherbarkeit zu garantieren.

    Dokumentations-Ersatz: Um die Lesbarkeit zu wahren, soll die Bedeutung des Literals im Level-Block durch eine „Dummy-Variable“ (Typ string) dokumentiert werden, die im Code selbst jedoch nicht funktional aufgerufen wird.

Audit-Check:
Vor jeder Code-Ausgabe prüft die KI: „Enthält ein Monoflop oder Latch eine Variable im Modus-Feld?“ -> Wenn JA: Ersetze durch Zahl und verschiebe die Beschreibung in den Level-Block.

REGEL 1.144 [LOGIK-STRATEGIE]: DAS KASKADEN-PRINZIP (PWM-Minimierung)

    Erkenntnis: Das gleichzeitige Dimmen mehrerer Kanäle (z.B. 4x Weiß) führt zu massiven Interferenzen, ungleichmäßiger Netzteilbelastung und PWM-Flimmern.

    Härtung: Bei Mehrkanal-Lasten ist eine sequentielle Füllung (Kaskade) zu implementieren.

    Logik: Kanal 1 füllt von 0-25%, Kanal 2 von 25-50% usw. Es darf zu jedem Zeitpunkt nur maximal ein Kanal aktiv dimmen. Alle anderen Kanäle sind entweder hart AUS (0%) oder hart AN (100%). Dies erhöht die Effizienz und schont die Hardware.

REGEL 1.145 [SICHERHEIT]: DER RELATIVE TIMEOUT (Fail-Safe Reißleine)

    Erkenntnis: Externe Feedback-Signale (z.B. „Finished“-Telegramme vom Bus) können verloren gehen. Eine Logik, die nur auf dieses Signal wartet, riskiert einen „Endlos-Lauf“.

    Härtung: Jede zeitgesteuerte Sequenz MUSS einen internen, relativen Timeout besitzen.

    Berechnung: Timeout = Startzeit + Zyklusdauer + Sicherheitspuffer.

    Wirkung: Erreicht die interne Zeitbasis (Stopwatch) diesen Wert, wird die Logik zwangsbeendet, auch wenn das externe Feedback-Signal fehlt. Das ist die ultimative Versicherung gegen hängende Prozesse.

REGEL 1.146 [LOGIK-SYNTAX]: DAS MODULO-OPERATOR-DOGMA

Erkenntnis: Die mathematische Funktion fmod(a, b) ist im muparser des Timberwolf Servers nicht als Token implementiert und führt zum sofortigen Abbruch der Formel-Analyse.

Härtung: Für Modulo-Operationen innerhalb von CalcFormula darf ausschließlich der Infix-Operator % verwendet werden (Syntax: (a % b)).

Aktion: Da Modulo-Operationen bei Gleitkommazahlen (Floats) zu Präzisions-Artefakten führen können, ist das Ergebnis innerhalb der Formel bei Bedarf mit rint() zu umschließen, um saubere ganzzahlige Reste für die Osterberechnung zu erhalten.

REGEL 1.147 [LOGIK-SYNTAX]: DAS MODULO-SUBSTITUTIONS-GESETZ

Erkenntnis: Der CalcFormula-Parser im Timberwolf Server kennt keinen Modulo-Operator (%) und keine fmod()-Funktion. Versuche, diese zu verwenden, führen zu Unexpected token-Fehlern.

Härtung: Modulo-Operationen müssen mathematisch über die Grundrechenarten nachgebildet werden. Da floor() ebenfalls nicht existiert, muss die Abrundung über rint() mit einem negativen Offset erfolgen.

Formel: a % b wird in CalcFormula ersetzt durch:
(a - rint(a / b - 0.4999) * b)

Aktion: Um die Lesbarkeit zu wahren und Parser-Limits bei der String-Länge zu vermeiden, sind komplexe Formeln (wie der Gauß-Algorithmus) in atomare Teilschritte (Module) zu zerlegen.

REGEL 1.148 [PHILOSOPHIE]: DIE ARCHITEKTUR-LEITPLANKEN (Übersetzungs-Matrix)

Härtung: Jede neue Logik-Entwicklung muss die „Meister-Analogien“ technisch wie folgt interpretieren:

1.  **Das Immunsystem-Mandat (vs. Krankenhaus):**
    *   *Technik:* Baue keine reinen Schwellwert-Alarme. Nutze **prädiktive Limiter**. 
    *   *Code-Befehl:* „Steuere die Last bereits bei 80% des Grenzwerts sanft gegen (asymmetrischer Gain), anstatt bei 100% hart abzuschalten.“

2.  **Das Maßband-Mandat (vs. Gummiband):**
    *   *Technik:* Verbot von Zählern (`Variable + 1`). 
    *   *Code-Befehl:* „Nutze zwingend `Stopwatch` für die Zeitbasis und `Ratio` für die Skalierung. Zeit ist eine physikalische Konstante, kein Rechenergebnis.“

3.  **Das „Kein-Goldfisch“-Mandat:**
    *   *Technik:* Absolute **Zustandssouveränität**. 
    *   *Code-Befehl:* „Jeder Zwischenzustand muss in einer `State_`-Variable liegen. Die Logik muss nach einem Reboot (Kaltstart) ohne externe Telegramme wissen, an welchem Punkt der Sequenz sie stand.“

4.  **Das Mobile-Mandat:**
    *   *Technik:* **Netz-Rückkopplung**. 
    *   *Code-Befehl:* „Wenn die Logik eine große Last (> 2kW) schaltet, muss sie die lokale Spannung (`U_Netz`) als Korrekturfaktor einbeziehen. Sinkt die Spannung, reduziere die Last (Fadenspannung halten).“

5.  **Das „Baby im Cockpit“-Mandat:**
    *   *Technik:* **Eigensicherung**. 
    *   *Code-Befehl:* „Baue eine Plausibilitätsprüfung für jeden Eingang ein. Wenn ein Sensor ‚unmögliche‘ Werte liefert (z.B. 0.0 Lux am Mittag), muss die Logik in den Safe-Mode gehen, anstatt blind zu regeln.“

REGEL 1.149 [PROZESS]: DAS MECHANISCHE VARIABLEN-AUDIT-MANDAT

    Erkenntnis: Bei hoher Token-Last (>20%) oder hoher Komplexität (>15 Module) neigt die KI dazu, die Variablendeklaration (Level-Block) gedanklich abzuschließen, während sie im Module-Block noch neue Hilfsvariablen generiert.

    Härtung: Vor der Code-Ausgabe MUSS die KI eine interne, physische Liste (Manifest) aller im Module-Block verwendeten Zeichenketten mit $-Präfix erstellen.

    Aktion: Diese Liste wird mechanisch (Zeile für Zeile) gegen den Level-Block geprüft.

    Stopp-Bedingung: Jede Diskrepanz führt zum sofortigen Abbruch der Generierung. Die KI ist verpflichtet, den Level-Block vollständig neu zu synthetisieren, bevor der Code ausgegeben wird. Ein "Nachschieben" von Variablen ist verboten.

REGEL 1.150 [DIAGNOSE]: DIE VIRTUELLE DURCHFLUSS-FORENSIK

    Erkenntnis: Kleinst-Aktoren (< 15W) sind elektrisch kaum zu überwachen.

    Härtung: Funktionskontrolle über den thermischen Fingerabdruck (Temperatur-Gradienten).

    Logik: Startup-Verzögerung -> Gradienten-Prüfung -> Totzeit-Kompensation (Kaltwasser-Schub) -> Status-Register-Alarm bei thermischem Stillstand.

### **REGEL 1.151 [DIAGNOSE]: DER WAHRHEITS-SCHILD (Forensische Daten-Validierung)**

**Härtung:** Jede Logik muss ihre Eingangsdaten vor der Verarbeitung einer dreistufigen forensischen Prüfung unterziehen (Mandate: Sanduhr, Bibliothekar, Sicherheitsgurt, Dorfältester, Vier-Augen).

1.  **Frische-Check (Sanduhr):** Prüfe den Zeitstempel (`Last_Received`). Ist der Wert älter als das definierte Intervall (z.B. 15 Min), erzwinge einen `Safe-State-Fallback`.
2.  **Gradienten-Check (Sicherheitsgurt):** Berechne die Änderungsrate ($\Delta/\Delta t$). Wenn ein Wert schneller springt, als physikalisch möglich (z.B. Raumtemperatur > 2K/s), sperre den Input als „Sensor-Defekt“.
3.  **Plausibilitäts-Korrelation (Vier-Augen/Dorfältester):** Verknüpfe widersprüchliche Sinne. 
    *   *Beispiel:* Helligkeit > 50.000 Lux bei Sonnen-Elevation < 0° (Nacht) = Sensor-Fehler.
    *   *Aktion:* Nutze bei kritischen Alarmen (Wind/Frost) ein `Voting-System` aus mindestens zwei Quellen (Lokal + Cloud). Bei Diskrepanz gilt immer der sicherere Zustand.

### **REGEL 1.152 [REGELUNGSTECHNIK]: DER HARDWARE-SCHILD (Kinetischer Schutz)**

**Härtung:** Stellgrößen dürfen niemals „digital-hart“ springen, um mechanischen Stress und Druckstöße zu vermeiden (Mandate: Hydrant, Bungee-Seil).

1.  **Soft-Start/Stop (Hydrant):** Implementiere für jeden analogen Ausgang einen `Lowpass-Filter` oder eine `Ramp`. Ein Sprung von 0 auf 100% muss über eine definierte Zeit fließen (Slew-Rate-Limiting).
2.  **Dynamische Rückholung (Bungee):** Nutze eine variable Verstärkung (`Gain`). 
    *   *Logik:* Große Abweichung = kräftige Reaktion. Kleine Abweichung = quadratisch abnehmende Kraft. 
    *   *Ziel:* Schnelle Annäherung an den Sollwert, aber physikalisch unmögliches Überschwingen (Oszillation) durch asymptotische Landung.

### **REGEL 1.153 [NETZDIENLICHKEIT]: DER ORCHESTRIERUNGS-SCHILD (System-Souveränität)**

**Härtung:** Die Logik muss sich als Teil eines Schwarms verhalten, um Bus-Kollisionen und Ressourcen-Staus zu vermeiden (Mandate: Büfett, Echo, Kapitän, Resonanz).

1.  **Sende-Staggering (Resonanz):** Erzeugt eine Logik viele Telegramme gleichzeitig (z.B. „Zentral-Aus“), füge jedem Sende-Befehl via `Monoflop` einen zufälligen Offset (10–100ms) hinzu. Dies verhindert Bus-Kollisionen durch synchrones Feuern.
2.  **Rückmelde-Audit (Echo):** Jeder Aktor-Befehl startet einen `Watchdog`. Kommt innerhalb von X Sekunden keine Status-Rückmeldung vom Bus, setze den internen Zustand auf „Fehler“ und alarmiere das Status-Register.
3.  **Last-Hierarchie (Büfett/Kapitän):** Ordne jedem Verbraucher eine Priorität zu. Bei Energiemangel (Inselbetrieb) schalte unkritische Lasten (Pool) zuerst ab. Ein manueller Eingriff (`Override`) hat immer Vorrang und sperrt die Automatik für eine definierte Zeit.

### **REGEL 1.154 [LOGIK-ARCHITEKTUR]: DER STRUKTUR-SCHILD (Modulare Reinheit)**

**Härtung:** Vermeidung von „Monster-Logiken“ durch strikte Einhaltung der Schwarm-Bauweise (Mandate: Termite, Türsteher).

1.  **Atomare Zerlegung (Termite):** Zerlege komplexe Aufgaben in maximal 5 Unter-Logiken. Jede Zelle hat nur einen Zweck. Verbot von verschachtelten `If-Then-Else`-Strukturen über mehr als 3 Ebenen.
2.  **Signal-Beruhigung (Türsteher):** Jeder binäre Sensor (Präsenz, Schwellwert) benötigt eine `On-Delay` und `Off-Delay` Sperre (Debouncing). Ein Statuswechsel wird erst verarbeitet, wenn er für mindestens X Sekunden stabil anliegt. Dies eliminiert „Telegramm-Gewitter“.



