# Kapitel 7: Die Meister und die Schüler

## Prolog: Die Schule des Architektenordens

*„Weisheit, die nicht weitergegeben wird, stirbt. Und mit ihr stirbt jedes System, das sie trug.“*

– Aus den Chroniken des Architektenordens

---

Der alte Architekt des Architektenordens stand vor einer Tür. Glas. Darauf in weißer Schrift: „Onboarding – V3 System“. Drinnen: sechs neue Gesichter. Müde von Bewerbungsgesprächen, glänzend vor Hoffnung.

Qion Varr legte die Hand auf den Türgriff. Atmete. Öffnete.

**Artefakt auf dem Tisch:** Ein Heft. Schwarz. Darauf: „Die fünf Fragen der Verantwortung“.

```text
QIONS NOTIZBUCH – KOPIE FÜR SCHÜLER

1) Was ist die EINE Verantwortung dieses Services?
2) Wo endet er? (Explizite Grenzen)
3) Wie wird er deployt? (getrenntes Schicksal)
4) Woran messen wir „korrekt“? (Business-Metriken)
5) Was tun wir, wenn „easy“ gesagt wird?
```

„Willkommen“, sagte Qion. Seine Stimme war ruhig. „Heute geht es nicht um Tools. Nicht um Azure, nicht um Sonar, nicht um Service Bus. Heute geht es darum, Systeme zu bauen, die bleiben – auch wenn ihr geht.“

---

## I. Der erste Unterricht

Montag, 09:03 Uhr. Whiteboard. Marker. Kaffee.

Arik Dane stand neben Qion. „Wir fangen klein an“, sagte Arik. „Eine neue Anforderung: Tenant‑spezifische Validierung für Dokumente. Klingt harmlos. Ist der Anfang von allem.“

Er schrieb:

```text
STORY: As Tenant X, I need custom validation rules

Akzeptanzkriterien:
- Größe < 25 MB
- Nur PDF & PNG
- Für Tenant X: zusätzliche Wasserzeichenprüfung
```

Eine Hand ging hoch. Finn, neu im Team. „Das ist doch easy? Ein if (tenantId == X) und fertig?“

Qion sah ihn an. Sanft. „Sag das Wort noch einmal. Aber höre zu, was es mit dir macht.“

Finn lächelte. „Easy.“

Qion legte den Marker ab. „Wenn ‚easy‘ gesprochen wird, beginnt der Unterricht. Nicht weil ihr falsch wärt. Sondern weil das Wort einen Schatten wirft: die Zukunft.“

---

## II. Das Beispiel aus der Vergangenheit

Qion öffnete Git. Ein altes Repo. `DmsUploader (legacy)`.

```text
commit a3f8e92 - feat: Added API Beta support
commit 9b2e4f3 - feat: Add OneDrive target
commit 7c1a9d2 - feat: Add validation (size, type)
commit 2d8e7a4 - feat: Add per-tenant exception (Tenant 143)
```

„Hier“, sagte er. „Ein if‑Statement für Tenant 143. Dann eins für 247. Dann eins für 501. 2.400 Zeilen später fragt dich niemand mehr, warum es brennt. Alle fragen nur noch, warum du nicht schneller löschst.“

Er klappte den Laptop zu. „Wir machen es diesmal anders.“

---

## III. Die Übung: Grenzen ziehen

Workshop. Zwei Gruppen. Jeweils ein Flipchart.

Aufgabe: „Entwerft eine Lösung, in der Tenant‑Regeln hinzugefügt werden können, ohne den Orchestrator anzufassen.“

Gruppe 1 (Cargo‑Cult): „Wir machen eine Config‑Datei. JSON. Pro Tenant Regeln. Der Orchestrator liest die Datei und verzweigt.“

Gruppe 2 (Ordnung): „Wir definieren ein Interface `IValidationPolicy`. Pro Tenant eine Implementierung. Registriert per DI. Der Orchestrator kennt nur `IValidationPolicyFactory`.“

Qion ging zwischen den Tafeln hin und her. Schwieg. Hörte zu.

Nach 20 Minuten bat er um die Stifte. Schrieb über beide Entwürfe:

```text
Gruppe 1: Konfiguration statt Architektur
→ Wenn der Orchestrator Regeln ‚liest‘, kennt er sie.
→ Kennt er sie, trägt er Verantwortung, die ihm nicht zusteht.

Gruppe 2: Verantwortung kapseln
→ Neue Regeln = neue Klasse, eigener Test, eigener Deploy
→ Orchestrator bleibt blind (und das ist gut)
```

