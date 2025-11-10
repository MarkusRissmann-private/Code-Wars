# Kapitel 8: Die Strangler Migration  

## (Oder: Wie man einen Elefanten isst – Stück für Stück, bis man an den Stoßzähnen erstickt)

## Prolog: Der Feigenbaum und die alte Eiche

*„Der Strangler Fig ist ein Wunder der Natur. Er wächst um einen alten Baum herum. Langsam. Geduldig. Jahr für Jahr. Bis der alte Baum stirbt. Und der Feigenbaum steht. Allein. Siegreich. Aber manchmal vergisst man: Der Feigenbaum ist ein Parasit. Und manchmal stirbt der Wirt zu schnell. Oder zu langsam. Oder beide sterben zusammen.“*

– Martin Fowler über das Strangler Fig Pattern, adaptiert von einem erschöpften Architekten

---

Der alte Architekt des Architektenordens zeigte dem Schüler zwei Bäume. Nebeneinander im Wald.

**Links:** Eine alte Eiche. Hundert Jahre alt. Stark. Tief verwurzelt. Aber krank. Die Blätter braun. Der Stamm morsch.

**Rechts:** Ein junger Feigenbaum. Rankt sich um die Eiche herum. Grün. Vital. Wächst.

„Das“, sagte der Alte, „ist das Strangler‑Fig‑Pattern. Ein neues System wächst um das alte. Langsam. Funktion für Funktion. Bis das alte stirbt. Und das neue übernimmt.“

„Klingt elegant“, sagte der Schüler.

„Ja“, sagte der Alte. „In der Theorie.“

Er zeigte auf einen dritten Baum. Weiter hinten im Wald.

Dort stand... etwas.

Die alte Eiche war tot. Aber sie stand noch. Umrankt vom Feigenbaum. Der auch tot war. Beide tot. Beide noch stehend. Zusammengewachsen. Ein Skelett aus Holz und Wurzeln.

"Das," sagte der Meister, "ist das Strangler Fig Pattern. In der Praxis. Wenn beide Systeme zu lange parallel leben. Wenn niemand den Mut hat, das alte System zu töten. Wenn die Migration zur ewigen Migration wird."

---

## I. Die Facade wird gebaut

Vier Wochen nach dem CTO-Meeting.

Das Team hatte eine Entscheidung getroffen: **Option D - Die Pragmatische Facade.**

Nicht ein vollständiger Rewrite. Nicht das Belassen des Legacy-Systems. Sondern eine moderne Fassade um die alten Stored Procedures.

Der Plan war einfach:

```text
╔════════════════════════════════════════════════╗
║     BPP CALCULATION SERVICE v2.0               ║
║           (The Facade Pattern)                 ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  [Modern C# Microservice]                      ║
║   ├─ REST API (OpenAPI documented)             ║
║   ├─ Health Checks & Readiness Probes          ║
║   ├─ Circuit Breaker Pattern                   ║
║   ├─ Structured Logging                        ║
║   ├─ Distributed Tracing                       ║
║   └─ Kubernetes-ready Deployment               ║
║                                                 ║
║   │                                            ║
║   └─→ Anti-Corruption Layer                    ║
║          │                                     ║
║          └─→ [Old SQL Stored Procedures]       ║
║               ├─ sp_CalculateBPP_Main          ║
║               ├─ sp_GetCalculationDetails      ║
║               ├─ sp_ImportCalculation          ║
║               └─ ... (87 more)                 ║
║                                                 ║
║  BENEFIT: Modern on the outside                ║
║  REALITY: Legacy on the inside                 ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Arik Dane präsentierte stolz das erste Deployment:

```csharp
// BppCalculationService.cs - The Facade

public class BppCalculationService
{
    private readonly ISqlConnectionFactory _sqlFactory;
    private readonly ICircuitBreaker _circuitBreaker;
    private readonly ILogger<BppCalculationService> _logger;
    
