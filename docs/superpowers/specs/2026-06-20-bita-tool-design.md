# BITA-tool — Design Spec

**Datum:** 20 juni 2026  
**Auteur:** Ralph Wagter (M&I/Partners)  
**Status:** Goedgekeurd voor implementatie

---

## 1. Doel

Een interactieve tool waarmee een consultant (of organisatie zelf) op basis van vijf invoerdimensies een op maat gemaakte Business IT Alignment-aanpak genereert. De tool vervangt generieke adviezen met een aanpak die aansluit bij hoe de organisatie feitelijk is ingericht en wat de stakeholders drijft.

**Directe aanleiding:** NEN 7510/BIO 2-implementaties stranden structureel wanneer de aanpak niet aansluit bij het type organisatie. Hetzelfde geldt voor IT governance-inrichting en IV-strategie-trajecten.

---

## 2. Deployment

- **Bestand:** `bita-tool.html` — één zelfstandig HTML-bestand
- **Hosting:** GitHub Pages (of als lokaal bestand / e-mailbijlage)
- **Afhankelijkheden:** geen — geen npm, geen CDN, geen framework
- **Offline:** volledig werkend zonder internetverbinding

---

## 3. Invoerdimensies

De tool vraagt vijf keuzes in volgorde:

### Stap 1 — Module

| Waarde | Label |
|---|---|
| `iv-strategie` | IV-strategie / I-visie |
| `it-governance` | IT Governance |
| `nen7510` | NEN 7510 / BIO 2 — Informatiebeveiliging |

### Stap 2 — Doel

| Waarde | Label | Toelichting |
|---|---|---|
| `visie` | Visie | Richting bepalen — wat willen we zijn? |
| `strategie` | Strategie | Plan voor realisatie — hoe komen we daar? |
| `implementatie` | Implementatie | Framework of structuur opzetten en uitvoeren |
| `toetsing` | Toetsing | Gap-analyse of audit — waar staan we? |

### Stap 3 — Organisatietype (spreektaal, geen Mintzberg-jargon)

| Waarde | Label | Kenmerk | Mintzberg-equivalent |
|---|---|---|---|
| `klein-direct` | Kleine gemeente, één managementlaag | Gemeentesecretaris en directie sturen direct aan; één managementlaag eronder | Eenvoudige structuur |
| `procedureel` | Procedureel en hiërarchisch | Vaste processen, formele besluitvorming, rollen duidelijk belegd | Machinebureaucratie |
| `vakgericht` | Vakgericht en professioneel | Specialisten werken autonoom op basis van vakkennis, niet op mandaat | Professionele bureaucratie |
| `projectmatig` | Projectmatig en coalitiegericht | Werkt in teams en pilots, snelheid boven procedure | Adhocratie |
| `gedecentraliseerd` | Gedecentraliseerd met meerdere diensten | Zelfstandige eenheden onder één bestuurlijk dak | Divisiestructuur |

### Stap 4 — Stakeholdercultuur (Quinn)

| Waarde | Label | Kenmerk |
|---|---|---|
| `hierarchie` | Regel & structuur | Procedures, formele accountability, audittrail |
| `output` | Output & resultaat | Doelmatigheid, kosten, aantoonbaar rendement |
| `vernieuwing` | Vernieuwing & experiment | Zichtbare vernieuwing, coalitie, ambitie |
| `samenwerking` | Samenwerking & vertrouwen | Peers, collectieve normen, sectorreferenties |

> **Theoretische noot:** Quinn's adhocratie-cultuur (vernieuwing) en clancultuur (samenwerking) zijn bewust gescheiden kwadranten. Vernieuwing is extern gericht; samenwerking is intern gericht. Het 'familaire' gevoel zit bij samenwerking, niet bij vernieuwing.

### Stap 5 — Volwassenheidsniveau

| Niveau | Label | Kenmerk |
|---|---|---|
| 1 | Ad hoc | Processen werken niet of nauwelijks — ook als er beleid op papier staat |
| 2 | Startend | Beleid aanwezig én een begin van uitvoering; nog inconsistent |
| 3 | Gestructureerd | Processen ingericht en aantoonbaar gevolgd |
| 4 | Lerend | Continue verbetering, gemeten en bijgestuurd |

---

## 4. Architectuur (bestandsstructuur)

```
bita-tool.html
├── CONTENT-BLOK    JS-object bovenaan — alle moduleinhoud, bewerkbaar zonder HTML-kennis
├── CSS             M&I house style — inline in <style>
├── HTML            wizard-shell (5 stap-containers + output-container)
└── JS              wizard-engine + output-renderer — inline in <script>
```

