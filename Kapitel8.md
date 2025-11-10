# Kapitel 8: Die Strangler Migration

## (Oder: Wie man einen Elefanten isst – Stück für Stück, bis man an den Stoßzähnen erstickt)

## Prolog: Der Feigenbaum und die alte Eiche

*„Der Strangler Fig ist ein Wunder der Natur. Er wächst um einen alten Baum herum. Langsam. Geduldig. Jahr für Jahr. Bis der alte Baum stirbt. Und der Feigenbaum steht. Allein. Siegreich. Aber manchmal vergisst man: Der Feigenbaum ist ein Parasit. Und manchmal stirbt der Wirt zu schnell. Oder zu langsam. Oder beide sterben zusammen."*

— Martin Fowler über das Strangler Fig Pattern, adaptiert von einem erschöpften Architekten

---

Der alte Architekt des Architektenordens zeigte dem Schüler zwei Bäume. Nebeneinander im Wald.

**Links:** Eine alte Eiche. Hundert Jahre alt. Stark. Tief verwurzelt. Aber krank. Die Blätter braun. Der Stamm morsch.

**Rechts:** Ein junger Feigenbaum. Rankt sich um die Eiche herum. Grün. Vital. Wächst.

„Das", sagte der Alte, „ist das Strangler‑Fig‑Pattern. Ein neues System wächst um das alte. Langsam. Funktion für Funktion. Bis das alte stirbt. Und das neue übernimmt."

„Klingt elegant", sagte der Schüler.

„Ja", sagte der Alte. „In der Theorie."

Er zeigte auf einen dritten Baum. Weiter hinten im Wald.

Dort stand... etwas.

Die alte Eiche war tot. Aber sie stand noch. Umrankt vom Feigenbaum. Der auch tot war. Beide tot. Beide noch stehend. Zusammengewachsen. Ein Skelett aus Holz und Wurzeln.

„Das", sagte der Meister, „ist das Strangler Fig Pattern. In der Praxis. Wenn beide Systeme zu lange parallel leben. Wenn niemand den Mut hat, das alte System zu töten. Wenn die Migration zur ewigen Migration wird."

---

## I. Der Neuanfang (sechs Monate nach Kapitel 7)

Das BPP Calculation System lief. Stabil. Ehrlich. Als Monolith.

Aber es war Zeit.

Der CTO rief das Team zusammen: „Ihr habt BPP stabilisiert. Gut. Jetzt rewriten wir. Richtig."

Qion Varr nickte. „Das Strangler Fig Pattern. Wie besprochen."

„Erkläre es dem Team."

Qion zeichnete auf das Whiteboard:

```
╔════════════════════════════════════════════════╗
║       STRANGLER FIG PATTERN - THE PLAN         ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  BPP hat 12 Calculation Types:                 ║
║  • STANDARD                                    ║
║  • ADVANCED                                    ║
║  • HISTORICAL                                  ║
║  • FORECAST                                    ║
║  • RECURSIVE                                   ║
║  • BATCH                                       ║
║  • ... (6 weitere)                             ║
║                                                 ║
║  Der Plan:                                     ║
║  1. Wähle EINEN Type                           ║
║  2. Verstehe ihn vollständig                   ║
║  3. Schreibe ihn neu (C#, modern, testbar)     ║
║  4. Routing: Alt oder Neu?                     ║
║  5. Schalte um, wenn 100% Parity erreicht      ║
║  6. Wiederhole für nächsten Type               ║
║                                                 ║
║  Timeline: 1-2 Monate pro Type                 ║
║  Total: 12-24 Monate                           ║
║                                                 ║
║  Risk: MANAGED (not zero, but managed)         ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

„Das", sagte Qion, „ist wie man einen Elefanten isst. Stück für Stück."

Arik Dane schaute skeptisch: „Aber 24 Monate? Zwei Jahre?"

„Ja", sagte der CTO. „Zwei Jahre ehrlicher Arbeit. Oder zehn Jahre Notlösungen. Eure Wahl."

Es war keine Wahl.

---

## II. Die erste Scheibe (Calculation Type: STANDARD)

Das Team wählte den einfachsten Type: **STANDARD**.

„Warum der einfachste?" fragte Palpatine.

„Weil", erklärte Obi‑Wan, „wir das Pattern lernen müssen. Die Tools. Den Flow. Wenn wir mit dem komplexesten beginnen, sterben wir."

Phase 1: **Verstehen**

Das Team verbrachte drei Wochen damit, die alte Stored Procedure zu verstehen.

```sql
-- sp_CalculateBPP_Standard (vereinfacht dargestellt)
CREATE PROCEDURE sp_CalculateBPP_Standard
    @DocumentID UNIQUEIDENTIFIER,
    @EffectiveDate DATETIME