    public async Task<CalculationResult> CalculateAsync(
        CalculationRequest request)
    {
        _logger.LogInformation(
            "BPP Calculation requested for Tenant {TenantId}", 
            request.TenantId);
        
        return await _circuitBreaker.ExecuteAsync(async () =>
        {
            // The modern part: Validation, logging, monitoring
            ValidateRequest(request);
            
            using var connection = await _sqlFactory
                .CreateConnectionAsync();
            using var command = connection.CreateCommand();
            
            // The truth: We still call the stored procedure
            command.CommandType = CommandType.StoredProcedure;
            command.CommandText = "sp_CalculateBPP_Main";
            command.CommandTimeout = 30; // Prayer timeout
            
            // Map modern C# object to ancient SQL parameters
            MapRequestToParameters(command, request);
            
            // Execute and hope for the best
            using var reader = await command.ExecuteReaderAsync();
            return MapResultToObject(reader);
        });
    }
    
    private void ValidateRequest(CalculationRequest request)
    {
        if (request.TenantId <= 0)
            throw new ValidationException("Invalid TenantId");
        
        if (request.CalculationType == null)
            throw new ValidationException("CalculationType required");
            
        // Modern validation wrapping ancient chaos
    }
}
```

"Das," sagte Arik, "ist Phase 1. Eine moderne API. Monitoring. Circuit Breaker. Kubernetes. Alles, was wir brauchen."

Qion Varr starrte auf den Code.

"Und die Stored Procedures?"

"Bleiben," sagte Arik. "Vorerst. Aber wir haben einen Plan."

---

## II. Der Strangler-Plan

Das Team präsentierte **The Strangler Migration Roadmap:**

```text
╔════════════════════════════════════════════════╗
║     STRANGLER MIGRATION PLAN (12 Monate)       ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  PHASE 1 (Monat 1-2): The Facade               ║
║  ✓ Moderne API-Schicht                         ║
║  ✓ Alle Calls gehen durch C# Service           ║
║  ✓ Aber: SPs dahinter                          ║
║                                                 ║
║  PHASE 2 (Monat 3-5): First Strangling         ║
║  → Migriere einfache Calculation-Types         ║
║  → 3-5 von 23 Types komplett in C#             ║
║  → Routing-Layer: Feature Flag pro Type        ║
║                                                 ║
║  PHASE 3 (Monat 6-9): Middle Strangling        ║
║  → Migriere mittlere Calculation-Types         ║
║  → 8-10 weitere Types in C#                    ║
║  → 50% Traffic auf neuem Code                  ║
║                                                 ║
║  PHASE 4 (Monat 10-12): Final Strangling       ║
║  → Migriere komplexe Types                     ║
║  → Alle Types in C#                            ║
║  → Delete Stored Procedures                    ║
║  → Victory                                     ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Oben Kell studierte den Plan. „Das sieht solide aus. Graduell. Kontrolliert. Low‑risk.“

"Ja," sagte Qion leise. "Zu solide."

"Was meinst du?"

"Zwölf Monate," sagte Qion. "Zwölf Monate in denen wir ZWEI Systeme betreiben. Das alte SP-System. Das neue C#-System. Und einen Routing-Layer dazwischen."

"Das ist doch der Punkt von Strangler," sagte Arik. "Graduell. Sicher. Kein Big Bang."

"Ja," sagte Qion. "Aber graduell über zwölf Monate bedeutet: Zwölf Monate parallele Entwicklung. Zwölf Monate doppelte Bugs. Zwölf Monate Feature-Entscheidungen: 'Alt oder Neu?'"

Er zeichnete auf das Whiteboard:

