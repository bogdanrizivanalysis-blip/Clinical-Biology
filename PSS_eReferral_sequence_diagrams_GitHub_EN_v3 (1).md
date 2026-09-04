# PSS – eReferral Clinical Biology

This document visualizes three workflow variants for integrating **Prescription Search Support (PSS)** into the **RIZIV-INAMI eReferral platform** for clinical biology prescriptions.

The revised workflows incorporate the review feedback by:

- retrieving the same standard baseline for every patient without a PSS-triggered EHR request;
- making indication selection and manual indication entry explicit;
- retrieving additional PSS parameters from the EHR on demand;
- returning the selected tests and any required justification to PSS;
- allowing the prescriber to continue without selecting a PSS-recommended test;
- showing PSS recommendations prominently before the clinical biology templates; and
- using templates as a manual fallback when PSS is unavailable, rather than as the starting point of the PSS workflow.

The Mermaid diagrams below are rendered directly by GitHub.

---

## Scenario 1 – PSS-first with sufficient baseline data

```mermaid
sequenceDiagram
    actor GP as General Practitioner
    participant ER as eReferral
    participant EHR as EHR
    participant PSS as PSS engine
    participant T as eReferral template repository

    GP->>ER: Start clinical biology prescription

    ER->>EHR: Retrieve standard baseline data
    Note over ER,EHR: Standard baseline for every patient:<br/>age, gender, problem list and active medication
    EHR-->>ER: Standard baseline data

    ER-->>GP: Show suspected indications and prescription purpose
    GP->>ER: Select screening, monitoring or diagnosis
    GP->>ER: Confirm, select or manually add indication(s)

    ER->>PSS: Request additional data requirements<br/>using baseline and selected indication(s)
    PSS-->>ER: On-demand parameters and linked patient variables
    ER->>EHR: Retrieve available on-demand parameters
    EHR-->>ER: Available matched patient data
    ER-->>GP: Show prefilled additional parameters

    opt Prescriber completes or overrides parameters
        GP->>ER: Add or modify parameter values
    end

    ER->>PSS: Submit completed patient context
    PSS-->>ER: Color-coded test recommendations
    ER-->>GP: Show PSS recommendations prominently

    Note over GP,PSS: Green = strongly recommended<br/>Yellow = acceptable, not optimal<br/>Red = not recommended, justification required<br/>Tests from one or more color categories may be selected

    alt Prescriber selects one or more PSS tests
        GP->>ER: Select desired PSS tests<br/>(green and/or yellow and/or red)

        opt At least one red test is selected
            ER-->>GP: Request justification
            GP->>ER: Provide justification
        end

        ER->>PSS: Submit selected tests and justification, if applicable
        PSS-->>ER: Selection feedback recorded

        ER->>T: Distribute selected tests across relevant template(s)
        T-->>ER: Blood, Microbiology and/or Special Fluids
        ER-->>GP: Show templates prefilled with selected tests

    else Prescriber selects no PSS tests
        GP->>ER: Continue without selecting a PSS test
        ER->>PSS: Submit no-selection outcome
        ER->>T: Open clinical biology templates
        T-->>ER: Blood, Microbiology and Special Fluids
        ER-->>GP: Show templates without PSS-prefilled tests
    end

    opt Prescriber modifies the test selection
        GP->>ER: Select additional tests
        GP->>ER: Deselect selected or prefilled tests
        ER->>T: Update template(s)
        T-->>ER: Template(s) updated
        ER-->>GP: Show updated template(s)
    end

    GP->>ER: Publish prescription
    ER-->>GP: Prescription published
```

---

## Scenario 2 – Insufficient baseline data: manual indication entry and three outcomes

