# PSS – eReferral Clinical Biology

This document visualizes three possible workflow variants for the integration of **Prescription Search Support (PSS)** within the **RIZIV-INAMI eReferral platform** for clinical biology prescriptions.

The Mermaid diagrams below are rendered directly by GitHub in Markdown.

---

## Scenario 1 – PSS-first: PSS test selection before the templates

```mermaid
sequenceDiagram
    actor GP as General Practitioner
    participant ER as eReferral
    participant EHR as EHR
    participant PSS as PSS engine
    participant T as Clinical Biology Templates

    GP->>ER: Start clinical biology prescription

    PSS->>ER: Trigger request for relevant patient context
    ER->>EHR: Search relevant patient data
    EHR-->>ER: Diagnoses, symptoms, test history, age, ...
    ER-->>PSS: Provide available patient context
    PSS-->>ER: PSS recommendations available

    ER-->>GP: Show PSS test selection

    Note over GP,PSS: PSS test selection shows recommended tests:<br/>Green = strongly recommended<br/>Yellow = acceptable, not optimal<br/>Red = not recommended, justification required<br/>(A combination of colors or selection from all colors is also possible)

    GP->>ER: Select desired PSS tests

    opt Red test selected
        ER-->>GP: Request justification
        GP->>ER: Provide justification
    end

    ER->>T: Distribute selected tests across the appropriate template(s)
    T-->>ER: Blood / Microbiology / Special Fluids
    ER-->>GP: Show prefilled templates

    opt GP modifies test selection
        GP->>ER: Add additional tests
        GP->>ER: Deselect suggested tests
        ER->>T: Update template(s)
    end

    GP->>ER: Publish prescription
    ER-->>GP: Prescription published
```

---

## Scenario 2 – Insufficient EHR data: manual diagnosis entry and three possible outcomes

```mermaid
sequenceDiagram
    actor GP as General Practitioner
    participant ER as eReferral
    participant EHR as EHR
    participant PSS as PSS engine
    participant T as Clinical Biology Templates

    GP->>ER: Start clinical biology prescription

    PSS->>ER: Trigger request for relevant patient context
    ER->>EHR: Search relevant patient data
    EHR-->>ER: Insufficient relevant information
    ER-->>PSS: Provide available patient context
    PSS-->>ER: No recommendation possible

    ER-->>GP: Offer manual diagnosis entry
    GP->>ER: Enter diagnosis manually

    GP->>ER: Click PSS button
    ER->>PSS: Request new PSS analysis using manual diagnosis
    PSS->>ER: Trigger request for updated patient context
    ER->>EHR: Search available EHR data
    EHR-->>ER: Available patient data
    ER-->>PSS: Provide EHR data + manually entered diagnosis

    alt OPTION 1 – GP selects PSS tests
        rect rgb(220, 235, 255)
            PSS-->>ER: Color-coded test recommendations
            ER-->>GP: Show PSS test selection

            Note over GP,PSS: Green = strongly recommended<br/>Yellow = acceptable, not optimal<br/>Red = not recommended, justification required<br/>(A combination of colors or selection from all colors is also possible)

            GP->>ER: Select green PSS test(s)
            ER->>T: Prefill selected tests
        end

    else OPTION 2 – GP selects no PSS tests
        rect rgb(235, 225, 250)
            PSS-->>ER: Color-coded test recommendations
            ER-->>GP: Show PSS test selection

            Note over GP,PSS: PSS provides recommendations,<br/>but the GP chooses not to select any tests

            GP->>ER: Continue without selecting PSS tests
        end

    else OPTION 3 – PSS still cannot generate a recommendation
        rect rgb(235, 235, 235)
            PSS-->>ER: No PSS recommendations available
            ER-->>GP: No PSS test selection available
        end
    end

    ER->>T: Open clinical biology templates
    T-->>ER: Blood / Microbiology / Special Fluids
    ER-->>GP: Show templates, with or without prefilled tests

    opt GP modifies templates
        GP->>ER: Select additional tests
        GP->>ER: Deselect tests
        ER->>T: Update template(s)
    end

    GP->>ER: Publish prescription
    ER-->>GP: Prescription published
```

---

## Scenario 3 – Start from a prefilled template, followed by PSS suggestions

```mermaid
sequenceDiagram
    actor GP as General Practitioner
    participant ER as eReferral
    participant EHR as EHR
    participant PSS as PSS engine
    participant T as Clinical Biology Templates

    GP->>ER: Open template menu
    GP->>ER: Select prefilled template

    ER->>T: Load selected template
    T-->>ER: Prefilled template
    ER-->>GP: Show prefilled template

    PSS->>ER: Trigger request for relevant patient context
    ER->>EHR: Search relevant patient data
    EHR-->>ER: Diagnoses, symptoms, test history, age, ...
    ER-->>PSS: Provide available patient context
    PSS-->>ER: Color-coded recommendations

    ER-->>GP: Show PSS suggestions at the bottom of the template

    Note over GP,PSS: PSS suggestions at the bottom of the template:<br/>Green = strongly recommended<br/>Yellow = acceptable, not optimal<br/>Red = not recommended, justification required<br/>(A combination of colors or selection from all colors is also possible)

    alt GP uses PSS recommendations
        GP->>ER: Select yellow PSS test
        ER->>T: Add yellow test to the relevant template
        T-->>ER: Template updated
        ER-->>GP: Show updated template

        opt GP further modifies test selection
            GP->>ER: Select additional tests
            GP->>ER: Deselect selected or prefilled tests
            ER->>T: Update template(s)
            T-->>ER: Template updated
            ER-->>GP: Show updated template
        end

    else GP does not use PSS
        GP->>ER: Keep existing template

        opt GP modifies template manually
            GP->>ER: Select additional tests
            GP->>ER: Deselect tests
            ER->>T: Update template(s)
        end
    end

    GP->>ER: Publish prescription
    ER-->>GP: Prescription published
```

---

## Actors and components

| Actor / component | Role |
|---|---|
| **General Practitioner** | Prescriber |
| **eReferral** | Platform in which the prescription is created and published; retrieves relevant patient context from the EHR when triggered by PSS |
| **EHR** | Source of diagnoses, symptoms, test history, age and other relevant patient data |
| **PSS engine** | Triggers eReferral to retrieve relevant patient context and generates color-coded clinical biology test recommendations |
| **Clinical Biology Templates** | Blood, Microbiology and Special Fluids |

## PSS color coding

- **Green** – strongly recommended
- **Yellow** – acceptable, not optimal
- **Red** – not recommended; justification required
- **Combinations are possible** – the GP may select tests from multiple color categories, including all three