```text
STRANGLER PATTERN - THE HIDDEN COSTS:

Month 1-3:
├─ New System: 20% functionality
├─ Old System: 100% functionality
└─ Bug in Old? Fix in Old.

Month 4-6:
├─ New System: 40% functionality
├─ Old System: 100% functionality
└─ Bug in Shared Logic? Fix in BOTH.

Month 7-9:
├─ New System: 60% functionality
├─ Old System: 100% functionality
└─ New Feature? Where? OLD or NEW?

Month 10-12:
├─ New System: 90% functionality
├─ Old System: Still needed for 10%
├─ Bug? Which version?
└─ Deploy? Both versions?

Month 13+:
├─ "Just 5% left in old system"
├─ "We'll migrate it next quarter"
└─ Narrator: They didn't.
```

"Das," sagte Qion, "ist die Strangler-Falle. Ihr startet mit einem System. Ihr endet mit anderthalb Systemen. Für immer."

---

## III. Phase 1: Die ersten Erfolge

Aber das Team war optimistisch.

**Woche 4:** Der erste Calculation-Type wurde migriert.

```csharp
// SimplePercentageCalculation.cs
// Previously: 180 lines of T-SQL
// Now: 45 lines of clean C#

public class SimplePercentageCalculation : ICalculationType
{
    public CalculationResult Calculate(CalculationInput input)
    {
        // Validate
        if (input.BaseAmount <= 0)
            throw new ArgumentException("BaseAmount must be positive");
        
        // Calculate
        var percentage = input.PercentageRate / 100m;
        var result = input.BaseAmount * percentage;
        
        // Round according to German VAT rules
        result = Math.Round(result, 2, MidpointRounding.AwayFromZero);
        
        return new CalculationResult 
        {
            Result = result,
            CalculationType = "SimplePercentage",
            CalculationDate = DateTime.UtcNow
        };
    }
}
```

Das Team feierte.

"Seht ihr?" sagte Arik. "Das ist sauber. Testbar. Verständlich. Das ist, was wir wollen."

Qion musste zustimmen. Der Code war gut.

Aber dann kam die nächste Frage.

---

## IV. Die Routing-Hölle

"Wie entscheiden wir, welcher Code ausgeführt wird? Alt oder Neu?"

Das Team baute einen **Routing-Layer:**

```csharp
public class CalculationRouter
{
    private readonly IFeatureFlagService _featureFlags;
    private readonly ICalculationServiceOld _oldService;
    private readonly ICalculationServiceNew _newService;
    
    public async Task<CalculationResult> RouteCalculation(
        CalculationRequest request)
    {
        // Check feature flag for this calculation type
        var useNewImplementation = await _featureFlags
            .IsEnabledAsync(
                $"Calculation.{request.CalculationType}.UseNew", 
                request.TenantId);
        
        if (useNewImplementation)
        {
            _logger.LogInformation(
                "Routing to NEW implementation for {Type}", 
                request.CalculationType);
            return await _newService.CalculateAsync(request);
        }
        else
        {
            _logger.LogInformation(
                "Routing to OLD implementation for {Type}", 
                request.CalculationType);
            return await _oldService.CalculateAsync(request);
        }
    }
}
```

„Elegant“, sagte Oben Kell. „Feature Flags. Pro Calculation‑Type. Pro Tenant sogar.“

"Ja," sagte Qion. "Und jetzt multipliziere das mit 23 Calculation-Types. Mit 500 Tenants. Mit 10 Entwicklern, die alle fragen: 'Ist das Flag aktiv für meinen Test?'"

Er schrieb auf das Whiteboard:

```text
FEATURE FLAG MATRIX:

23 Calculation Types
× 500 Tenants
× 2 Implementations (Old, New)
= 23,000 potential configurations

Questions every day:
├─ "Which tenants are on new code?"
├─ "Can I test against old or new?"
├─ "Bug in prod – which implementation?"
├─ "Deploy new version – update flags?"
└─ "What's the rollback plan?"

Cognitive Load: EXTREME
```

---

## V. Der erste Bug

**Woche 7.**

Production Alert:

