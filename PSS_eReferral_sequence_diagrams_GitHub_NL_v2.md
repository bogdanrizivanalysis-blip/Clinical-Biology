# PSS – eReferral Klinische Biologie

Dit document visualiseert drie mogelijke workflowvarianten voor de integratie van **Prescription Search Support (PSS)** binnen het **RIZIV-INAMI eReferral-platform** voor voorschriften klinische biologie.

De Mermaid-diagrammen hieronder worden rechtstreeks door GitHub gerenderd in Markdown.

---

## Scenario 1 – PSS-first: PSS-testselectie vóór de templates

```mermaid
sequenceDiagram
    actor HA as Huisarts
    participant ER as eReferral
    participant EPD as EPD
    participant PSS as PSS-engine
    participant T as Templates Klinische Biologie

    HA->>ER: Start voorschrift klinische biologie

    PSS->>ER: Trigger aanvraag relevante patiëntcontext
    ER->>EPD: Doorzoek relevante patiëntgegevens
    EPD-->>ER: Diagnoses, symptomen, testhistoriek, leeftijd, ...
    ER-->>PSS: Bezorg beschikbare patiëntcontext
    PSS-->>ER: PSS-aanbevelingen beschikbaar

    ER-->>HA: Toon PSS-testselectie

    Note over HA,PSS: PSS-testselectie toont aanbevolen testen:<br/>Groen = sterk aanbevolen<br/>Geel = aanvaardbaar, niet optimaal<br/>Rood = niet aanbevolen, motivatie vereist<br/>(Een combinatie van kleuren of een selectie uit alle kleuren is ook mogelijk)

    HA->>ER: Selecteer gewenste PSS-testen

    opt Rode test geselecteerd
        ER-->>HA: Vraag motivatie
        HA->>ER: Geef motivatie
    end

    ER->>T: Verdeel geselecteerde testen over de juiste template(s)
    T-->>ER: Bloed / Microbiologie / Speciale vochten
    ER-->>HA: Toon vooringevulde templates

    opt Huisarts wijzigt testselectie
        HA->>ER: Voeg bijkomende testen toe
        HA->>ER: Deselecteer voorgestelde testen
        ER->>T: Werk template(s) bij
    end

    HA->>ER: Publiceer voorschrift
    ER-->>HA: Voorschrift gepubliceerd
```

---

## Scenario 2 – Onvoldoende EPD-data: manuele diagnose-invoer en drie mogelijke uitkomsten

```mermaid
sequenceDiagram
    actor HA as Huisarts
    participant ER as eReferral
    participant EPD as EPD
    participant PSS as PSS-engine
    participant T as Templates Klinische Biologie

    HA->>ER: Start voorschrift klinische biologie

    PSS->>ER: Trigger aanvraag relevante patiëntcontext
    ER->>EPD: Doorzoek relevante patiëntgegevens
    EPD-->>ER: Onvoldoende relevante informatie
    ER-->>PSS: Bezorg beschikbare patiëntcontext
    PSS-->>ER: Geen aanbeveling mogelijk

    ER-->>HA: Bied manuele diagnose-invoer aan
    HA->>ER: Vul diagnose manueel in

    HA->>ER: Klik op PSS-knop
    ER->>PSS: Vraag nieuwe PSS-analyse o.b.v. manuele diagnose
    PSS->>ER: Trigger aanvraag bijgewerkte patiëntcontext
    ER->>EPD: Doorzoek beschikbare EPD-data
    EPD-->>ER: Beschikbare patiëntgegevens
    ER-->>PSS: Bezorg EPD-data + manueel ingevoerde diagnose

    alt OPTIE 1 – Huisarts selecteert PSS-testen
        rect rgb(220, 235, 255)
            PSS-->>ER: Kleurgecodeerde testaanbevelingen
            ER-->>HA: Toon PSS-testselectie

            Note over HA,PSS: Groen = sterk aanbevolen<br/>Geel = aanvaardbaar, niet optimaal<br/>Rood = niet aanbevolen, motivatie vereist<br/>(Een combinatie van kleuren of een selectie uit alle kleuren is ook mogelijk)

            HA->>ER: Selecteer groene PSS-test(en)
            ER->>T: Vul geselecteerde testen vooraf in
        end

    else OPTIE 2 – Huisarts selecteert geen PSS-testen
        rect rgb(235, 225, 250)
            PSS-->>ER: Kleurgecodeerde testaanbevelingen
            ER-->>HA: Toon PSS-testselectie

            Note over HA,PSS: PSS geeft aanbevelingen,<br/>maar de huisarts kiest ervoor geen testen te selecteren

            HA->>ER: Ga verder zonder PSS-testen te selecteren
        end

    else OPTIE 3 – PSS kan nog steeds geen aanbeveling genereren
        rect rgb(235, 235, 235)
            PSS-->>ER: Geen PSS-aanbevelingen beschikbaar
            ER-->>HA: Geen PSS-testselectie beschikbaar
        end
    end

    ER->>T: Open templates klinische biologie
    T-->>ER: Bloed / Microbiologie / Speciale vochten
    ER-->>HA: Toon templates, al dan niet vooringevuld

    opt Huisarts wijzigt templates
        HA->>ER: Selecteer bijkomende testen
        HA->>ER: Deselecteer testen
        ER->>T: Werk template(s) bij
    end

    HA->>ER: Publiceer voorschrift
    ER-->>HA: Voorschrift gepubliceerd
```