```mermaid
sequenceDiagram
    actor GP as General Practitioner
    participant ER as eReferral
    participant EHR as EHR
    participant PSS as PSS engine
    participant T as eReferral template repository

    GP->>ER: Start clinical biology prescription

    ER->>EHR: Retrieve standard baseline data
    Note over ER,EHR: Standard baseline for every patient:<br/>age, gender, problem list and active medication
    EHR-->>ER: Incomplete baseline data<br/>or no usable indication

    ER-->>GP: Offer manual indication entry
    GP->>ER: Select screening, monitoring or diagnosis
    GP->>ER: Enter indication or suspected diagnosis manually

    ER->>PSS: Request additional data requirements<br/>using available baseline and manual indication
    PSS-->>ER: On-demand parameters and linked patient variables
    ER->>EHR: Retrieve available on-demand parameters
    EHR-->>ER: Available matched patient data
    ER-->>GP: Show available prefilled parameters<br/>and fields requiring manual completion

    opt Prescriber completes or overrides parameters
        GP->>ER: Add or modify parameter values
    end

    ER->>PSS: Submit completed patient context

    alt OPTION 1 - PSS generates recommendations and tests are selected
        rect rgb(220, 235, 255)
            PSS-->>ER: Color-coded test recommendations
            ER-->>GP: Show PSS recommendations prominently

            Note over GP,PSS: Green = strongly recommended<br/>Yellow = acceptable, not optimal<br/>Red = not recommended, justification required<br/>Tests from one or more color categories may be selected

            GP->>ER: Select desired PSS tests<br/>(green and/or yellow and/or red)

            opt At least one red test is selected
                ER-->>GP: Request justification
                GP->>ER: Provide justification
            end

            ER->>PSS: Submit selected tests and justification, if applicable
            PSS-->>ER: Selection feedback recorded

            ER->>T: Distribute selected tests across relevant template(s)
            T-->>ER: Blood, Microbiology and/or Special Fluids
            ER-->>GP: Show templates prefilled with selected tests
        end

    else OPTION 2 - PSS generates recommendations but no tests are selected
        rect rgb(235, 225, 250)
            PSS-->>ER: Color-coded test recommendations
            ER-->>GP: Show PSS recommendations prominently
            GP->>ER: Continue without selecting a PSS test
            ER->>PSS: Submit no-selection outcome

            ER->>T: Open clinical biology templates (option 2)
            T-->>ER: Blood, Microbiology and Special Fluids
            ER-->>GP: Show templates without PSS-prefilled tests
        end

    else OPTION 3 - PSS cannot generate a recommendation
        rect rgb(235, 235, 235)
            PSS-->>ER: No PSS recommendations available
            ER-->>GP: Explain that no PSS recommendation is available

            ER->>T: Open clinical biology templates (option 3)
            T-->>ER: Blood, Microbiology and Special Fluids
            ER-->>GP: Show templates without PSS-prefilled tests
        end
    end

    opt Prescriber modifies or completes the test selection
        GP->>ER: Select additional tests
        GP->>ER: Deselect selected or prefilled tests
        ER->>T: Update template(s)
        T-->>ER: Template(s) updated
        ER-->>GP: Show updated template(s)
    end

    GP->>ER: Publish prescription
    ER-->>GP: Prescription published
```

---

## Scenario 3 – PSS unavailable: manual-template fallback

```mermaid
sequenceDiagram
    actor GP as General Practitioner
    participant ER as eReferral
    participant EHR as EHR
    participant PSS as PSS engine
    participant T as eReferral template repository

    GP->>ER: Start clinical biology prescription

    ER->>EHR: Retrieve standard baseline data
    Note over ER,EHR: Standard baseline for every patient:<br/>age, gender, problem list and active medication
    EHR-->>ER: Available baseline data

    ER-->>GP: Show suspected indications and prescription purpose
    GP->>ER: Select screening, monitoring or diagnosis
    GP->>ER: Confirm, select or manually add indication(s)

    ER->>PSS: Request PSS data requirements or recommendations
    PSS--xER: PSS unavailable or request timed out
    ER-->>GP: Explain that PSS is unavailable<br/>and offer the manual fallback

    GP->>ER: Open clinical biology template menu
    ER->>T: Load current clinical biology templates
    T-->>ER: Blood, Microbiology and Special Fluids
    ER-->>GP: Show templates for manual completion

    GP->>ER: Select desired tests manually

    opt Prescriber modifies the test selection
        GP->>ER: Select additional tests
        GP->>ER: Deselect tests
        ER->>T: Update template(s)
        T-->>ER: Template(s) updated
        ER-->>GP: Show updated template(s)
    end

    Note over GP,PSS: No PSS recommendations are shown in fallback mode

    GP->>ER: Publish prescription
    ER-->>GP: Prescription published
```

---

## Actors and components

| Actor / component | Role |
|---|---|
| **General Practitioner** | Prescriber who selects the prescription purpose and indication, reviews PSS recommendations, selects tests and publishes the prescription |
| **eReferral** | Orchestrates the prescription workflow, retrieves baseline and on-demand patient data from the EHR, displays PSS recommendations and captures the prescriber's choices |
| **EHR** | Source of standard baseline data and available on-demand patient parameters |
| **PSS engine** | Determines which additional parameters are needed, generates color-coded recommendations and records selection feedback |
| **Template repository (eReferral)** | Provides the current Blood, Microbiology and Special Fluids templates used after PSS selection or as a manual fallback |

## PSS color coding

- **Green** – strongly recommended
- **Yellow** – acceptable, not optimal
- **Red** – not recommended; justification required
- **Combinations are possible** – the prescriber may select tests from multiple color categories, including all three

## Modelling assumption to validate

The diagrams model the **clinical biology template repository as an eReferral-managed component**, rather than as content stored in the EHR. This makes template ownership explicit in response to the review question, but the final ownership and maintenance responsibility should be confirmed with the solution architect and business owner.