```text
🔥 CRITICAL: Calculation mismatch detected
Tenant: 471
Calculation Type: SimplePercentage
Old Result: 1,234.56
New Result: 1,234.57
Difference: €0.01
```

Ein Cent Unterschied.

Das Team debuggte.

**Die Ursache:** Rounding.

Die alte Stored Procedure:

```sql
-- Old SP: Banker's Rounding (Round-to-Even)
SELECT ROUND(@Result, 2)
```

Der neue C#-Code:

```csharp
// New C#: Away-from-Zero Rounding
Math.Round(result, 2, MidpointRounding.AwayFromZero)
```

Bei `1234.565` produziert das:

- **Banker's Rounding:** `1234.56` (abrunden bei gerader Ziffer)
- **Away-from-Zero:** `1234.57` (immer aufrunden bei .5)

"Ein Cent," sagte der Product Owner. "Das ist doch nicht so schlimm?"

"Ein Cent mal 500 Tenants mal 10,000 Calculations pro Tag," rechnete Qion. "Das sind 50,000 Cent. 500 Euro. Pro Tag. Fehler. In Buchhaltungs-Software."

Stille.

"Fix it," sagte der CTO.

Das Team fixte. Passte den neuen Code an. **Backward-compatible zu falschem Rounding-Verhalten.**

```csharp
// New C#: Match old behavior exactly
// Even if technically wrong
Math.Round(result, 2, MidpointRounding.ToEven)
```

"Das," murmelte Qion, "ist der Preis der Migration. Wir migrieren nicht nur Code. Wir migrieren Bugs."

---

## VI. Events erscheinen

**Woche 9.**

Das Team brauchte einen Weg, verschiedene Services zu informieren, wenn eine Calculation fertig war.

Die alte Lösung: Polling. Jeder Service fragt alle 30 Sekunden: "Ist meine Calculation fertig?"

Die neue Lösung: **Events.**

```csharp
public class CalculationCompletedEvent
{
    public Guid CalculationId { get; set; }
    public int TenantId { get; set; }
    public string CalculationType { get; set; }
    public decimal Result { get; set; }
    public DateTime CompletedAt { get; set; }
}
```

Sora Nyra implementierte:

```csharp
public async Task CompleteCalculation(Guid calculationId)
{
    var calculation = await _repo.GetAsync(calculationId);
    
    // Persist to database
    await _repo.SaveAsync(calculation);
    
    // Publish event
    await _eventBus.PublishAsync(new CalculationCompletedEvent
    {
        CalculationId = calculation.Id,
        TenantId = calculation.TenantId,
        CalculationType = calculation.Type,
        Result = calculation.Result,
        CompletedAt = DateTime.UtcNow
    });
    
    _logger.LogInformation(
        "Calculation {Id} completed and event published", 
        calculationId);
}
```

„Das ist gut“, sagte Oben Kell. „Decoupling. Event‑driven. Modern.“

Und es war gut.

**Bis andere Services dasselbe taten.**

---

## VII. Die Event-Kultur entsteht

**Woche 11.**

Das Import-Service brauchte auch Events.

```csharp
// Events vom Import Service:
DocumentReceivedEvent
ValidationStartedEvent
ValidationCompletedEvent
TransformationStartedEvent
TransformationCompletedEvent
ImportStartedEvent
ImportCompletedEvent
```

Das Project-Service auch:

```csharp
// Events vom Project Service:
ProjectCreatedEvent
ProjectUpdatedEvent
ProjectArchivedEvent
ProjectMemberAddedEvent
ProjectMemberRemovedEvent
```

Das Calculation-Service erweiterte seine Events:

```csharp
// Calculation Service Events (erweitert):
CalculationStartedEvent          // ← neu
CalculationValidatedEvent        // ← neu
CalculationTypeRoutedEvent       // ← neu (für Monitoring)
CalculationProgressEvent         // ← neu (für UI)
CalculationCompletedEvent        // ← original
CalculationFailedEvent           // ← neu
CalculationCancelledEvent        // ← neu
```

