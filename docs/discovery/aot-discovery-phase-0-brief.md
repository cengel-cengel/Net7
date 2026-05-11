# AOT Discovery — Phase-0 Brief
## Unblock 3 H-Risk OQs for Sprint-2 Kickoff

**Mission:** Sprint-2 Implementation kann nicht starten bevor OQ-049 (Public Visibility), OQ-051 (Document Scope Mapping), OQ-053 (List-Price Anchor) durch AOT-Stakeholder-Entscheidungen geschlossen sind. Dieses Dokument liefert die fokussierten Fragebatterien, Stakeholder-Mappings und Capture-Templates für diese 3 Sessions.

**Foundation:** Vollständiger Discovery-Kit (3 Tracks à 60-90 Min, 42 Fragen) liegt parallel vor — diese Phase-0 ist die schlanke Vorab-Sequenz für H-Risk-Unblocks. Discovery-Kit kommt danach für die übrigen 12 OQs.

**Date:** 2026-05-11
**Status:** Active — Sessions to be scheduled

---

## §1 Die 3 H-Risk OQs auf einen Blick

| OQ | Frage (kurz) | Primary Stakeholder | Blocking |
|----|--------------|---------------------|----------|
| OQ-049 | Welche Produkte sind public-visible? Alle `aktiv`, oder nur mit NET7 WebService-Flag, oder per anderem Filter? | AOT-Sales + AOT-PM | Catalog-Projection-Logic, Public Sitemap |
| OQ-051 | Wie mappet NET7's B2B-Dokumentenanzeige-Dropdown auf die 4 DocumentScopes (Public/B2B/Internal/Customer-Specific)? | AOT-Compliance + Legal | Documents-Adapter-Logic; Audit-Risiko bei Fehlentscheidung |
| OQ-053 | Was ist der Public-List-Price-Anchor: Staffel-1 VK frei Haus, "ab €X" Range, oder anderes? | AOT-Mgmt + Sales | Pricing-Disclosure, Schema.org-Markup, Pricing-Module-Logic |

Risk-of-no-decision per OQ siehe §5/§6/§7 unten.

---

## §2 Stakeholder-Mapping und Meeting-Empfehlung

### Empfehlung: 2 Sessions, nicht 3

**Session A — Strategy + Operations (90 Min)**
- Teilnehmer: Geschäftsführung (oder strategischer Vertreter), AOT-Sales-Lead, optional AOT-PM
- Deckt: OQ-049 + OQ-053
- Format: Walkthrough strategic-foundation §1 (Assumption Challenges) als Pre-Read, dann fokussierte Q&A
- Ziel: Decision-Record für beide OQs

**Session B — Compliance + Regulatory (60 Min)**
- Teilnehmer: AOT-Compliance/QM, optional Legal
- Deckt: OQ-051
- Format: NET7-DMS-Walkthrough als Anchor, dann Mapping-Tabelle erstellen
- Ziel: Decision-Record + Default-Fail-Safe-Bestätigung

**Optional zusammenführen wenn Personaldecke knapp:**
3-OQ-Combined-Session 120 Min, alle Stakeholder, sequentiell. Risiko: Compliance-Fokus geht in Strategy-Diskussion unter.

### Wenn AOT-Single-Contact-Pattern

Falls du nur einen Single-Person-Contact bei AOT hast, der für mehrere Funktionen spricht: 1 Session 90-120 Min, sequentiell OQ-049 → OQ-053 → OQ-051. Person muss vorher die Pre-Read § lesen.

---

## §3 Pre-Read 1-Pager (für AOT-Stakeholder)

