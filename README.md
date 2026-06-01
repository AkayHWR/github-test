# VOLLSTÄNDIGES PRÄSENTATIONSSKRIPT
## SKP Coding Challenge — Kreditausfallvorhersage
### Termin: 01.06.2026, 11:00 Uhr | Gesprächspartner: Lionel Tankouan + Teamleiter

---

> **WIE DU DAS SKRIPT NUTZT:**
> Alles in „Anführungszeichen" sagst du laut.
> *Kursiv* = Regieanweisung was du im Notebook tust.
> [PAUSE] = kurz inne halten, damit die anderen lesen/schauen können.

---

## ── PHASE 0: EINSTIEG (bevor du teilst) ─────────────────────────────

*Warte bis alle im Call sind. Kamera an. Dann:*

> „Guten Morgen, Herr Tankouan, guten Morgen — schön, dass wir heute zusammenkommen.
> Ich freue mich sehr über die Einladung und die Möglichkeit, mich und meine Lösung vorzustellen.
> Ich würde direkt einsteigen — ich teile jetzt meinen Bildschirm und gehe dann gemeinsam mit Ihnen durch mein Notebook."

*Bildschirm teilen → Jupyter öffnen → ganz nach oben scrollen → Imports-Zelle sichtbar.*

> „Das Notebook ist so aufgebaut, dass jeder Schritt für sich sprechen soll:
> Code, Kommentare direkt im Code, und Markdown-Zellen die erklären warum ich etwas so gemacht habe —
> nicht nur was ich getan habe. Ich gehe jetzt Schritt für Schritt durch."

---

## ── PHASE 1: IMPORTS & ÜBERBLICK (ca. 1 Minute) ────────────────────

*Scrolle zur ersten Markdown-Zelle.*

> „Die erste Markdown-Zelle beschreibt das Gesamtziel:
> Wir haben einen Datensatz zu Hauskrediten — sogenannte Home Equity Loans —
> und wollen vorhersagen, ob ein Kreditnehmer ausfällt oder nicht.

> „Der Aufbau meiner Lösung folgt genau der Aufgabenstellung:
> Daten laden, Typen definieren, Fehlende Werte behandeln,
> PSI prüfen, zwei Modelle bauen, und die Ergebnisse vergleichen."

*Zeige die Import-Zelle (erste Code-Zelle).*

> „Ganz oben stehen alle Imports — ich habe sie in drei Gruppen kommentiert:
> pandas und numpy für die Datenarbeit, sklearn für die Modellierung, und matplotlib plus seaborn für Visualisierungen.
> Das ist bewusst so sortiert, damit man auf einen Blick sieht, was gebraucht wird — ohne die Imports einzeln zu suchen."

---

## ── PHASE 2: DATEN LADEN (ca. 1,5 Minuten) ─────────────────────────

*Scrolle zur Lade-Zelle. Zeige die Ausgabe — df.head() und df.info().*

> „Ich lade den Datensatz aus der Excel-Datei über pandas read_excel.
> Der Datensatz hat 6199 Zeilen und 21 Spalten."

*Zeige df.head() Ausgabe.*

> „Schauen wir kurz auf die ersten Zeilen — man sieht sofort:
> Es gibt eine ID-Spalte, ein Referenzdatum, ein Ausfallsdatum,
> das DEFAULT_FLAG als unsere Zielvariable — also 1 für ausgefallen, 0 für nicht ausgefallen —
> und dann verschiedene Features: Kreditbetrag, Produkttyp, Vertriebskanal, Jobbeschreibung und so weiter."

*Zeige df.info() Ausgabe.*

> „Das info() zeigt uns den Rohzustand: fast alle Spalten sind noch als object oder float geladen,
> und man sieht schon hier fehlende Werte — zum Beispiel hat DEBTINC nur 4836 von 6199 Einträgen.

[PAUSE — lass sie kurz schauen]

---

## ── PHASE 3: DATENTYPKONVERTIERUNG (ca. 2 Minuten) ─────────────────

*Scrolle zur Konvertierungs-Zelle.*

> „Der nächste Schritt ist Datentypkonvertierung — das ist wichtig, weil pandas alleine
> nicht weiß, ob eine Spalte kategorisch ist oder numerisch. Wenn ich das nicht definiere,
> behandelt es zum Beispiel JOB als einen beliebigen Text-String — was bei One-Hot-Encoding
> zu falschen Ergebnissen führen würde."

