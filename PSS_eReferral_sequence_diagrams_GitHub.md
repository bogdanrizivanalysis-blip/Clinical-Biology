# PSS – eReferral Klinische Biologie

Dit document visualiseert drie mogelijke workflow-varianten voor de integratie van **Prescription Search Support (PSS)** binnen het **RIZIV-INAMI eReferral-platform** voor voorschriften klinische biologie.

De Mermaid-diagrams hieronder worden rechtstreeks door GitHub gerenderd in Markdown.

---

## Scenario 1 – PSS-first: PSS-testselectie vóór de templates

```mermaid
sequenceDiagram
    actor H as Huisarts
    participant ER as eReferral
    participant EPD as EPD
    participant PSS as PSS-engine
    participant T as Templates klinische biologie

    H->>ER: Start voorschrift klinische biologie

    ER->>PSS: Start PSS-analyse op achtergrond
    PSS->>EPD: Doorzoek patiëntgegevens
    EPD-->>PSS: Diagnoses, symptomen, testhistoriek, leeftijd, ...
    PSS-->>ER: PSS-aanbevelingen beschikbaar

    ER-->>H: Toon PSS-testselectie

    Note over H,PSS: PSS-testselectie toont aanbevolen testen:<br/>Groen = sterk aanbevolen<br/>Geel = aanvaardbaar, niet optimaal<br/>Rood = niet aanbevolen, motivatie vereist

    H->>ER: Selecteer gewenste PSS-testen

    opt Rode test geselecteerd
        ER-->>H: Vraag motivatie
        H->>ER: Geef motivatie
    end

    ER->>T: Verdeel geselecteerde testen over juiste template(s)
    T-->>ER: Bloed / Microbiologie / Speciale vochten
    ER-->>H: Toon vooringevulde templates

    opt Huisarts wijzigt testselectie
        H->>ER: Voeg extra testen toe
        H->>ER: Deselecteer voorgestelde testen
        ER->>T: Werk template(s) bij
    end

    H->>ER: Publiceer voorschrift
    ER-->>H: Voorschrift gepubliceerd
```

---

## Scenario 2 – Onvoldoende EPD-data: manuele diagnose en drie mogelijke uitkomsten

```mermaid
sequenceDiagram
    actor H as Huisarts
    participant ER as eReferral
    participant EPD as EPD
    participant PSS as PSS-engine
    participant T as Templates klinische biologie

    H->>ER: Start voorschrift klinische biologie

    ER->>PSS: Start PSS-analyse op achtergrond
    PSS->>EPD: Doorzoek patiëntgegevens
    EPD-->>PSS: Onvoldoende relevante informatie
    PSS-->>ER: Geen aanbeveling mogelijk

    ER-->>H: Bied manuele diagnose-invoer aan
    H->>ER: Vul diagnose manueel in

    H->>ER: Klik op PSS-knop
    ER->>PSS: Vraag nieuwe PSS-analyse
    PSS->>EPD: Combineer beschikbare EPD-data met diagnose
    EPD-->>PSS: Beschikbare patiëntgegevens

    alt OPTIE 1 – Huisarts selecteert PSS-testen
        rect rgb(220, 245, 220)
            PSS-->>ER: Kleurgecodeerde testaanbevelingen
            ER-->>H: Toon PSS-testselectie

            Note over H,PSS: Groen = sterk aanbevolen<br/>Geel = aanvaardbaar, niet optimaal<br/>Rood = niet aanbevolen, motivatie vereist

            H->>ER: Selecteer groene PSS-test(en)
            ER->>T: Vul geselecteerde testen vooraf in
        end

    else OPTIE 2 – Huisarts selecteert geen PSS-testen
        rect rgb(255, 245, 200)
            PSS-->>ER: Kleurgecodeerde testaanbevelingen
            ER-->>H: Toon PSS-testselectie

            Note over H,PSS: PSS geeft aanbevelingen,<br/>maar de huisarts kiest ervoor niets te selecteren

            H->>ER: Ga verder zonder PSS-testen te selecteren
        end

    else OPTIE 3 – PSS kan nog steeds geen aanbeveling genereren
        rect rgb(255, 225, 225)
            PSS-->>ER: Geen PSS-aanbevelingen beschikbaar
            ER-->>H: Geen PSS-testselectie beschikbaar
        end
    end

    ER->>T: Open templates klinische biologie
    T-->>ER: Bloed / Microbiologie / Speciale vochten
    ER-->>H: Toon al dan niet vooringevulde templates

    opt Huisarts past templates aan
        H->>ER: Selecteer bijkomende testen
        H->>ER: Deselecteer testen
        ER->>T: Werk template(s) bij
    end

    H->>ER: Publiceer voorschrift
    ER-->>H: Voorschrift gepubliceerd
```

---

## Scenario 3 – Vooraf ingevuld sjabloon eerst, PSS-suggesties daarna

```mermaid
sequenceDiagram
    actor H as Huisarts
    participant ER as eReferral
    participant EPD as EPD
    participant PSS as PSS-engine
    participant T as Templates klinische biologie

    H->>ER: Open sjabloonmenu
    H->>ER: Selecteer vooraf ingevuld sjabloon

    ER->>T: Laad gekozen sjabloon
    T-->>ER: Vooraf ingevulde template
    ER-->>H: Toon ingevulde template

    ER->>PSS: Start PSS-analyse na selectie sjabloon
    PSS->>EPD: Doorzoek patiëntgegevens
    EPD-->>PSS: Diagnoses, symptomen, testhistoriek, leeftijd, ...
    PSS-->>ER: Kleurgecodeerde aanbevelingen

    ER-->>H: Toon PSS-suggesties onderaan template

    Note over H,PSS: PSS-suggesties onderaan de template:<br/>Groen = sterk aanbevolen<br/>Geel = aanvaardbaar, niet optimaal<br/>Rood = niet aanbevolen, motivatie vereist

    alt Huisarts gebruikt PSS-aanbevelingen
        H->>ER: Selecteer gele PSS-test
        ER->>T: Voeg gele test toe aan relevante template
        T-->>ER: Template bijgewerkt
        ER-->>H: Toon bijgewerkte template

    else Huisarts gebruikt PSS niet
        H->>ER: Behoud bestaande template
    end

    H->>ER: Publiceer voorschrift
    ER-->>H: Voorschrift gepubliceerd
```

---

## Actoren

| Actor / component | Rol |
|---|---|
| **Huisarts** | Voorschrijver |
| **eReferral** | Platform waarin het voorschrift wordt opgesteld en gepubliceerd |
| **EPD** | Bron van diagnoses, symptomen, testhistoriek, leeftijd en andere relevante patiëntgegevens |
| **PSS-engine** | Genereert kleurgecodeerde aanbevelingen voor klinische biologie |
| **Templates klinische biologie** | Bloed, Microbiologie en Speciale vochten |

## PSS-kleurcodering

- **Groen** – sterk aanbevolen
- **Geel** – aanvaardbaar, niet optimaal
- **Rood** – niet aanbevolen; motivatie vereist