### M&I house style (non-negotiable)

```
--purple:  #883486
--teal:    #006782
Font:      "Segoe UI", Calibri, system-ui
Toon:      zakelijk neutraal — geen marketingtaal, geen metaforen
Taal:      Nederlands voor alle gebruikersgerichte content
```

Referentie voor stijl: `NEN7510-per-organisatietype.html` (bestaande HTML).

---

## 5. Data-structuur (gelaagd)

Content wordt niet per combinatie uniek opgeslagen (dat levert 960 cellen op). In plaats daarvan drie lagen:

### Laag 1 — Kerninhoud (module × doel × organisatietype)

3 modules × 4 doelen × 5 typen = **60 contentblokken**.  
Per blok:

```javascript
{
  intro: "Korte contextbeschrijving (2–3 zinnen)",
  wel:  ["Wat werkt", "..."],
  niet: ["Wat niet werkt", "..."],
  tip:  "Praktische tip voor dit type"
}
```

### Laag 2 — Communicatiemodifier (Quinn-cultuur)

Per organisatietype × Quinn-cultuur: welk argument en welke taal bij deze stakeholder past.

```javascript
quinn: {
  hierarchie:   { argument: "...", eerste_stap: "..." },
  output:       { argument: "...", eerste_stap: "..." },
  vernieuwing:  { argument: "...", eerste_stap: "..." },
  samenwerking: { argument: "...", eerste_stap: "..." }
}
```

### Laag 3 — Startpuntmodifier (volwassenheid)

Per combinatie: welke eerste stap past bij het huidige volwassenheidsniveau.

```javascript
volwassenheid: {
  1: { toelichting: "...", eerste_stap: "..." },
  2: { toelichting: "...", eerste_stap: "..." },
  3: { toelichting: "...", eerste_stap: "..." },
  4: { toelichting: "...", eerste_stap: "..." }
}
```

### Placeholders

IT Governance en IV-strategie: volledige structuur aanwezig, content-velden zijn `null`.  
Renderer toont: *"Deze module is nog niet uitgewerkt — inhoud volgt."*

---

## 6. Output-pagina

Na stap 5 wordt de output direct gerenderd (geen tussenscherm).

### Opbouw output

1. **Koptekst** — samenvatting van de vijf keuzes als badges (module · doel · org-type · cultuur · volwassenheid)
2. **Context** — 2–3 zinnen die de situatie typeren op basis van org-type
3. **Implementatielogica** — kernboodschap: wat is de juiste insteek voor dit type?
4. **Wat werkt / Wat niet werkt** — WEL/NIET kolommen (uit laag 1)
5. **Hoe aanspreken** — communicatietip op basis van Quinn-cultuur (laag 2)
6. **Eerste stap** — concrete actie op basis van volwassenheidsniveau (laag 3)
7. **Printknop** — boven en onder de output; print-CSS verbergt wizard-navigatie

### Print-CSS

- Navigatiebalk en wizard-stappen worden verborgen (`display: none`)
- Output-pagina vult het A4-formaat
- Marges: 1.5cm rondom
- Kleurblokken blijven zichtbaar (`-webkit-print-color-adjust: exact`)

---

## 7. Module-inhoud: NEN 7510 / BIO 2

De bestaande `NEN7510-per-organisatietype.html` bevat uitgewerkte content voor drie organisatietypen (procedureel, vakgericht, projectmatig) en één stakeholdertype (output). Deze content wordt als startpunt gebruikt en aangevuld voor:
- De twee ontbrekende organisatietypen (klein-direct, gedecentraliseerd)
- Alle vier Quinn-communicatiemodifiers per org-type
- Alle vier volwassenheidsstartpunten per combinatie
- De vier doelen (visie, strategie, implementatie, toetsing) als aparte contentblokken

---

## 8. Buiten scope (v1)

- PDF-export (printknop is voldoende)
- Opslaan van sessie of resultaat
- Gebruikersaccounts of login
- Server-side logica
- Animaties of transities

---

## 9. Technische randvoorwaarden

- Geen `any` types (niet van toepassing — puur JS, geen TypeScript)
- Geen externe CDN-afhankelijkheden
- Werkt in Chrome, Firefox, Edge (laatste twee versies)
- Responsive tot 768px breed (tabletweergave voor klantgesprek)
- Printbaar op A4 (staand)