> „Ich definiere drei Gruppen:
> Erstens die kategorischen Spalten — DATA_TYPE, PRODUCT_TYPE, DISTRIBUTION_CHANNEL,
> COVER_TYPE, REASON_LOAN und JOB — diese konvertiere ich auf category-dtype.
> Zweitens die Datumsspalten — REFERENCE_DATE und DATE_OF_DEFAULT — die werden zu datetime.
> Und drittens alle übrigen Spalten außer ID_CUSTOMER werden numerisch."

*Zeige die Schleife über date_cols_requested.*

> „Beim Datum nutze ich errors='coerce' — das bedeutet: wenn ein Wert nicht als Datum parsebar ist,
> wird er zu NaT, also einem leeren Datumswert, anstatt einen Fehler zu werfen.

*Zeige df.info() nach Konvertierung.*

> „Nach der Konvertierung sieht man im info(): Datumsspalten sind datetime64,
> kategorische Spalten sind category-dtype, und die numerischen sind float64 oder int64.
> Das ist der Zustand den ich brauche für alle weiteren Schritte."

[PAUSE]

---

## ── PHASE 4: ZIELVARIABLE UND FEATURES TRENNEN (30 Sekunden) ────────

*Zeige y = df['DEFAULT_FLAG'] / X = df.drop(...).*

> „Jetzt trenne ich Zielvariable und Features sauber.
> y ist das DEFAULT_FLAG — das ist das was wir vorhersagen wollen.
> X sind alle anderen Spalten — das sind die Informationen, die das Modell nutzen darf.

---

## ── PHASE 5: FEHLENDE WERTE IN KATEGORISCHEN SPALTEN (ca. 2 Minuten) ──

*Scrolle zur impute_categorical_missing()-Funktion.*

> „Hier ist eine eigene Funktion, die ich gebaut habe. Die Aufgabe:
> fehlende Werte in kategorischen Spalten ersetzen — mit einer neuen Kategorie namens 'MISSING'."

> „Warum das? Weil 'Information fehlt' selbst eine Information sein kann.
> Wenn jemand seinen Job nicht angibt, ist das vielleicht kein Zufall —
> das könnte ein Muster sein, das mit Ausfallrisiko zusammenhängt.
> Wenn man einfach mit dem häufigsten Job auffüllt, verliert man dieses Signal."

> „Die Funktion prüft außerdem über isinstance mit pd.CategoricalDtype,
> ob eine Spalte wirklich kategorisch ist.

*Zeige die Ausgabe nach der Funktion — X.info().*

> „Nach der Anwendung sieht man: keine NaN-Werte mehr in den kategorischen Spalten —
> stattdessen gibt es jetzt überall eine MISSING-Kategorie."

[PAUSE]

---

## ── PHASE 6: FILTERN UND TRAIN-TEST-SPLIT (ca. 1 Minute) ───────────

*Zeige Filter + train_test_split.*

> „Jetzt filtere ich auf DATA_TYPE gleich RDS.
> RDS steht für Reference Data Set — das ist die Stichprobe, auf der das Modell trainiert wird.
> Der APP-Datensatz — also Application — bleibt separat. Den brauche ich gleich für den PSI."

> „Den Split mache ich 80 zu 20 — 80 Prozent Training, 20 Prozent Test —
> mit random_state=42 für Reproduzierbarkeit.
> Jeder der dieses Notebook ausführt bekommt die gleiche Aufteilung."

> „Ich nutze .copy() beim Filtern — das verhindert den SettingWithCopyWarning,
> bei dem pandas warnt dass man eine Kopie statt das Original verändert.
> Das ist gutes pandas-Handwerk."

---

## ── PHASE 7: PSI-ANALYSE (ca. 3 Minuten — zentraler Block) ─────────

*Scrolle zur calculate_psi()-Funktion.*

> „Das ist der methodisch interessanteste Teil — die PSI-Analyse.
> PSI steht für Population Stability Index.
> Die Frage dahinter ist: Hat sich die Verteilung meiner Features zwischen
> dem Trainingsdatensatz und dem APP-Datensatz signifikant verändert?"