AS
BEGIN
    -- 847 Zeilen Code
    -- Davon 230 Zeilen undokumentierte Business Rules
    -- Davon 47 Zeilen "HACK - TODO"
    -- Davon 12 Zeilen in Deutsch (Kommentare von 2017)
    
    -- Nach drei Wochen Reverse Engineering:
    -- Das Team verstand: 85%
    -- Das Team erriet: 10%
    -- Das Team ignorierte: 5% ("legacy quirks")
END
```

Phase 2: **Neu schreiben**

```csharp
// BppCalculationService.cs - The NEW Way

public class StandardCalculationService : ICalculationService
{
    private readonly IHistoricalDataRepository _historyRepo;
    private readonly IBusinessRuleEngine _ruleEngine;
    private readonly ILogger _logger;
    
    public async Task<CalculationResult> CalculateAsync(
        CalculationRequest request)
    {
        _logger.LogInformation(
            "Standard calculation started for {DocumentId}", 
            request.DocumentId);
        
        // Step 1: Validate (explicit, testable)
        var validation = ValidateRequest(request);
        if (!validation.IsValid)
            throw new ValidationException(validation.Errors);
        
        // Step 2: Fetch historical data (no cursor, just LINQ)
        var historicalData = await _historyRepo
            .GetHistoricalDataAsync(
                request.DocumentId, 
                request.EffectiveDate);
        
        // Step 3: Apply business rules (extracted, documented)
        var result = await _ruleEngine
            .ApplyStandardRulesAsync(historicalData, request);
        
        _logger.LogInformation(
            "Standard calculation completed for {DocumentId}: {Result}", 
            request.DocumentId, 
            result.Value);
        
        return result;
    }
    
    // 200 Zeilen statt 847
    // 100% Test Coverage
    // Dokumentiert
    // Verständlich
}
```

Das Team war stolz. Modern. Clean. Testbar.

„Jetzt", sagte Qion, „kommt der schwere Teil."

---

## III. Die Brücke zwischen zwei Welten

Das Problem: Wie entscheidet das System, ob es die alte Stored Procedure oder den neuen Service aufruft?

**Option A: Feature Flag**

```csharp
if (_featureFlags.IsEnabled("UseNewStandardCalculation"))
{
    return await _newCalculationService.CalculateAsync(request);
}
else
{
    return await _legacyStoredProcedure.ExecuteAsync(request);
}
```

„Einfach", sagte Arik.

„Zu einfach", warnte Qion. „Was ist mit Monitoring? Comparison? Rollback?"

**Option B: Routing Layer + Shadow Mode**

```csharp
public class CalculationRouter
{
    public async Task<CalculationResult> RouteAsync(CalculationRequest request)
    {
        var strategy = DetermineStrategy(request.CalculationType);
        
        switch (strategy)
        {
            case RoutingStrategy.LegacyOnly:
                return await _legacyService.CalculateAsync(request);
            
            case RoutingStrategy.NewOnly:
                return await _newService.CalculateAsync(request);
            
            case RoutingStrategy.ShadowMode:
                // Call BOTH, compare, return legacy (for now)
                var legacyResult = await _legacyService.CalculateAsync(request);
                var newResult = await _newService.CalculateAsync(request);
                
                await _comparisonService.CompareAndLogAsync(
                    legacyResult, 
                    newResult, 
                    request);
                
                // Return legacy (safe)
                return legacyResult;
            
            case RoutingStrategy.NewWithFallback:
                try
                {
                    return await _newService.CalculateAsync(request);
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "New service failed, falling back");
                    return await _legacyService.CalculateAsync(request);
                }
        }
    }
}
```

„Das ist komplexer", sagte Arik.

„Ja", bestätigte Qion. „Aber sicherer. Und das brauchen wir für 12 Calculation Types."

Das Team baute die Routing Layer.

Es dauerte zwei Wochen. Länger als der neue Service selbst.

*Das erste Warnsignal. Niemand beachtete es.*

---

## IV. Die Proliferation beginnt

Zwei Monate später. STANDARD Type lief im neuen Service. Erfolgreich.

Das Team begann mit Type #2: **ADVANCED**.

Aber ADVANCED war... komplizierter.

„ADVANCED", erklärte Obi‑Wan, „ruft STANDARD auf. Intern. In der alten SP."

„Okay", sagte Arik. „Dann rufen wir den neuen STANDARD Service auf."

„Aber wie erfährt der neue ADVANCED Service, dass STANDARD fertig ist?"

Pause.

„Wir... könnten es hard‑coden?"

„Nein", sagte Qion. „Was wenn wir STANDARD zurückrollen müssen? Oder was wenn ein anderer Type auch STANDARD braucht?"

Das Team diskutierte. Stunden.

Dann hatte jemand eine Idee: **„Events."**

```csharp
// Der neue ADVANCED Service