> **Was AOT-Mgmt vor Session A wissen muss:**
>
> Die neue digitale Plattform ersetzt nicht das Portal — sie ergänzt es um eine öffentliche Discovery-Ebene. Anonyme Web-Besucher sollen AOT-Produkte über Google finden können (SEO-getrieben). Authentifizierte B2B-Kunden bekommen wie bisher kundenspezifische Preise.
>
> **Drei Entscheidungen sind für den Kickoff erforderlich:**
>
> 1. **Welche Produkte zeigen wir der Welt?** Alle 800+ aktiven Artikel — oder nur eine kuratierte Auswahl? NET7 hat bereits einen WebService-Flag pro Artikel — verwenden wir den, oder definieren wir eine andere Regel?
>
> 2. **Welche Dokumente sind public-zugänglich?** SDS (Sicherheitsdatenblätter) sind oft Public-Standard in der Industrie. COAs (Chargenanalysen) typischerweise nicht. Wie übersetzen wir NET7's bestehende B2B-Dokumentenanzeige-Klassifikation in unsere 4-Stufen-Sichtbarkeit?
>
> 3. **Welcher Preis erscheint öffentlich?** Anonyme Besucher sollen einen Anker-Preis sehen (für SEO-Schema.org-Anforderungen). Der höchste Listenpreis (Staffel-1 VK frei Haus) ist eine Option — aber das könnte abschreckend wirken. Alternative: "ab €X" Range. Oder: kein Preis, nur "Auf Anfrage".
>
> Diese Entscheidungen sind reversibel, aber kostspielig wenn später geändert. Wir möchten heute alle drei treffen.
>
> Vorbereitung: ca. 15 Min Pre-Read, dann 90 Min strukturierte Diskussion.

---

## §4 Risiko der Nicht-Entscheidung

Per ADR-0004 Rule 6 (Triplet-Closer) müssen OQs aktiv geschlossen werden — Stillstand ist keine Option. Was passiert wenn diese OQs offen bleiben:

| OQ | Risk-of-no-decision |
|----|---------------------|
| OQ-049 | Engineering muss Worst-Case-Annahme treffen: nur explizit WebService-flagged Produkte werden public. Wenn AOT später entdeckt dass das zu restriktiv war (oder zu offen) — Catalog-Re-Projection nötig, plus SEO-Index-Refresh (Crawl-Budget-Verlust). |
| OQ-051 | Default-Fail-Safe greift: alle ambiguous Documents werden B2B-only. SDS/MSDS-Public-Access geht verloren → AOT-Kunden müssen sich für Sicherheitsdatenblätter einloggen → vermeidbare Kundenfriktion und mögliche Regulatorik-Lücke. |
| OQ-053 | Engineering kann nicht Pricing-Disclosure-Frontend bauen. Public-Catalog-Implementation blockiert. SEO-Schema.org-Markup nicht baubar. → Sprint-2 V1 verzögert sich. |

---

## §5 OQ-049 — Fragebatterie: Public Catalog Visibility

### Kontext (Anchor for the Discussion)

NET7 hat pro Artikel-Stammdatensatz einen **WebService-Flag** (visual evidence Screenshot 1: Flag "W") und einen **B2B-Dokumentenanzeige-Dropdown**. Sprint-1 hat das als read-only-mirror modelliert. Sprint-2 muss entscheiden: was bedeutet "public-visible" konkret?

### Fragebatterie

1. **Wie viele Artikel sind aktuell als `aktiv` in NET7 geführt?** (Größenordnung — 200? 800? 1500?)
2. **Wie viele davon haben aktuell den WebService-Flag gesetzt?** (Falls bekannt; sonst Schätzung)
3. **Was bedeutet der WebService-Flag heute funktional?** Steuert er Anzeige im aktuellen B2B-Portal? Steuert er sonst etwas?
4. **Gibt es Artikel-Kategorien die NICHT public sein sollen?** Zum Beispiel: Custom-Formulierungen, Pre-Release-Produkte, Auslauf-Artikel?
5. **Gibt es regulatorische Gründe Produkte zu verbergen?** (Cosmetic-Notification-Pflichten, Restricted-Substances?)
6. **Wer entscheidet zukünftig welche Produkte public werden?** Einzel-Person, Workflow, automatisch via Flag-Wert?
7. **Soll der WebService-Flag im SAME-NET7-Feld bleiben, oder brauchen wir ein zusätzliches "Public"-Feld?** (Hat Implikationen für TopM-Integration)

