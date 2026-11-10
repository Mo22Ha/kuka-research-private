# Healthy-Modell: Pilot, P0–P5-Audit und chronologische Studie V2

Aktueller Stand 21.10.2026: Studie V2 abgeschlossen, 6 Trainingstage, 3 Validierungstage,
10 QC-geeignete Testtage. Mittlerer Test-RMSE 0,013822 m/s², bedingtes
95%-Blockintervall [0,012982; 0,014626]. Kleiner Vorteil gegenüber Nullvorhersage;
gegenüber Trainingstemplate nicht eindeutig. Keine physikalisch eindeutige
Identifikation oder validierte Verschleißerkennung. Vollständige V2-Ergebnisse
und Einschränkungen stehen am Ende dieses Berichts.

## Historischer Pilot vor dem Audit

Die folgenden Pilotzahlen stammen aus `healthy-pilot-refined/`, der vollständige
Orientierungsscan aus `healthy-pilot/`. Sie bleiben als historische Ergebnisse
erhalten; die späteren V2-Zahlen verwenden andere Daten und eine korrigierte Verarbeitung.

| Größe | Ergebnis | Unsicherheit | Status | Datenbasis |
|---|---|---|---|---|
| Healthy-Modell | Ebenes Grey-Box-Modell, J3/50%, 0,7–5 Hz | Montage und Messkette ungeklärt | IMPLEMENTIERT, BERECHNET; NICHT VALIDIERT | Training 25.10., Prüfung 01.11.2022 |
| κ₀ | −0,2704709882 rad/s² | OFFEN; kein belastbares Intervall | Bedingter Fit | 1 Trainingsfingerprint |
| κ₁ | 0,1256953242 rad/(s² A) | Kein belastbares Konfidenzintervall | Bedingter Fit | wie oben |
| γ | 5,7811·10⁻²⁸ s⁻¹, numerisch an Untergrenze 0 | Randlösung; kein Reibungsnachweis | Bedingter Fit | wie oben |
| λc | 0,04474454549 rad/s² | Vorzeichen hängt unter anderem von Koordinatenwahl ab | Bedingter Fit | wie oben |
| λs | −0,1601335438 rad/s² | wie λc | Bedingter Fit | wie oben |
| ρx | 0,02977421365 m | Kein vermessener Sensorhebelarm | Bedingter Fit | wie oben |
| ρy | 0,2149505748 m | Kein vermessener Sensorhebelarm | Bedingter Fit | wie oben |
| Q | `[[0,0,1],[0,-1,0],[1,0,0]]` | Nachgewiesen nicht eindeutig | BERECHNET, nicht als Montage validiert | 24 Q × 2 Startwerte |
| Healthy-Validierungs-RMSE | 0,01498209481 m/s² | Kein Generalisierungsintervall aus einem Testfingerprint | BERECHNET; Nullbaseline besser | 272 Zeitpunkte × 3 Kanäle im Auswertefenster |

## Daten und Eingrenzung

- Training: `October 2022.zip`, `F_25_Oct_2022_13_37_17.mat`.
- Unabhängige spätere Prüfung: `November 2022.zip`, `F_01_Nov_2022_14_31_59.mat`.
- SHA256, Manifesthash, Kanäle und Einstellungen:
  `healthy-pilot-refined/config.json`; genaue Acc-/Stromfrequenzen zusätzlich in
  `healthy-pilot/signal-quality.json`.
- Label ausschließlich `healthy_provisional_by_filename_chronology`.
- `motion_joint=3`, `sensor_joint=3`, Geschwindigkeit 50%; Acc-Spalten 4–6,
  Stromspalte 2 (jeweils einsbasiert). Keine Verwechslung mit J2-Bewegung.
- J3-Temperaturen 65,13776 °C und 65,04343 °C. Der erste Fingerprint vom
  13.10. wurde wegen seiner kühleren ~38,7 °C nicht zum Fit verwendet.
- Beide Frequenzregime enthalten: Acc ~10.240 Hz im Training, ~25.600 Hz im Test.
  Das begrenzt die Interpretation; ein weiterer Test bei gleicher Rate fehlt.
- Signalprüfung bei 10/50/100% für beide Dateien: keine NaN/Inf in den verwendeten
  J3-Beschleunigungs-/Stromkanälen. Detailwerte in `healthy-pilot/signal-quality.json`.
- Es wurden keine rohen Messdaten geändert. `data/cache/mat/` enthält die Originalbytes
  mit Herkunft und Prüfsumme; Rohdaten sind von Git ausgeschlossen.

## Modell, Verarbeitung und Optimierung

`src/kuka_pm/healthy.py` implementiert die dokumentierte Gleichung

`θ̈ = κ₀ + κ₁ I(t) − γ θ̇ − λc cosθ + λs sinθ`.

Die Starrkörperbeschleunigung und spezifische Kraft werden mit dem körperfesten
Sensorhebelarm berechnet. Q bildet Sensorkoordinaten auf Körperkoordinaten ab;
für Zeilenvektoren wird deshalb `f_body @ Q` verwendet. x/y bilden die Bewegungsebene,
z deren Normale. Einheiten der Rohsignale folgen dem Autoren-README.

**Bedingte Annahmen:** Sensor sitzt auf dem mitbewegten Körper; Segmentstarts sind
synchron; der tiefpassgefilterte gemessene Phasenstrom ist ein linearer empirischer
Anregungsproxy. Keine dieser Annahmen wurde durch Montage-/Steuerungsdaten kalibriert.
Das Modell ist deshalb ein überprüfbarer Pilotkandidat und kein identifiziertes
Herstellermodell. Der Strom wurde nicht als gemessenes Drehmoment ausgegeben.

Messsignale werden vor dem Resampling auf 40 Hz bei 5 Hz tiefpassgefiltert.
Ein gemeinsames Offline-Analyseband 0,7–5 Hz wird auf Messung und Modell angewendet.
Dies ist **keine identifizierte IEPE-/DAQ-Übertragungsfunktion**. Das zusätzliche
Antialiasfilter der Messung, unbekannte analoge Filter und Randtransienten bleiben
Modellgrenzen. Pro Segment wird die erste und letzte Sekunde vom Score ausgeschlossen.
Die Filterung ist nicht kausal; Online-Leistungsfähigkeit wird nicht behauptet.

Fit: SciPy `least_squares`, Trust Region Reflective, lineare quadratische Zielfunktion,
Trainingsresiduen skaliert mit deren gemessenem Trainings-RMS. Sieben globale Parameter
plus θ₀/ω₀ werden geschätzt. Grenzen, Skalen, beide Startwerte und Toleranzen stehen
vollständig in `config.json`; sie sind numerische Suchgrenzen, keine Herstellerwerte.
Alle 48 Läufe sind in `healthy-pilot/fit-attempts.jsonl` gespeichert. 30 meldeten
erfolgreiche Terminierung, 18 erreichten ihr Auswertungsbudget. Ein globales Minimum
ist damit nicht bewiesen.

Der auf dem Training beste Kandidat wurde anschließend **nur auf Trainingsdaten**
mit zentralen Differenzen und strengeren Toleranzen nachoptimiert. Das verbesserte
die Zielfunktion geringfügig. Die Validierungsdaten beeinflussten weder Q-Auswahl
noch diese Nachoptimierung. Die ursprünglichen Ergebnisse bleiben unverändert erhalten.