Das Team liebte Events.

"Events sind die Lösung," wurde zum Team-Mantra.

- Zwei Services müssen kommunizieren? **Event.**
- Status-Update für UI? **Event.**
- Logging für Analytics? **Event.**
- Debugging-Info für Entwickler? **Event.**

Qion beobachtete. Besorgt.

Er erinnerte sich an ein anderes Projekt. Vor Jahren. Wo Events auch die Lösung waren.

Bis sie das Problem wurden.

---

## VIII. Die ersten Anzeichen

**Woche 13.**

Azure Service Bus Dashboard:

```text
╔════════════════════════════════════════════════╗
║         SERVICE BUS METRICS                    ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Messages/Day: 847,000                         ║
║  Peak Rate: 28 messages/second                 ║
║  Queue Depth: Average 240                      ║
║  Dead Letter Queue: 18 messages                ║
║                                                 ║
║  Top Topics:                                   ║
║  1. calculation-events (412k/day)              ║
║  2. import-events (287k/day)                   ║
║  3. project-events (148k/day)                  ║
║                                                 ║
║  Cost: €340/month                              ║
║  Status: 🟡 Warning: Growing rapidly           ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

„847.000 Events pro Tag“, las Oben Kell laut. „Das ist viel.“

"Das ist elf Events pro Sekunde," rechnete Arik. "Durchschnitt. Das ist okay."

"Aber 28 Events pro Sekunde im Peak," sagte Qion. "Und schaut euch den Trend an."

Er zeigte ein Grafana-Dashboard. **Events per Day - Last 30 Days:**

```text
Day  1: 120,000 events
Day  7: 240,000 events (2x)
Day 14: 480,000 events (4x)
Day 21: 720,000 events (6x)
Day 28: 847,000 events (7x)
```

"Das ist exponentielles Wachstum," sagte Qion leise. "7x in vier Wochen."

"Aber es funktioniert," verteidigte Sora. "Kein Performance-Problem. Kein Downtime."

"Noch nicht," sagte Qion. "Aber extrapoliert das. Vier weitere Wochen. 7x von 847,000."

Er schrieb an die Tafel:

```text
Week  4: 847,000 events/day
Week  8: 5,929,000 events/day (projected)
Week 12: 41,503,000 events/day (projected)
Week 16: 290,521,000 events/day (projected)

Cost projection (Week 16):
├─ Service Bus: €12,000/month
├─ Storage: €3,400/month
├─ Data Transfer: €2,100/month
└─ Total: €17,500/month

Current: €340/month
Increase: 5,147%
```

"Das," sagte Qion, "kann nicht funktionieren."

"Aber es ist nur eine Projektion," sagte Arik. "Vielleicht flacht es ab."

Qion schüttelte den Kopf. "Es flacht nie ab. Nicht von selbst. Nicht ohne Intervention."

---

## IX. Die Governance-Frage

Das Team hielt ein Emergency-Meeting.

"Wir brauchen Regeln," sagte Qion. "Governance. Für Events."

"Was für Regeln?"

Qion schrieb auf das Whiteboard:

```text
EVENT GOVERNANCE (Vorschlag):

1. Event Budget:
   Max 5 events per user request
   
2. Event Review:
   Jedes neue Event braucht Approval
   
3. Event Taxonomy:
   Commands vs Events vs Notifications
   
4. Event Monitoring:
   Pflicht: Track Events Published vs Consumed
   
5. Event Cleanup:
   Quarterly Review: Welche Events sind noch nötig?
