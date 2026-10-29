# HIPAA, PHI, and Safe Harbor: a plain-language primer

> Legal statements cite official US government sources: the Code of Federal Regulations (CFR) and guidance from HHS, the US Department of Health and Human Services. Statements without a citation are examples or project analysis, not legal claims; footnotes say which. CFR text was checked against eCFR (version dated 2025-01-01). HHS guidance passages were checked against the official HHS page.

## What HIPAA protects

HIPAA is the US law about health information.[^privacy-rule] It restricts **PHI** (protected health information): health information that is **tied to a specific person**.[^phi-def] In a care company, PHI is held by the organization,[^held] and the information:

- **relates to someone's health, care, or payment for care.**[^health-prong] Even "this person is a client of a care company" counts.[^clinic]
- **can identify that person.**[^identify-prong]

Both parts have to be true.[^both] "A 40-year-old had a broken arm" is health information, but it doesn't identify anyone. "John Smith" identifies someone but says nothing about health. "John Smith had a broken arm" is PHI.[^examples]

## De-identification is the way out

If you remove the "identifies a person" part, the data is no longer PHI, and HIPAA doesn't restrict it.[^deid] You can then send it anywhere, including a model not covered by a BAA.[^baa]

The law gives two official ways to show data is de-identified:[^two-methods]

1. **Expert Determination.** A statistics expert analyzes the data and certifies that the risk of identifying anyone is "very small".[^expert] It's flexible, but it costs money and depends on judgment.[^practical]
2. **Safe Harbor.** You follow a fixed checklist.[^sh]

## Safe Harbor

A safe harbor is legal shorthand for "follow these exact steps and you're protected, no arguments."[^sh-meaning] Here the steps are:

**A. Remove all 18 types of identifiers** about the person, and about their relatives, employer and household:[^relatives]

- **Names**[^A]
- **Locations smaller than a state**, such as street, city, county or full ZIP. You can keep the first 3 digits of a ZIP code if that area has more than 20,000 people.[^B]
- **Dates other than the year**, such as birth, admission, discharge or death. Ages over 89 become "90 or older".[^C]
- **Contact details**: phone, fax, email[^DEF]
- **ID numbers**: SSN, medical record number, health plan ID, account number, license number[^GK]
- **Vehicle and device IDs**: license plates, serial numbers[^LM]
- **Web identifiers**: URLs, IP addresses[^NO]
- **Body and photo identifiers**: fingerprints, voiceprints, full-face photos[^PQ]
- **Any other unique number, characteristic, or code** (a catch-all)[^R]

**B. "No actual knowledge":** even after removing all of that, the result doesn't count if you *know* the remaining data could still identify someone.[^actual] An example is "the only 7-foot-tall patient in town".[^seven-foot]

Some catches:
- **Partial identifiers count.** Initials or the last 4 digits of an SSN are still identifiers.[^partial]
- **Plain hashes don't work.** Running a name through a hash function without a secret key still counts as an identifier.[^hash]
- **Free text gets no exemption.** A name buried in a case note counts the same as a name in a "name" column.[^freetext]

## How it relates to your project

Your project needs a checker that reads a transcript and decides whether it can go to the cheaper model. In legal terms, the question is: **"Is this transcript de-identified?"**[^q]

Safe Harbor is why that's possible:

- **It's the only part of the law that's already close to a checklist.** Most of HIPAA is vague, with phrases like "reasonable basis to believe".[^vague] Safe Harbor is a concrete list, so it's a natural starting point for Phase 1.[^analysis]
- **It's legally sufficient on its own.** A transcript that truly meets Safe Harbor isn't PHI.[^sufficient] So rules built on it meet your "at least as strict as the law" requirement automatically.[^analysis]
- **Phase 1 fills in the parts it leaves fuzzy.** These are the "any other unique characteristic" catch-all, the "actual knowledge" condition, combinations of harmless-looking facts, free text, and your company's own IDs. Each one needs a concrete rule.[^analysis]
- **It's stricter than necessary in places, and that's your cost gap.** For example, removing every exact date blocks some transcripts that probably wouldn't identify anyone. Those go to the expensive model. That's the "gap" the README says to track, and loosening it later, with the expert's approval, is where you'd save money.[^analysis]

The PHI checker is essentially Safe Harbor turned into a strict, testable checklist for transcripts.

---

## Sources