Nachoptimierung: 7 Funktionsauswertungen, `ftol`-Terminierung, Optimalitätsmaß 0,0209883.
Das ist keine behauptete Erfüllung einer strengen Gradienten-Nullbedingung.
γ liegt an seiner unteren Grenze. Die Trainings-Anfangsbedingungen sind
θ₀=1,2969494422 rad und ω₀=0,5948934666 rad/s. Sie werden im Test unverändert verwendet;
kein Anpassungsschritt verwendet das vollständige Testsignal.

## Unsicherheit und nachgewiesene Mehrdeutigkeit

Es wurde **kein Standardabweichungsvektor und keine Kovarianzmatrix geschätzt**.
Eine solche Größe würde beispielsweise bei einer gültigen Kovarianzmatrix C als
`σᵢ = sqrt(Cᵢᵢ)` entstehen; sie wäre noch kein Konfidenzintervall. Die hier stark
korrelierten Residuen, eine Randlösung, die unbekannte Messkette und nur ein
Trainingsfingerprint rechtfertigen keine einfache iid-Kovarianz oder ein Bootstrap
auf unabhängiger Fingerprintebene. Die CSV enthält deshalb `NOT_ESTIMATED`.

Die skalierte lokale Jacobi-Matrix ist beim gewählten Q numerisch nicht rangdefizient;
`σ_min/σ_max ≈ 0,002074`. Daraus wird **keine lokale vollständige
Nichtidentifizierbarkeit aller sieben Parameter** abgeleitet. Insbesondere ist
dies kein Nachweis physikalisch richtiger Parameter oder globaler Eindeutigkeit.

Eine globale diskrete Mehrdeutigkeit ist dagegen exakt nachgewiesen. Mit
`D=diag(-1,-1,1)` liefern

```text
θ₀' = θ₀ + π
(λc', λs', ρx', ρy') = −(λc, λs, ρx, ρy)
Q' = D Q = [[0,0,-1],[0,1,0],[1,0,0]]
```

dieselben Sensorsignale. Auf den echten Trainingsinputs beträgt die maximale
numerische Abweichung ungefähr `8,26e-15 m/s²`. Beide Matrizen sind rechtshändig.
Ein unabhängiger Winkelnullpunkt beziehungsweise eine Montageorientierung wird
benötigt, um diese Darstellungsmehrdeutigkeit zu entfernen. Die übrigen
optimierten Q-Kandidaten und ihre Trainingsscores sind ebenfalls vollständig gespeichert.

## Unabhängige Vorhersage und Residuen

RMSE in m/s², ausschließlich im beschriebenen Analyseband/Auswertefenster:

| Verfahren | x | y | z | Zusammengefasster RMSE |
|---|---:|---:|---:|---:|
| Modell, Training | 0,009524 | 0,019094 | 0,009706 | 0,013534 |
| Modell, spätere Prüfung | 0,010960 | 0,021108 | 0,010379 | **0,014982** |
| Nullvorhersage, spätere Prüfung | 0,010960 | 0,020863 | 0,009670 | **0,014707** |
| Trainings-Template, spätere Prüfung | 0,014807 | 0,033746 | 0,016377 | 0,023283 |

MAE zusammengefasst: Modell 0,00982482, Nullbaseline 0,00932975 m/s².
Der Modell-RMSE liegt etwa 1,87% über der Nullbaseline. Das Template wird ohne
an Testdaten angepasste Zeitverschiebung auf gemeinsame Segmentzeit interpoliert.
Es ist eine einfache Baseline, kein optimal ausgerichtetes Templateverfahren.

Die Modellresiduen sind deutlich zeitlich korreliert (Lag-1 etwa 0,91–0,95).
Die ausgewählte Ebene sagt für den Sensor-x-Kanal identisch null voraus; dort
ist der Fehler deshalb exakt der Nullbaseline gleich. Das ist eine Einschränkung
des ebenen Modells, kein Beleg eines funktionslosen Messkanals.

**Keine Aussage über vollständige Hochfrequenzsignale:** Die Roh-RMS der drei
Test-Beschleunigungskanäle betragen etwa 3,943 / 1,699 / 1,353 m/s², während die
ausgewerteten Band-RMS nur etwa 0,0110 / 0,0209 / 0,00967 m/s² betragen. Die kleinen
Fitfehler dürfen nicht als gelungene Rekonstruktion der Rohvibration dargestellt werden.

Halbierung des RK4-Zeitschritts bei unverändertem Auswertefilter verändert die
Trainingsvorhersage nur um rund `1,70e-11 m/s²` RMSE. Der beobachtete Missfit wird
damit nicht durch diesen Integrationsschritt erklärt. Dies ist eine lokale
numerische Kontrolle, keine Validierung sämtlicher Modellannahmen.

![Unabhängige Healthy-Prüfung](healthy-pilot-refined/healthy-validation.png)

Die Abbildung wurde visuell geprüft. Grau markierte Randbereiche gehen nicht
in die Metriken ein. Numerische Kurven und Masken liegen in `predictions.npz`.

## Prüfmemo P0–P5: Signalverarbeitung und Residuen, 21.10.2026

[GETESTET] Das vollständig gelesene Prüfmemo vom 21.10.2026 wurde mit Code,
Original-MATs, V4-CSV und gespeicherten Vorhersagen abgeglichen. Der neue Lauf
`healthy-pipeline-audit-v1/` ist ein **Audit des eingefrorenen Piloten, kein Fit**.
Die alten Ergebnisdateien und Originalmessungen bleiben unverändert.
Reproduktion: `.\\.decoder-venv\\Scripts\\python.exe -m kuka_pm.healthy_audit --output reports/healthy-pipeline-audit-repeat`.
Code-/Konfigurations-/Vorhersagehashes stehen in `audit.json`, MAT-Hashes in
`signal-quality.csv`. Der erste abgebrochene Auditordner enthält keine fertigen
Ergebnisse: Ein UTF-8-BOM im V4-CSV wurde beim Einlesen korrigiert.

### P0/P1: Filter, Frequenzen und Synchronisation

[FAKT: lokale Messdaten] V4 ist für die vier relevanten Kanäle je Fingerprint
mit Hann-Welch, 50% Überlappung, linearem Detrending und an Bandgrenzen
interpolierter Trapezintegration reproduziert. Mit exakten Props-Raten statt
gerundeter Paper-Raten beträgt die größte Abweichung im 0,7–5-Hz-Anteil
**0,0007524 Prozentpunkte**. 24 Vergleichszeilen stehen in `v4-comparison.csv`.
Die 48 PSD-Diagnosezeilen in `raw-psd.csv` vergleichen zusätzlich konstantes
Detrending. Sie enthalten getrennte Integrale 0–500 Hz und bis zur nativen
Nyquistfrequenz; alle Prozentwerte haben weiterhin ausschließlich 0–500 Hz als Nenner.

