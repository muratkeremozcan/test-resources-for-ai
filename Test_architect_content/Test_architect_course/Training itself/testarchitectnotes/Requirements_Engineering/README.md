# REQUIREMENTS ENGINEERING

## Sentence Template for System Requirements
Given When Then

![Sentence Template for System Requirements](../slides/sentenceTemplateSystemRequirements.jpg)

## Characteristics of Good Requirements

![Characteristics of Good Requirements](../slides/goodRequirements.jpg)

## Requirements Traceability
![ Requirements Traceability](../slides/requirementsTraceability.jpg)

Tracing Usage:

|Analysis | Description | Process
|---|---|---|
| Derivation  | *Why is this here?*  | cost-benefit analysis |
| Impact  | *What if this changed?*  | change management  |
| Coverage  | *Have I covered all reqs?*  | management report   |

## NFRs / Quality Attributes

[Software Quality Characteristics pdf](./softwareQualityCharacteristics.pdf)

![ISO IEC 25010 Product Quality Model](../slides/ISO_IEC_25010.jpg)

### Step 1: Utility Tree
![Utility Tree](../slides/utilityTree.jpg)

### Step 2: Scenario Description Template / Workflows
To reduce ambiguity and increase testability, each measurable scenario in the Utility Tree gets described with this template.

![Scenario Description Template](../slides/scenarioDescriptionTemplate.jpg)

### Step 3: Refine Scenarios
![Scenario Refinement Table](../slides/scenarioRefinement.jpg)

### Step 4: Implement Design Strategies to support the NFRs
.. and use Design patterns when needed...

![Design Strategy Template](../slides/designStrategy.jpg)

![Design Strategy Example](../slides/designStrategyExample.jpg)
![Design Pattern Example](../slides/designPatternExample.jpg)

## Testing NFRs

![Testing NFRs](../slides/testingNFRs.jpg)
![NFR Testing Approaches](../slides/NFRtestingApproaches.jpg)

### KANO Model
For the Product Manager KANO is a tool to find the right mix between the  different feature types.
For the Test Architect KANO helps to identify risks.

![KANO Model](../slides/KANO_model.jpg)

## Configuration Management
Source Control, configurations, environment...

**Devops**:  continious integration, deployment, testing...

![Attributes of Configuration Management](../slides/configManagement.jpg)

## Change Request Management
**Change Request**: any issue that cannot go in a patch or point rev, has to go into the product asap.

**Change Management**: because requirements are incomplete, erroneous. ambiguous.

**Defect Management**: because tests reveal defects that need correction.

Tool -> Feature -> trace & Test

**CCB**:
![CCB](../slides/CCB.jpg)
