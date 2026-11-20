# Legal and data protection

Read for step 1 when the right to analyse is unclear, when personal data is involved, or when the user is in the EU. This is orientation for the agent and the user, not legal advice; for a commercial bridge the user should have a lawyer confirm the basis once.

## Reverse engineering for interoperability

- **EU**: Article 6 of Directive 2009/24/EC allows a licensed user to decompile a program when that is indispensable to obtain the information needed for interoperability with an independently created program, the information is not readily available, and the work is confined to the parts needed. Article 8 makes contract terms that exclude this void. The information obtained may serve only that interoperability: not other goals, and not a program substantially similar in its expression. In Germany this is § 69e UrhG.
- **US**: 17 U.S.C. § 1201(f) permits circumventing a technical measure for the sole purpose of interoperability under narrow conditions; licence terms and other law still apply. Treat any circumvention as out of scope for this skill.
- Reading data the customer already owns, through access the customer's administrator grants, is the strongest position. Decompiling is for understanding the data layer, never for copying the vendor's code into the bridge.

Record in `BRIDGE.md`: who holds the licence, who authorised the work, that the vendor offers no adequate interface (with what was checked), and which parts of the program were analysed.

## Personal data

- **GDPR**: the customer is the controller. Whoever runs or maintains the bridge for them is usually a processor and needs a data processing agreement (Art. 28). Health data is a special category (Art. 9); minimise fields, keep data on site, and encrypt anything that leaves it.
- **Germany, professional secrecy**: doctors, lawyers, tax advisers and others are bound by § 203 StGB. Service providers who get access to such data must be bound to confidentiality in writing (§ 203 (3) and (4) StGB).
- **US health data**: a bridge that touches PHI for a covered entity requires a Business Associate Agreement under HIPAA.

`DATA-PROTECTION.md` lists: every personal-data field the bridge can return, every destination (local only, a named client, a cloud service), retention of logs and caches, where the bridge runs, and the agreements in place.

## Fixtures and screenshots

Replace personal data before anything leaves the test machine: fixtures, logs, issue reports, screenshots and messages to an AI model. Test installations restored from a real backup contain real patients and customers.
