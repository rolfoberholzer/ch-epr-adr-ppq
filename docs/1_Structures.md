# 1. Structure of policies and policy sets

Each policy or policy set has the following elements:

* ID of the policy or policy set — must be a UUID in URN format.
* A set of so-called "target constraints" determining whether a given policy or policy set shall be evaluated when
  making the access decision for a given request. These constraints can apply to:
    * Resources — objects being tried to be accessed or manipulated,
    * Actions — actions being tried to be performed on the resources,
    * Subjects — users trying to perform these actions,
    * Environments — circumstances under which the request occurs.
* An optional additional condition.
* Effect, i.e. whether the access shall be denied or granted when all constraints are met.
* ID of the combining algorithm which determines how the Authorization Decision Provider shall combine effects
  of all evaluated policies and policy sets when making the final access decision. In EPR, this value is
  always `urn:oasis:names:tc:xacml:1.0:rule-combining-algorithm:deny-overrides`, which means that a decision to deny
  the access overrules any other decisions.

Additionally, policy sets may reference and/or embed other policies or policy sets that will be evaluated recursively.

## 1.1. Structure of target constraints

This subsection describes how target constraints are defined in XACML policies and policy sets. In these descriptions,
the placeholder `${Type}` will be used to represent one of the following values, correspondingly to the possible
objects of target constraints:

* `Resource`,
* `Action`,
* `Subject`,
* `Environment`.

The XACML element `Target` of a policy or policy set contains sub-elements named after plural
forms of `${Type}`, i.e. `Resources`, `Actions`, `Subjects`, and `Environments`. Each of them consists
from *1–n* sub-elements `${Type}` (in singular), and they, on their part —
from *1–m* sub-elements `${Type}Match`. For example:

```xml

<xacml:PolicySet xmlns:xacml="urn:oasis:names:tc:xacml:2.0:policy:schema:os"
                 PolicyCombiningAlgId="urn:oasis:names:tc:xacml:1.0:policy-combining-algorithm:deny-overrides"
                 PolicySetId="urn:uuid:58bbfa76-4d65-4fa1-b0af-c862b52a20d4" Version="1.0">
    <xacml:Target>
        <xacml:Subjects>
            <xacml:Subject>
                <xacml:SubjectMatch.../>
            </xacml:Subject>
            <xacml:Subject>
                <xacml:SubjectMatch.../>
                <xacml:SubjectMatch.../>
                <xacml:SubjectMatch.../>
            </xacml:Subject>
        </xacml:Subjects>
        <xacml:Resources>
            <xacml:Resource>
                <xacml:ResourceMatch.../>
            </xacml:Resource>
        </xacml:Resources>
        <xacml:Environments>
            <xacml:Environment>
                <xacml:EnvironmentMatch.../>
                <xacml:EnvironmentMatch.../>
            </xacml:Environment>
        </xacml:Environments>
    </xacml:Target>
    <xacml:PolicySetIdReference>urn:e-health-suisse:2015:policies:tralala</xacml:PolicySetIdReference>
</xacml:PolicySet>
```

All elements `${Type}Match` have the same purpose and the same structure — they define, how a so-called
"designated attribute" (a part of the policy evaluation context, e.g. a value contained in the CH:ADR request)
shall be matched against a specified expected value. In that way, each of the elements `${Type}Match` contains:

* A match ID, i.e. a name of the comparator function to be applied — attribute `@MatchID`.
* A fixed expected value of the designated attribute — sub-element `AttributeValue`. It will
  be used as the first parameter of the comparator function.
* A definition of the designated attribute, consisting from the attribute ID and data type — sub-element
  `${Type}AttributeDesignator`. The actual value of this attribute will be determined dynamically when evaluating the
  policy or policy set and used as the second parameter of the comparator function. The data type must match the one of
  the expected value.

For example, the following definition prescribes to apply the function `urn:hl7-org:v3:function:II-equal`
(comparator of HL7v3 instance identifiers) on two parameters of the type `urn:hl7-org:v3#II` — the value of the
evaluation context attribute named `urn:e-health-suisse:2015:epr-spid` and the instance identifier
`761337612483458721` with the assigning authority OID `2.16.756.5.30.1.127.3.10.3`.

```xml

<xacml:ResourceMatch MatchId="urn:hl7-org:v3:function:II-equal">
    <xacml:AttributeValue DataType="urn:hl7-org:v3#II">
        <InstanceIdentifier xmlns="urn:hl7-org:v3" extension="761337612483458721"
                            root="2.16.756.5.30.1.127.3.10.3"/>
    </xacml:AttributeValue>
    <xacml:ResourceAttributeDesignator AttributeId="urn:e-health-suisse:2015:epr-spid" DataType="urn:hl7-org:v3#II"/>
</xacml:ResourceMatch>
```

The order of sub-elements `${Type}Match` inside of an element `${Type}` as well as the order of elements `${Type}`
inside of `${Type}s` does not matter.

There is a logical conjunction among sub-elements `${Type}Match` of the same `${Type}`, and a logical
disjunction among elements `${Type}` of the same policy or policy set. In that way, a policy set will become relevant
for producing an access decision only if all sub-elements `SubjectMatch` in at least one element `Subject` of this
policy set will resolve to `true`, and the same for `Action`, `Resource`, and `Environment`.

Elements `SubjectMatch` used in EPR policies and policy sets can be furthermore classified as matchers of:

* group IDs (OIDs),
* subject IDs (GLNs, EPR-SPIDs, representative's IDs),
* subject ID qualifier codes (person ID types),
* subject role codes,
* purpose of use codes.

Each of them has a specific fixed combination of match ID, designated attribute, and data type. Depending on the target
subject type, an element `Subject` may contain up to three elements `SubjectMatch` addressing different criteria
of the subject identification — for example, the following definition will match a user in the role "patient",
who has an ID "761384759483458721" of the type "EPR-SPID":

```xml

<xacml:Subject>
    <xacml:SubjectMatch MatchId="urn:oasis:names:tc:xacml:1.0:function:string-equal">
        <xacml:AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">761384759483458721
        </xacml:AttributeValue>
        <xacml:SubjectAttributeDesignator AttributeId="urn:oasis:names:tc:xacml:1.0:subject:subject-id"
                                          DataType="http://www.w3.org/2001/XMLSchema#string"/>
    </xacml:SubjectMatch>
    <xacml:SubjectMatch MatchId="urn:oasis:names:tc:xacml:1.0:function:string-equal">
        <xacml:AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">urn:e-health-suisse:2015:epr-spid
        </xacml:AttributeValue>
        <xacml:SubjectAttributeDesignator AttributeId="urn:oasis:names:tc:xacml:1.0:subject:subject-id-qualifier"
                                          DataType="http://www.w3.org/2001/XMLSchema#string"/>
    </xacml:SubjectMatch>
    <xacml:SubjectMatch MatchId="urn:hl7-org:v3:function:CV-equal">
        <xacml:AttributeValue DataType="urn:hl7-org:v3#CV">
            <CodedValue xmlns="urn:hl7-org:v3" code="PAT" codeSystem="2.16.756.5.30.1.127.3.10.6"/>
        </xacml:AttributeValue>
        <xacml:SubjectAttributeDesignator AttributeId="urn:oasis:names:tc:xacml:2.0:subject:role"
                                          DataType="urn:hl7-org:v3#CV"/>
    </xacml:SubjectMatch>
</xacml:Subject>
```
