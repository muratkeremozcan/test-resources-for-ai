# TESTING & QUALITY 

# Value of Testing
### What is Testing? : investigation of the SUT to provide **information** that results in **improvements**.
### The 5 Dimensions of Testing

>Coverage: an assessment for the thoroughness or completeness of testing with respect to our test model - *Paul Gerrard*

![5 dimensions of Testing](../slides/dimensions.jpg)

### ROI of Testing
![ROI of testing](../slides/roi_of_testing.jpg)

### Cost of Quality
![Cost_of_quality](../slides/Cost_of_quality.jpg)

# Test Strategy

![testStrategy](../slides/testStrategy.jpg)

Testing serves a purpose (*test mission*) that has goals (*test policy*) 
and requires a map (*test strategy*).

![testPolicy](../slides/testPolicy.jpg)

### Test levels – V model with architecture testing
![V mdoel with architecture](../slides/v_model_with_architecture_testing.jpg)

# Risk Based Testing
Risk Profile

![risk profile](../slides/riskProfile.jpg)

Risk Based Testing Worksheet. You can download the worksheet in xls .
![RBT Workseet](../slides/RiskBasedTesting.jpg)


Relations in RBT worksheet
![RBT summary](../slides/RiskBasedTestingSUMMARY.jpg)

# Design for Testability

Goal : Controllable, Observable, Reliable : *Instrinsic Testability* . [More On Heuristics of Testability](http://www.satisfice.com/tools/testability.pdf)

Why : reduce the cost of testing, diagnosis, maintenance.

Who : system, software and test architects

How : TDD, Loose Coupling, Inversion of control, SOLID, follow the best practices of [clean code & architecture](https://clean-code-developer.com/weitere-infos/solid/).

![Testability](../slides/testability.jpg)

# Test Exit Criteria
![Test Exit Criteria](../slides/testExitCriteria.jpg)

# Performance Testing and Scalability
![Performance Testing](../slides/performanceTesting.jpg)

At a certain load, the response time sky-rockets.

![Scalability Testing](../slides/scalabilityTesting.jpg)

# TDD
From Req. to unit test level.
The most effective way of specifying something is to describe how you would test it.

# Test Design Techniques
[Test Design Techniques pdf](./testDesignTechniques.pdf)
* **Black-box**: req. based, workflow, statistical/markov, eq.class & boundary value, state-based, combinatorial, model based
* **Gray-box**: interfaces between components, services, systems
* **White-box**: statement, branch, path: cyclomatic complexity *(Edges-Nodes-2 = independent paths)*
* **Fault-based**: exploratory, fuzzing, mutation. [Data Type Attacks and Web Tests pdf](./dataTypeAttacks.pdf)
* **Regression**: Risk Based Testing, testing firewall (re-test parts influenced by changes)

![Test Design Tecniques](../slides/testDesignTechniques.jpg)

## [Test Automation Patterns website](http://testautomationpatterns.wikispaces.com)
[Test Automation Design Patterns paper](http://testautomationpatterns.wikispaces.com/)

## Test Environment
**Test environment**: test rig

**Test infrastructure**: test rig + tools + office network etc.

**Test suite architecture**: test  levels

![Microservice Test Architecture](../slides/microserviceArchitecture.jpg)


# Internal Quality
Negative efects:
* Slows development with unplanned activities
* Rising cost of  maintenance, new features, change
* Rising cost of regression testing, system testing for hotfixes
* Rising cost of onboarding
* Complex & risky integration

![Internal vs External Quality](../slides/internal&ExternalQuality.jpg)


## Technical Debt
Lack of internal quality reseults in technical debt.

![Technical Debt](../slides/technicalDebt.jpg)

## Measuring and Driving Internal Quality
To measure internal quality
* Static code analysis, linters etc.
* Req. trace
* On-boarding feedback
* Visualize with tools,  reviews
* Test gap analysis
* Automated document analysis

![DSM](../slides/DSM.jpg)

To drive internal quality, you must monetize it:

![monetize](../slides/monetize.jpg)

## Test Code & Architecture Quality Management

Test Code Quality at different levels:

| | | |
| --- | --- | --- |
| Micro | code | tools
| Macro | hacky code | review
| Architecture | UML | review, some tools for architecture analysis

![Test Code Quality](../slides/STCE.jpg)