[FAKT] Mit identischem 2-s-Verfahren macht selbst das gesamte Band 0–500 Hz
im November nur 0,281 / 0,567 / 1,063% der bis zur nativen Nyquistfrequenz
integrierten Acc3/4/5-PSD aus. Im Oktobertraining sind es 1,392 / 1,296 /
2,787%. Der Nennerwechsel ist daher wesentlich. Wegen unterschiedlicher
Nyquistgrenzen und ungeklärter Messkette ist dies kein isolierter Nachweis
einer mechanischen Zustandsänderung.

[FAKT] Im Novemberstrom ergibt lineares Detrending 38,64% im Analyseband bei
2-s-Fenstern, aber 8,61% bei 8-s-Fenstern. Konstantes Detrending ergibt 32,50%
bzw. 9,49%. Unter 0,7 Hz liegen beim 8-s-Verfahren rund 89–90% der
0–500-Hz-Leistung. Die beiden sichtbaren langsamen Stromverläufe und langsame
Acc-Anteile werden durch den Hochpass stark verändert. Diese Beobachtung
beweist weder einen kalibrierten Gravitationsanteil noch einen Sensorfehler.
Ein einziges 8-s-Fenster und sieben überlappende 2-s-Fenster sind keine
unabhängigen Wiederholungen. Fenster-/Trendempfindlichkeit bleibt erheblich.

[FAKT: Code/Frequenzgang] Messung: nativer Butterworth-Tiefpass 4. Ordnung,
5 Hz, vorwärts/rückwärts; lineare Interpolation auf 40 Hz; Butterworth-Bandpass
3. Ordnung 0,7–5 Hz vorwärts/rückwärts. Modell: nur dieser Bandpass auf der
berechneten spezifischen Kraft. Der Strominput wird nur tiefpassgefiltert;
die bandpassgefilterte Stromkurve ist eine zusätzliche Darstellung, kein neuer Input.
Damit unterscheiden sich die Messoperatoren systematisch:

| Frequenz | zusätzliche Amplitude LP im Messpfad | Differenz gegenüber Modellpfad |
|---|---:|---:|
| 2 Hz | 0,99935 | −0,00569 dB |
| 3 Hz | 0,98348 | −0,14468 dB |
| 4 Hz | 0,85633 | −1,34716 dB |
| 5 Hz | 0,50000 | −6,02060 dB |

Dies ist die stationäre digitale Filterantwort, ohne Randbehandlung und
Interpolationsfehler. Bei 20 Hz beträgt die zusätzliche Dämpfung rund −96,33 dB.
Das spricht für starke digitale Anti-Alias-Unterdrückung vor dem 40-Hz-Raster,
beweist aber weder ideale Aliasfreiheit noch die analoge Anti-Alias-Funktion
vor der ursprünglichen Abtastung. Beide Original-Abtastraten wurden berücksichtigt.
Vorwärts/rückwärts hebt die stationäre Phase auf, erzeugt aber Randtransienten
und verwendet zukünftige Samples. Keine kausale Echtzeitvalidierung.

[ENTSCHEIDUNG] 0,7 Hz ist weiterhin **nicht physikalisch begründet oder optimal**.
Eine Sensordatenblatt-Grenze allein kalibriert nicht die ganze IEPE-/DAQ-Kette.
Vor neuer Optimierung sind ein symmetrischer Messoperator für Modell/Messung,
die tatsächliche analoge Kopplung, längere Vor-/Nachlaufdaten und eine
begründete untere Auswertegrenze nötig. Den Modellpfad jetzt ungeprüft zu
ändern würde die Interpretation des alten Fits ändern; der Audit erhält ihn.

[FAKT] Acc-/Curr-Endzeiten unterscheiden sich bei Props-Raten um nur etwa
0,08–0,10 ms. Das bestätigt konsistente Längen, **nicht** einen gemeinsamen
Hardware-Zeitnullpunkt. Der Decoder liefert Aufnahmezeiten, keine verifizierten
Sample-Zeitvektoren. Auf gemeinsamen angenommenen Segmentachsen zeigen die
Plots die Rohsignale, LP5 und LP5+Bandpass samt Einheiten und Vorzeichen.
Die absolute Sensorpolarität ist ohne Montagebezug weiterhin offen.
Eine explorative Kreuzkorrelation von Band-Acc mit LP-Strom im Bereich ±0,5 s
hat kleine, kanalabhängige Maxima (|r| ≤ 0,167) und wechselnde Lags
(−0,375 bis +0,175 s). Unterschiedliche Bandbreiten und mechanische Dynamik
verhindern eine Interpretation als Clock-Offset. Es wurde nichts verschoben.

![Signalverarbeitung November](healthy-pipeline-audit-v1/F_01_Nov_2022_14_31_59-signals.png)
![Digitale Filterantwort](healthy-pipeline-audit-v1/filter-response.png)

### P2: Residuen primär gegen null

[FAKT: eingefrorene Vorhersagen] `r = Messung − Modell`, 272 Punkte je Kanal,
unverändertes 1-s-Randfenster. Bias, Standardabweichung und RMSE in m/s²:

| Novemberkanal | Bias | std (ddof=0) | RMSE | RMS Modell / RMS Messung | ACF bei 0,025 s |
|---|---:|---:|---:|---:|---:|
| x / Acc3 | −0,00013686 | 0,01095905 | 0,01095991 | 0 | 0,95394 |
| y / Acc4 | +0,00015824 | 0,02110728 | 0,02110787 | 0,42346 | 0,89159 |
| z / Acc5 | +0,00015857 | 0,01037798 | 0,01037920 | 0,57223 | 0,90374 |

Die Identität RMSE² = Bias² + std² gilt mit ddof=0. Der kleine Bias bedeutet
keine kleinen momentanen Fehler; der Fehler besteht überwiegend aus Streuung.
Der Bandpass begünstigt kleine Mittelwerte. Zentrierte ACF:
`sum(r_centered[t]*r_centered[t+k])/sum(r_centered**2)` über gültige Paare;
keine Lag-spezifische Renormierung. Die alten Lag-1-Werte nutzten stattdessen
Pearson-Korrelation der gekürzten Paare. Daher unterscheiden sich insbesondere
die y-Werte; dies ist eine explizite Definitionsdifferenz, keine Datenänderung.
Keine iid-Signifikanzbänder oder Gleichsetzung von 272 Samples mit 272 Versuchen.

`residual-metrics.csv` enthält auch Training, Median, 95%-Quantil von |r|,
Modell-/Mess-RMS und Residuen-Bandleistung. Vollständige ACF und PSD liegen
je Kanal als CSV vor. Die Residuen-PSD verwendet Hann, 2 s, 50% Überlappung,
konstantes Detrending und nur das ausgeschlossene-Ränder-Auswertefenster.
Sie beschreibt die **bereits gefilterten** Residuen bis 20 Hz. Ihr Welch-Integral
ist wegen Fensterung/Detrending nicht identisch mit dem Zeitbereichs-RMSE².

[FAKT] Randausschluss 0,5/1/1,5/2 s ergibt für November-y RMSE
0,026676 / 0,021108 / 0,019842 / 0,020845. Es wurde kein bestes Fenster ausgewählt.
Der Verlauf ist sensitiv und nicht monoton; er trennt Filterartefakte nicht
eindeutig von realen Bewegungstransienten. 1 s ist kein nachgewiesenes
Einschwingkriterium. Roh-/Filterplots und Residuenabbildung wurden visuell geprüft.