### Decision-Record-Template

```
OQ-049 Decision Record
Date: ___________
Attendees: ___________
Decision:
  Public-Visibility-Rule: ___________________________
  (z.B. "Alle Artikel mit aktiv-Status UND
   WebService-Flag W UND ABC-Class A oder B")
Open Questions surfaced: ___________
Trigger-for-Revisit: ___________
Action items:
  - [ ] Engineering: implementiere Visibility-Filter in Catalog-Adapter
  - [ ] AOT-PM: Audit der ~N Produkte die unter neue Regel fallen
  - [ ] Marketing: SEO-Sitemap-Scope-Definition
```

---

## §6 OQ-051 — Fragebatterie: DocumentScope Mapping

### Kontext

NET7 DMS hat per-Dokument zwei Sichtbarkeits-Flags (Visual Discovery Screenshot 1): **WebService-Flag** und **B2B-Dokumentenanzeige-Dropdown**. Plattform-seitig sollen Dokumente in 4 Scopes klassifiziert werden:

- **Public** — sichtbar für anonyme Besucher (z.B. SDS)
- **B2B** — sichtbar für jeden authentifizierten B2B-Kunden
- **Customer-Specific** — sichtbar nur für bestimmte Kunden
- **Internal** — niemals auf Web (interne QM-Docs)

### Fragebatterie

1. **Welche Werte hat der B2B-Dokumentenanzeige-Dropdown heute?** (Liste aller möglichen Werte mit ihrer Bedeutung)
2. **Welche Dokumenttypen sind aktuell für anonyme Besucher gedacht?** (SDS? MSDS? COA? TDS? Allergen Statement? Flowchart?)
3. **Gibt es Dokumente die customer-spezifisch sind?** Beispiel: kundenspezifische COA mit speziellen Parametern. Wie werden die in NET7 markiert?
4. **Welche Dokumente sind nie für Web bestimmt?** (Interne QM, Lieferanten-Dokus, etc.)
5. **Wer ist verantwortlich für Document-Scope-Klassifikation in NET7?** (QM? Marketing? Compliance?)
6. **Wie oft ändert sich Document-Scope?** (Selten / pro Version / pro Audit?)
7. **Gibt es regulatorische Pflicht bestimmte Dokumente public zu machen?** (REACH? CLP? Kosmetik-Verordnung?)
8. **Falls Scope-Mapping unklar ist — ist Default-Fail-Safe (B2B-only) akzeptabel?** (Per documents-sprint-2-extension §2.3 constitutional)

### Decision-Record-Template

```
OQ-051 Decision Record
Date: ___________
Attendees: ___________
Mapping-Rule:
  B2B-Dropdown-Value "____" → Scope: Public/B2B/Internal/Customer-Specific
  B2B-Dropdown-Value "____" → Scope: ___
  B2B-Dropdown-Value "____" → Scope: ___
  ...
Default-Fail-Safe: confirmed = B2B-only (when mapping ambiguous)
Document-Types-by-Default-Scope:
  SDS/MSDS:  ___________
  COA:       ___________
  TDS:       ___________
  Flowchart: ___________
  Allergen:  ___________
Regulatorische-Pflicht-Liste: ___________
Open Questions surfaced: ___________
Action items:
  - [ ] Engineering: implementiere DocumentScope-Adapter
  - [ ] AOT-Compliance: Audit-Stichprobe von 20 Dokumenten zur Mapping-Validierung
  - [ ] Engineering: AnonymousAccessAuditEntry mit DSGVO-Retention 6 Monate
```

---