```

Das Team diskutierte.

"Max 5 Events per Request?" fragte Arik. "Was ist mit unserem Import-Flow? Der braucht 7 Events!"

"Dann ist der Import-Flow zu granular," sagte Qion. "Oder die Events sind falsch geschnitten."

"Aber wir haben doch gerade erst angefangen mit Events," protestierte Sora. "Jetzt schon Limits?"

"Ja," sagte Qion fest. "JETZT ist der richtige Zeitpunkt. Bevor ihr 40 Millionen Events pro Tag habt."

„Das ist doch übertrieben“, sagte Refactorist Prime.

Qion drehte sich zu ihm. "Ist es das? Vor vier Wochen: 120,000 Events. Heute: 847,000 Events. Nächsten Monat?"

Stille.

---

## X. Der Strangler stockt

**Woche 16.**

Stand-Up.

"Status der Strangler-Migration?" fragte der Tech Lead.

Arik präsentierte:

```text
MIGRATION STATUS (Week 16):

Calculation Types migriert: 4 von 23 (17%)
Traffic auf neuem Code: 8%
Geschätzte Zeit bis Completion: 47 Wochen

Original Plan: 12 Monate
Current Projection: 22 Monate
```

"Warum so langsam?" fragte der Product Owner.

"Mehrere Gründe," sagte Arik. Er listete auf:

```text
SLOWDOWN FACTORS:

1. Routing-Complexity
   └─ Feature Flags, Tenant-spezifisch, Debugging

2. Bug-Kompatibilität
   └─ Neuer Code muss alte Bugs nachbilden

3. Event-Proliferation
   └─ Jede Migration fügt 5-7 neue Events hinzu

4. Parallel Development
   └─ Bugs in Old? Fix in Old.
      Bugs in New? Fix in New.
      Bug in Both? Fix in Both.

5. Testing-Overhead
   └─ Alte Implementation testen
      Neue Implementation testen
      Routing-Layer testen
      Feature-Flags testen

6. Cognitive Load
   └─ "Bin ich im alten oder neuen Code?"
      "Ist dieses Flag aktiv?"
      "Welcher Bug ist das?"
```

"Das ist mehr als doppelt so lang wie geplant," stellte der CTO fest.

"Ja," sagte Arik leise. "Strangler ist langsamer als gedacht."

Qion sprach, was alle dachten: "Strangler ist nicht langsam. Strangler ist teuer. In Zeit. In Komplexität. In Cognitive Load."

Er ging ans Whiteboard.

"Ihr habt gelernt, was ich befürchtet habe. Strangler Pattern ist elegant in der Theorie. Aber in der Praxis..."

Er zeichnete:

```text
STRANGLER PATTERN - DIE REALITÄT:

Month 1-2: "This is great! Modern API!"
           └─ Honeymoon Phase

Month 3-4: "Why is routing so complex?"
           └─ Complexity Awakening

Month 5-6: "Do we fix the bug in old or new?"
           └─ Parallel System Tax begins

Month 7-8: "Should we add this feature to old?"
           └─ Decision Paralysis

Month 9-12: "When can we delete the old code?"
            └─ The Question Nobody Answers

Month 13+: "Let's just keep both forever."
           └─ The Acceptance of Defeat