- **45 CFR 160.103** (definitions): https://www.ecfr.gov/current/title-45/section-160.103
- **45 CFR 164.502** (uses and disclosures): https://www.ecfr.gov/current/title-45/section-164.502
- **45 CFR 164.514** (de-identification): https://www.ecfr.gov/current/title-45/section-164.514
- **HHS, *Guidance Regarding Methods for De-identification of Protected Health Information*** ("HHS De-identification Guidance"): https://www.hhs.gov/hipaa/for-professionals/special-topics/de-identification/index.html ([PDF version](https://www.hhs.gov/sites/default/files/ocr/privacy/hipaa/understanding/coveredentities/De-identification/hhs_deid_guidance.pdf))
- **HHS, *Summary of the HIPAA Privacy Rule***: https://www.hhs.gov/hipaa/for-professionals/privacy/laws-regulations/index.html

[^privacy-rule]: HHS, *Summary of the HIPAA Privacy Rule*. The Privacy Rule implements the Health Insurance Portability and Accountability Act of 1996 (Pub. L. 104-191) and is codified at 45 CFR Part 160 and Part 164, Subparts A and E.

[^phi-def]: 45 CFR 160.103, "Protected health information": "*individually identifiable health information* ... (i) Transmitted by electronic media; (ii) Maintained in electronic media; or (iii) Transmitted or maintained in any other form or medium." The definition excludes FERPA education records, employment records held by a covered entity in its role as employer, and records about a person who has been deceased for more than 50 years.

[^held]: 45 CFR 160.103, "Individually identifiable health information": information that "(1) Is created or received by a health care provider, health plan, employer, or health care clearinghouse." "PHI" is that information as "transmitted or maintained" (see the note above). Whether a particular care company is covered depends on its role. It must be a covered entity, or a business associate acting for one.

[^health-prong]: 45 CFR 160.103, "Individually identifiable health information," paragraph (2): "Relates to the past, present, or future physical or mental health or condition of an individual; the provision of health care to an individual; or the past, present, or future payment for the provision of health care to an individual."

[^clinic]: HHS De-identification Guidance: "If such information was listed with health condition, health care provision or payment data, such as an indication that the individual was treated at a certain clinic, then this information would be PHI." HHS's example is a clinic. Applying it to a care company's client list is our reading of the same principle.

[^identify-prong]: 45 CFR 160.103, "Individually identifiable health information," paragraphs (2)(i)–(ii): information "(i) That identifies the individual; or (ii) With respect to which there is a reasonable basis to believe the information can be used to identify the individual."

[^both]: 45 CFR 160.103. The definition joins the health prong and the identification prong with "and". HHS De-identification Guidance gives a contrast example: "A health plan report that only noted the average age of health plan members was 45 years would not be PHI."

[^examples]: Illustrative examples, not taken from a source. They apply the two prongs defined in the notes above.

[^deid]: 45 CFR 164.514(a): "Health information that does not identify an individual and with respect to which there is no reasonable basis to believe that the information can be used to identify an individual is not individually identifiable health information." 45 CFR 164.502(d)(2): "The requirements of this subpart do not apply to information that has been de-identified in accordance with the applicable requirements of § 164.514." The exception is that disclosing a re-identification code counts as disclosing PHI, and re-identified data becomes PHI again.