> „Warum ist das relevant? Wenn ich ein Modell auf historischen Daten trainiere,
> aber die Daten in der Zukunft anders verteilt sind — zum Beispiel weil sich
> der Kundenmix geändert hat — dann verliert das Modell genau dort seine Vorhersagekraft,
> ohne dass man es merkt. PSI macht das sichtbar."

> „Die Interpretation ist standardisiert:
> Unter 0,1 — stabil, kein Handlungsbedarf.
> Zwischen 0,1 und 0,2 — moderate Verschiebung, beobachten.
> Über 0,2 — signifikante Verschiebung, das Feature sollte kritisch geprüft werden."

*Zeige den if is_numeric / elif is_categorical Block.*

> „Die Funktion behandelt numerische und kategorische Features unterschiedlich:
> Bei numerischen Features schneide ich die Daten in Bins — Intervalle —
> und vergleiche die Anteile pro Bin.
> 
> Bei kategorischen Features vergleiche ich direkt die Anteile pro Kategorie.
> Für jedes Feature gibt es auch einen Plot — Trainingsdaten blau, APP-Daten orange —
> damit man die Verschiebung visuell sieht."

> „Ich füge außerdem ein kleines Epsilon — also eine winzige Zahl, hier 1e-6 —
> zu den Anteilen hinzu, bevor ich den Logarithmus berechne.
> Das verhindert log(0) oder Division durch null, was zu Infinity oder NaN führen würde."

*Zeige die PSI-Ausgabetabelle, sortiert nach PSI.*

> „Das Ergebnis: DEROG und DELINQ haben PSI-Werte über 0,2.
> Das sind Variablen für abgewiesene Kredite und Zahlungsverzüge.
> Diese Features sind zwischen Training und APP-Stichprobe signifikant anders verteilt —
> das Modell würde dort instabil sein."

*Zeige den Drop-Code.*

> „Deshalb entferne ich DEROG und DELINQ aus Training und Test —
> zusammen mit den Identifier-Spalten ID_CUSTOMER, REFERENCE_DATE,
> DATE_OF_DEFAULT und DATA_TYPE, die kein Modell-Feature sein sollten."

> „Ich nutze errors='ignore' beim Drop — falls eine Spalte bereits fehlt,
> wirft das keinen Fehler. Das macht den Code robuster."

[PAUSE — das ist ein guter Moment für Rückfragen, lass kurz Raum]

---

## ── PHASE 8: PIPELINE LOGISTISCHE REGRESSION (ca. 2 Minuten) ────────

*Scrolle zur Pipeline-Zelle.*

> „Jetzt bauen wir das erste Modell — und ich nutze dafür eine sklearn-Pipeline.

> „Der Hauptvorteil einer Pipeline: Vorverarbeitung und Modell sind eine Einheit.
> Das verhindert Data Leakage — also den Fall, wo Testdaten die Vorverarbeitung beeinflussen.
> Wenn ich zum Beispiel den Scaler auf allen Daten fitteen würde — Training und Test zusammen —
> dann hätte das Modell beim Testen indirekten Zugang zu Testinformationen.
> Die Pipeline stellt sicher: fit() passiert nur auf Trainingsdaten."

*Zeige numerical_transformer.*

> „Die Pipeline hat zwei Hauptstufen.
> Erstens Preprocessing — aufgeteilt über einen ColumnTransformer,
> der für numerische und kategorische Spalten verschiedene Schritte ausführt.
> Für numerische Features: Imputation mit dem Median, dann StandardScaler.
> Ich nutze den Median statt dem Mittelwert, weil Finanzdaten typischerweise
> Ausreißer haben — ein einziger sehr hoher Kreditbetrag würde den Mittelwert stark verzerren."

*Zeige categorical_transformer.*

> „Für kategorische Features nutze ich One-Hot-Encoding.
> Das bedeutet: jede Kategorie wird zu einer eigenen Binärspalte — zum Beispiel
> wird JOB_Office zu 1 wenn der Job Office ist, sonst 0.
> handle_unknown='ignore' ist dabei wichtig:
> wenn im Testdatensatz eine Kategorie auftaucht, die im Training nicht vorkam,
> setzt das Encoding einfach alle Werte auf 0 — statt einen Fehler zu werfen."

