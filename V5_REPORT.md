# V5 — implementiertes und empirisch geprüftes J3-Grey-Box-Modell

## Fragestellung
Verbessert eine konsistente Kombination aus Starrkörperdynamik, kausalem Antrieb,
Sensorabbildung und stochastischer Winkelstörung die Healthy-Prognose gegenüber AR?

## Voraussetzungen
[ENTSCHEIDUNG] J2 steht exakt still, J3 bleibt starr. Keine J2-Störung und keine
elastische Körpermode. Modelle gelten bedingt auf B3-Sensormontage und die
explizite historische Beobachtungshypothese; deren physikalische Kalibrierung
wird nicht behauptet. V4-Dateninventur und Spektren übernommen, keine Wiederholung.
J3/50/Sensor3XYZ: sechs Trainingstage, drei Kalibriertage, zwölf bekannte spätere
Healthy-Tage. Historische Evaluation ist kein neuer Blindtest.

## Methode
[IMPLEMENTIERT] Vollständige Kette Antrieb→J3-Winkelzustand→Körperbeschleunigung
→Sensorrotation/spezifische Kraft→kausales IEPE-Arbeitsmodell. Einheiten m/s²;
Residuen gemessene minus vorhergesagte Sensorbeschleunigung.
Sechs Antriebskandidaten mit gruppierter CV; Auswahl H3_200ms.
Für die AR-Ordnung von C werden mechanische Fold-Fits verwendet. Die Ordnungen
2/8/16/24 wurden erneut geprüft; A und C wählen auf allen drei Achsen24.
MLE und zwei MAP-Priorstärken untersucht. D ist ein echter dreizuständiger
Winkel-/Winkelgeschwindigkeits-/Anregungsprozess mit Kalman-Likelihood.
Globale Fits und Alarmschwellen wurden vor der bekannten Evaluation eingefroren.

## Ergebnisse
T1: mittlerer Tages-RMSE der drei Achsen auf derselben nativen1,5–7s-Maske:

| Modell | RMSE [m/s²] |
|---|---:|
| zero | 2.832966 |
| template | 3.009817 |
| A_V4 | 1.452982 |
| A_MLE | 1.452605 |
| B | 2.833019 |
| C_MLE | 1.452393 |
| D_MLE | 2.821173 |

C verbessert A nur um **0.0146%**.
D verbessert B um **0.418%**, bleibt aber weit
hinter A/C. Die MAP-Varianten ändern diese Rangfolge nicht. C ist numerisch
minimal besser; ein praktisch überzeugender mechanischer Zusatznutzen ist nicht belegt.
T1 hat nur39,06µs Horizont, keine nachgewiesene praktische Vorwarnzeit.

T2: vorher festgelegte51 Ursprünge je Tag, alle100ms, jeweils gleiche Zielpunkte:

| Horizont | Null | A | B | C | D |
|---|---:|---:|---:|---:|---:|
| 10ms | 2.796642 | 2.798359 | 2.796701 | 2.797918 | 2.796701 |
| 100ms | 2.755976 | 2.755781 | 2.756635 | 2.755540 | 2.756635 |
| 500ms | 2.769286 | 2.768906 | 2.769907 | 2.768735 | 2.769907 |

Hier liegt kein praktisch relevanter Abstand zur Nullvorhersage vor.
Bekannte verzögerte Ströme werden weitergenutzt; unbekannte zukünftige Ströme
werden gehalten. Keine echten zukünftigen Zwischenwerte in AR/Kalman-Historien.
T1 und T2 haben unterschiedliche Auswertedichten; Zahlen nur innerhalb der
jeweiligen Tabellenzeile unmittelbar vergleichen.

**D-Befund:** Die drei D-Schätzungen konvergieren, treffen aber die vorab
gesetzte Frequenzobergrenze4kHz. Das ist keine gemessene Eigenfrequenz.
Der eingefrorene Sensorrahmen lässt im führenden Winkelbeschleunigungsanteil
keine X-Komponente zu; ein großer Teil der realen X-Energie verbleibt daher
im Rauschmodell. Das ist eine konkrete Grenze dieser getesteten Beobachtung,
kein allgemeiner Gegenbeweis gegen Starrkörper-Winkelstörungen.

**Anomalien:** Die vorab aus drei Kalibriertagen festgelegten NLL-Maximalschwellen
werden bei A, B, C und D jeweils von **12/12** späteren Healthy-Tagen überschritten.
Beobachtete dateibezogene Fehlalarmquote100%. Dies wurde anhand der Einzelwerte
geprüft; keine Schwelle nachträglich erhöht. Bei A steigen NLL-Werte von maximal
4,9953 in der Kalibrierung auf5,0412–5,1767 später. Schon moderate Varianz-/Spektral-
verschiebungen überschreiten die enge Drei-Tage-Referenz.
Die V4-Quote25% verwendete einen anderen Score und andere Bedingungen; die
Prozentwerte sind kein kontrollierter Vergleich zweier identischer Alarmverfahren.

## Mathematische Bedeutung
[GETESTET] Sensorpunktformeln, Rotationskonvention, Gravitationsableitungen,
kausale Eingangsverarbeitung, AR-Mehrschrittgleichung und Kalman-Likelihood
sind numerisch geprüft. Ein synthetischer stochastischer Versuch erkennt die
bekannte Prozessvarianz. Das validiert die Software, nicht die Robotermodellhypothese.
Für dieselben Koeffizienten ist C exakt äquivalent zu
c+sum(phi*a_past)+(m−sum(phi*m_past)); Gleichwertigkeit folgt in beide Richtungen
durch Einsetzen/Ausmultiplizieren. Aus dieser Identität folgt keine Verbesserung.

Bei positiver Referenz-RMSE gilt G>0 ⇔ RMSE_Modell<RMSE_Referenz. Das belegt
einen Vorteil auf den geprüften Daten, ist jedoch nicht hinreichend für
mechanische Validität oder gute Alarmkalibrierung. Frequenzgrenzkontakt und
schlechte Generalisierung der NLL ⇒ keine belastbare Interpretation der
geschätzten D-Frequenz als physikalischer Robotermodus.
S>Schwelle ⇔ Alarmflag ist lediglich die definierte Entscheidungsregel;
Alarmflag ⇏ bestätigter Verschleiß.

## Schlussfolgerung
[ENTSCHEIDUNG] Direktes AR bleibt die sparsamere native Kurzfrist-Referenz.
C wird als getesteter Grey-Box-Kandidat erhalten, sein Zusatznutzen ist sehr klein.
D ist tatsächlich implementiert und geschätzt, aber in der getesteten Form
nicht konkurrenzfähig. Für10–500ms liefern die Modelle keinen belastbaren
Rohwellenform-Vorhersagegewinn; die V4-Bandenergie-Referenz bleibt eine getrennte,
sinnvolle Zielgröße. Keine nachträgliche Optimierung auf diesen zwölf Tagen.
Keine lokale Oil-Leaking-/Washout-Auswertung möglich; keine Schadenslabels erfunden.
Nächster Untersuchungsschritt: neue, zeitlich breitere Healthy-Kalibrierungs- und
Blindaufnahmen mit der eingefrorenen statistischen Referenz prüfen, einschließlich
Temperatur- und Spektraldrift. Aktuell keine Alarmfreigabe.

Artefakte: PARAMETER_FITS.json, MODEL_COMPARISON.csv, FREQUENCY_BAND_SCORES.csv,
ANOMALY_SCORES.csv, CHECKPOINT.json. Tests/Ressourcen/Integrität in VERIFICATION.json.