## §7 OQ-053 — Fragebatterie: Public List-Price Anchor

### Kontext

NET7 hat per-Artikel 7-Staffel × 5-Preisarten-Matrix (Visual Discovery Screenshot 6). Anonyme Besucher sehen heute nichts. Für Public-Catalog brauchen wir einen Anker-Preis (SEO-Schema.org Product.offers.price erfordert nominal-Wert).

### Fragebatterie

1. **Wie kommunizieren Wettbewerber (Mibelle, Provital, etc.) Preise public?** "Auf Anfrage" oder konkreter Anker? (Marktbeobachtung-Input)
2. **Was ist die strategische Positionierung des Public-Preises?** Lockt sie an, oder schreckt sie ab vs. Wettbewerb?
3. **Wie groß ist die Spreizung zwischen Staffel-1 (39,90) und Staffel-5 (19,10) typisch?** (Faktor 2-3 verbreitet?)
4. **Welche Preisart-Spalte ist der "richtige" Public-Anker?** VK frei Haus (höchster), VK ex works, oder anderes?
5. **Wie verhält es sich mit "A.Anfrage"-Staffeln (≥190 kg)?** Public sehen Login-CTA, Range, oder Hide-Tier?
6. **Bestätige: Per-Kunden-Preis darf NIE public sichtbar werden.** (Pricing-Extension §2.4 constitutional)
7. **Akzeptanz von Range-Display ("ab €X")?** Alternative zu fixem Anker. UX-Vorteil: signalisiert Volumen-Rabatt-Möglichkeit.
8. **Wie reagiert AOT-Sales darauf dass Anonyme einen Anker-Preis sehen?** (Compliance, Vertrieb-Argument-Vorbereitung)

### Decision-Record-Template

```
OQ-053 Decision Record
Date: ___________
Attendees: ___________
Public-List-Price-Strategy:
  Option chosen: [Staffel-1 VK frei Haus] / [Range "ab €X"] /
                 [other: ___________]
  Display-Format: __________
A-Anfrage-Tier-UX:
  Anonymous sees: [Login-CTA] / [RFQ-CTA] / [Hide-Tier] /
                  [Placeholder "Auf Anfrage"]
Edge-Cacheable: confirmed Yes per pricing-ext INV-014
Per-Customer-Price-Leak-Risk: re-confirmed NEVER edge-cached
Open Questions surfaced: ___________
Action items:
  - [ ] Engineering: implementiere PublicListPrice-Projection in pricing module
  - [ ] Engineering: Schema.org Product.offers Markup
  - [ ] Marketing: Public-Pricing-FAQ-Page draft
```

---

## §8 Outreach-Templates

### Mail an AOT-Geschäftsführung (Session A)

```
Betreff: Sprint-2 Kickoff — 90 Min Workshop zur Public-Plattform-
         Strategie

Hallo [Name],

wir sind bereit für den Sprint-2 Kickoff der neuen digitalen
Plattform. Bevor Engineering startet, brauchen wir Deine
Entscheidung zu zwei strategischen Fragen:

1. Welche Produkte sollen public-discoverable sein (SEO-getrieben)?
2. Welcher Preis erscheint anonym (Anker für SEO und User-Trust)?

Beide Entscheidungen sind reversibel aber teuer in der Korrektur.
Ich schlage einen 90-Min-Workshop vor.

Pre-Read: 2-seitig, ca. 15 Min Zeitbedarf — sende ich vorab.
Teilnehmer: Du, [AOT-Sales-Lead], optional [AOT-PM].
Vorschlag-Termine: [3 Slots]

LG, [Name]
```

### Mail an AOT-Compliance/QM (Session B)