„Blindheit“, sagte er, „ist eine Tugend. Ein Orchestrator, der alles ‚weiß‘, ist nur höflicher Spaghetti‑Code.“

---

## IV. Der Spiegel des Linters

Der Linter betrat den Raum. Unsichtbar. Unbarmherzig.

Arik startete die Pipeline.

```text
SONARQUBE QUALITY GATE

Coverage: 93.7% ✓
Duplication: 2.8% ✓
Cognitive Complexity (Orchestrator): 7 ✓
Security Vulnerabilities: 0 ✓

STATUS: PASSED
```

Jubel. Kurz. Zu kurz.

Qion hob die Hand. „Der Linter ist ein Spiegel. Er zeigt euch, ob ihr geschminkt seid. Nicht, ob ihr lebt.“

Er zeigte auf die Business‑Metriken:

```text
BUSINESS METRICS – letzte Stunde
Tenant 1: 99.2% Success
Tenant 143: 88.7% Success (⚠︎ Zero‑Results: 9)
Tenant 247: 100% Success

Top Error: ValidationPolicy.MissingWatermark (Tenant 143)
```

„Das ist die Wahrheit“, sagte er. „Nicht grün oder rot im Tool. Sondern: was beim Nutzer ankommt.“

---

## V. Die Falltür

Mittwoch, 16:42 Uhr. Finn öffnete einen PR.

```text
feat: Add quick exception for CFO (urgent)

// TODO remove next week
if (userRole == "CFO") return Ok();
```

Der Raum wurde kalt.

Oben Kell kommentierte: „Nein.“

Finn antwortete: „Nur für diese Woche. Zeitsparend. Easy.“

Arik schrieb: „Wenn wir das mergen, lehren wir: Regeln gelten, bis sie wehtun. Dann beugen wir sie. Das ist die Geburtsstunde jedes Monolithen.“

Der PR wurde geschlossen. Nicht gemerged. Finn senkte den Blick. Qion klopfte ihm auf die Schulter. „Gute Entscheidung. Von uns. Und von dir.“

---

## VI. Die Übergabe

Freitag. 17:00 Uhr. Die neuen Entwickler deployen ihre erste Änderung. Keine Incidents. Keine Ausreden. Nur ein stilles Nicken.

Die Commit‑Bruderschaft speicherte die ADR:

```markdown
# ADR‑018: Tenant Validation via Strategy Pattern

Context:
Tenant‑spezifische Validierung ohne Orchestrator‑Verzweigungen.

Decision:
IValidationPolicy pro Tenant; Factory im Orchestrator.

Consequences:
Neue Regeln → neue Klasse, neuer Test, eigener Deploy.
Orchestrator bleibt blind für Regeln.

Status: Accepted
```

Qion schloss den Laptop. „Die Meister sind keine, die alles wissen. Die Meister sind die, die Grenzen sehen – und sie lehren.“

---

*Du sitzt jetzt vor deinem Screen. Ein neuer Kollege fragt dich nach einem ‚easy‘ Workaround. Die Deadline atmet dir in den Nacken. Der Linter ist grün. Dein Bauch nicht.*

*Was wirst du sagen?*

*„Easy“?*

*Oder: „Stopp. Die Grenze bleibt. Auch heute.“*

# Kapitel 7: Die Geister der Legacy

## Prolog: Die Schuld der Väter

*"Legacy ist nicht das, was alt ist. Legacy ist das, was niemand mehr versteht. Und das Gefährlichste an Legacy ist nicht, dass sie existiert. Das Gefährlichste ist die Illusion, dass man sie modernisieren kann, ohne sie zu verstehen."*

— Aus den Lehren des Architektenordens

---

Der alte Meister des Ordens öffnete ein Deployment-Diagramm. Datiert: Vor acht Monaten.