![Residuen, ACF und PSD](healthy-pipeline-audit-v1/validation-residuals.png)

[ENTSCHEIDUNG] Nullvorhersage und Trainings-Template stehen separat in
`baselines-separate.csv`. Der frühere kombinierte Vergleich bleibt unverändert:
Modell 0,0149821, Null 0,0147072, Template 0,0232827 m/s². Für x ist die
Modellvorhersage konstruktionsbedingt null; dieser Vergleich liefert dort
keinen zusätzlichen Nachweis. Die Modellresiduen werden unabhängig davon
gegen den physikalischen Idealwert null beurteilt.

### P3: unabhängige algebraische Prüfung der Orientierung

[MATHEMATISCH NACHGEWIESEN] Direktes Einsetzen von
`sin(θ+π)=−sinθ`, `cos(θ+π)=−cosθ` und negierten λc/λs erhält θ̈ und ω.
Mit negiertem ρ kehren beide ebenen Komponenten der spezifischen Kraft ihr
Vorzeichen um: `f' = f D`. Weil `Q2 = D Q1` und `D²=I`, folgt
`f' Q2 = f D D Q1 = f Q1`. Q1/Q2 sind orthogonal mit Determinante +1;
`Q2 Q1.T = D = diag(−1,−1,1)` ist eine 180°-Drehung.
Lineare identische Filter erhalten diese Gleichheit.

[NUMERISCH BESTÄTIGT] Maximale Differenz auf den realen Inputs:
Training 8,26e−15 und November 8,33e−15 m/s². Invariant unter **dieser**
diskreten Transformation sind κ₀, κ₁, γ, ω₀ und insbesondere
`||ρ||=0,217002888 m`, `||λ||=0,166267334 rad/s²`,
`λ·ρ=−0,033088564` und `λc ρy−λs ρx=0,014385716`.
Invarianz ist kein Nachweis einer physikalisch korrekt geschätzten Größe.
Weitere Daten desselben Beobachtungstyps brechen diese Symmetrie nicht.

[OFFEN] Die gezielte Prüfung des vorhandenen mechanischen Entwurfs
(`docs/sources/project_history/mechanical_model.tex`, Sensor Kinematics und
Orientation Validation) liefert die Montageplattenbeschreibung, aber keine
vermessene Sensorkörper-Transformation. Erforderlich sind Montagefoto/-zeichnung
mit Achsen und bewegter/statischer Seite sowie Controller-Winkel mit bekannter
Nullpunkt-/Vorzeichenkonvention oder eine äquivalente dynamische Referenzmessung.
Keine statische 1-g-Kalibrierung für die ungeklärte AC-gekoppelte Kette.

### P4/P5: Datenbedarf und Freigabegates

[ENTSCHEIDUNG] `configs/healthy_followup_split.json` legt vor Signalinspektion
03.11. und 07.11. als neue Entwicklungs-/QC-Aufnahmen fest; 14.11. als
reservierte Validierung; 21.11. und 28.11. als ungesehene spätere Holdouts.
Alle exakten Dateinamen wurden im bestehenden Manifest gefunden. Die Auswahl
erfolgt nach Datum, nicht nach günstigem Modellfehler. QC: J3/50%, endliche
Kanäle, Acc 25.600 Hz und J3-Temperatur 65,1±2 °C. Bei QC-Ausfall Ausschluss
dokumentieren, keine nach Score gesteuerte Ersatzwahl.
November 1 bleibt gesehen und ist für neue Filterentscheidungen kein Blindtest.
Die neuen Novemberdateien passen zum November-Samplingregime, nicht zum alten
Oktober-Training; ein späteres Referenzexperiment muss Training und Prüfung
innerhalb eines Regimes halten oder den Regimeeffekt ausdrücklich untersuchen.

[EXPERIMENTELL BEOBACHTET] Die neue Datei `F_03_Nov_2022_12_09_03` wurde per
Einzeldatei-Range vollständig geladen (84.466.556 Bytes, SHA256 in
`healthy-pipeline-audit-v1/nov03/additional-quality.json`), mit 64,93943 °C und Acc 25.600 Hz.
Alle J3/50%-Acc-, Curr- und RTD-Werte sind endlich; die vorher festgelegte QC
ist erfüllt. Bei unveränderten Parametern, Trainingsanfangszuständen und
Auswertefenster betragen die drei Residuen-RMSE 0,0100059 / 0,0211759 /
0,0096623 m/s²; ACF1 0,9404 / 0,9097 / 0,8805. Weitere robuste Merkmale
stehen in `healthy-pipeline-audit-v1/nov03/additional-residual-metrics.csv`.
Separater Kontrollvergleich: zusammengefasster Modell-RMSE 0,0146276,
Nullvorhersage 0,0152538 m/s². Der Vorteil auf diesem einzelnen neuen
Entwicklungstag widerlegt keinen früheren Missfit und begründet noch keinen
allgemeinen Nutzen. Keine erneute Parameterschätzung oder Bandwahl.

[EXPERIMENTELL BEOBACHTET] Auch `F_07_Nov_2022_12_19_10` ist vollständig
geladen (84.442.802 Bytes) und QC-geeignet: 64,32769 °C, Acc 25.600 Hz,
endliche J3/50%-Kanäle. Provenance und Merkmale liegen analog unter `nov07/`.
Es liegen jetzt fünf Original-MATs im Cache; vier warme Aufnahmen einschließlich
Oktobertraining und eine kalte Diagnoseaufnahme. Der Abruf beider neuen Dateien
endete erfolgreich, ohne HTTP-403 in diesem Lauf. Daraus folgt keine Klärung
des früheren Januar-403. Keine reservierte Validierungs-/Holdoutdatei wurde geladen.

Deskriptiver Vergleich bei unveränderter Verarbeitung und Parametrisierung:

| Datum | Modell-RMSE kombiniert (m/s²) | separat: Nullbaseline (m/s²) |
|---|---:|---:|
| 01.11. | 0,0149821 | 0,0147072 |
| 03.11. | 0,0146276 | 0,0152538 |
| 07.11. | 0,0140684 | 0,0147215 |

Die kanalweisen RMSE-Spannen über diese drei gesehenen Novembertage betragen
x: 0,0100059–0,0109599; y: 0,0196690–0,0211759; z: 0,0096623–0,0103792 m/s².
`november-descriptive-summary.json` enthält zusätzlich Bias-, |r|-q95- und
ACF-Spannen. Diese Spannen sind **keine** Konfidenz-/Vorhersageintervalle oder
Alarmschwellen. Zwei neue Entwicklungstage verbessern die empirische Basis,
reichen aber nicht zur statistischen Freigabe. Alle Parameter und
Anfangszustände blieben unverändert; eine sinkende RMSE wird nicht als
Verbesserung des mechanischen Zustands interpretiert.

[OFFEN] Getrennte Tage sind sinnvolle Aufnahmeblöcke, beweisen aber keine
statistische Unabhängigkeit. Betriebs-/Abkühlungsereignisse, Source-Labels und
Wartungsprotokolle sind vor einer Healthy-Freigabe zu prüfen. Benötigt werden
mehrere solcher Blöcke für Entwicklung und getrennte Schwellenkalibrierung,
mit Temperaturabdeckung innerhalb jedes Splits. Eine universell ausreichende
Anzahl für die sieben Parameter folgt aus den verfügbaren Daten nicht.