```
Betreff: Document-Scope-Klassifikation für neue Plattform (60 Min)

Hallo [Name],

die neue Web-Plattform wird Produkt-Dokumente sowohl public als
auch B2B-scoped ausliefern. NET7 hat dafür bereits einen
B2B-Dokumentenanzeige-Dropdown — wir müssen klären, wie dessen
Werte auf unsere 4 Sichtbarkeits-Stufen (Public/B2B/Internal/
Customer-Specific) gemappt werden.

Fokus: SDS, COA, TDS, Flowchart, Allergen Statement.
Plus: Default-Fail-Safe-Regel bei Mehrdeutigkeit.

Pre-Read: 1-seitig, sende ich vorab.
Teilnehmer: Du, optional Rechtsberatung.
Vorschlag-Termine: [3 Slots]

LG, [Name]
```

---

## §9 Triplet-Closer Erinnerung (per ADR-0004 Rule 6)

Jede OQ-Schließung produziert ein **Triplet:**

1. **Decision** — eingetragen in OQ-Master mit Status "Closed"
2. **Artifact-Update** — relevante Aggregate-Extension oder ADR-Section aktualisiert
3. **Test/Validation** — Engineering-Item im Backlog der die Decision implementiert UND validiert

Beispiel-Closure für OQ-049:

```
OQ-049 Closure Triplet:
1. Decision (recorded in OQ-Master row, status "Closed"):
   "Public-visibility = aktiv AND WebService-flag-W AND
    ABC-class IN (A,B)"
2. Artifact-update:
   catalog-sprint-2-extension §2.1 → INV-008 final formulation
3. Test-validation:
   Engineering-Story: implementiere Visibility-Filter +
   integration-test mit 3 Edge-cases (flag-missing,
   ABC-missing, aktiv-but-no-flag)
```

Ohne dieses Triplet ist die OQ formell nicht geschlossen.

---

## §10 Post-Session Workflow

Nach jedem Closure:

1. Decision-Record in `/docs/discovery/oq-closures/` ablegen (V1: einzelne MD-Files pro OQ)
2. OQ-Master-Extension updaten (Status: Open → Closed mit Closure-Triplet)
3. Affected Aggregate-Extension oder ADR aktualisieren (PR mit Sprint-1-Workflow)
4. Engineering-Backlog: korrespondierende Stories anlegen

**Triggers Sprint-2 Phase-0 Implementation-Start:** alle 3 H-Risk-OQs geschlossen mit vollständigem Triplet.

---

## §11 Was nach den 3 H-Risk-Closures kommt

Sobald OQ-049 + OQ-051 + OQ-053 geschlossen sind:

- **NET7-Adapter-Spike** kann starten (Phase 0 in ADR-0005)
- **Event-Bus-Setup** parallel
- **Read-Model-Skeleton** für catalog + documents
- **Andere OQs** (047, 048, 050, 052, 054-061) können dann in Standard-Cadence abgearbeitet werden (kein Phase-0-Blocker mehr)

Du kannst nach den 3 Closures direkt in Sprint-2-Implementation eintreten.

---

## §12 References

- /docs/architecture/strategic-foundation.md §13.2 (Sprint-2-Kickoff-Criteria)
- /docs/adr/0004-open-question-governance.md (Rule 6 Triplet-Closer)
- /docs/adr/0005-strangler-fig.md (Phase-0 entry-criteria)
- /docs/domain/open-questions-master-sprint-2-extension.md (OQ-049, OQ-051, OQ-053 H-Risk markings)
- /docs/domain/aggregates/catalog-sprint-2-extension.md §2.1 (PublicVisibilityFlag — depends on OQ-049)
- /docs/domain/aggregates/documents-compliance-sprint-2-extension.md §2.2 (DocumentScope 4-tier — depends on OQ-051)
- /docs/domain/aggregates/pricing-contracts-sprint-2-extension.md §2.1 (PublicListPrice — depends on OQ-053)
- Visual Discovery: Screenshots 1 (NET7 DMS WebService-flag), 3 (Webshop-Beschreibung), 6 (Staffelpreise), 8 (public product page)