> „Für den Scaler habe ich StandardScaler gewählt.
> Der bringt jedes Feature auf Mittelwert 0 und Standardabweichung 1.
> Das ist ideal für Logistische Regression, weil der Optimierer —
> also der Algorithmus der die Koeffizienten berechnet —
> dann gleichmäßig über alle Features konvergiert.
> Ohne Skalierung würden Features mit großen Werten wie INITIAL_LOAN_AMOUNT
> den Lernprozess dominieren."

> „Zweitens der Klassifikator — LogisticRegression mit solver='liblinear'.
> liblinear ist gut für kleinere Datensätze und unterstützt
> sowohl L1- als auch L2-Regularisierung. Außerdem war dies der einzig mir bekannte."

---

## ── PHASE 9: COMPARE_MODEL FUNKTION (ca. 1 Minute) ──────────────────

*Scrolle zur compare_model()-Funktion.*

> „Ich habe eine eigene Funktion für den Modellvergleich geschrieben.
> Sie ist so gebaut, dass man beliebig viele Modelle übergeben kann —
> als Listen von wahren Labels und vorhergesagten Wahrscheinlichkeiten.
> Für jedes Modell berechnet sie die ROC-Kurve und den AUC-Wert,
> und trägt beides in einen gemeinsamen Plot ein."

> Die Kurve zeigt das Verhältnis zwischen True Positive Rate — also wie oft
> echte Ausfälle erkannt werden — und False Positive Rate —
> wie oft gute Kredite fälschlicherweise als Ausfall eingestuft werden.
> Man will möglichst oben links sein — also viele echte Ausfälle erkennen,
> ohne viele Fehlalarme zu produzieren."

> „AUC ist die Fläche unter dieser Kurve.
> Ein Wert von 0,5 bedeutet: das Modell ist so gut wie Münzwerfen — zufällig.
> Ein Wert von 1,0 wäre perfekte Trennung.

> „Die gestrichelte Diagonale im Plot ist das Zufallsmodell — die Untergrenze.
> Alles was oberhalb liegt ist besser als raten."

---

## ── PHASE 10: TRAINING UND ERGEBNISSE (ca. 1,5 Minuten) ───────────

*Zeige pipe_logistic_regression.fit() und dann den ROC-Plot.*

> „Ich trainiere das Modell mit fit() auf den Trainingsdaten.
> Dann sage ich für Training- und Testdaten die Ausfallwahrscheinlichkeiten voraus —
> mit predict_proba, das für jeden Kreditnehmer zwei Werte zurückgibt:
> die Wahrscheinlichkeit für kein Ausfall und die Wahrscheinlichkeit für Ausfall.
> Ich greife mit [:, 1] auf die zweite Spalte zu — also die Ausfallwahrscheinlichkeit."

*Zeige den ROC-Plot Logistische Regression.*

> „Im Plot sieht man: der Trainings-AUC und der Test-AUC liegen nah beieinander.
> Das ist ein gutes Zeichen.

---

## ── PHASE 11: ENTSCHEIDUNGSBAUM UND VERGLEICH (ca. 1,5 Minuten) ────

*Scrolle zur Entscheidungsbaum-Pipeline.*

> „Als optionalen Schritt habe ich einen Entscheidungsbaum hinzugefügt.
> Ich reuse dabei denselben Preprocessor — also denselben ColumnTransformer —
> und ersetze nur den Classifier.
> Ich nutze max_depth=5, um Overfitting zu begrenzen.
> Ein unbeschränkter Baum würde die Trainingsdaten perfekt auswendig lernen —
> das wäre ein AUC von fast 1,0 auf Training, aber deutlich schlechter auf Test."

*Zeige den kombinierten ROC-Plot.*

> „Im direkten Vergleich sieht man etwas sehr interessantes:
> Der Entscheidungsbaum hat auf den Trainingsdaten einen höheren AUC als die Logistische Regression.
> Aber auf den Testdaten fällt er stärker ab.
> Das ist klassisches Overfitting — der Baum hat sich zu sehr an die Trainingsdaten angepasst."

---

## ── PHASE 12: ABSCHLUSS (ca. 1 Minute) ─────────────────────────────

*Letzte Markdown-Zelle zeigen oder einfach Blickkontakt zur Kamera.*

> „Zum Abschluss meine ehrliche Einschätzung des Ansatzes."