[IMPLEMENTIERT] Robuste Merkmale pro Fingerprint sind berechenbar; bisherige
Zahlen bleiben deskriptive Pilotdiagnostik. [OFFEN] Belastbare Zwischenblock-
Intervalle, Parameterunsicherheit und Fehlalarmrate bleiben gesperrt, bis
Messoperator, Synchronisation, Anfangszustandsregel und Healthy-Labels geklärt,
das Modell auf Entwicklung eingefroren und genügend unabhängige Kalibrierungs-
und Testblöcke vorhanden sind. Kein Sample-Bootstrap als Ersatz.
Als bedingtes Planungsbeispiel: Bei null Fehlalarmen in n unabhängigen,
repräsentativen Healthy-Prüfungen beträgt die einseitige exakte 95%-Obergrenze
`1−0,05**(1/n)`. Für eine Grenze unter 5% benötigt man mindestens 59 solche
Prüfungen; zwei reservierte Holdouts reichen dafür ausdrücklich nicht.
Dies ist keine bereits gemessene Fehlalarmrate und garantiert keine Gültigkeit
bei serieller Abhängigkeit oder ungeprüften Labels. Keine Verschleiß-/RUL-Aussage.

## Ergebnis und konkret nächste Untersuchung

**Ja**, das bedingte physikalische Modell wurde implementiert und an echten
Messdaten angepasst. **Nein**, eine physikalisch eindeutige Parameteridentifikation,
eine eindeutige Sensororientierung und ein nützlicher unabhängiger
Vorhersagevorteil sind damit nicht nachgewiesen. Der negative Baselinevergleich
ist ein reales wissenschaftliches Pilotergebnis.

Vor Ausbau: Sensorposition relativ zum bewegten Gelenk und Messkettenfilter klären;
Segment-/Stromausrichtung mit mehreren warmen Healthy-Fingerprints prüfen;
vorzugsweise gleiche Samplingrate verwenden; ausreichend unabhängige Trainings-
und Prüfaufnahmen für Unsicherheitsanalyse beschaffen. Danach erst komplexere
Modelle oder Verschleißinterpretation. Kein RUL- oder Wartungszeitpunktnachweis.

Historische Fit-Reproduktion (derzeit **nicht ausführen**: Das Prüfmemo sperrt
neue Optimierung bis zur Klärung der Messverarbeitung; neue Ausgabeordner wären erforderlich):

```powershell
.\.decoder-venv\Scripts\python.exe -m kuka_pm.healthy_pilot --train F_25_Oct_2022_13_37_17 --validation F_01_Nov_2022_14_31_59 --output reports/healthy-pilot-repeat --max-nfev 80
.\.decoder-venv\Scripts\python.exe -m kuka_pm.healthy_pilot --train F_25_Oct_2022_13_37_17 --validation F_01_Nov_2022_14_31_59 --output reports/healthy-pilot-refinement-repeat --refine-from reports/healthy-pilot-repeat --max-nfev 300
```

## Erweiterte Studie V2 — genehmigtes Mehrfingerprint-Fitting und Prüfung

[ENTSCHEIDUNG, 21.10.2026] Der neue Nutzerauftrag erlaubt erneutes Fitting nach
Methodenprüfung. Die frühere Optimierungssperre gilt für diesen begründeten,
separaten Studienlauf nicht mehr. Alte Pilot-/Auditdateien bleiben erhalten.
Protokoll: `configs/healthy_study_v2.json`; Ergebnisse: `healthy-study-v2/`.

### Vorab festgelegtes Design und Präzisionsziel

[ENTSCHEIDUNG] Sechs Trainingstage 01./03./05./07./09./11.11., drei spätere
Validierungstage 14./16./18.11. und zwölf Testtermine vom 21.11. bis 25.12.
Exakte Manifest-IDs stehen im Protokoll. Je Tag wird der Eintrag nächst 12 Uhr
gewählt; zuvor reservierte IDs werden beibehalten. Der 07.12. fehlt im Manifest,
daher wurde vor Signalinspektion der 08.12. festgelegt. Keine Auswahl nach Score.
Die bekannten Novembertage sind jetzt ausdrücklich Entwicklung/Training.