public class AdvancedCalculationService
{
    private readonly IEventBus _eventBus;
    
    public async Task<CalculationResult> CalculateAsync(
        CalculationRequest request)
    {
        // Publish event: "I need a STANDARD calculation"
        var standardRequest = new StandardCalculationRequested
        {
            DocumentId = request.DocumentId,
            EffectiveDate = request.EffectiveDate,
            RequestId = Guid.NewGuid()
        };
        
        await _eventBus.PublishAsync(standardRequest);
        
        // Wait for response event
        var standardResult = await _eventBus
            .WaitForAsync<StandardCalculationCompleted>(
                e => e.RequestId == standardRequest.RequestId,
                timeout: TimeSpan.FromSeconds(30));
        
        // Use the result
        return ApplyAdvancedLogic(standardResult.Data);
    }
}
```

„Das ist elegant", sagte Arik. „Entkoppelt. Service weiß nicht, ob STANDARD alt oder neu ist."

Qion nickte. „Ja. Aber..."

„Aber was?"

„Nichts. Macht es."

*Das zweite Warnsignal. Wieder ignoriert.*

---

## V. Month 6: Die Event‑Landschaft

Sechs Monate in die Strangler Migration.

Das Whiteboard im Team‑Raum zeigte den Fortschritt:

```
╔════════════════════════════════════════════════╗
║     BPP STRANGLER MIGRATION - STATUS           ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Calculation Types:                            ║
║  ✅ STANDARD (New Service, 100%)               ║
║  ✅ ADVANCED (New Service, 100%)               ║
║  ✅ HISTORICAL (New Service, 85%)              ║
║  🔄 FORECAST (In Progress, 40%)                ║
║  ⏳ RECURSIVE (Not Started)                    ║
║  ⏳ BATCH (Not Started)                        ║
║  ⏳ ... (6 more types)                         ║
║                                                 ║
║  Progress: 3.5 / 12 Types (29%)                ║
║  Timeline: On Track                            ║
║  Status: GOOD... ?                             ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Aber ein anderes Whiteboard – kleiner, in der Ecke – zeigte etwas anderes:

```
╔════════════════════════════════════════════════╗
║            EVENT LANDSCAPE                     ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Events created (last 6 months):               ║
║                                                 ║
║  StandardCalculationRequested                  ║
║  StandardCalculationCompleted                  ║
║  StandardCalculationFailed                     ║
║  AdvancedCalculationRequested                  ║
║  AdvancedCalculationCompleted                  ║
║  AdvancedCalculationFailed                     ║
║  HistoricalDataRequested                       ║
║  HistoricalDataRetrieved                       ║
║  HistoricalDataNotFound                        ║
║  ForecastCalculationRequested                  ║
║  ForecastCalculationStarted                    ║
║  ForecastCalculationInProgress                 ║
║  ForecastCalculationCompleted                  ║
║  ValidationRequested                           ║
║  ValidationCompleted                           ║
║  ValidationFailed                              ║
║  BusinessRuleApplied                           ║
║  BusinessRuleSkipped                           ║
║  RoutingDecisionMade                           ║
║  LegacyServiceCalled                           ║
║  NewServiceCalled                              ║
║  ComparisonResultAvailable                     ║
║  FallbackTriggered                             ║
║  ... (23 weitere Events)                       ║
║                                                 ║
║  Total: 47 Event Types                         ║
║  Average per Calculation Type: 13 Events       ║
║  Projected total (12 types): ~150 Events       ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Obi‑Wan starrte auf die Zahlen. „47 Events. In sechs Monaten. Für 3,5 Calculation Types."

„Ja", sagte Palpatine. „Aber das ist okay, oder? Events sind gut. Entkopplung."

Qion schwieg. Er rechnete.

12 Types × 13 Events pro Type = 156 Events.

Plus Infrastruktur‑Events.  
Plus Error‑Handling‑Events.  
Plus Monitoring‑Events.

*200+ Events. Für ein Calculation System.*

„Das ist zu viel", sagte er leise.

„Was?"

„Nichts. Weitermachen."

---

## VI. Month 9: Die Flut

FORECAST Type ging live. Der komplexeste bisher.

Er brauchte STANDARD, ADVANCED, und HISTORICAL. Alle drei.

Die Event‑Kette:

```
ForecastCalculationRequested
  └─▶ StandardCalculationRequested
       └─▶ HistoricalDataRequested
            └─▶ HistoricalDataRetrieved
       └─▶ StandardCalculationCompleted
  └─▶ AdvancedCalculationRequested
       └─▶ StandardCalculationRequested (wieder!)
            └─▶ HistoricalDataRequested (wieder!)
                 └─▶ HistoricalDataRetrieved (wieder!)
            └─▶ StandardCalculationCompleted
       └─▶ AdvancedCalculationCompleted
  └─▶ HistoricalCalculationRequested
       └─▶ HistoricalDataRequested (zum dritten Mal!)
            └─▶ HistoricalDataRetrieved
       └─▶ HistoricalCalculationCompleted
  └─▶ ForecastCalculationCompleted

Total Events pro FORECAST Request: 14
Total Latency: 2.3 seconds (war: 0.4s in legacy SP)
```

„Warum ist FORECAST so langsam?" fragte der Product Owner.

„Die Events", antwortete Arik. „Jeder Event braucht Zeit. Serialization. Message Bus. Deserialization."

„Aber die alte SP war schneller?"

„Ja. Weil alles in einem Prozess lief. In‑Memory."

„Also", der Product Owner lehnte sich zurück, „wir ersetzen ein schnelles Legacy‑System mit einem langsamen modernen System?"

Stille.

„Das ist temporär", verteidigte sich der Tech Lead. „Wir optimieren."

„Wann?"

„Nach der Migration."

„Ihr seid bei Type 4 von 12. Das dauert noch ein Jahr."

„Ja."

„Ein Jahr mit einem langsameren System?"

„...ja."

Der Product Owner stand auf. „Fix it."

---

## VII. Die Optimierungsspirale

Das Team versuchte, FORECAST zu beschleunigen.

**Versuch 1: Event‑Batching**

```csharp
// Statt mehrere Events sequenziell:
var standardResult = await GetStandardAsync();
var advancedResult = await GetAdvancedAsync();
var historicalResult = await GetHistoricalAsync();

// Alle parallel:
var tasks = new[]
{
    GetStandardAsync(),
    GetAdvancedAsync(),
    GetHistoricalAsync()
};
var results = await Task.WhenAll(tasks);
```

Resultat: Latency von 2.3s → 1.1s.

Besser. Aber immer noch langsamer als Legacy (0.4s).

**Versuch 2: Event‑Caching**

```csharp
// Cache häufige Requests
var cacheKey = $"STANDARD_{documentId}_{effectiveDate}";
var cached = await _cache.GetAsync(cacheKey);
if (cached != null)
    return cached;

