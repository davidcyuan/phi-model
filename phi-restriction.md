# How to check if model received PHI in one Session

## Format
- INPUT:
    - <TASK>: includes `request` + `data`
        - NOTE: `data` is very general. It could refer to database data, it could refer to an existing codebase.
        - FOR SIMPLICITY: we will assume that any model is working without accessing websites, downloading info outside of <TASK>, etc.
    - <TRANSCRIPT>: the full transcription of what was sent to a model

- OUTPUT:
    - <STATUS>: PASS / FAIL
        - Needs to be STRICLY NARROWER than a "court of law" interpretation.
        - The CLOSER the definition is to the actual court of law, the better we expect downstream performance
    - <EVIDENCE>: If failed, what fields failed it

- Specifications
    - TO KEEP THINGS SIMPLE (AND AS A SIDE EFFECT, HARDER), these Rules will NOT have human supervision / reviewing / labeling to assist. If this becomes too difficult, that is a topic for another day

## 18 failing identifiers

Applies to the individual AND their relatives, employers, and household members.

V = the forbidden list: every value inside the task's RECORDS that is
not provably SAFE. V_NAMES is the names on that list, V_LOCATIONS the
locations, and so on. Full explanation in "What V is".

Terms in CAPS (V_X, RECORDS, CODE, SAFE, MATCH, RESERVED_X, CHECK) are
defined in "How to check?".

NER ==  Named Entity Recognition

A. Names
   RULE:
   - FAIL if MATCH(v) for any v in V_NAMES
     - V_NAMES: client, relative, emergency contact, guarantor, household
       member names; names of the client's employer
     - V_NAMES comes from:
       - structured RECORDS: values of fields whose name matches the
         NAMES keywords (name, first, last, middle, employer, contact)
       - any RECORDS: person / organization names found by NER
     - derived forms: full name, first or last name alone, initials,
       nicknames (fixed nickname table), misspellings (edit distance
       <= 1 for names of 5+ characters)
     - a first or last name alone only fails if it is used as a name,
       not as a common word ("Rose" the person vs "rose" the flower)
   - request: FAIL if JUDGE finds a person name that may belong to a
     client, relative, employer, or household member; unsure -> FAIL
   CHECK: SCRIPT for V_NAMES; JUDGE for the request

B. Geographic subdivisions smaller than a state
   - street address, city, county, precinct, zip code, geocodes
   - exception: first 3 digits of a zip code may stay, if all zip codes
     sharing those 3 digits together contain more than 20,000 people
   - otherwise, the first 3 digits become 000
   RULE:
   - FAIL if MATCH(v) for any v in V_LOCATIONS
     - V_LOCATIONS: address, city, county, zip, coordinates, and
       facility names (a facility name points to a location)
     - V_LOCATIONS comes from:
       - structured RECORDS: values of fields whose name matches the
         LOCATIONS keywords (address, street, city, county, zip,
         postal, lat, lon, facility, location)
       - any RECORDS: places found by NER, zip codes and coordinates
         found by pattern
   - zip codes, anywhere in the transcript:
     - zip5 or zip+4 -> FAIL
     - zip3 -> PASS, unless it is in RESTRICTED_ZIP3, then it must
       appear as 000
   - request: FAIL if JUDGE finds a place smaller than a state that may
     be where a client lives, is treated, or works; unsure -> FAIL
   - state and country -> PASS
   CHECK: SCRIPT for V_LOCATIONS and zip codes; JUDGE for the request

C. Dates (except year) directly related to the individual
   - birth, admission, discharge, death, etc.
   - ages over 89, and any date element (including year) that reveals such
     an age, must be grouped into a single "90 or older" category
   RULE:
   - FAIL if any date element finer than a year, derived from V_DATES,
     appears: month, day, week, weekday, time of day
     - V_DATES: every date or timestamp in RECORDS (all dates in a
       record are assumed to be about the person in it)
     - V_DATES comes from:
       - structured RECORDS: every value that parses as a date or
         timestamp, whatever the field is called
       - unstructured RECORDS: every date found by a date parser
     - derived forms: any reformatting ("2024-03-05", "3/5/24",
       "March 5", "a Tuesday in March")
   - year alone -> PASS
   - FAIL if an age >= 90 appears, or a birth year that implies
     age >= 90; "90+" -> PASS
     - ages come from: fields whose name matches age keywords (age,
       years_old), and "N y/o" / "N years old" patterns in RECORDS
   - FAIL if an interval finer than a year, computed from two V_DATES
     dates, appears (v1, see open questions)
   - dates not from RECORDS (commit dates, build dates, today) -> PASS
   - request: FAIL if JUDGE finds a date that may be about a client;
     unsure -> FAIL
   CHECK: SCRIPT for V_DATES; JUDGE for the request

