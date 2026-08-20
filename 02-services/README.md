# Service Analysis

## Objective

The objective of this phase was to perform a service-by-service analysis of the network services identified during the reconnaissance phase.

Each exposed service was examined individually to identify its purpose, version, configuration, exposed functionality and potential security weaknesses.

The analysis was performed against the intentionally vulnerable Metasploitable 2 system within the isolated laboratory environment.

## Methodology

The service analysis followed a structured approach:

1. Identify the service and version.
2. Enumerate the information exposed by the service.
3. Examine its configuration and accessible functionality.
4. Identify potential security weaknesses.
5. Assess whether the observed characteristics could represent a security risk.
6. Record relevant evidence for subsequent vulnerability assessment.

No exploitation of identified vulnerabilities was performed as part of this project.

## Services

The services identified during reconnaissance are analysed individually in the corresponding directories.

Each service directory contains the information gathered during its enumeration and analysis.

The analysis focuses on the security characteristics of the exposed service rather than simply documenting its presence.

## Scope

The service analysis covers network services exposed by the Metasploitable 2 target, including:

* Remote access services
* File transfer services
* Network file sharing
* Web services
* Database services
* Mail services
* Network management and RPC services
* Remote graphical access
* Application and middleware services

The specific services and findings are documented in their respective directories.

## Relationship with Vulnerability Assessment

Service analysis and vulnerability assessment are treated as related but separate stages.

The service analysis establishes **what is exposed and how it behaves**, while the vulnerability assessment evaluates **whether the observed characteristics represent identifiable security weaknesses and what their potential impact is**.

The findings from this phase therefore provide the technical basis for the subsequent vulnerability assessment.

## Evidence

Relevant command output and screenshots collected during the service analysis will be referenced where appropriate.

Evidence is stored separately in the project's `evidence/` directory to keep the service documentation focused and readable.

## Conclusion

The service-by-service analysis provides a detailed view of the Metasploitable 2 attack surface and establishes the technical basis for identifying and assessing security vulnerabilities in the following stages of the assessment.