var result = await CalculateAsync(request);
await _cache.SetAsync(cacheKey, result, TimeSpan.FromMinutes(5));
return result;
```

Resultat: Latency für gecachte Requests: 0.05s. Für neue: 1.1s.

„Gut", sagte der Product Owner. „Aber was ist die Cache‑Hit‑Rate?"

„12%."

„Das heißt 88% der Requests sind langsamer als Legacy?"

„...ja. Aber wir arbeiten daran."

**Versuch 3: Direct Service Calls**

„Moment", sagte Arik. „Warum nutzen wir Events für interne Kommunikation? FORECAST ist im selben Service wie STANDARD. Wir könnten direkt aufrufen."

```csharp
// Direct call statt Event
var standardResult = await _standardCalculationService
    .CalculateAsync(request);
```

Resultat: Latency: 0.3s.

„Schneller als Legacy!" jubelte jemand.

„Aber", unterbrach Qion, „jetzt ist FORECAST direkt gekoppelt an STANDARD. Wenn wir STANDARD ändern, müssen wir FORECAST neu deployen."

„Ist das schlimm?"

„Es widerspricht der ganzen Event‑Architektur."

„Aber es ist schnell."

Qion seufzte. „Macht es. Fürs Erste."

---

## VIII. Month 12: Die Retrospektive der Schmerzen

Ein Jahr in die Migration.

Status:

```
╔════════════════════════════════════════════════╗
║        BPP STRANGLER - YEAR 1 REVIEW           ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Types Migrated: 5 / 12 (42%)                  ║
║  Timeline: Behind (planned: 50%)               ║
║                                                 ║
║  Technical Debt Accumulated:                   ║
║  • 89 Event Types created                      ║
║  • 34 Event Handlers                           ║
║  • 12 Direct Service Calls (bypassing events)  ║
║  • 7 Cache Layers                              ║
║  • 3 Different Event Patterns                  ║
║  • 2 Message Bus Implementations               ║
║                                                 ║
║  Complexity: HIGH                              ║
║  Maintainability: DECLINING                    ║
║  Performance: MIXED                            ║
║                                                 ║
║  Team Sentiment: EXHAUSTED                     ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Das Team saß im Retrospective. Niemand sprach.

Schließlich, Arik: „Wir haben ein Problem."

„Was?" fragte der Tech Lead.

„Wir haben das Legacy‑System nicht ersetzt. Wir haben ein zweites Legacy‑System gebaut. Daneben."

„Das ist nicht fair—"

„Doch. Schau dir den Code an. Wir haben:

- Eine Routing Layer, die niemand versteht.
- 89 Events, ohne klare Ownership.
- Services, die manchmal Events nutzen, manchmal nicht.
- Caching auf 7 verschiedene Arten.
- Performance‑Probleme, die wir mit Workarounds fixen."

Er zeigte auf das Whiteboard:

```
╔════════════════════════════════════════════════╗
║              DIE WAHRHEIT                      ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Wir haben nicht ein System ersetzt.           ║
║  Wir haben zwei Systeme parallel.              ║
║                                                 ║
║  System 1: Legacy SP (7 Types, 47% Traffic)    ║
║  System 2: New Services (5 Types, 53% Traffic) ║
║                                                 ║
║  Beide müssen gewartet werden.                 ║
║  Beide haben Bugs.                             ║
║  Beide haben Incidents.                        ║
║                                                 ║
║  Und dazwischen:                               ║
║  • Die Routing Layer                           ║
║  • Die Events                                  ║
║  • Die Comparison Logic                        ║
║  • Die Fallbacks                               ║
║                                                 ║
║  Complexity: 3× das Original                   ║
║  Team Capacity: Fully consumed                 ║
║  End in Sight: Not really                      ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Qion Varr nickte langsam. „Das Strangler Fig Pattern hat ein Geheimnis. Das niemand laut sagt."

„Was?"

„Es funktioniert nur, wenn die Strangler‑Phase kurz ist. Sehr kurz. Drei Monate. Maximal sechs."

„Wir sind bei zwölf."

„Ja. Und wir haben noch zwölf vor uns. Mindestens."

„Was schlägst du vor?"

Qion zeichnete zwei Optionen:

**Option A: Abort Mission**

```
╔════════════════════════════════════════════════╗
║  STOP THE STRANGLER                            ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  1. Freeze migration                           ║
║  2. Stabilize what we have (5 types new)       ║
║  3. Accept: 7 types stay in Legacy forever     ║
║  4. Clean up: Remove routing, remove events    ║
║  5. Optimize: Make both systems work well      ║
║                                                 ║
║  Result: Hybrid system (documented, stable)    ║
║  Honesty: 100%                                 ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