```text
╔════════════════════════════════════════════════╗
║           V3 SYSTEM - MODERN STACK             ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  [Orchestrator Service]  ←─────┐              ║
║  [Auth Service]                 │              ║
║  [Document Processing]          │              ║
║  [Target Service]               │              ║
║  [Notification Service]         │              ║
║                                  │              ║
║                                  │              ║
║  [BPP Calculation Service] ←────┘              ║
║   Status: LEGACY                                ║
║   Tech: SQL Server Stored Procedures           ║
║   Lines: ~8,000 SP Code                        ║
║   Age: 7 years                                  ║
║   Knowledge Holder: NONE (developer left)      ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

"Das," sagte Qion Varr, "war der Geist, den niemand sehen wollte."

Der junge Schüler starrte auf das Diagramm. "Aber... sie haben doch V3 gebaut? Clean Architecture? Microservices?"

"Ja," sagte Qion. "Sie haben fünf neue Services gebaut. Modern. Sauber. Cloud-native."

"Und?"

"Und sie haben vergessen, dass der sechste Service—der wichtigste—nicht modernisiert wurde. Er war zu kompliziert. Zu riskant. Zu scary."

Er scrollte zum nächsten Diagramm. Heute.

```
╔════════════════════════════════════════════════╗
║        V3 SYSTEM - THE UNCOMFORTABLE TRUTH     ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  ┌─────────────────────────────────────────┐  ║
║  │   MODERN MICROSERVICES                  │  ║
║  │   (Cloud, K8s, Event-Driven)            │  ║
║  └──────────────┬──────────────────────────┘  ║
║                  │                              ║
║                  │ Every calculation request   ║
║                  ↓                              ║
║  ┌─────────────────────────────────────────┐  ║
║  │   BPP CALCULATION SERVICE               │  ║
║  │   (On-Prem SQL Server, 1 VM)            │  ║
║  │   • 8,000 lines Stored Procedures       │  ║
║  │   • No tests                             │  ║
║  │   • No documentation                     │  ║
║  │   • No one understands it                │  ║
║  │   • Single Point of Failure              │  ║
║  └─────────────────────────────────────────┘  ║
║                                                 ║
║  Status: "Working" (untouchable)               ║
║  Reality: Distributed Monolith                 ║
╚════════════════════════════════════════════════╝
```

"Sie bauten eine Microservices-Fassade," sagte Qion leise. "Um einen Monolithen, den sie nicht wagten anzufassen."

"Und?"

"Und der Monolith hatte das letzte Wort. Immer."

---

## I. Das Erwachen

Zehn Monate nach dem V3-Launch.

Das Team saß im Planning. Neue Features. Neue Hoffnung. Die Moral war gut.

Dann kam das Ticket.

**Feature Request #892:** "BPP Calculation Performance Degradation - P1"

```
Customer Report:
- Calculation requests take 15-45 seconds
- Sometimes timeout completely
- Affecting 40% of users
- Business critical

Expected: <2 seconds
Actual: 15-45 seconds (P95: timeout)
```

Arik Dane öffnete das Monitoring. Die neuen V3 Services: Alle grün. Latenz: <200ms.

BPP Calculation Service: 🔴 Red. Response Time: 23 seconds average.

"Wann ist das passiert?" fragte der Tech Lead.

"Schwer zu sagen," antwortete Oben Kell. "Wir haben kein richtiges Monitoring für BPP. Es ist... außerhalb unseres Stacks."

"Außerhalb?"

"On-Prem SQL Server. Windows Server 2012. Kein Kubernetes. Kein Grafana. Nur basic Performance Counter."

Qion Varr, der bisher geschwiegen hatte, sprach: "Wir haben es aufgeschoben. Zu lange."

"Was?"

"Die BPP-Migration. Wir haben V3 gebaut und gesagt: 'BPP migrieren wir später.' Jetzt ist später."

---

## II. Die Expedition ins Unbekannte

Der Tech Lead rief ein Emergency-Meeting ein. Titel: "BPP Legacy Assessment".

"Okay," begann er. "Wir müssen BPP modernisieren. Wie schwer kann es sein?"

Stille.

"Jemand?"

Mehr Stille.

"Hat IRGENDWER jemals den BPP-Code gesehen?"

Schließlich, Oben Kell: "Ich habe mal reingeschaut. Vor zwei Jahren. Kurz."

"Und?"

"Und ich habe wieder rausgeschaut. Schnell."

Er teilte seinen Screen. Öffnete SQL Server Management Studio. Navigierte zur BPP-Datenbank.

```
BPP_Calculation_DB
├── Stored Procedures (47)
│   ├── sp_CalculateBPP_Main (1,847 lines)
│   ├── sp_CalculateBPP_Recursive (892 lines)
│   ├── sp_GetHistoricalData (456 lines)
│   ├── sp_ApplyBusinessRules (1,234 lines)
│   ├── sp_ValidateInputs (234 lines)
│   ├── sp_TransformResults (678 lines)
│   └── ... (41 more procedures)
├── Tables (23)
├── Views (12)
├── Functions (34)
└── Triggers (8)