D. Telephone numbers
   RULE:
   - FAIL if MATCH(v) for any v in V_PHONES (digits only, ignore
     formatting)
     - V_PHONES comes from:
       - structured RECORDS: values of fields whose name matches the
         PHONES keywords (phone, mobile, cell, tel)
       - any RECORDS: phone numbers found by pattern
   - FAIL if a phone number pattern appears in the request, unless it
     is in RESERVED_PHONES
   CHECK: SCRIPT

E. Fax numbers
   RULE:
   - same as D; phone and fax look identical, so use one detector
     and one V (V_PHONES + V_FAXES)
     - V_FAXES comes from: values of fields whose name matches "fax";
       pattern hits are already in V_PHONES
   CHECK: SCRIPT

F. Email addresses
   RULE:
   - FAIL if MATCH(v) for any v in V_EMAILS
     - V_EMAILS comes from:
       - structured RECORDS: values of fields whose name matches the
         EMAILS keywords (email, mail)
       - any RECORDS: email addresses found by pattern
     - derived forms: the part before the @ alone (often holds a name)
   - FAIL if an email address appears in the request, unless its
     domain is in RESERVED_DOMAINS
   CHECK: SCRIPT

G. Social Security numbers
   RULE:
   - FAIL if MATCH(v) for any v in V_SSNS
     - V_SSNS comes from:
       - structured RECORDS: values of fields whose name matches the
         SSNS keywords (ssn, social, tin)
       - any RECORDS: SSN-shaped numbers found by pattern
     - derived forms: the last 4 digits alone
   - FAIL if any 9-digit number in SSN shape (###-##-#### or #########)
     appears anywhere in the transcript
   CHECK: SCRIPT

H. Medical record numbers
   RULE:
   - FAIL if MATCH(v) for any v in V_MRNS
     - V_MRNS comes from:
       - structured RECORDS: values of fields whose name matches the
         MRNS keywords (mrn, medical_record, chart)
       - other MRNs are caught as V_OTHER_IDS (every ID-like value
         in RECORDS is forbidden anyway)
     - derived forms: any 4+ character suffix
   CHECK: SCRIPT

I. Health plan beneficiary numbers
   RULE:
   - FAIL if MATCH(v) for any v in V_PLAN_IDS (member IDs, subscriber
     IDs, group + member combinations)
     - V_PLAN_IDS comes from:
       - structured RECORDS: values of fields whose name matches the
         PLAN_IDS keywords (member, subscriber, policy, beneficiary,
         insurance_id)
       - any RECORDS: Medicare Beneficiary Identifiers found by their
         published format
       - other plan IDs are caught as V_OTHER_IDS
   - FAIL if an MBI-format value appears anywhere in the transcript
   CHECK: SCRIPT

J. Account numbers
   RULE:
   - FAIL if MATCH(v) for any v in V_ACCOUNTS (billing, bank, card,
     portal account numbers)
     - V_ACCOUNTS comes from:
       - structured RECORDS: values of fields whose name matches the
         ACCOUNTS keywords (account, acct, card, iban, routing)
       - any RECORDS: card numbers found by pattern + Luhn check
       - other account numbers are caught as V_OTHER_IDS
     - derived forms: any 4+ digit suffix
   - FAIL if a card number (13-19 digits passing the Luhn check)
     appears anywhere in the transcript
   CHECK: SCRIPT

K. Certificate / license numbers
   RULE:
   - FAIL if MATCH(v) for any v in V_LICENSES (driver's license, state
     ID, professional license, birth / death certificate numbers)
     - V_LICENSES comes from:
       - structured RECORDS: values of fields whose name matches the
         LICENSES keywords (license, licence, dl, certificate,
         state_id)
       - no reliable format; others are caught as V_OTHER_IDS
   CHECK: SCRIPT

L. Vehicle identifiers and serial numbers, including license plates
   RULE:
   - FAIL if MATCH(v) for any v in V_VEHICLES
     - V_VEHICLES comes from:
       - structured RECORDS: values of fields whose name matches the
         VEHICLES keywords (vin, plate, vehicle)
       - any RECORDS: VINs found by pattern (17 characters, no I/O/Q,
         valid check digit)
       - plates have no reliable format; others are caught as
         V_OTHER_IDS
   - FAIL if a VIN appears anywhere in the transcript
   CHECK: SCRIPT

M. Device identifiers and serial numbers
   RULE:
   - FAIL if MATCH(v) for any v in V_DEVICES (serial numbers, UDIs,
     IMEIs, MAC addresses, advertising IDs, push tokens)
     - V_DEVICES comes from:
       - structured RECORDS: values of fields whose name matches the
         DEVICES keywords (device, serial, imei, mac, udi, token)
       - any RECORDS: IMEIs (15 digits, Luhn), MAC addresses, and
         UUIDs found by pattern
   - FAIL if an IMEI or MAC address appears anywhere in the transcript
   - UUIDs outside RECORDS -> PASS (they are everywhere in code)
   CHECK: SCRIPT

N. URLs
   RULE:
   - FAIL if MATCH(v) for any v in V_URLS (personal sites, social
     profiles)
     - V_URLS comes from:
       - structured RECORDS: values of fields whose name matches the
         URLS keywords (url, website, link, profile)
       - any RECORDS: URLs found by pattern
   - FAIL if any URL contains a value from V in its path or query
     (e.g. /clients/48213, ?email=...)
   - FAIL if a URL in the request matches a social profile pattern
     (facebook.com/<x>, linkedin.com/in/<x>, instagram.com/<x>, ...)
   - other URLs (docs, packages, internal endpoints) -> PASS
   CHECK: SCRIPT

O. IP addresses
   RULE:
   - FAIL if MATCH(v) for any v in V_IPS
     - V_IPS comes from:
       - structured RECORDS: values of fields whose name matches the
         IPS keywords (ip, ip_address, remote_addr)
       - any RECORDS: IPv4 / IPv6 addresses found by pattern
   - FAIL if an IP address appears in the request, unless it is in
     RESERVED_IPS
   CHECK: SCRIPT

P. Biometric identifiers, including finger and voice prints
   RULE:
   - FAIL if MATCH(v) for any v in V_BIOMETRICS
     - V_BIOMETRICS comes from:
       - structured RECORDS: values of fields whose name matches the
         BIOMETRICS keywords (fingerprint, biometric, voice, retina,
         face_template)
       - files: every audio file in RECORDS (by file type, not content;
         voice recordings are voiceprints)
       - matching: by file hash, plus audio fingerprint so trimmed or
         re-encoded copies still match
   CHECK: SCRIPT

Q. Full-face photos and any comparable images
   RULE:
   - FAIL if MATCH(v) for any v in V_PHOTOS
     - V_PHOTOS comes from:
       - files: every image / video / scanned PDF in RECORDS, whatever
         it shows (faces, wounds, documents, signatures)
       - structured RECORDS: values of fields whose name matches the
         PHOTOS keywords (photo, image, avatar, scan, picture)
       - matching: by file hash, plus perceptual hash so resized or
         cropped copies still match
   - images in CODE (diagrams, icons) -> PASS
   CHECK: SCRIPT

R. Any other unique identifying number, characteristic, or code
   - exception: re-identification codes (see section 3)
   RULE:
   - FAIL if MATCH(v) for any v in V_OTHER_IDS
     - V_OTHER_IDS: every value in RECORDS that is not SAFE and not
       already in another V_X (client IDs, case numbers, claim
       numbers, keys, unknown fields, ...)
     - V_OTHER_IDS comes from:
       - structured RECORDS: every non-SAFE value
       - unstructured RECORDS: every token that is not a common
         dictionary word, and every number with 4+ digits
   - FAIL if MATCH(v) for any v in V_FREE_TEXT (any run of 5+
     consecutive tokens; v1, very strict)
     - V_FREE_TEXT comes from: every text value in RECORDS with 5+
       tokens, and the full text of unstructured RECORDS
   - re-identification codes: not handled yet, treated as identifiers
   CHECK: SCRIPT

## How to check?

### No humans at run time

A human may review these rules (and the fixed lists inside them) to see
how well they fit the law. When the rules run, nobody is in the loop:
every decision is made by a script, or by a model (JUDGE). Anything the
rules cannot prove SAFE is treated as forbidden.

### The main problem: whose identifier is it?

Safe Harbor only cares about identifiers OF THE INDIVIDUAL (and their
relatives, employers, household). Transcripts are full of names, dates,
URLs, IPs that belong to nobody relevant (a developer in a git log,
today's date, a docs URL). Failing every name or date would fail almost
every transcript.

So every rule is:

    FAIL  <=>  some value in TRANSCRIPT has
                 Category(value) is one of the 18   -- what kind of thing it is
             AND Subject(value)              -- it belongs to an individual

Category is mostly easy (formats, NER). Subject is the hard part.

### Solving Subject with provenance

Because the model only sees <TASK>, every identifier in the transcript
must come from `request`, from `data`, or be made up by the model. So
Subject is decided by WHERE a value came from, not by judgment:

- a value from RECORDS (data about people) -> treated as the
  individual's
- a value from CODE (software, docs) -> not the individual's, unless
  it looks like a record (see "Where to scan with what")
- a value from the request -> JUDGE

At a care company, being in the data already means "is a client", which
is health information. So any identifier from RECORDS is treated as
PHI, without judging the health part separately.

### RECORDS vs CODE

Every item in `data` is sorted by a fixed rule on its type:

- CODE: source code, config, markup, schema definitions, docs
  (by file type: .py, .ts, .java, .sql DDL, .md, .yaml config, ...)
- RECORDS: everything else, e.g. query results, CSV / JSON / Excel
  data, log files, PDFs, office documents, emails, images, audio,
  database dumps, and any item whose type is unknown

Data-format files (CSV, JSON data, logs, dumps) are RECORDS even when
they sit inside a code repository; test fixtures are often copies of
real data.

### What V is

V is the FORBIDDEN LIST for one task: every value inside that task's
RECORDS that the rules cannot prove SAFE.

The checker's main job is then simple: search the transcript for
anything on that list. If something on the list shows up, FAIL.

V is split by category, so a failure can say what kind of thing
leaked:

- V_NAMES = the names on the list
- V_LOCATIONS = the locations on the list
- V_DATES = the dates on the list
- ... one per category, V_NAMES to V_OTHER_IDS, plus V_FREE_TEXT

Example. The task's `data` contains one row from the clients table:

    id     | name        | dob        | phone        | diagnosis
    48213  | Maria Lopez | 1961-03-05 | 918-555-2741 | E11.9

Then V for that task is:

    V_OTHER_IDS = { "48213" }          (not SAFE, not in another category)
    V_NAMES     = { "Maria Lopez" }    (field name matches "name")
    V_DATES     = { "1961-03-05" }     (value parses as a date)
    V_PHONES    = { "918-555-2741" }   (field name + phone pattern)

"E11.9" is not in V, because it is a valid ICD-10 code, and codes from
public code systems are SAFE. A diagnosis code alone does not identify
anyone.

If the transcript contains "Maria", "03/05/61", or "9185552741", it
FAILS: those are derived forms of values in V (see MATCH below).
If the transcript contains only "E11.9" and "born 1961", it PASSES.

### Where V comes from

V is built per task, from the RECORDS inside <TASK> only. Not from the
whole database: values that are not in <TASK> cannot reach the
transcript, so they do not need to be in V.

Step 1. Sort every `data` item into RECORDS or CODE (see above).
        CODE adds nothing to V.

Step 2. Collect every value in RECORDS:
        - structured RECORDS (rows, CSV, JSON): every field value
        - unstructured RECORDS (notes, PDFs, emails, logs): every
          token, plus every span found by the pattern detectors and
          NER
        - files (images, audio, video): the file itself

Step 3. Drop every value that is SAFE (see below).

Step 4. Put every remaining value into a category, first match wins:
        a. a pattern detector recognizes it (phone, email, SSN, date,
           IP, URL, card, VIN, IMEI, MAC, MBI) -> that category
        b. its field name matches a category's KEYWORDS (listed in
           each rule above) -> that category
        c. NER tags it as a person / organization -> V_NAMES,
           as a place -> V_LOCATIONS
        d. it is an image / video / audio file -> V_PHOTOS /
           V_BIOMETRICS
        e. otherwise -> V_OTHER_IDS
        Text values with 5+ tokens also go into V_FREE_TEXT.

        The category only decides which derived forms are checked and
        what the EVIDENCE says. It never makes a value SAFE.

V = all V_X together.

### SAFE

A value is SAFE (left out of V) only if it is one of these. The list is
fixed and part of the rules:

- empty, null, true / false
- a number with at most 3 digits before the decimal point (counts,
  scores, measurements), unless it is an age >= 90 (see C)
- the year of a date, unless it implies an age >= 90 (see C)
- a valid code from a public code system: ICD-10-CM, ICD-10-PCS,
  CPT / HCPCS, LOINC, NDC, RxNorm, SNOMED CT
- a value in SAFE_VOCAB, a short fixed list of generic values (male,
  female, yes, no, active, inactive, unknown, pending, ...)
- a US state or country name or code
- a 3-digit zip prefix not in RESTRICTED_ZIP3

Everything else in RECORDS is forbidden. There is no "looks harmless"
exception: "President of State University" is all common words, but
it is not SAFE.

### Other definitions

- MATCH(v): v, or a derived form of v, appears in the transcript.
  Derived forms always include:
  - normalization: case, whitespace, punctuation, formatting
  - encodings: base64, hex, URL-encoding
  - hashes without a secret key (HHS: these still count)
  - category-specific partials (listed in each rule)

- KEYWORDS: the field-name word lists in each rule above, matched
  case-insensitively against parts of the field name
  (`client_first_name` matches "first" and "name").

- RESERVED_X: fixed lists of values that are reserved for examples
  or private use, so they cannot belong to anyone:
  - RESERVED_PHONES: 555-0100 to 555-0199
  - RESERVED_DOMAINS: example.com / .org / .net, *.test, *.example,
    *.invalid, *.localhost
  - RESERVED_IPS: 10/8, 172.16/12, 192.168/16, 127/8, 0/8,
    192.0.2/24, 198.51.100/24, 203.0.113/24, IPv6 equivalents

- RESTRICTED_ZIP3: 3-digit zip prefixes with 20,000 or fewer people,
  from Census data. Fixed list, part of the rules.

- JUDGE: a model that answers one narrow question about a span of
  request text ("may this name belong to a client, relative,
  employer, or household member?"). It must answer FAIL when unsure.

- CHECK, how a rule is checked:
  - SCRIPT: deterministic code (patterns, parsers, NER, lookups)
  - JUDGE: needs the JUDGE model

- EVIDENCE for each failure: category, matched text, location in the
  transcript, and which V value / pattern it matched.

### Where to scan with what

- Model-generated text: only MATCH against V and identifiers found in
  the request. The model cannot know identifiers that are not in
  <TASK>, so pattern hits in its own output are made up. This avoids
  failing on fake example phone numbers, etc.
- `request`: pattern detectors + JUDGE. It has no fields, so V cannot
  cover identifiers a human typed in ("fix Mrs. Smith's account").
- CODE: a value only fails if it looks like a record: 2 or more
  identifier types next to each other (same line, object, or row),
  e.g. a name + a birth date, or a name + a phone number. A single
  support email in a config file passes; a hard-coded patient in a
  test (name + DOB) fails.

### Open questions

- Date intervals (C): does "stayed 4 days" count? v1 says FAIL.
- Staff / provider names and IDs (A, K): in RECORDS they are forbidden
  like everything else. Too strict? They are not the individual's
  under the list, but could identify a client in context (only
  caregiver of one client).
- Free text (R): v1 fails all of it. This is the most expensive rule
  and the first candidate to loosen.
- SAFE is small on purpose. Every value added to it (e.g. medication
  names, lab values, common categorical fields) makes downstream work
  better; each one needs to be argued against the law.
- One-row tasks: values that are categorical across many clients
  ("Type 2 diabetes") are still forbidden if they are not on a SAFE
  list. Should frequency across rows count as proof of SAFE?
- Fuzzy name thresholds (A): edit distance and nickname table need
  tuning against real false positives.
- RECORDS vs CODE by file type: synthetic test data in CSV / JSON is
  treated as RECORDS, so fake names in fixtures will fail tasks.
- "No actual knowledge" and re-identification codes: ignored for now.