**Option B: Aggressive Push**

```
╔════════════════════════════════════════════════╗
║  FINISH THE MIGRATION - FAST                   ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  1. Set HARD deadline: 6 months                ║
║  2. Accept: Quick & dirty                      ║
║  3. No optimization during migration           ║
║  4. No new events                              ║
║  5. Copy logic, don't perfect it               ║
║  6. KILL legacy after 6 months, no exceptions  ║
║                                                 ║
║  Result: Complete migration (messy but done)   ║
║  Refactor: After legacy is dead                ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

„Welche wählst du?" fragte der CTO, der plötzlich im Call war.

Qion: „Option B. Aber nur, wenn ihr mir den Mut gebt, schnell und dreckig zu sein. Und die Erlaubnis, Legacy zu töten. Gnadenlos."

Der CTO nickte. „Ihr habt sechs Monate. Dann stirbt Legacy. Ob die Migration fertig ist oder nicht."

---

## IX. Die letzten sechs Monate (Schnelldurchlauf)

Was folgte, war die intensivste Zeit der Migration.

**Month 13-14:** RECURSIVE und BATCH migriert. Keine Events. Direkte Calls. Dirty, aber schnell.

**Month 15:** COMPLEX migriert. Code kopiert statt verstanden. Tests: Minimal. „Funktioniert" = gut genug.

**Month 16-17:** Die letzten vier Types. Parallel gebaut. Code‑Quality: Gesunken. Velocity: Gestiegen.

**Month 18, Tag 1:** Letzter Type migriert.

**Month 18, Tag 2:** Legacy SP wurde read‑only gesetzt.

**Month 18, Tag 7:** Legacy SP wurde gelöscht.

8,000 Zeilen T‑SQL. Sieben Jahre Wissen. Verschwunden.

Das Team sah zu. Niemand jubelte. Alle waren zu erschöpft.

„Es ist vorbei", sagte Obi‑Wan.

„Nein", korrigierte Qion. „Es fängt erst an."

„Was?"

„Der Cleanup. Wir haben 89 Events. 34 Handler. 7 Cache‑Layer. Eine Routing Layer, die wir nicht mehr brauchen. Das ist der neue Legacy."

„Wie lange?"

„Für den Cleanup? Sechs Monate."

„Also zwei Jahre total. Für einen Rewrite."

„Ja."

---

## Epilog: Die Lektion des Feigenbaums

Zwei Jahre und einen Monat nach Start.

Das BPP System lief. Komplett neu. Modern. Testbar. 

Aber der Weg war nicht elegant gewesen.

Qion Varr schrieb ein Dokument. Titel: **„Strangler Fig: Lessons from the Trenches"**

```
╔════════════════════════════════════════════════╗
║                                                 ║
║       THE STRANGLER FIG REALITY                ║
║                                                 ║
║  Strangler Fig Pattern ist nicht sanft.        ║
║  Es ist nicht elegant.                         ║
║  Es ist ein Krieg. Auf zwei Fronten.          ║
║                                                 ║
║  What the books say:                           ║
║  "Gradually replace functionality"             ║
║  "One piece at a time"                         ║
║  "Safe and incremental"                        ║
║                                                 ║
║  What really happens:                          ║
║  • Two systems to maintain                     ║
║  • Exponential complexity                      ║
║  • Events everywhere                           ║
║  • Routing hell                                ║
║  • Performance degradation                     ║
║  • Team exhaustion                             ║
║                                                 ║
║  Strangler Fig works IF:                       ║
║  1. The strangler phase is SHORT (3-6 months)  ║
║  2. You have the discipline to kill legacy     ║
║  3. You accept: Messy is okay during migration ║
║  4. You resist: Over-engineering the bridge    ║
║                                                 ║
║  Strangler Fig FAILS if:                       ║
║  1. Migration takes > 12 months                ║
║  2. You perfect each piece before moving on    ║
║  3. You build complex infrastructure (events!) ║
║  4. You lose the will to kill legacy           ║
║                                                 ║
║  Our mistakes:                                 ║
║  • We built 89 events (needed: ~20)            ║
║  • We took 18 months (should: 9 months)        ║
║  • We optimized during migration (should: after)║
║  • We treated the bridge as permanent (it's not)║
║                                                 ║
║  What we learned:                              ║
║  • Fast and dirty beats slow and perfect       ║
║  • Parallel systems are hell (minimize time)   ║
║  • Events can become their own legacy          ║
║  • The courage to kill legacy is everything    ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Er schickte es an das Team. Und ans Management.

Der CTO antwortete: „Publiziert das. Andere sollen lernen, was ihr gelernt habt."

---

## Anhang: Die Warnsignale der endlosen Strangler‑Migration

🔴 **Erkenne die Falle, bevor du reinfällst:**

⚠️ **„Wir migrieren Stück für Stück, ganz sanft"**
- Sanft = langsam
- Langsam = zwei Systeme lange parallel
- Parallel = Komplexität × Leiden

⚠️ **Event‑Proliferation während Migration**
- Jedes neue Piece braucht neue Events
- Events als Kommunikation zwischen alt und neu
- Plötzlich: Event‑Chaos

⚠️ **„Wir perfektionieren jeden migrierten Teil"**
- Perfektionismus verlängert Migration
- Je länger Migration, desto mehr leiden
- Dirty code ist okay während Migration

⚠️ **Keine harte Deadline für Legacy‑Tod**
- „Wir migrieren, bis fertig" = nie fertig
- Legacy wird nicht sterben ohne Execution Date

⚠️ **Routing‑Layer wird komplexer und komplexer**
- Feature Flags
- Shadow Mode
- Fallbacks
- Comparison Logic
- A/B Testing
- → Die Routing‑Layer wird selbst zum Legacy

⚠️ **Performance‑Regression wird akzeptiert**
- „Das neue System ist langsamer, aber moderner"
- User merken das
- Business merkt das

⚠️ **Team ist erschöpfter, nicht energiegeladener**
- Strangler sollte motivieren
- Wenn Team erschöpft ist nach 6 Monaten: Abort

⚠️ **„Nur noch ein paar Pieces"**
- Die letzten 20% dauern 80% der Zeit
- Zeno's Paradox der Migration

---

## Die Regel für Strangler‑Migrationen

```
╔════════════════════════════════════════════════╗
║                                                 ║
║        THE STRANGLER FIG LAW                   ║
║                                                 ║
║  If you use Strangler Fig Pattern:             ║
║                                                 ║
║  1. SET HARD DEADLINE (6-9 months max)         ║
║  2. MINIMIZE BRIDGE COMPLEXITY                 ║
║     - Simple routing, no fancy infrastructure  ║
║     - Avoid events if possible                 ║
║     - Accept: Direct calls during migration    ║
║  3. FAST AND DIRTY during migration            ║
║     - Perfection comes AFTER legacy is dead    ║
║  4. KILL LEGACY at deadline (no exceptions)    ║
║     - Even if migration isn't perfect          ║
║  5. CLEANUP AFTER (6 months for refactoring)   ║
║                                                 ║
║  If migration takes > 12 months:               ║
║  → You failed                                  ║
║  → Abort or go aggressive                      ║
║                                                 ║
║  Remember:                                     ║
║  Strangler Fig is not gentle.                  ║
║  It's a race against complexity.               ║
║  Win fast. Or lose slowly.                     ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

---

*„Der Feigenbaum ist kein Freund des alten Baums. Er ist sein Mörder. Langsam, ja. Aber gnadenlos. Das Problem ist, wenn der Mord zu lange dauert. Dann sterben beide. Und du stehst in einem Wald aus Skeletten, die einmal Hoffnung waren."*

— Qion Varr, Überlebender der Strangler‑Migration

---

**Nächstes Kapitel:** „Die zwei Welten" – Wenn V2 Legacy und V3 Modern parallel laufen, und das Team die Lektion nochmal lernen muss, auf größerem Scale.