Total Lines of T-SQL: ~8,000
Documentation: None
Tests: None
Last Modified: 2 months ago (by whom: unknown)
Original Author: Left company in 2021
```

Arik starrte auf die Zahlen. "8,000 Zeilen Stored Procedures?"

"Ja."

"Und keine Tests?"

"Correct."

"Und keine Dokumentation?"

"Auch correct."

"Und der ursprüngliche Entwickler?"

"Arbeitet jetzt bei einem Konkurrenten. Hat einen NDA. Darf nicht helfen."

Der Tech Lead lehnte sich zurück. "Okay. Wir schauen uns die Main-Procedure an."

Oben Kell öffnete `sp_CalculateBPP_Main`. Scrollte. Und scrollte. Und scrollte.

```sql
CREATE PROCEDURE sp_CalculateBPP_Main
    @DocumentID UNIQUEIDENTIFIER,
    @CalculationType VARCHAR(50),
    @EffectiveDate DATETIME,
    @OverrideFlag BIT = 0,
    @RecursionDepth INT = 0,
    -- ... 23 weitere Parameter
AS
BEGIN
    SET NOCOUNT ON;
    
    -- Initialize temp tables (Why? Nobody knows)
    CREATE TABLE #TempCalc1 (/*...*/)
    CREATE TABLE #TempCalc2 (/*...*/)
    CREATE TABLE #TempCalc3 (/*...*/)
    CREATE TABLE #TempCalc4 (/*...*/)
    CREATE TABLE #TempCalc5 (/*...*/)
    
    -- Step 1: Validate inputs (maybe)
    IF @CalculationType = 'STANDARD'
    BEGIN
        EXEC sp_ValidateInputs @DocumentID, @CalculationType
        -- But what if it fails? Nobody checks.
    END
    ELSE IF @CalculationType = 'ADVANCED'
    BEGIN
        -- Different validation? Or no validation?
        -- Comments say "TODO: Implement" (from 2018)
    END
    
    -- Step 2: Get historical data (with a cursor. Yes, a cursor.)
    DECLARE @HistoricalValue DECIMAL(18,4)
    DECLARE history_cursor CURSOR FOR
        SELECT Value FROM HistoricalData 
        WHERE DocumentID = @DocumentID
        AND EffectiveDate <= @EffectiveDate
        -- This query scans 2.3M rows. Every time.
    
    OPEN history_cursor
    FETCH NEXT FROM history_cursor INTO @HistoricalValue
    
    WHILE @@FETCH_STATUS = 0
    BEGIN
        -- Do something with @HistoricalValue
        -- (What? It's not clear)
        INSERT INTO #TempCalc1 VALUES (/*...*/)
        FETCH NEXT FROM history_cursor INTO @HistoricalValue
    END
    
    CLOSE history_cursor
    DEALLOCATE history_cursor
    
    -- Step 3: Apply business rules (recursive)
    IF @RecursionDepth < 10 -- Magic number. Why 10?
    BEGIN
        EXEC sp_CalculateBPP_Recursive 
            @DocumentID, 
            @RecursionDepth + 1,
            -- ... 15 more parameters
    END
    
    -- Step 4: ??? (There are 1,400 more lines)
    -- Including:
    -- - Dynamic SQL generation
    -- - String concatenation for SQL injection fun
    -- - Undocumented business rules
    -- - Comments in German from 2017
    -- - A section titled "HACK - FIX LATER" (still not fixed)
    
    -- Step ?: Return results (probably)
    SELECT * FROM #TempCalc5 -- But which columns? Who knows?
    
    DROP TABLE #TempCalc1
    DROP TABLE #TempCalc2
    -- ... sometimes we forget to drop all temp tables
END
```

Die Stille im Raum war greifbar.

"Das," sagte Arik langsam, "ist nicht maintainable."

"Nein," bestätigte Oben Kell.

"Das ist nicht testable."

"Auch nein."

"Das ist nicht—"

"Es ist nichts von alledem," unterbrach Qion. "Aber es ist eines: Es funktioniert. Seit sieben Jahren. Und niemand versteht warum."

---

## III. Die unheilige Allianz

Das Team stand vor einer Entscheidung.

**Option A: Big Bang Rewrite**

```
Plan:
1. Verstehe alle 8,000 Zeilen SP Code
2. Extrahiere alle Business Rules
3. Schreibe alles neu in C#
4. Teste alles (wie?)
5. Deploy (und bete)

Time Estimate: 6-9 months
Risk: 🔥🔥🔥🔥🔥 (EXTREME)
Chance of Success: ???
```

**Option B: Strangler Fig Pattern**

```
Plan:
1. Neue BPP Service (C#, cloud-native)
2. Migriere Calculation-Types einer nach dem anderen
3. Routing-Layer entscheidet: Neu oder Alt?
4. Eventuell alles neu

Time Estimate: 12+ months
Risk: 🔥🔥🔥 (HIGH)
Complexity: 🔥🔥🔥🔥 (VERY HIGH)
```

**Option C: Leave It Alone**

```
Plan:
1. Don't touch it
2. Seriously, don't
3. Just optimize the SQL Server
4. Add monitoring
5. Hope it doesn't die

Time Estimate: 2 weeks
Risk: 🔥 (Current state continues)
Technical Debt: +9000
```

Der CTO war im Call. Er hörte die Optionen. Dann sagte er etwas, das niemand erwartet hatte:

"Option D."

"Es gibt keine Option D."

"Jetzt schon. Ihr baut eine Fassade. Einen modernen BPP-Service. In C#. Cloud-native. Microservice-ready."

"Und der Backend?"

"Der Backend bleibt. Der Stored Procedure Code. Der ruft euer neuer Service auf."

Stille.

"Das ist..." Arik suchte nach Worten.

"Das ist pragmatisch," sagte der CTO. "Ihr kriegt:

- Moderne API
- Cloud Deployment
- Monitoring
- Resilience Patterns
- Ohne die Business Logic anzufassen"

"Aber das ist eine Facade," protestierte Arik. "Wir wickeln nur ein moderneres Interface um Legacy-Code!"

"Ja," sagte der CTO. "Und?"

Qion Varr sprach, leise aber deutlich: "Das ist gefährlich. Wir täuschen uns selbst. Wir tun so, als hätten wir Microservices. Aber unter der Haube—da lebt der Monolith. Und er kann jederzeit sterben."

Der CTO nickte. "Ich weiß. Aber was ist die Alternative? Sechs Monate kein neues Feature, weil wir einen Rewrite machen, der vielleicht scheitert?"

"Wir würden Lügen," sagte Qion.

"Nein," korrigierte der CTO. "Wir würden akzeptieren, dass Modernisierung iterativ ist. Und manchmal unvollständig."

---

## IV. Die Fassade wird gebaut

Vier Wochen später.

Das Team präsentierte: **BPP Calculation Service v2.0**

```csharp
// BppCalculationService.cs - The Facade

public class BppCalculationService
{
    private readonly ISqlConnectionFactory _sqlFactory;
    private readonly ICircuitBreaker _circuitBreaker;
    private readonly ILogger _logger;
    
    public async Task<CalculationResult> CalculateAsync(
        CalculationRequest request)
    {
        _logger.LogInformation("BPP Calculation requested");
        
        return await _circuitBreaker.ExecuteAsync(async () =>
        {
            // The modern part: Validation, logging, monitoring
            ValidateRequest(request);
            
            using var connection = await _sqlFactory.CreateConnectionAsync();
            using var command = connection.CreateCommand();
            
            // The truth: We still call the stored procedure
            command.CommandType = CommandType.StoredProcedure;
            command.CommandText = "sp_CalculateBPP_Main";
            command.CommandTimeout = 30; // Prayer timeout
            
            // Map modern C# object to ancient SQL parameters
            MapRequestToParameters(command, request);
            
            // Execute and hope for the best
            using var reader = await command.ExecuteReaderAsync();
            
            // Parse the results (format still unclear)
            return ParseResults(reader);
        });
    }
    
    private void ValidateRequest(CalculationRequest request)
    {
        // Modern validation logic
        if (string.IsNullOrEmpty(request.DocumentId))
            throw new ValidationException("DocumentId required");
        
        // But we can't validate business rules
        // because they're hidden in 8,000 lines of T-SQL
    }
    
    private CalculationResult ParseResults(SqlDataReader reader)
    {
        // Try to parse whatever the SP returns
        // Format changes based on CalculationType
        // No schema documentation exists
        // This method is 400 lines of "try-catch-guess"
        
        try
        {
            // Attempt 1: Standard format
            return ParseStandardFormat(reader);
        }
        catch
        {
            try
            {
                // Attempt 2: Legacy format
                return ParseLegacyFormat(reader);
            }
            catch
            {
                // Attempt 3: Whatever this is
                _logger.LogWarning("Unknown result format, using fallback");
                return ParseFallbackFormat(reader);
            }
        }
    }
}
```

Das Deployment-Diagramm wurde aktualisiert:

```
╔════════════════════════════════════════════════╗
║           V3 SYSTEM - "MODERNIZED"             ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  [Modern Microservices]                        ║
║         ↓                                       ║
║  [BPP Calculation Service v2.0] ✨             ║
║   • C# / .NET 6                                ║
║   • Kubernetes                                  ║
║   • Circuit Breakers                            ║
║   • Monitoring                                  ║
║   • Health Checks                               ║
║         ↓                                       ║
║         ↓ (HTTP call to legacy)                ║
║         ↓                                       ║
║  [BPP Calculation Service v1.0] 💀             ║
║   • SQL Server Stored Procedures               ║
║   • On-Prem VM                                  ║
║   • 8,000 lines T-SQL                          ║
║   • No one understands it                       ║
║   • Still the source of truth                   ║
║                                                 ║
║  Status: "Modern"* (*footnote: not really)     ║
╚════════════════════════════════════════════════╝
```

Der Product Owner war begeistert: "Microservices! Cloud! Modern!"

Qion Varr war es nicht: "Lipstick. On a pig."

---

## V. Der erste Riss

Es dauerte drei Wochen.

Ein normaler Dienstag. 14:23 Uhr.

Die SQL Server VM hatte einen Memory Leak. Langsam. Unsichtbar. Der Performance Counter zeigte es nicht.

Um 14:47 Uhr: SQL Server begann zu swappen. Stored Procedures liefen langsamer. 5 Sekunden. 10 Sekunden. 20 Sekunden.

Um 14:53 Uhr: Das Kubernetes Cluster sah: "BPP Service unhealthy" (weil die SP-Calls timeouteten).

Kubernetes Reaktion: "Kill the pod. Restart."

Neuer Pod startet. Versucht Calculation. Ruft SQL Server. Timeout. Pod unhealthy. Restart.

Loop.

Um 15:03 Uhr: 47 BPP Service Pods. Alle im Restart-Loop. Alle hämmern auf den SQL Server.

SQL Server: 100% CPU. Deadlocks. Connection Pool exhausted.

Um 15:12 Uhr: **Total System Failure**

Nicht nur BPP. Alle Services, die BPP brauchten (und das waren ALLE), failten.

Die schöne Microservices-Architektur? Gestorben. Weil ein 7 Jahre alter Stored Procedure auf einer Windows Server 2012 VM ein Memory-Problem hatte.

---

## VI. Die Post-Mortem (die schmerzhafte Wahrheit)

Zwei Tage nach dem Incident.

Das Team saß im Konferenzraum. Müde. Desillusioniert.

Der CTO war auch da. Er sah älter aus als vor einer Woche.

"Root Cause?" fragte er.

Der Tech Lead antwortete: "SQL Server Memory Leak. On-Prem VM hatte nicht genug Resources."

"Das war nicht die Root Cause," unterbrach Qion.

Alle sahen ihn an.

"Die Root Cause," fuhr er fort, "war unsere Entscheidung. Vor vier Monaten. Als wir sagten: 'Wir bauen eine Fassade um Legacy, und nennen es modern.'"

Er stand auf. Ging zum Whiteboard.

```
╔════════════════════════════════════════════════╗
║              DIE WAHRHEIT                      ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Wir haben NICHT Microservices gebaut.         ║
║                                                 ║
║  Wir haben:                                    ║
║  • Eine moderne API-Schicht gebaut             ║
║  • Um einen monolithischen SP-Backend          ║
║  • Der ein Single Point of Failure ist         ║
║  • Den niemand versteht                         ║
║  • Den niemand warten kann                      ║
║  • Der jederzeit sterben kann                   ║
║                                                 ║
║  Das ist kein "Microservice mit Legacy".       ║
║  Das ist ein "Distributed Monolith".           ║
║                                                 ║
║  Der schlimmste Hybrid:                        ║
║  • Komplexität von Microservices               ║
║  • Risiko von Monolithen                        ║
║  • Benefits von keinem                          ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

"Wir haben uns selbst belogen," sagte Qion. "Wir sagten: 'Wir modernisieren iterativ.' Aber wir modernisierten nie. Wir verpackten neu."

Der CTO nickte langsam. "Was schlägst du vor?"

"Ehrlichkeit."

"Erkläre."

Qion zeichnete zwei Optionen:

**Option 1: Ehrlicher Monolith**

```
╔════════════════════════════════════════════════╗
║  ACCEPT THE TRUTH                              ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  [BPP Calculation Service]                     ║
║   - Monolithic                                  ║
║   - Stored Procedures                           ║
║   - Documented as such                          ║
║   - Treated as critical infrastructure          ║
║   - High availability setup                     ║
║   - Real monitoring                             ║
║   - Dedicated team                              ║
║                                                 ║
║  Benefits:                                     ║
║  • No false microservices                      ║
║  • Clear operational model                     ║
║  • Honest about limitations                    ║
║  • Can invest in stability                     ║
║                                                 ║
║  Downside:                                     ║
║  • Still legacy                                 ║
║  • But at least we're honest                   ║
╚════════════════════════════════════════════════╝
```

**Option 2: Echter Rewrite (aber diesmal richtig)**

```
╔════════════════════════════════════════════════╗
║  THE HARD WAY                                  ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Phase 1: Understand (3 months)                ║
║  • Reverse engineer ALLE business rules        ║
║  • Document everything                         ║
║  • Build test suite against OLD system         ║
║  • Get domain expert (hire if needed)          ║
║                                                 ║
║  Phase 2: Rewrite (6 months)                   ║
║  • One calculation type at a time              ║
║  • Parallel run: Old + New                     ║
║  • Compare results on EVERY request            ║
║  • Accept this will take time                   ║
║                                                 ║
║  Phase 3: Migrate (3 months)                   ║
║  • Strangler pattern for real                  ║
║  • Keep old system running as fallback         ║
║  • Gradual cutover                             ║
║  • Monitoring everything                        ║
║                                                 ║
║  Total: 12 months                              ║
║  Risk: Still high, but manageable              ║
║  Honesty: 100%                                 ║
╚════════════════════════════════════════════════╝
```

Der CTO las beide Optionen. Lang. Sorgfältig.

Dann sagte er: "Option 1. Sofort. Option 2. Nächstes Jahr."

"Warum nicht Option 2 jetzt?"

"Weil," der CTO lehnte sich vor, "wir erst lernen müssen, ehrlich zu sein. Mit uns selbst. Mit dem Business. Mit den Limits."

"Wir bauen keinen falschen Microservice mehr. Wir akzeptieren, dass BPP ein Monolith ist. Wir behandeln ihn als solchen. Wir investieren in seine Stabilität."

"Und DANN—wenn er stabil ist, wenn wir ihn verstehen—dann rewriten wir. Richtig."

---

## VII. Die Rückkehr zur Ehrlichkeit

Sechs Wochen später.

Das Team präsentierte das neue Setup:

```
╔════════════════════════════════════════════════╗
║      BPP CALCULATION SERVICE - HONEST          ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Architecture: MONOLITHIC                      ║
║  (We stopped lying about it)                   ║
║                                                 ║
║  Infrastructure:                               ║
║  • SQL Server Always-On Cluster (3 nodes)      ║
║  • Dedicated hardware (no sharing)             ║
║  • Memory: 128GB per node                      ║
║  • SSD storage                                  ║
║  • Load balancer                                ║
║                                                 ║
║  Monitoring:                                   ║
║  • SQL Server Performance Counters             ║
║  • Custom SP execution tracing                 ║
║  • Memory/CPU/Disk real-time alerts            ║
║  • Query plan analysis                         ║
║  • Deadlock detection                          ║
║                                                 ║
║  Operational:                                  ║
║  • Dedicated on-call rotation                  ║
║  • Runbooks for common issues                  ║
║  • Quarterly maintenance windows               ║
║  • Performance tuning schedule                 ║
║                                                 ║
║  Documentation:                                ║
║  • "We don't fully understand this"            ║
║  • "But we know how to keep it running"        ║
║  • "Rewrite planned for 2026"                  ║
║                                                 ║
║  Status: STABLE & HONEST                       ║
╚════════════════════════════════════════════════╝
```

Die C# Facade wurde entfernt. Alle Services riefen direkt das BPP SQL Server Cluster auf.

"Das sieht nicht modern aus," bemerkte jemand.

"Nein," bestätigte Qion. "Aber es ist ehrlich. Und Ehrlichkeit ist der erste Schritt zur echten Modernisierung."

---

## Epilog: Die Lektion der Geister

Ein Jahr später.

Das Team hatte begonnen, BPP wirklich zu rewriten. Langsam. Sorgfältig. Mit Tests. Mit Verständnis.

Arik Dane reflektierte: "Wir haben ein Jahr verloren."

"Nein," korrigierte Qion. "Wir haben ein Jahr gewonnen. Durch Ehrlichkeit."

"Wie meinst du das?"

"Der Facade-Ansatz war eine Illusion. Er gab uns das Gefühl von Fortschritt, ohne echten Fortschritt. Er machte uns blind für das echte Problem."

"Erst als wir akzeptierten—WIRKLICH akzeptierten—dass BPP ein Monolith ist, konnten wir ihn richtig behandeln. Stabilisieren. Verstehen. Und dann—erst dann—rewriten."

Qion zeigte ein Dokument: **"BPP Rewrite Progress - Q3"**

```
╔════════════════════════════════════════════════╗
║        BPP MODERNISIERUNG - REAL               ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Calculation Types: 12 total                   ║
║  Migrated: 3 (25%)                             ║
║  In Progress: 2                                ║
║  Remaining: 7                                   ║
║                                                 ║
║  Approach: Strangler Fig (for real this time)  ║
║  • New service written from understanding      ║
║  • Not from wishful thinking                   ║
║  • Tests compare Old vs New on every request   ║
║  • Migration only when 100% parity achieved    ║
║                                                 ║
║  Timeline: 18 months remaining                 ║
║  Confidence: HIGH                              ║
║  Why? Because we understand the problem now    ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

"Das," sagte Qion, "ist echter Fortschritt. Langsam. Aber real."

---

## Anhang: Die Warnsignale der falschen Modernisierung

🔴 **Erkenne die Facade, bevor du sie baust:**

⚠️ **"Wir wrappen Legacy in eine moderne API"**

- Wrapping ist kein Modernisieren
- Lipstick on a pig

⚠️ **"Das ist Phase 1, später rewriten wir"**

- "Später" kommt nie
- Phase 1 wird permanent

⚠️ **"Zu riskant für Big Bang, also iterativ"**

- Iterativ ist gut
- Aber "iterativ" ist nicht "nie das Kern-Problem anfassen"

⚠️ **"Der Client sieht ja nur die moderne API"**

- Single Point of Failure ändert sich nicht
- Operations Reality ändert sich nicht

⚠️ **"Wir haben keine Zeit für einen Rewrite"**

- Dann habt ihr auch keine Zeit für eine ehrliche Facade
- Die wird genauso lange dauern + technische Schuld

⚠️ **Niemand im Team versteht die Legacy-Logic**

- Das ist kein Grund zum Wrappen
- Das ist ein Grund zum Stoppen

⚠️ **"Moderne" Architektur-Diagramme vs. Deployment-Reality**

- Diagramm zeigt Microservices
- Realität ist Monolith mit API-Gateway

⚠️ **Das neue System hat dieselben Failure Modes wie das alte**

- Neues Interface ändert nichts an alter Fragilität

---

## Die Regel für Legacy-Modernisierung

```
╔════════════════════════════════════════════════╗
║                                                 ║
║        THE HONEST MODERNIZATION LAW            ║
║                                                 ║
║  Before "modernizing" legacy:                  ║
║                                                 ║
║  1. Do we UNDERSTAND the business logic?       ║
║  2. Can we TEST the current behavior?          ║
║  3. Do we have time to do it RIGHT?            ║
║  4. Are we solving the REAL problem?           ║
║  5. Or are we just painting the facade?        ║
║                                                 ║
║  If "No" to #1-4 and "Yes" to #5:             ║
║  → Don't modernize yet                         ║
║  → First: Stabilize & Understand               ║
║  → Then: Modernize for real                    ║
║                                                 ║
║  Better Options:                               ║
║  • Honest Monolith (stable, documented)        ║
║  • Real Rewrite (slow, tested, understood)     ║
║  • No Half-Measures                            ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

---

*"Die Geister der Legacy sind nicht das Problem. Das Problem ist, wenn wir tun, als wären sie nicht da. Wenn wir moderne Fassaden bauen, um alte Wahrheiten zu verstecken. Ehrlichkeit ist der erste Schritt. Nicht der letzte. Aber ohne den ersten—wird es keinen letzten geben."*

— Qion Varr, Architektenorden

---

**Nächstes Kapitel:** Kapitel 8: Die Strangler Migration  
Die Brücke vom Beschluss zur Tat: Fassade jetzt, Governance gleich – oder nie.
