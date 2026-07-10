# Unified Namespace for WEBUILD Attestations

**Authors:**

- [Your Name/Affiliation], WEBUILD Consortium
- Jonas Toeniss, Brønnøysundregistrene (BRC), Norway

**Obsoletes:** N/A

**Obsoleted by:** N/A

## Context

WEBUILD relies on various attestations to facilitate interoperability between business wallets and relying parties. Currently, there is a number of namespaces in use as defined in the [Rulebooks](https://github.com/webuild-consortium/webuild-attestation-rulebooks-catalog/tree/main/rulebooks).

The namespaces range from WEBUILD bound namespaces such as `eu.we-build:companyinfo:1` to ARF namespaces on the pid: `urn:eudi:pid:1` which are to be used in production. 

The WEBUILD architecture assembly in June 2026 decided to extend the mandatory PID attributes as documented in the resulting [ADR](https://github.com/webuild-consortium/wp4-architecture/pull/223).

This means that we now have a collisison of definiton for the same identifier. The namespaces are supposed to be collision resistant in their assigment.
The ARF opens for national extensions of a scheme through a localiser `urn:eudi:pid:de:1`. 

This leads to three options: 
1. Keep the divergent namespaces and include a webuild specific localiser such as `wb` which is not assigned in ISO 3166-2  
2. Keep the divergent namespaces and include an increment the PID version to `urn:eudi:pid:2`.
3. Standardise the namespaces used in webuild. 

Option 1 and 2 keep us inline with the ARF definitions, but introduces a divergence from the expected production definitions. 
Option 3 would clearly indicate that the work belongs to and used in webuild, and not the production environment. This would also enforce the defintion of what webuild does and does not deliver ([as stated in WeBuild bluprint section 1.2.1](https://webuild-consortium.github.io/wp4-architecture/blueprint/blueprint.html#bridging-arf-gaps-through-specifications-and-testing)) and prevent possible future confusion. 



## Decision

WEBUILD will move all consortium-specific attestations to a dedicated, agreed-upon namespace.

This namespace will be for SD-JWT vcts : `urn:eu:webuild.<attestation-type>:<version-number>`
For mdoc the namespace will be `eu.webuild.<attestation-type>:<version-number>`


## Consequences

**Positive:**
- **Improved Interoperability:** A clear, unified namespace ensures that all participants have a common understanding of attestation claims, source of documentation, and usage area. Preventing any confusion between production and webuild environments. 
- **Explicit Scope:** Clearly distinguishes WEBUILD-specific work from production-ready EUDI ARF namespaces, reducing the risk of accidental misuse in production environments.

**Negative/Risks:**
- **Migration Effort:** Existing rulebooks and issuance/receiver implementations will need to be updated to support the new namespace.
- **Short-term Overhead:** Participants already using existing divergent namespaces will need to handle a transition period.

## Advice