Primäres Präzisionsziel: Halbbreite eines bedingten 95%-Intervalls für den
mittleren Fingerprint-RMSE höchstens 0,0025 m/s². Planungsstandardabweichung
0,002 m/s² (mehr als viermal die empirische SD der drei bereits gesehenen
November-Pilotwerte), nicht als bekannte Populations-SD ausgegeben.
Die Planungsformel `t(0,975;n−1)*0,002/sqrt(n)` unterschreitet bei n=5 das Ziel.
Planungsfaktor 2 für zeitliche Abhängigkeit ergibt zehn Aufnahmen; zwölf feste
Termine erlauben einige QC-Ausfälle. Alle zwölf geeigneten Fälle werden ausgewertet,
kein Stoppen bei günstigem p-Wert. Bei zu großer tatsächlicher Intervallbreite
wird das Ziel als verfehlt berichtet, nicht durch optionales Nachladen erzwungen.
Formelgrundlage: [NIST, Confidence Limits for the Mean](https://www.itl.nist.gov/div898/handbook/eda/section3/eda352.htm).
Dies ist eine bedingte Planung, kein Beweis von Unabhängigkeit oder Teststärke.

QC vorab: Acc 25.600 Hz, J3/50%, endliche Acc-/Curr-/RTD-Kanäle, Temperatur
65,1±2 °C, Segmentdauer 8–10 s. Ausschlüsse werden mit Grund erhalten, ohne
scoregesteuerte Ersatzwahl. ORDA-README klassifiziert den Zeitraum bis
19.01.2023 als Healthy; die hier ausgewählten November-/Dezemberdaten liegen
davor und vor den dort aufgelisteten gezielten Abkühlereignissen. Dies ist ein
Quellenlabel, keine unabhängige Zustandsdiagnose jedes aufgenommenen Tages.

### Korrigierte Messgleichung und verbleibende Annahmen

[IMPLEMENTIERT/GETESTET] Native Beschleunigung wird mit einem Butterworth-LP20
8. Ordnung vorwärts/rückwärts anti-alias-gefiltert und auf 100 Hz interpoliert.
Das Modell wird bei 100 Hz integriert und ebenfalls LP20-gefiltert. Danach
folgen auf beiden Pfaden **identische** LP5- und BP0,7–5-Operatoren bei 100 Hz;
die Scorepunkte werden auf 40 Hz interpoliert. Der native vs. 100-Hz-LP20 hat
im Band maximal 1,9921e−10 Unterschied der stationären Amplitudenantwort.
Native LP20-Dämpfung bei 50 Hz: −127,342 dB. Ergebnis in `operator-check.json`.
Ein synthetischer Test mit 1/4-Hz-Nutzsignal und 1-kHz-Störung prüft
Signalübertragung und Aliasunterdrückung; er ist keine echte Robotermessung.

Strom bleibt LP5-gefilterter empirischer Anregungsproxy. Die Bandgrenze 0,7 Hz
wird als bisheriger diagnostischer Schätzgegenstand beibehalten, nicht als
physikalisch kalibrierte oder nach Scores optimierte Grenze. Erste/letzte
1,5 s werden vorab ausgeschlossen; das reduziert, beweist aber nicht die
Abwesenheit von Randartefakten. Hardware-Uhren und analoge Übertragung fehlen.

Sieben globale Parameter plus zwei **gemeinsame Trainings-Anfangszustände**
werden geschätzt. Neue Aufnahmen übernehmen beide Zustände unverändert;
keine Vollsignal- oder Testpräfix-Anpassung und kein Score-gesteuertes Alignment.
Gemeinsamer Segmentstart und wiederholbare Startbewegung bleiben Hypothesen.
Auf Validierung werden ausschließlich vorab festgelegte Verzögerungen
−25/0/+25 ms diagnostiziert; die primäre Verzögerung bleibt 0, unabhängig vom
besten Validierungsscore. Q1 bleibt die bisherige Entwicklungskonvention.
Die mathematische Q1/Q2-Symmetrie und fehlende physische Montageeindeutigkeit
werden durch das neue Fitting nicht aufgehoben.

Gleiche Gewichtung der Fingerprint-MSE, drei fest definierte Startwerte,
TRF-least_squares, numerische Grenzen und Toleranzen im Protokoll/Code.
Auswahl allein nach Trainings-MSE. Leave-one-training-day-out untersucht
Stabilität, nicht nominelle Parameter-Konfidenz. Min/Max der Parameter sowie
unter der bekannten diskreten Symmetrie invariante Kombinationen werden
berichtet; eine iid-Residuenkovarianz wird nicht als physikalische Unsicherheit ausgegeben.

### Statistik und Sperren gegen Test-Leakage

[ENTSCHEIDUNG] Primär Residuen gegen null: Bias, std, RMSE, Quantile, ACF und PSD
je Kanal/Fingerprint. Separat Nullvorhersage und Mittelwert der Trainingstemplates.
RMSE-Differenzen werden gepaart auf denselben Tagen verglichen, mit absoluten,
relativen und standardisierten Effektgrößen. Keine künstliche Vervielfachung
von Stichproben durch die 40-Hz-Zeitpunkte.

Bedingte 95%-Intervalle: zirkulärer Moving-Block-Bootstrap von Tageskennwerten,
Blocklänge 2, vorab festgelegte Sensitivitäten 1 und 3, 10.000 Resamples und
Seed 27368490. Der Ansatz setzt ausreichende Stationarität/kurzreichweitige
Abhängigkeit voraus; unregelmäßige Tagesabstände, Trends und eine einzige
Roboterhistorie begrenzen die nominale Abdeckung. Siehe
[Politis/Romano, Circular Block-Resampling](https://statistics.stanford.edu/technical-reports/circular-block-resampling-procedure-stationary-data)
und [NIST, Autocorrelation](https://itl.nist.gov/div898/handbook/eda/section3/eda35c.htm).
Die Lag-1-Korrelation der ausgewählten Tage ist nur eine Indexdiagnose, keine
kalibrierte kontinuierliche Zeitkorrelation bei unregelmäßigen Abständen.
Unbedingte Signifikanz-/p-Wert-Aussagen sind bei diesen Voraussetzungen nicht
vorgesehen. Drei Validierungstage reichen nicht für belastbare Fehlalarmkalibrierung.

Vor Validierungszugriff werden Trainingsparameter, Protokoll und wissenschaftlicher
Code gehasht; vor Testzugriff zusätzlich die endgültige Freigabe gespeichert.
Der Downloader prüft diese Sperren. Nach Testöffnung kein Fitting und keine
Änderung von Band, Randfenster, Q, Initialisierung oder Statistik.

### V2: tatsächlich berechnete Trainings- und Validierungsergebnisse

[EXPERIMENTELL BEOBACHTET] Alle sechs Trainingstage sind QC-geeignet, J3/50%, Acc 25.600 Hz, Temperatur 64,32769–65,07907 °C. Alle drei Validierungstage sind ebenfalls geeignet (64,68457–65,06578 °C). Einzelne Dateigrößen, SHA256, Temperaturen und Quellzustände stehen in `train-quality.json` und `validation-quality.json`. Die Zeiten dienen als Aufnahmetags-IDs aus Dateinamen, nicht als neue MATLAB-/Hardware-Uhrvalidierung.

| Größe | Vollständiges Training | Min–Max beim Weglassen eines Tages |
|---|---:|---:|
| κ₀ [rad/s²] | -0.37934536 | -0.41668424 bis -0.31848527 |
| κ₁ [rad/(s² A)] | 0.11887224 | 0.10964633 bis 0.15141425 |
| γ [s⁻¹] | 0.10267106 | 0.0055335429 bis 0.15675933 |
| λc [rad/s²] | 0.013216463 | 0.0087802117 bis 0.02950654 |
| λs [rad/s²] | -0.11804328 | -0.16343319 bis -0.10353021 |
| ρx [m] | 0.14057017 | 0.10621452 bis 0.16049663 |
| ρy [m] | 0.29366894 | 0.21622863 bis 0.34073742 |
| θ₀ gemeinsam [rad] | 2.012677 | 1.8948709 bis 2.1530991 |
| ω₀ gemeinsam [rad/s] | 0.59123525 | 0.4451963 bis 0.68578517 |

Die Spannweiten sind **Stabilitätsdiagnosen, keine Konfidenzintervalle**. Zwei Anfangszustände sind zusätzliche gemeinsame Nuisance-Parameter. Insbesondere γ schwankt stark relativ zu seinem Volltrainingswert; eine präzise physikalische Dämpfung ist nicht nachgewiesen. Auch Hebelarme sind keine vermessene Geometrie. Vergleiche zum alten Pilotwert sind durch gleichzeitig geänderte Verarbeitung, Datenbasis und Randfenster beeinflusst.

Zwei Startwerte terminieren mit ftol in sehr ähnlicher Lösung, der dritte erreicht 200 Auswertungen ohne Konvergenz und mit 1.238% höherer MSE. Gewählt wurde ausschließlich die kleinste Trainings-MSE. Der gewählte Fit hat Optimalitätsmaß 0.0011928, keine aktiven Parametergrenzen und σ_min/σ_max=0.00328161. Kein globales Minimum oder physisch eindeutiger Parametersatz ist dadurch bewiesen. Alle sechs Auslassungs-Fits terminieren erfolgreich.

Unter der bereits mathematisch nachgewiesenen diskreten Q-Symmetrie invariante Kombinationen:

| Kombination | Volltrainingswert | Auslassungsspanne |
|---|---:|---:|
| rho_norm | 0.3255786 | 0.25610234 bis 0.36724911 |
| lambda_norm | 0.11878085 | 0.10447348 bis 0.16607542 |
| lambda_dot_rho | -0.032807805 | -0.035492832 bis -0.02839198 |
| lambda_cross_rho | 0.020474629 | 0.015865744 bis 0.028808997 |

Diese Invarianz gilt unter der geprüften Transformation; sie beweist keine Kalibrierung gegenüber beliebigen realen Montage-/Messkettenfehlern. Beim Halbschritt-Test des Integrators bleiben die Messoperatoren unverändert; größte Vorhersageabweichung 4,7567e−13 m/s² RMSE. Ein numerischer Integrationsfehler dieser Größenordnung erklärt die realen Residuen nicht.

| Validierungstag | Modell-RMSE | separat Nullbaseline | separat Trainingstemplate |
|---|---:|---:|---:|
| F_14_Nov_2022_13_13_51 | 0.0149776 | 0.0152951 | 0.0168844 |
| F_16_Nov_2022_05_55_03 | 0.0143738 | 0.0148216 | 0.0170697 |
| F_18_Nov_2022_12_10_03 | 0.0151976 | 0.0157802 | 0.0183253 |

Alle Werte in m/s². Der vorab festgelegte +25-ms-Diagnoseversatz senkt auf diesen drei Fällen den RMSE gegenüber delay=0. Der Primärversatz bleibt dennoch null; weder ein optimaler Delay noch ein Hardware-Clock-Offset wurde identifiziert. Die offene Synchronisationsannahme ist deshalb weiterhin materiell.

[GETESTET] `final-freeze.json` wurde am 21.10.2026 um 09:12:24 UTC vor dem ersten Testabruf gespeichert. Protokoll, Auswertecode, Trainingsdatenrepräsentation und Parameter sind per SHA256 eingefroren. Testdaten dürfen diese Größen nicht mehr verändern.

Die Validierung zeigt eine mittlere Modell-minus-Null-RMSE-Differenz von -0.00044925 m/s²; die diagnostische Änderung durch +25 ms beträgt -0.00040785 m/s². Die beiden Effekte sind ähnlich groß. Das ist eine Sensitivitätsbeobachtung und kein geschätztes Intervall des tatsächlichen Hardwareversatzes. Ein eventueller positiver Testvergleich belegt deshalb zunächst bedingte Vorhersageleistung, keine physikalische Kalibrierung.


### V2: endgültige Testauswertung und Datenherkunft

[FAKT/GETESTET] 21 Originaldateien wurden vorab ausgewählt: 6 Training,
3 Validierung, 12 Test. Davon wurden 18 neu per ZIP-Range-Zugriff geladen
(1.521.448.952 Byte Original-MAT-Dateien); alle 21 zusammen umfassen
1.774.865.564 Byte. Diese Größen sind gespeicherte Originaldateien, keine
Messung des Netzwerkverkehrs inklusive Wiederholungen. Kein Monatsvollarchiv.
Der vollständige [Fingerprint-Nachweis](healthy-study-v2/fingerprint-inventory.csv)
enthält Dateinamen, Split, Archiv-ID, Manifest-Hash, SHA256, Größe, Temperatur,
Abtastrate, Betriebszustand und Ausschlussgründe. Herkunft:
[ORDA 27368490 v1](https://doi.org/10.15131/shef.data.27368490.v1).
CRC und SHA256 sichern die Transportintegrität; sie validieren keine Healthy-Labels.

Alle 6 Trainings- und 3 Validierungstage bestanden QC. Von 12 Testtagen bleiben
10 nach der unveränderten Temperaturgrenze 63,1–67,1 °C. Ausschlüsse:

| Aufnahme | Mittlere Temperatur | Vorab definierter Ausschluss |
|---|---:|---|
| F_16_Dec_2022_12_06_53 | 63,04821 °C | unter 63,1 °C |
| F_25_Dec_2022_11_52_41 | 63,09630 °C | unter 63,1 °C |

Der zweite Wert liegt nur etwa 0,0037 °C unter der Grenze. Unbekannte
RTD-Messunsicherheit verhindert eine physikalische Interpretation dieses
Grenzunterschieds; die vorab festgelegte Regel wurde trotzdem unverändert
angewandt. Die Ausschlüsse sind keine Diagnose eines ungesunden Roboters.
Keine Ersatzaufnahmen und keine Scoreberechnung für die ausgeschlossenen Fälle.

[GETESTET] Die endgültige Testmethode wurde vor Testabruf eingefroren und genau
mit diesen Parametern ausgewertet. Alle Testaufnahmen sind jetzt gesehen und
für eine spätere Methodenentwicklung **keine neuen Blindtests**.

### Primäre Residuenprüfung gegen den Idealwert null

Der primäre Schätzer ist der Mittelwert des pro Fingerprint über drei Kanäle
berechneten RMSE. Die zehn Fingerprints, nicht die einzelnen Zeitpunkte, sind
die Untersuchungseinheiten. Alle folgenden Intervalle sind bedingte
95%-Intervalle des vorab definierten zirkulären Block-Bootstraps.

| Kennwert | Testresultat |
|---|---:|
| Geeignete Testaufnahmen | 10 |
| Mittelwert RMSE | 0,01382205 m/s² |
| Stichproben-SD der Tages-RMSE | 0,00118537 m/s² |
| Primäres Intervall, Blocklänge 2 | [0,01298195; 0,01462639] m/s² |
| Sensitivität Blocklänge 1 | [0,01313128; 0,01451131] m/s² |
| Sensitivität Blocklänge 3 | [0,01296573; 0,01459600] m/s² |
| Primäre Intervallhalbbreite | 0,00082222 m/s² |
| Vorab festgelegtes Präzisionsziel | höchstens 0,0025 m/s² |

[GETESTET/INFERENZ] Die geplante Mindestzahl von zehn Aufnahmen und das
Präzisionsziel sind rechnerisch unter den Intervallannahmen erreicht.
**Dies ist keine bestätigte nominale 95%-Abdeckung:** Lag-1 der zeitgeordneten
Tages-RMSE beträgt 0,48470; die zweite Hälfte hat einen um 0,00135549 m/s²
niedrigeren mittleren RMSE. Die mittlere Temperatur beträgt im Training
64,80699 °C, in der Validierung 64,86871 °C und im geeigneten Test 63,87571 °C.
Der Test ist somit im Mittel 0,93128 °C kühler. Zeit-/Temperaturabhängigkeit,
unregelmäßige Aufnahmeabstände und eine einzelne Roboterhistorie begrenzen
die Stationaritätsannahme. Kein nachträgliches Nachfitten oder Temperaturkorrektur.

Kanalweise Mittelwerte der innerhalb jedes Fingerprints berechneten Kennwerte:

| Kanal | Bias | Standardabweichung (ddof=0) | RMSE | Mittelwert des 95%-Quantils des Absolutresiduums |
|---|---:|---:|---:|---:|
| x | 0,00005343 | 0,00841649 | 0,00841898 | 0,01871366 |
| y | −0,00076766 | 0,01996048 | 0,01997908 | 0,04521937 |
| z | 0,00005122 | 0,01007976 | 0,01008310 | 0,02353820 |

Alle Werte in m/s². Quantilmittelwerte sind keine garantierten Alarmgrenzen.
Das bedingte Biasintervall von y ist [−0,00099634; −0,00050942] m/s²; x und z
schließen null ein. Die zugehörigen Intervalle sämtlicher Kanalgrößen stehen in
[test-inference.json](healthy-study-v2/test-inference.json). Die Lag-1-ACF der
Zeitresiduen beträgt x 0,9393–0,9611, y 0,9003–0,9202 und z 0,8962–0,9232.
Die Residuen sind somit stark gefärbt; Filterung trägt dazu bei. Sichtbare
wiederkehrende Ausschläge belegen verbleibende Struktur, nicht deren Ursache
und insbesondere keinen Verschleiß. Die x-Vorhersage ist im ebenen Q1-Modell
identisch null; dort entsteht kein zusätzlicher Modellnutzen.

### Separate Referenzmodellvergleiche

| Referenz | Mittlerer Referenz-RMSE | Modell minus Referenz | Bedingtes 95%-Intervall der Differenz | Relative Differenz | Standardisierter gepaarter Effekt |
|---|---:|---:|---|---:|---:|
| Nullvorhersage | 0,01426828 | −0,00044623 | [−0,00068449; −0,00023928] | −3,1274% | −1,4024 |
| Trainingstemplate | 0,01488513 | −0,00106308 | [−0,00299722; +0,00152977] | −7,1419% | −0,3087 |

RMSE und Differenzen in m/s²; relative Differenz bezogen auf den jeweiligen
Referenzmittelwert. Standardisierung: mittlere gepaarte Differenz geteilt durch
deren Stichproben-SD, keine Populationsbehauptung. Das Modell übertrifft die
Nullvorhersage an neun von zehn geeigneten Testtagen. Das bedingte Intervall
schließt null aus; wegen der genannten Voraussetzungen wird daraus keine
unbedingte Signifikanzbehauptung abgeleitet und kein p-Wert berichtet.
Gegenüber dem Trainingstemplate ist ein Vorteil nicht eindeutig belegt.
Der bisherige negative Pilotvergleich bleibt gültig für dessen damalige
Verarbeitung/Daten; die neue Differenz isoliert keinen einzelnen Korrektureffekt.

![Chronologische RMSE und separate Referenzen](healthy-study-v2/chronological-rmse.png)

![Testresiduen, empirische Verteilungen und Autokorrelation](healthy-study-v2/test-residual-distribution.png)

![Parameterstabilität bei Weglassen eines Trainingstages](healthy-study-v2/parameter-stability.png)

### Wissenschaftliche Schlussfolgerung und verbleibende Schritte

[GETESTET] Die Filterasymmetrie ist im neuen Rechenpfad korrigiert; Fit,
chronologische Prüfung und bedingte Unsicherheitsanalyse sind reproduzierbar
berechnet. Softwareprüfung: 85 Tests bestanden, compileall und Ruff F/E9
bestanden. Diese Prüfungen bestätigen Softwareeigenschaften, keine Physik.

[FAKT] Die bereits bewiesene Q1/Q2-Mehrdeutigkeit bleibt bestehen.
[INFERENZ] Der kleine bedingte Vorhersagevorteil gegenüber null rechtfertigt
eine weitere Untersuchung, aber keine eindeutige mechanische Interpretation.
[OFFEN] Eine belastbar physikalisch kalibrierte Healthy-Referenz, unabhängige
Healthy-Zustandsbestätigung, Alarmfehlerrate, Verschleiß-/RUL-Validierung und
Übertragbarkeit auf andere Roboter sind weiterhin nicht erreicht.

Nächste Schritte in fachlicher Reihenfolge:

1. Montage/Koordinaten, analoge IEPE-/DAQ-Übertragung, Anti-Alias-Hardware,
   gemeinsame Hardwarezeit und Strom-/Bewegungssynchronisation durch
   Originaldokumente oder Referenzmessung klären. Der ±25-ms-Diagnosetest
   ersetzt keine Zeitkalibrierung.
2. Bewegungskinematik und wiederholbare Anfangszustände unabhängig messen;
   Strom als Drehmomentproxy kalibrieren. Bis dahin Parameter und symmetrieinvariante
   Kombinationen nur bedingt interpretieren; insbesondere γ bleibt instabil.
3. Für ein neues, vorab eingefrorenes Protokoll bislang ungenutzte Healthy-Blöcke
   mit überlappenden Temperaturbereichen an mehreren Tagen und möglichst
   unabhängigen Läufen/Robotern vorsehen. Zahl anhand eines fachlich begründeten
   Präzisionsziels und blockbezogener Streuung neu planen; zehn Aufnahmen sind
   keine allgemeine Garantie. Temperatur-/Zeitmodell nur auf neuen Entwicklungsdaten
   entwickeln und anschließend auf neuen Holdouts testen.
4. Erst mit ausreichenden unabhängigen Kalibrierungsblöcken ein Alarmkriterium
   samt Fehlalarmpräzision prospektiv festlegen; danach tatsächlich bestätigte
   Degradationsdaten prüfen. Keine Wartungsentscheidung aus diesem Test ableiten.

### Reproduzierbarkeit und Abschlussartefakte

Protokoll: `configs/healthy_study_v2.json`; Auswertung:
`src/kuka_pm/healthy_study.py`; Transport: `data_access/study_fetch.py` und
`data_access/resumable_range.py`. Die ausgeführten Phasen waren der Reihe nach
`python -m kuka_pm.data_access.study_fetch --split train`,
`python -m kuka_pm.healthy_study train`, entsprechend `validation`, dann `test`,
unter `.decoder-venv/Scripts/python.exe`. Phase train ist nach final-freeze
absichtlich gesperrt. Eine unabhängige Reproduktion benötigt eine getrennte
Arbeitskopie der eingefrorenen Eingaben; bestehende Ergebnisdateien und Sperren
nicht löschen, um neu zu optimieren. Darstellungen können ohne Fit oder
Testneuauswertung mit `python -m kuka_pm.healthy_study_figures` erzeugt werden.

`train-freeze.json` und `final-freeze.json` dokumentieren die Freigaben vor
Validierung/Test. `completion-provenance.json` dokumentiert Abschluss-Hashes
und Laufzeitversionen. `*-quality.json`, `*-residual-metrics.csv`,
`*-baselines-separate.csv`, `test-inference.json` und gespeicherte Vorhersagen
halten Einzelresultate fest. Originalmessdaten wurden nicht verändert;
Transportwiederholungen betrafen dieselben vorab gewählten Dateien.
Keine Git-Commits oder Pushes.

## Fortsetzung Phase 4 — 23.10.2026

[GETESTET] Der nachfolgend ausdrücklich freigegebene H1/H3/H6-Vergleich nutzt
nur die sechs bisherigen Entwicklungstage. Vollständige Ergebnisse und
Fortsetzungsdiagnostik: [Phase-4-Ergebnisbericht](torque-hypothesis-study/phase4_result_report.md).
Der alte V2-Test bleibt historisch abgeschlossen und wurde nicht erneut ausgewertet.
H3 verbessert die interne Tagesauslassung gegenüber H1 um rund 6,24% und gegenüber
separat gefittetem Stromtemplate um rund 0,00098 m/s²; das direkte Acc-Template
bleibt im Mittel besser. Keine neue unabhängige Validierung und keine absolute Nm-Kalibrierung.
Natives Stromzeitprofil, Geschwindigkeitsdiagnosen, Residuen, Messkettenproblem
und Modellentscheidung stehen im verlinkten Bericht. STOPP vor H2/H4/H5.