> „Was ich als Stärken sehe:
> Die Pipeline-Struktur sorgt für Sauberkeit und verhindert Data Leakage.
> Die ROC-Funktion ist generisch und wiederverwendbar.
> Das Data Cleanen ist auf andere Datensätze ebenfalls anwendbar sobald man einmal die kategorischen und datenspalten definiert hat."

> „Was ich verbessern würde:
> Erstens Kreuzvalidierung statt einfachem Split — das gibt robustere AUC-Schätzungen.
> Zweitens class_weight='balanced' bei der Logistischen Regression,
> weil Kreditausfälle typischerweise seltener sind als Nicht-Ausfälle —
> ein unbalancierter Datensatz kann das Modell verzerren.
> Drittens würde ich als nächsten Modellschritt XGBoost testen —
> das sehe ich auch in der Stellenbeschreibung als relevante Technologie,
> und es kombiniert Stärken von Entscheidungsbäumen mit deutlich besserer Generalisierung."

> „Das war meine Lösung. Ich freue mich sehr auf Ihre Fragen und Ihr Feedback."

---
---
---

# ═══════════════════════════════════════════════════════════════
# TEIL 2: KRITISCHE CODESTELLEN — AUF DIESE FRAGEN MUSST DU VORBEREITET SEIN
# ═══════════════════════════════════════════════════════════════

---

## 🔴 CODESTELLE 1: errors='coerce' bei Datumskonvertierung

```python
df[col] = pd.to_datetime(df[col], errors='coerce')
```

**Fragen die kommen WERDEN:**
- „Was passiert wenn errors='coerce' nicht gesetzt wäre?"
- „Was ist NaT?"

**Deine Antwort:**
„Ohne errors='coerce' würde pandas bei einem ungültigen Datumswert — zum Beispiel dem String 'n/a' oder einem leeren Feld — eine ValueError-Exception werfen und das ganze Notebook stoppt.
Mit coerce wird der ungültige Wert stattdessen zu NaT — Not a Time — das ist das datetime-Äquivalent zu NaN.
Das ist defensive Programmierung: der Code läuft durch und ich sehe danach wie viele Werte nicht parsebar waren."

---

## 🔴 CODESTELLE 2: cat.add_categories() vor fillna()

```python
if 'MISSING' not in df_copy[col].cat.categories:
    df_copy[col] = df_copy[col].cat.add_categories('MISSING')
df_copy[col] = df_copy[col].fillna('MISSING')
```

**Fragen die kommen WERDEN:**
- „Warum add_categories vor fillna?"
- „Was passiert wenn man das weglässt?"
- „Warum überhaupt 'MISSING' als Kategorie und nicht einfach löschen?"

**Deine Antwort zu add_categories:**
„Bei category-dtype verwaltet pandas intern eine Liste erlaubter Werte — die sogenannten Kategorien.
Wenn ich versuche einen Wert einzufügen der nicht in dieser Liste steht — wie 'MISSING' —
wirft pandas einen ValueError.
Deshalb muss ich 'MISSING' zuerst als erlaubte Kategorie registrieren,
und erst danach kann ich fillna('MISSING') aufrufen.
Ich prüfe außerdem mit 'not in cat.categories' ob es schon existiert — sonst würde ich es doppelt hinzufügen."

**Deine Antwort zu warum MISSING:**
„Man könnte die Zeilen löschen oder mit dem häufigsten Wert auffüllen.
Aber 'fehlt' kann selbst ein Signal sein — wenn jemand seinen Job nicht angibt,
könnte das mit einem höheren Ausfallrisiko zusammenhängen.
Mit der MISSING-Kategorie bleibt diese Information erhalten und das Modell kann daraus lernen."

---

## 🔴 CODESTELLE 3: isinstance(df_copy[col].dtype, pd.CategoricalDtype)

```python
if isinstance(df_copy[col].dtype, pd.CategoricalDtype):
```

**Fragen die kommen WERDEN:**
- „Warum isinstance statt dtype == 'category'?"
- „Was ist der Unterschied?"

**Deine Antwort:**
„Beides funktioniert in den meisten Fällen, aber isinstance ist die robustere Methode.
pd.CategoricalDtype ist ein Typ-Objekt, und isinstance prüft direkt dagegen.
dtype == 'category' ist ein String-Vergleich — der ist anfälliger wenn sich interne Pandas-Implementierungen ändern.
Außerdem ist isinstance lesbarer und entspricht eher Python-Konventionen."

