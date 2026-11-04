# Signal the attestation legal category using ETSI TS 119 472-1

**Authors:**

- Jonas Toennis, Brønnøysund Registry Center, Norway

## Context

Annex V point (a) and Annex VII point (a) of [Regulation (EU) 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj) require a QEAA and a PuB-EAA to carry an indication, in a form suitable for automated processing, of how they were issued. Relying Parties depend on this signal, so it has to be spelled and encoded the same way in every attestation.

Three sources currently disagree on how to express it:

| Source | Status | Attribute | Values |
| --- | --- | --- | --- |
| [ETSI TS 119 472-1 V1.2.1](https://www.etsi.org/deliver/etsi_ts/119400_119499/11947201/01.02.01_60/ts_11947201v010201p.pdf), clause 4.2.2 and format clauses | Clauses 2 to 6 referenced from Annex II of Implementing Regulation (EU) 2024/2979, as replaced by [Implementing Regulation (EU) 2026/1731](https://eur-lex.europa.eu/eli/reg_impl/2026/1731/oj) | `category` | `urn:etsi:esi:eaa:eu:qualified`, `urn:etsi:esi:eaa:eu:pub`. Non-qualified EAA **shall not** include it (EAA-5.2.2.1-01, EAA-6.2.2.1-01) |
| [ARF v3.0.0](https://eudi.dev/latest/annexes/annex-2/annex-2.02-high-level-requirements-by-topic/) Topic 12, ARB_25 (cited in the catalog as EW-DM-12-029) | Non-binding guidance | `attestation_legal_category` | As set by the Attestation Rulebook template |
| [EU rulebook template v1.5](https://github.com/eu-digital-identity-wallet/eudi-doc-attestation-rulebooks-catalog/blob/main/template/attestation-rulebook-template.md) and the WE BUILD generic template | Templates | `category` | The two ETSI URNs, plus an invented `eaa:eu:non-qualified`, which contradicts ETSI ([open EU issue #44](https://github.com/eu-digital-identity-wallet/eudi-doc-attestation-rulebooks-catalog/issues/44)) |

ETSI does not rename or remove `attestation_legal_category`: the name never occurs in TS 119 472-1. The attribute comes from the ARF, while ETSI defines its own `category` attribute with URN values for two categories only. A non-qualified EAA is signalled by the absence of the attribute.

### Current state of the catalog

At commit `4cade93` of the [attestation rulebooks catalog](https://github.com/webuild-consortium/webuild-attestation-rulebooks-catalog), the generic template says `attestation_legal_category` SHALL NOT be used, but all 41 rulebooks still use it, 39 of them recording a deviation or open point for the rulebook quality assurance group.

| Aspect | What is in use |
| --- | --- |
| Attribute in rulebooks | `attestation_legal_category`, `attestationLegalCategory`, `issuer.attestation_legal_category`, `legal_category` |
| Attribute in schemas | Legacy attribute in 30 of 32 schemas and 32 of 35 samples; the ETSI `category` attribute in none |
| Values allowed by schemas | `EAA` (17), `QEAA` (14), `PUB-EAA` (5), `non-qualified-EAA` (5), `PuB-EAA` (2), `PubEAA` (1) |
| PuB-EAA spellings, all files | `PuB-EAA` (176), `Pub-EAA` (21), `PUB-EAA` (17), `PubEAA` (6), `pubEAA` (2), `urn:etsi:esi:eaa:eu:pub` (3) |
| Non-qualified spellings, all files | `non-qualified EAA` (268), `non-qualified-EAA` (94), `Non-qualified EAA` (12), `EAA` as a value |
| Stated migration target | 31 rulebooks name `eaa:eu:non-qualified`, which ETSI forbids |
| PID | The rulebook definition says optional and its encoding table says `PID`, while its example and both schemas say `PUB-EAA`, and both schemas require the attribute |
| EBWOID | `TO AGREE` in the template; `QEAA` or `PUB-EAA` in the schema |

These differences break interoperability, not just style: a verifier matching `PuB-EAA` exactly rejects `PUB-EAA`, and no QEAA or PuB-EAA built from these schemas would conform to ETSI TS 119 472-1 V1.2.1, because none of them defines the ETSI `category` attribute.

### Options considered

| Option | Assessment |
| --- | --- |
| A. Keep `attestation_legal_category` and only normalise its spelling | Follows the ARB_25 wording, but a QEAA or PuB-EAA would still lack the `category` that ETSI requires. Rejected. |
| B. Carry both attributes | Conforms during migration, but leaves two signals that can disagree and keeps the verifier ambiguity. Rejected. |
| C. Use `category`, with `eaa:eu:non-qualified` for non-qualified EAA | Matches both templates, and gives an explicit signal as the ARB_12 SHOULD suggests, but breaks EAA-5.2.2.1-01 and EAA-6.2.2.1-01. The value is not a URN and is not defined by ETSI. Rejected. |
| D. Use `category` exactly as ETSI specifies, and omit it where ETSI requires | Conforms to the referenced standard and gives one signal. Deviates from the literal ARB_25 and ARB_12 wording, which is addressed below. **Chosen.** |

Waiting for the ARF or the EU template to settle was also rejected: migration only gets more expensive as rulebooks and pilots mature, and option D keeps a stable result for QEAA and PuB-EAA whichever way those discussions end.

## Decision

WE BUILD uses the ETSI TS 119 472-1 `category` attribute as the only machine-readable signal of the legal category. This applies to all rulebooks, schemas, samples, and examples in the catalog, and to WE BUILD issuers and verifiers.

### Values

| Legal category | `category` value | Rule |
| --- | --- | --- |
| QEAA | `urn:etsi:esi:eaa:eu:qualified` | SHALL be present |
| PuB-EAA | `urn:etsi:esi:eaa:eu:pub` | SHALL be present |
| Non-qualified EAA, including self-issued (S-EAA) | none | SHALL NOT be present, wherever the issuer is established |
| PID | none | SHALL NOT be present; a PID is identified by its `vct` or doctype and its PID Provider |

- `attestation_legal_category`, `legal_category`, `issuer.attestation_legal_category`, and `eaa:eu:non-qualified` SHALL NOT be used. No rulebook may define another value.
- ETSI limits the prohibition to issuers registered in the European Union. WE BUILD applies it to all issuers so that behaviour does not differ between Member States, EEA countries, and third countries.
- EBWOID is an attestation type, not a legal category. An EBWOID takes the value matching how it was issued: QEAA or PuB-EAA. If it is issued as a non-qualified EAA, `category` is omitted. This closes the `TO AGREE` entry in the generic template.
- A rulebook allowing several categories lists the applicable URNs and states which issuers use which.

### Encoding

| Format | Encoding |
| --- | --- |
| SD-JWT VC | Top-level `category` claim of type string (clause 5.2.2), not selectively disclosable |
| ISO/IEC mdoc | `category` data element of type `tstr` in namespace `org.etsi.01947201.010101` (clause 6.2.2) |
| W3C VCDM | An element of the `type` array (clause 7.2.2), not a separate property. The `attestationLegalCategory` vocabulary term is not used as a signal |

Non-disclosure of the SD-JWT VC claim is a rulebook rule taken from the EU template, not an ETSI requirement. ISO/IEC mdoc has no always-disclosed data elements, so a Relying Party that needs the signal SHALL request it. Clause 7 is outside the clauses referenced by the implementing regulation, so W3C VCDM follows it as a consortium profile.

In JSON Schema, a QEAA or PuB-EAA rulebook adds `category` to `required` and constrains the value with `const` or `enum`. A non-qualified EAA or PID schema sets `"category": false` under `properties`.

### Verification

The `category` value is an assertion by the issuer. It does not prove the issuer's status by itself. Relying Parties:

1. SHALL accept `urn:etsi:esi:eaa:eu:qualified` or `urn:etsi:esi:eaa:eu:pub` only if the issuer's status is confirmed through the [trusted lists](trusted-lists.md).
2. SHALL treat an attestation without `category` as a non-qualified EAA, unless it is a PID, even if the issuer is a QTSP or public body.
3. SHALL NOT treat any other value as a QEAA or PuB-EAA, and SHALL ignore `attestation_legal_category` for trust decisions.

### Written form

Rulebooks, schema descriptions, and WE BUILD documents use these forms:

| Concept | Use | Do not use |
| --- | --- | --- |
| The concept | legal category | attestation category, legal status, legal classification |
| Qualified | QEAA, QEAAs | qualified EAA, Qualified EAA |
| Public sector | PuB-EAA, PuB-EAAs | Pub-EAA, PUB-EAA, PubEAA, pubEAA |
| Non-qualified | non-qualified EAA, at the start of a sentence Non-qualified EAA | non-qualified-EAA, or EAA where non-qualified is meant |
| Self-issued | S-EAA, defined once as a self-issued non-qualified EAA | non-qualified self EAA |

The rulebook header `Category:` remains a label for people and the catalog explorer. It takes `PID`, `EBWOID`, `QEAA`, `PuB-EAA`, `non-qualified EAA`, or `S-EAA`, or several of these separated by commas. It is not an attribute value.

Other attributes named `category` keep their meaning and are not affected, such as the SD-JWT VC Type Metadata `category` of TS12 (`urn:eu:europa:ec:eudi:sua:sca`, see [CS-12](../conformance-specs/cs-12-sca-payments.md)) and `entity.category` in the ownership and control schemas. Rulebooks that use a Type Metadata `category` SHALL keep it distinct from the legal category.

### Relation to the ARF

The ETSI requirements are met through a referenced standard under an implementing regulation, while the ARF is non-binding. Where they differ, WE BUILD follows ETSI. Annex V point (a) and Annex VII point (a) remain met, as the notes to clause 4.2.2 state. ARB_11, ARB_12, and ARB_25 are recorded as met or deviated from in this ADR once, so rulebooks replace their individual deviation notes with a reference here. WE BUILD raises the difference with the ARF editors and supports EU issue #44.

### Migration

1. Correct the generic template: remove `eaa:eu:non-qualified`, set the non-qualified EAA and EBWOID rows, and refine the mdoc disclosure text.
2. Update all schemas and samples, and change their versions according to catalog practice.
3. Update all rulebooks: chapters 2 to 5, examples, verification sequences, compliance tables, and deviation notes. The PID verification sequence moves to `vct` or doctype and the PID Provider.
4. Add a catalog check that fails on the forbidden attributes, values, and spellings, except where this ADR or a changelog quotes them.
5. Ask the WP4 Semantics group to deprecate `attestationLegalCategory` in the EBW vocabulary.

New rulebooks and schemas apply this ADR immediately. Issuers apply it at their next rulebook version. Attestations issued earlier remain valid until they expire or are re-issued, which verification rule 3 accommodates for non-qualified attestations and PID.

This ADR is reviewed if ETSI publishes EN 319 472-1 (in [early draft](https://portal.etsi.org/Portal_WI/Form1.asp?SupCrit=ESI+++++++++++++++++++++++19472-169600&PersonId=0&TbId=0&TabId=&SubTB=&WIcritID=18&NbToDisplay=30) as version 1.2.2), if the ARF changes ARB_11, ARB_12, or ARB_25, or if an implementing act changes the referenced clauses.

## Consequences

What becomes easier?

- Attestations can conform to the standard referenced by the implementing regulation.
- Verifiers apply one rule across all rulebooks, with a fixed set of values.
- Rulebook authors no longer need to record the same deviation, and the rulebook quality assurance group can close it.
- EBWOID and PID no longer need a value of their own.

What becomes more difficult?

- All 41 rulebooks and most schemas and samples change, and this is a breaking change for issuers and verifiers built on them. Schema changes alone do not update deployed pilots.
- A non-qualified EAA has no positive signal. Absence cannot distinguish it from an attribute that was omitted in error.
- WE BUILD knowingly deviates from the wording of ARB_25 and the ARB_12 recommendation, and from the EU template until issue #44 is resolved.

How do we address the risks introduced by this change?

- Absence is safe by default: it never grants QEAA or PuB-EAA treatment, and a schema requiring `category` catches omission for those categories at issuance.
- A rulebook or Relying Party needing to recognise a specific non-qualified EAA uses its `vct` or doctype and issuer, not a category value. Any scheme-specific signal uses the scheme's own namespace outside `category`.
- The migration is carried out in the catalog in the order above, so template, schemas, and rulebooks do not diverge again, and the catalog check prevents regression.
- If the ARF or EU template settles differently, only the review above is triggered. The QEAA and PuB-EAA values already come from the referenced standard.

## Advice

Once merged, this is our consortium’s decision. This does not mean all
participants agree it is the best possible decision. In the decision
making process, we have heard the following advice.
