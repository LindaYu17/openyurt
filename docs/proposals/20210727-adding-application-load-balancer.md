---
title: Proposal Template
authors:
  - "@lindayu17"
  - "@gnunu"
reviewers:
  - "@YYY"
creation-date: 2021-07-27
last-updated: yyyy-mm-dd
status: provisional
---

# A compute resource based application load balancer  

## Table of Contents

A table of contents is helpful for quickly jumping to sections of a proposal and for highlighting
any additional information provided beyond the standard proposal template.
[Tools for generating](https://github.com/ekalinin/github-markdown-toc) a table of contents from markdown are available.

- [Title](#title)
  - [Table of Contents](#table-of-contents)
  - [Glossary](#glossary)
  - [Summary](#summary)
  - [Motivation](#motivation)
    - [Goals](#goals)
    - [Non-Goals/Future Work](#non-goalsfuture-work)
  - [Proposal](#proposal)
    - [User Stories](#user-stories)
      - [Story 1](#story-1)
      - [Story 2](#story-2)
    - [Requirements (Optional)](#requirements-optional)
      - [Functional Requirements](#functional-requirements)
        - [FR1](#fr1)
        - [FR2](#fr2)
      - [Non-Functional Requirements](#non-functional-requirements)
        - [NFR1](#nfr1)
        - [NFR2](#nfr2)
    - [Implementation Details/Notes/Constraints](#implementation-detailsnotesconstraints)
    - [Risks and Mitigations](#risks-and-mitigations)
  - [Alternatives](#alternatives)
  - [Upgrade Strategy](#upgrade-strategy)
  - [Additional Details](#additional-details)
    - [Test Plan [optional]](#test-plan-optional)
  - [Implementation History](#implementation-history)

## Glossary

Refer to the [Cluster API Book Glossary](https://cluster-api.sigs.k8s.io/reference/glossary.html).

## Summary

Other than traditional load balancer in Kubernetes, this application load balancer collects system metrics information, such as CPU usage, GPU usage, memory, etc., and dispatches requests with heavy workloads to proper POD according to available resources and system loads, user defined rules, if present, will be taken into consideration as well.

## Motivation

Some workloads, such as video analytics, cloud gaming are of sustaining resource comsuming. Reuqests for such kind of workloads should not be distributed randomly, so this application load balancer is designed to cope with this situation, it dispatches requests to proper PODs on proper nodes with sufficient resources to make sure optimal performance.

### Goals

- Collect system metrics through metrics monitoring services such as Prometheus;
- Analyze and verify the requests and match them with cluster's system capabilities;
- Dispatch requests to proper PODs according to specific algorithms and policies.

### Non-Goals/Future Work

- Metrics services for OpenYurt is not part of this proposal.

## Proposal

The application load balancer will be implemented as an operator, and its CRD definition lists below:

```go
type UseCase struct {
    UseCaseClass            string `json:"useCaseClass,omitempty"`      //AI, media, gaming...
    UseCaseName             string `json:"useCaseName"`                 //detect, classify...
    DeploymentName          string `json:"deploymentName"`
    Device                  string `json:"device,omitempty"`            //CPU, GPU...
    Policy                  string `json:"policy,omitempty"`            //squeeze, manual
    HorizontalScalable      bool   `json:"horizontalScalable,omitempty"`
}

type LoadBalancerSpec struct {
    Usecase                 UseCase `json:"usecase"`
}
//    UseCaseClass            string `json:"useCaseClass,omitempty"`        //AI, media, gaming...
//    UseCaseName             string `json:"useCaseName,omitempty"`         //detect, classify...
//    DeploymentName          string `json:"deploymentName,omitempty"`
//    Device                  string `json:"device,omitempty"`              //CPU, GPU...
//    Policy                  string `json:"policy,omitempty"`              //squeeze, manual
//    HorizontalScalable      bool   `json:"horizontalScalable,omitempty"`

type LoadBalancerStatus struct {
    CurUsecase              UseCase `json:"curUsecase,omitempty"`
    AllUsecases             []UseCase `json:"allUsecases,omitempty"`
}

ConfigMap {

}
```

Development plan: hopefully this feature can be implementated and merged into OpenYurt Release 0.8.

### User Stories

#### Story 1

Requests of workloads will be dispatched to proper PODs according to specific algorithms and policies.

#### Story 2

Users want to be able to customize its own request dispatching rules.

#### Story 3

User want to be able to decide the target device for workloads.

### Requirements (Optional)

#### Functional Requirements

##### FR1

Application load balancer controller collects use case descriptions, user defined rules from CR, the PODs and the nodes' basic information, and store them into ConfigMap. 

##### FR2

Application load balancer receives requests from ingress of the cluster, it analyzes and verifies the requests and matches them with the cluster's system capabilities.

##### FR3

Application load balancer reads the mounted volume of the ConfigMap which includes the basic information of the PODs and the nodes in order to get the realtime metrics data from metrics server through metrics API.

##### FR4

TBD:---------Based on the metrics data and the dispatching algorithm, application load balancer distributes the request to the proper PODs of nodes.-------

#### Non-Functional Requirements

##### NFR1

We suppose metrics service is working correctly in OpenYurt nodepool environment, so Metrics services for OpenYurt is not part of this proposal.

##### NFR2

### Implementation Details/Notes/Constraints

The application load balancer consists of two parts, a load balancer controller and a load balancer worker. The load balancer controller is used for PODs and nodes' monitoring, and save related information to a ConfigMap for further usage, while the load balancer worker dispatches the requests of workloads to appropriate PODs according to metrics data.

### Risks and Mitigations

- What are the risks of this proposal and how do we mitigate? Think broadly.
- How will UX be reviewed and by whom?
- How will security be reviewed and by whom?
- Consider including folks that also work outside the SIG or subproject.

## Alternatives

The `Alternatives` section is used to highlight and record other possible approaches to delivering the value proposed by a proposal.

## Upgrade Strategy

If applicable, how will the component be upgraded? Make sure this is in the test plan.

Consider the following in developing an upgrade strategy for this enhancement:
- What changes (in invocations, configurations, API use, etc.) is an existing cluster required to make on upgrade in order to keep previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing cluster required to make on upgrade in order to make use of the enhancement?

## Additional Details

### Test Plan [optional]

## Implementation History

- [ ] MM/DD/YYYY: Proposed idea in an issue or [community meeting]
- [ ] MM/DD/YYYY: Compile a Google Doc following the CAEP template (link here)
- [ ] MM/DD/YYYY: First round of feedback from community
- [ ] MM/DD/YYYY: Present proposal at a [community meeting]
- [ ] MM/DD/YYYY: Open proposal PR