---

## 🔴 CODESTELLE 4: PSI-Formel und Epsilon

```python
epsilon = 1e-6
expected_pct = np.maximum(expected_pct, epsilon)
actual_pct = np.maximum(actual_pct, epsilon)
psi_value = np.sum((expected_pct - actual_pct) * np.log(expected_pct / actual_pct))
```

**Fragen die kommen WERDEN:**
- „Erklären Sie die PSI-Formel."
- „Warum das Epsilon?"
- „Was ist PSI überhaupt und warum ist es relevant?"
- „Welche Schwellenwerte nutzen Sie?"

**Deine Antwort zur Formel:**
„PSI berechnet für jede Bin oder Kategorie die Differenz der Anteile —
Trainingsanteil minus APP-Anteil — multipliziert mit dem natürlichen Logarithmus des Quotienten.
Das ist ähnlich wie die KL-Divergenz aus der Informationstheorie.
Je größer die Verschiebung in einem Bin, desto mehr trägt er zum Gesamt-PSI bei.
Am Ende summiere ich über alle Bins."

**Deine Antwort zu Epsilon:**
„Wenn ein Bin in einer Stichprobe komplett leer ist — also Anteil null —
würde ich durch null dividieren oder den Logarithmus von null berechnen.
Beides ist mathematisch undefiniert und gibt in numpy Infinity oder NaN.
Das Epsilon von 1e-6 — also 0,000001 — ist so klein dass es das Ergebnis kaum beeinflusst,
aber verhindert diesen numerischen Fehler."

**Deine Antwort zu Schwellenwerten:**
„Unter 0,1: stabile Verteilung — kein Handlungsbedarf.
0,1 bis 0,2: moderate Verschiebung — beobachten, eventuell nachschauen.
Über 0,2: signifikante Verschiebung — Feature kritisch prüfen oder entfernen.
Diese Schwellenwerte sind in der Kreditrisikopraxis Standard — sie kommen aus der Literatur und regulatorischen Leitfäden."

---

## 🔴 CODESTELLE 5: ColumnTransformer und remainder='passthrough'

```python
preprocessor = ColumnTransformer(
    transformers=[
        ('num', numerical_transformer, numerical_features),
        ('cat', categorical_transformer, categorical_features)
    ],
    remainder='passthrough'
)
```

**Fragen die kommen WERDEN:**
- „Was macht ColumnTransformer?"
- „Was bedeutet remainder='passthrough'?"
- „Warum nicht einfach alles mit einem Scaler transformieren?"

**Deine Antwort zu ColumnTransformer:**
„ColumnTransformer wendet verschiedene Transformationen auf verschiedene Spaltengruppen an — parallel.
Numerische Features bekommen Imputation und Skalierung.
Kategorische Features bekommen One-Hot-Encoding.
Das Ergebnis wird dann spaltenweise zusammengefügt zu einer einzigen Matrix, die das Modell bekommt."

**Deine Antwort zu remainder='passthrough':**
„Alle Spalten die nicht explizit in transformers aufgeführt sind, werden unverändert durchgereicht.
In meinem Fall sollte das keine Spalten betreffen — ich habe alle zugewiesen.
Aber es ist eine Sicherheitsnetz: falls ich eine Spalte vergessen hätte zu definieren,
würde sie nicht still verschwinden, sondern erhalten bleiben."

---

## 🔴 CODESTELLE 6: handle_unknown='ignore' beim OneHotEncoder

```python
OneHotEncoder(handle_unknown='ignore')
```

**Fragen die kommen WERDEN:**
- „Was passiert ohne handle_unknown='ignore'?"
- „Ist das immer die richtige Wahl?"

**Deine Antwort:**
„Ohne diesen Parameter würde sklearn beim Transform des Testdatensatzes einen ValueError werfen,
sobald eine Kategorie auftaucht, die im Training nicht vorkam.
Das passiert in der Praxis häufig — zum Beispiel wenn ein neuer Vertriebskanal hinzukommt.
Mit ignore werden alle Einträge dieser unbekannten Kategorie auf 0 gesetzt —
das ist konservativ: das Modell tut so als hätte es diese Information nicht.
Eine Alternative wäre, unbekannte Kategorien als MISSING zu behandeln — das wäre meine Erweiterungsidee."

