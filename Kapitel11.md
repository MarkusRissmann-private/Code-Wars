# Kapitel 11: Die Sonar‑Inquisition

## Prolog: Wenn Standards zu Ketten werden

*„Standards sind wie Karten. Ohne sie verirrst du dich. Mit ihnen auch – wenn du vergisst, dass sich das Gelände ändert.“*

– Aus den Chroniken des Architektenordens

---

Der alte Architekt des Architektenordens saß am Fenster. Unten die Stadt. Drinnen Stille. Auf seinem Bildschirm: drei Repositories, drei Teams, drei identische Pipelines – und drei sehr verschiedene Ergebnisse.

„Wir haben gewonnen“, sagte jemand hinter ihm. „Wir haben die Referenz‑Architektur. Alle Teams übernehmen sie.“

Qion Varr drehte sich um. „Dann beginnt jetzt unsere schwerste Aufgabe: zu verhindern, dass sie zur Religion wird.“

---

## I. Die Template‑Euphorie

Montag, 10:15 Uhr. All‑Hands im Engineering.

Der CTO: „Gute Nachrichten. V3 ist Reference Architecture. Ab sofort nutzen alle Teams das Template.“

Sora Nyra flüsterte zu Arik: „Das wird weh tun.“

Arik: „Warum? Es ist doch gut.“

Sora: „Weil ‚gut‘ in falschen Händen zu ‚gesetzlich‘ wird.“

---

## II. Das erste Team: Copy & Paste

Team Atlas klonte das Template.

```text
git clone reference‑architecture
replace 'service‑name' with 'atlas‑orders'
push
```

Der Linter nickte. Die Pipeline war grün. Die Architektur war richtig – auf dem Papier.

Am dritten Tag kam der erste PR:

```text
feat: Add CSV import for orders (temporary)

// quick hack until vendor API is ready
if (file.Extension == ".csv") {
    ProcessCsv(file); // bypasses validation
}
```

Der Linter schwieg. Die Business‑Metriken schrien. CSV‑Bestellungen ohne Validierung. Ein Team, das das Template befolgte – und den Geist verriet.

Qion kommentierte nur zwei Worte: „Nein. Warum?“

Der PR wurde umgebaut. Eine `IOrderImportPolicy` entstand. Der CSV‑Pfad wurde ein Test – nicht ein Ausnahmezustand.

---

## III. Das zweite Team: Der Linter als Thron

Team Boreas erhob den Linter zum König. Alles, was grün war, galt als gut. Alles, was rot war, war böse.

Arik besuchte ihr Stand‑up. Hörte Sätze, die er kannte – aus einem früheren Leben.

„Coverage ist nur 79,8%. Schreib noch zwei Tests.“

„Was testen wir?“

„Egal. Hauptsache 80%.“

Arik sah zu Boden. „So beginnt der Niedergang: Wenn das Maß zum Ziel wird.“

Er zeichnete an ihr Whiteboard:

```text
Metrik → Spiegel, nicht Bühne
Gate → Schutz, nicht Folter
Zahl → Hinweis, nicht Wahrheit
```

Dann zeigte er ihre Business‑Metriken:

```text
ATLAS ORDERS – letzte Stunde
Success: 99.1%
Zero‑Results: 0
Refund Rate Spike: +240% ⚠︎

Top Signal: DiscountPolicy mis‑applied (Tenant 12)
```

„Eure Tests sind grün“, sagte Arik. „Eure Kunden sind es nicht.“

---

## IV. Das dritte Team: Die heilige Checkliste

Team Circe schrieb eine Checkliste. 48 Punkte. Jeder PR musste alle abhaken.

Oben Kell las sie. Punkt 17: „ADR hinzufügen“. Punkt 18: „Diagramm aktualisieren“. Punkt 19: „Runbook pflegen“.

Er fragte: „Warum?“

Stille.

„Weil es draufsteht“, sagte jemand.

Oben nahm den Stift. Strich Punkt 17–19 durch. Schrieb darüber:

```text
Begründung → dann Dokument
Erkenntnis → dann Artefakt
Verantwortung → dann Gate
```

„Dokumentation ist nicht Bürokratie“, sagte er. „Sie ist Zeitreisen. Aber nicht jeder Spaziergang braucht eine Landkarte.“

---

## V. Der Rückschlag

Freitag, 21:47 Uhr. PagerDuty. Team Atlas. P1.

```text
CRITICAL: Order Calculation incorrect (Tenant 7)
Root Cause: Hotfix in PolicyFactory (bypass for CFO)
```

Arik atmete aus. „Das alte Gespenst.“

Qion schrieb in den Incident‑Channel:

```text
Kein Wutausbruch. Nur eine Frage:
Warum war es einfacher, die Fabrik zu verbiegen,
als eine legitime Ausnahme zu modellieren?
```

Stille. Dann kam die Antwort: „Weil der Review Montag ist.“

Qion: „Dann verschieben wir den Review. Nicht die Verantwortung.“

Der Hotfix wurde zurückgenommen. Eine `IRoleBasedExceptionPolicy` entstand – mit Tests, mit ADR, mit Deploy in eigener Einheit. Montag war spät. Aber richtig.

---

## VI. Der Rat: Standards ohne Ketten

Montag. Interner Talk. „Standards ohne Ketten.“

Sora Nyra zeigte drei Folien. Keine mehr.

```text
1) Prinzip vor Pattern
2) Verantwortung vor Zahl
3) Grenze vor Geschwindigkeit
```

Qion ergänzte:

```text
Der Compiler ist die Macht → Verstehen
Der Linter ist der Spiegel → Erkennen
Die Commit‑Bruderschaft ist das Gedächtnis → Erinnern
Das Architectum ist der Raum → Lehren
```

„Wenn ihr das Template nutzt“, sagte Sora, „fragt zuerst: Welches Prinzip schützt es? Wenn ihr das nicht sagen könnt, nutzt ihr eine Form – und verratet den Inhalt.“

---

## VII. Die Entscheidung

Zwei Wochen später. Drei Teams, drei Wege – wieder eins geworden. Nicht weil die Checkliste schwerer wurde. Sondern weil der Sinn zurückkehrte.

Die Commit‑Bruderschaft speicherte drei ADRs:

```markdown
# ADR‑021: Metrics as Decision Aid, not Target
# ADR‑022: Exceptions as Policies, not Hotfixes
# ADR‑023: Templates as Teaching, not Law
```

Qion schloss das Repository. „Standards sind Werkzeuge. Ketten sind Gefängnisse. Wenn ihr Geschwindigkeit wollt – lernt den Unterschied.“

---

*Du sitzt jetzt vor deinem Screen. Dein Team hat das neue Template übernommen. Die Pipeline ist grün. Der Linter lächelt. Jemand sagt: „Nur noch diese Zahl, dann sind wir compliant."*

*Was wirst du sagen?*

*„Easy"?*

*Oder: „Stopp. Erst das Prinzip. Dann die Zahl."*

---

**Nächstes Kapitel:** „Der Feature-Krieg" – Wenn das Bauen von Features wichtiger wird als das Erreichen von Outcomes.

**Ende von Kapitel 11.**

*Möge der Compiler mit dir sein.*