[^baa]: Business associate assurances are required to disclose *PHI* (45 CFR 164.502(e)(1)(i): "A covered entity may disclose protected health information to a business associate ... if the covered entity obtains satisfactory assurance that the business associate will appropriately safeguard the information."). De-identified data is outside the subpart (see the note above). "Send it anywhere" is a statement about HIPAA only. Contracts (including your company's own BAAs with clients) and other laws may still restrict it.

[^two-methods]: 45 CFR 164.514(b)(1) and (b)(2). HHS De-identification Guidance names them "Expert Determination and Safe Harbor."

[^expert]: 45 CFR 164.514(b)(1): a qualified person "determines that the risk is very small that the information could be used, alone or in combination with other reasonably available information, by an anticipated recipient to identify an individual who is a subject of the information; and (ii) Documents the methods and results of the analysis that justify such determination."

[^practical]: Practical observation, not a legal statement.

[^sh]: 45 CFR 164.514(b)(2).

[^sh-meaning]: "Safe Harbor" is the name HHS uses for the 164.514(b)(2) method (HHS De-identification Guidance). The "no arguments" part reflects 164.514(b): a covered entity "may determine that health information is not individually identifiable health information only if" it meets (b)(1) or (b)(2). Meeting (b)(2) is therefore sufficient by itself, with no statistical analysis required.

[^relatives]: 45 CFR 164.514(b)(2)(i): "The following identifiers of the individual or of relatives, employers, or household members of the individual, are removed."

[^A]: 45 CFR 164.514(b)(2)(i)(A): "Names."

[^B]: 45 CFR 164.514(b)(2)(i)(B): "All geographic subdivisions smaller than a State, including street address, city, county, precinct, zip code, and their equivalent geocodes, except for the initial three digits of a zip code if, according to the current publicly available data from the Bureau of the Census: (1) The geographic unit formed by combining all zip codes with the same three initial digits contains more than 20,000 people; and (2) The initial three digits of a zip code for all such geographic units containing 20,000 or fewer people is changed to 000."

[^C]: 45 CFR 164.514(b)(2)(i)(C): "All elements of dates (except year) for dates directly related to an individual, including birth date, admission date, discharge date, date of death; and all ages over 89 and all elements of dates (including year) indicative of such age, except that such ages and elements may be aggregated into a single category of age 90 or older."

[^DEF]: 45 CFR 164.514(b)(2)(i)(D)–(F): telephone numbers, fax numbers, electronic mail addresses.

[^GK]: 45 CFR 164.514(b)(2)(i)(G)–(K): Social Security numbers, medical record numbers, health plan beneficiary numbers, account numbers, certificate/license numbers.

[^LM]: 45 CFR 164.514(b)(2)(i)(L)–(M): "Vehicle identifiers and serial numbers, including license plate numbers"; "Device identifiers and serial numbers."

[^NO]: 45 CFR 164.514(b)(2)(i)(N)–(O): "Web Universal Resource Locators (URLs)"; "Internet Protocol (IP) address numbers."

[^PQ]: 45 CFR 164.514(b)(2)(i)(P)–(Q): "Biometric identifiers, including finger and voice prints"; "Full face photographic images and any comparable images."

[^R]: 45 CFR 164.514(b)(2)(i)(R): "Any other unique identifying number, characteristic, or code, except as permitted by paragraph (c) of this section." Paragraph (c) allows a re-identification code only if it "is not derived from or related to information about the individual" and its key is kept secret.

[^actual]: 45 CFR 164.514(b)(2)(ii): "The covered entity does not have actual knowledge that the information could be used alone or in combination with other information to identify an individual who is a subject of the information." HHS De-identification Guidance describes actual knowledge as clear and direct knowledge. Mere awareness of re-identification research "does not mean it has 'actual knowledge'."

[^seven-foot]: Illustrative example, not taken from a source. HHS's own example is a record listing the patient's occupation as "current President of State University". HHS says that detail plus almost any other data, such as age or state of residence, "would clearly lead to an identification of the patient," so simply removing the 18 identifiers would not be enough (HHS De-identification Guidance).

[^partial]: HHS De-identification Guidance: a data set that contained patient initials, or the last four digits of a Social Security number, would not meet the requirement. This follows from 164.514(b)(2)(i)(R) and (c)(1).

[^hash]: HHS De-identification Guidance: "a code derived from a secure hash function without a secret key (e.g., 'salt') would be considered an identifying element." See also 45 CFR 164.514(c)(1), which allows a re-identification code only if it "is not derived from or related to information about the individual."

[^freetext]: HHS De-identification Guidance: "an identifier listed in the Safe Harbor standard must be removed regardless of its location in a record if it is recognizable as an identifier." HHS gives an example: "The free text field of a patient's medical record notes that the patient is the Executive Vice President of the state university. The covered entity must remove this information."

[^q]: Project analysis. It applies 45 CFR 164.514(a)–(b) and 164.502(d)(2) to the project's routing question.

[^vague]: For example, 45 CFR 160.103 ("reasonable basis to believe the information can be used to identify the individual") and 45 CFR 164.514(b)(1) ("the risk is very small"). Calling most of HIPAA vague is our characterization.

[^sufficient]: 45 CFR 164.514(b)(2) together with 164.502(d)(2). Information meeting (b)(2) "is considered not to be individually identifiable health information, i.e., de-identified."

[^analysis]: Project analysis, not a legal statement.