---

## 🔴 CODESTELLE 7: StandardScaler — Warum dieser?

```python
StandardScaler()
```

**Fragen die kommen WERDEN:**
- „Warum StandardScaler und nicht MinMaxScaler oder RobustScaler?"
- „Braucht ein Entscheidungsbaum auch einen Scaler?"

**Deine Antwort zu StandardScaler:**
„StandardScaler bringt jedes Feature auf Mittelwert 0 und Standardabweichung 1.
Das ist für Logistische Regression wichtig, weil der dahinterliegende Optimierer —
Gradient Descent oder liblinear — schneller und stabiler konvergiert wenn alle Features
auf einer ähnlichen Skala sind.
MinMaxScaler wäre eine Alternative, ist aber anfälliger für Ausreißer —
ein einzelner extremer Wert würde die ganze Skala verschieben.
RobustScaler nutzt Median und IQR statt Mittelwert und Standardabweichung — das wäre bei
stark verzerrten Finanzdaten eigentlich noch robuster, ein guter Verbesserungsvorschlag."

**Deine Antwort zu Entscheidungsbaum:**
„Nein — Entscheidungsbäume brauchen keinen Scaler.
Sie treffen ihre Entscheidungen nur auf Basis von Schwellenwerten: ist Feature X größer als Y?
Die absolute Skala des Features ändert daran nichts.
In meiner Pipeline ist der Scaler trotzdem drin, weil ich denselben Preprocessor wiederverwendet habe.
Das schadet dem Baum nicht — es ist nur unnötig. In einer optimierten Version
würde ich einen separaten Preprocessor ohne Scaler für den Baum bauen."

---

## 🔴 CODESTELLE 8: predict_proba()[:, 1]

```python
y_train_pred_proba_logreg = pipe_logistic_regression.predict_proba(X_train_cleaned)[:, 1]
```

**Fragen die kommen WERDEN:**
- „Was gibt predict_proba() zurück?"
- „Warum [:, 1] und nicht [:, 0]?"
- „Warum predict_proba und nicht predict?"

**Deine Antwort:**
„predict_proba gibt eine Matrix zurück — eine Zeile pro Beobachtung, zwei Spalten.
Die erste Spalte ist die Wahrscheinlichkeit für Klasse 0 — kein Ausfall.
Die zweite Spalte ist die Wahrscheinlichkeit für Klasse 1 — Ausfall.
Mit [:, 1] greife ich auf alle Zeilen, zweite Spalte zu — also die Ausfallwahrscheinlichkeit für jeden Kreditnehmer.
Die beiden Spalten summieren sich immer zu 1 — das ist eine Eigenschaft von Wahrscheinlichkeiten.

Ich nutze predict_proba statt predict, weil roc_curve() und roc_auc_score() 
kontinuierliche Wahrscheinlichkeiten brauchen — keine binären 0/1-Vorhersagen.
Die ROC-Kurve entsteht dadurch, dass intern viele verschiedene Schwellenwerte durchprobiert werden —
bei 0,3 ist etwas ein Ausfall, bei 0,5, bei 0,7 — und für jeden wird TPR und FPR berechnet.
Mit binären Vorhersagen hätte man nur einen einzigen Punkt, keine Kurve."

---

## 🔴 CODESTELLE 9: ROC-Kurve und AUC erklären

```python
fpr, tpr, _ = roc_curve(y_empirical[i], y_predicted[i])
auc_score = roc_auc_score(y_empirical[i], y_predicted[i])
```

**Fragen die kommen WERDEN:**
- „Was ist TPR und FPR genau?"
- „Was sagt der AUC-Wert aus?"
- „Warum ist AUC besser als Accuracy für diesen Use-Case?"

**Deine Antwort zu TPR/FPR:**
„TPR — True Positive Rate — auch Sensitivität oder Recall — ist der Anteil der echten Ausfälle,
die das Modell korrekt als Ausfall identifiziert.
FPR — False Positive Rate — ist der Anteil der guten Kredite,
die das Modell fälschlicherweise als Ausfall eingestuft hat.
Man will einen hohen TPR bei niedrigem FPR — also viele Ausfälle erkennen ohne zu viele Fehlalarme."