---

## Scenario 3 – Vertrek vanuit een vooraf ingevulde template, gevolgd door PSS-suggesties

```mermaid
sequenceDiagram
    actor HA as Huisarts
    participant ER as eReferral
    participant EPD as EPD
    participant PSS as PSS-engine
    participant T as Templates Klinische Biologie

    HA->>ER: Open templatemenu
    HA->>ER: Selecteer vooraf ingevulde template

    ER->>T: Laad geselecteerde template
    T-->>ER: Vooraf ingevulde template
    ER-->>HA: Toon vooraf ingevulde template

    PSS->>ER: Trigger aanvraag relevante patiëntcontext
    ER->>EPD: Doorzoek relevante patiëntgegevens
    EPD-->>ER: Diagnoses, symptomen, testhistoriek, leeftijd, ...
    ER-->>PSS: Bezorg beschikbare patiëntcontext
    PSS-->>ER: Kleurgecodeerde aanbevelingen

    ER-->>HA: Toon PSS-suggesties onderaan de template

    Note over HA,PSS: PSS-suggesties onderaan de template:<br/>Groen = sterk aanbevolen<br/>Geel = aanvaardbaar, niet optimaal<br/>Rood = niet aanbevolen, motivatie vereist<br/>(Een combinatie van kleuren of een selectie uit alle kleuren is ook mogelijk)

    alt Huisarts gebruikt PSS-aanbevelingen
        HA->>ER: Selecteer gele PSS-test
        ER->>T: Voeg gele test toe aan relevante template
        T-->>ER: Template bijgewerkt
        ER-->>HA: Toon bijgewerkte template

        opt Huisarts wijzigt testselectie verder
            HA->>ER: Selecteer bijkomende testen
            HA->>ER: Deselecteer geselecteerde of vooringevulde testen
            ER->>T: Werk template(s) bij
            T-->>ER: Template bijgewerkt
            ER-->>HA: Toon bijgewerkte template
        end

    else Huisarts gebruikt PSS niet
        HA->>ER: Behoud bestaande template

        opt Huisarts wijzigt template manueel
            HA->>ER: Selecteer bijkomende testen
            HA->>ER: Deselecteer testen
            ER->>T: Werk template(s) bij
        end
    end

    HA->>ER: Publiceer voorschrift
    ER-->>HA: Voorschrift gepubliceerd
```

---

## Actoren en componenten

| Actor / component | Rol |
|---|---|
| **Huisarts** | Voorschrijver |
| **eReferral** | Platform waarin het voorschrift wordt aangemaakt en gepubliceerd; haalt relevante patiëntcontext op uit het EPD wanneer dit door PSS wordt getriggerd |
| **EPD** | Bron van diagnoses, symptomen, testhistoriek, leeftijd en andere relevante patiëntgegevens |
| **PSS-engine** | Triggert eReferral om relevante patiëntcontext op te halen en genereert kleurgecodeerde aanbevelingen voor klinische biologie |
| **Templates Klinische Biologie** | Bloed, Microbiologie en Speciale vochten |

## PSS-kleurcodering

- **Groen** – sterk aanbevolen
- **Geel** – aanvaardbaar, niet optimaal
- **Rood** – niet aanbevolen; motivatie vereist
- **Combinaties zijn mogelijk** – de huisarts kan testen selecteren uit meerdere kleurcategorieën, inclusief alle drie
