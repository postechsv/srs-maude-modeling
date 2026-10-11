[SRS Numbering]
Split the SRS into sentences, number them, and keep only the sentences that state system requirements.

# Input
inputs/BTMS.txt

# Output
inputs/srs_numbered.yaml — in the following format, containing only the kept sentences.

sentences:
  - id: S-01
    text: "<sentence>"

# Conditions
1. Number every sentence of the SRS body in order as S-01, S-02, ..., before excluding any. Do not number the title.
2. Exclude a sentence if it states neither data the system keeps nor behavior the system performs, for example a sentence that only describes the background or purpose of the system. Keep its id unused in srs_numbered.yaml.
3. Copy each sentence exactly as written. Do not rephrase, merge, or split it further.