**Deine Antwort zu AUC:**
„AUC ist die Fläche unter der ROC-Kurve — ein Wert zwischen 0 und 1.
Intuitiv: AUC ist die Wahrscheinlichkeit, dass das Modell einem zufällig gewählten Ausfall
eine höhere Wahrscheinlichkeit zuweist als einem zufällig gewählten Nicht-Ausfall.
0,5 bedeutet: nicht besser als Zufall.
1,0 bedeutet: perfekte Trennung.
In der Kreditrisikopraxis ist AUC über 0,7 akzeptabel, über 0,8 gut."

**Deine Antwort zu AUC vs Accuracy:**
„Bei Kreditausfällen ist die Zielklasse stark unbalanciert — es gibt viel mehr gute Kredite als Ausfälle.
Ein dummes Modell das immer 'kein Ausfall' vorhersagt, hätte vielleicht 95% Accuracy —
aber es erkennt keinen einzigen echten Ausfall.
AUC misst das Trennvermögen unabhängig vom Schwellenwert und ist robust gegenüber Klassenungleichgewicht.
Deshalb ist AUC im Kreditrisiko die Standardmetrik."

---

## 🔴 CODESTELLE 10: Data Leakage durch Pipeline verhindern

**Fragen die kommen WERDEN:**
- „Was ist Data Leakage?"
- „Wie verhindert die Pipeline das?"

**Deine Antwort:**
„Data Leakage bedeutet: das Modell hat beim Training indirekten Zugang zu Informationen
die es eigentlich nicht haben dürfte — zum Beispiel aus den Testdaten.
Das führt zu einem zu optimistischen AUC beim Training, der in der Realität nicht hält.

Ein typischer Fehler: ich fitte den StandardScaler auf dem gesamten Datensatz —
Training plus Test — und nutze dann die skalierten Testdaten zum Evaluieren.
Der Scaler kennt dann den Mittelwert und die Standardabweichung der Testdaten —
das sind Informationen die er in der Praxis nicht hätte.

Die Pipeline löst das: wenn ich pipeline.fit(X_train) aufrufe,
wird der Scaler nur auf X_train gefittet.
Wenn ich dann pipeline.predict(X_test) aufrufe,
transformiert der Scaler X_test mit den Parametern aus X_train — nicht aus X_test selbst.
Das ist korrekt und entspricht dem echten Einsatzszenario."

---

## 💡 BONUS: Fragen die aus dem Stellenprofil kommen könnten

**„Sie sehen XGBoost in der Stellenbeschreibung — wie würden Sie das hier einbauen?"**
„Ich würde eine dritte Pipeline erstellen — pipe_xgboost — mit demselben Preprocessor
und xgboost.XGBClassifier als Classifier. XGBoost ist ein Gradient Boosting Modell —
es baut iterativ viele schwache Entscheidungsbäume und korrigiert bei jedem Schritt die Fehler des vorherigen.
Es ist meist besser als ein einzelner Baum und oft besser als Logistische Regression bei nichtlinearen Zusammenhängen.
Für den Vergleich würde ich es einfach in compare_model mit den anderen Kurven darstellen."

**„Was sind PD und LGD — der Schwerpunkt der Stelle?"**
„PD ist Probability of Default — die Wahrscheinlichkeit dass ein Kreditnehmer ausfällt.
Genau das modellieren wir hier mit DEFAULT_FLAG als Zielvariable.
LGD ist Loss Given Default — wie viel Prozent des ausstehenden Betrags verliert die Bank wenn der Kredit ausfällt.
Das wäre ein separates Regressionsmodell. Beide Parameter zusammen mit EAD — Exposure at Default —
ergeben den erwarteten Verlust: EL = PD × LGD × EAD."

**„Was bedeutet regulatorisch konform in diesem Kontext?"**
„Im Kreditrisiko gibt es regulatorische Anforderungen — vor allem aus Basel III und der CRR.
Die fordern unter anderem, dass Modelle dokumentiert, validiert und stabil sein müssen.
PSI ist genau ein solches Validierungsinstrument. Interpretierbarkeit ist ebenfalls gefordert —
man muss erklären können warum ein Kredit abgelehnt wird, was bei Logistischer Regression gut möglich ist."