```

"Das," sagte Qion, "ist der Elefant im Raum. Ihr wollt einen Elefanten essen. Stück für Stück. Aber nach vier Wochen seid ihr satt. Und der Elefant steht noch da. Halb aufgegessen. Verwesend."

---

## XI. Das Wendepunkt-Meeting

Der CTO rief ein All-Hands an.

"Wir müssen reden," sagte er. "Über BPP. Über die Migration. Über die nächsten Schritte."

Das Team versammelte sich.

Der CTO teilte ein Slide. Eine einfache Frage:

```text
╔════════════════════════════════════════════════╗
║                                                 ║
║          KRITISCHE FRAGE                       ║
║                                                 ║
║  Strangler Migration:                          ║
║  ├─ Week 16: 17% migrated                      ║
║  ├─ Projection: 22 months total                ║
║  └─ Question: Can we afford that?              ║
║                                                 ║
║  Parallel Systems:                             ║
║  ├─ Old BPP (Stored Procedures)                ║
║  ├─ New BPP (C# Services)                      ║
║  ├─ Routing Layer (Feature Flags)              ║
║  └─ Question: For how long?                    ║
║                                                 ║
║  Event Proliferation:                          ║
║  ├─ Current: 847k events/day                   ║
║  ├─ Growth: 7x in 4 weeks                      ║
║  ├─ Projection: 5.9M events/day in 4 weeks     ║
║  └─ Question: Is this sustainable?             ║
║                                                 ║
║  Team Velocity:                                ║
║  ├─ 60% time: Strangler Migration              ║
║  ├─ 25% time: Bug-Fixes (both systems)         ║
║  ├─ 15% time: New Features                     ║
║  └─ Question: Is this acceptable?              ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

"Ich stelle keine rhetorischen Fragen," sagte der CTO. "Ich will echte Antworten. Von jedem von euch."

---

## XII. Die Optionen (wieder)

Qion stand auf. Er wusste, was kommen würde.

"Wir haben vier Optionen. Wieder."

Er schrieb an die Tafel:

```text
OPTION 1: STRANGLER FORTSETZEN
├─ Timeline: 22 Monate noch
├─ Kosten: €1.8M (Team + Infrastructure)
├─ Risiko: Event-Storm eskaliert
└─ Outcome: Vielleicht Erfolg. Vielleicht zwei Systeme für immer.

OPTION 2: STRANGLER BESCHLEUNIGEN
├─ Timeline: 6 Monate (aggressive Migration)
├─ Kosten: €600k + keine neuen Features
├─ Risiko: Qualität leidet
└─ Outcome: Abgeschlossen, aber hastig.

OPTION 3: STRANGLER STOPPEN
├─ Timeline: Sofort
├─ Kosten: €0 (+ Sunk Cost: €180k)
├─ Risiko: Alle Arbeit umsonst
└─ Outcome: Zurück zu Stored Procedures. Für immer.

OPTION 4: STRANGLER PAUSIEREN + EVENTS FIXEN
├─ Timeline: 2 Monate Event-Governance
├─ Kosten: €120k + verzögerte Migration
├─ Risiko: Event-Problem wird schlimmer
└─ Outcome: Fundament reparieren, dann weiter.
```

"Meine Empfehlung," sagte Qion, "ist Option 4. Pausiert die Migration. Zwei Monate. Repariert das Event-Problem. Baut Governance. Dann: beschleunigte Migration. Sechs Monate. All-in."

"Warum?" fragte der CTO.

"Weil," sagte Qion, "wir gerade zwei Brände gleichzeitig löschen. Strangler-Komplexität. Event-Proliferation. Wenn wir weitermachen, werden beide Brände außer Kontrolle. Wenn wir einen nach dem anderen löschen, haben wir eine Chance."

Der Tech Lead nickte langsam. "Das macht Sinn."

"Aber," warf Arik ein, "das bedeutet: noch länger parallele Systeme."

"Ja," sagte Qion. "Aber kontrolliert. Gemanaged. Mit Governance. Statt unkontrolliert in die Eskalation."

---

## XIII. Die Entscheidung

Der CTO schwieg lange.

Dann sagte er: "Option 4. Mit Bedingungen."

Er schrieb an die Tafel:

```text
BEDINGUNGEN:

1. Event-Governance innerhalb 4 Wochen
   └─ Event Budget
   └─ Event Review Board
   └─ Event Cleanup

2. Strangler-Beschleunigung-Plan innerhalb 2 Wochen
   └─ Realistischer Timeline
   └─ Klare Milestones
   └─ Rollback-Strategie

3. Parallel Systems Maximum: 6 Monate noch
   └─ Danach: One System Rule
   └─ Entweder Alt oder Neu
   └─ Aber nicht beide
```

"Sechs Monate," sagte er ernst. "Ihr habt sechs Monate. Dann muss eine Entscheidung gefallen sein: Entweder BPP ist komplett migriert. Oder wir stoppen und leben mit den Stored Procedures. Aber wir können nicht zwei Systeme für immer betreiben."

Stille.

"Verstanden?" fragte der CTO.

"Verstanden," sagte das Team im Chor.

---

## XIV. Epilog: Der Baum, der nicht stirbt

Drei Monate später.

Die Event-Governance war implementiert. Die Event-Proliferation unter Kontrolle.

Die Strangler-Migration lief wieder. Beschleunigt. Fokussiert.

Aber das Team hatte eine wichtige Lektion gelernt.

**Strangler Pattern ist keine Silver Bullet.**

Es ist ein Werkzeug. Ein Ansatz. Eine Strategie.

Aber wie jede Strategie: Es braucht Disziplin. Governance. Mut zum Abschluss.

Und die Erkenntnis, dass der schwerste Teil nicht der Anfang ist.

Der schwerste Teil ist das Töten des alten Systems.

Den Mut zu haben zu sagen: "Die alte Eiche muss weg. Jetzt. Nicht nächstes Jahr. Jetzt."

Denn sonst endet man mit zwei toten Bäumen. Zusammengewachsen. Skelette.

Das Team hatte den Fehler früh genug erkannt.

Aber andere Teams, in anderen Projekten, würden ihn wiederholen.

Und wieder.

Und wieder.

---

## Anhang: Die Lektionen der Strangler-Migration

```text
╔════════════════════════════════════════════════╗
║     STRANGLER PATTERN - DIE WAHRHEIT           ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  ✓ FUNKTIONIERT für:                           ║
║    ├─ Kleine Systeme (< 10 Module)             ║
║    ├─ Klare Grenzen (gut modularisiert)        ║
║    ├─ Kurze Migration (< 6 Monate)             ║
║    └─ Team-Commitment (alle fokussiert)        ║
║                                                 ║
║  ✗ SCHEITERT bei:                              ║
║    ├─ Großen Systemen (> 20 Module)            ║
║    ├─ Unklaren Grenzen (eng gekoppelt)         ║
║    ├─ Langer Migration (> 12 Monate)           ║
║    └─ Geteilter Aufmerksamkeit                 ║
║                                                 ║
║  ⚠ KRITISCHE ERFOLGS-FAKTOREN:                 ║
║    1. Zeitlimit setzen (6-12 Monate MAX)       ║
║    2. Governance von Tag 1                     ║
║    3. "One System Rule" durchsetzen            ║
║    4. Mut zum Töten des Alten                  ║
║                                                 ║
║  💀 TODESZEICHEN:                              ║
║    ├─ "Just ein paar Monate länger"            ║
║    ├─ "Beide Systeme funktionieren doch"       ║
║    ├─ "Wir migrieren when it makes sense"      ║
║    └─ "Some day we'll finish"                  ║
║                                                 ║
║  💰 VERSTECKTE KOSTEN:                         ║
║    ├─ Cognitive Load (2x Systeme verstehen)    ║
║    ├─ Testing-Overhead (2x Testsuites)         ║
║    ├─ Deployment-Complexity (Sync-Probleme)    ║
║    ├─ Bug-Fixes (in beiden Systemen)           ║
║    └─ Feature-Decisions (alt oder neu?)        ║
║                                                 ║
║  🎯 DER EINZIGE WEG ZUM ERFOLG:                ║
║    Start Fast. Migrate Faster. Finish Fastest. ║
║    Kill the Old. No Mercy. No "Just in Case".  ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

---

*"Man isst einen Elefanten Stück für Stück. Das ist wahr. Aber wenn man zu langsam isst, verwest der Elefant. Und dann isst man Aas. Und Aas macht krank."*

— Qion Varr, nach der Strangler-Migration

---

**Nächstes Kapitel:** Kapitel 9: Die Event-Storm  
Der Preis der Freiheit und warum "Event-Driven" ohne Budget eine Katastrophe ist.

---
