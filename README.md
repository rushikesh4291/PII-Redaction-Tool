PII Redaction Pipeline: Red Herring Prospectus
1. Approach & Architecture
To process the 128-page PDF, I utilized a hybrid NLP and rules-based architecture using Microsoft Presidio, spaCy, and Faker (localized to en_IN).
The Brain (NER): I  upgraded the NLP engine from spaCy's standard model to the RoBERTa Transformer (en_core_web_trf). This was necessary to  identify regional Indian names contextually without relying on hardcoded lists.
The Rules (Regex): For structured PII, I used custom PatternRecognizer instances into Presidio's registry. I wrote an ultra-permissive regex pattern to capture Indian phone numbers that were fragmented by unsual OCR line-breaks and irregular spacing (e.g., + 91 20 \n 45053237).
Consistent Replacement: I implemented a global Hash Map (pii_mapping). When an entity is detected, the script strips whitespace and stores the Faker generated replacement. This guarantees that "Sarthak Malvadkar" consistently maps to the exact same synthetic name across all 128 pages.
2. Precision, Recall, & Explicit Choices (Trade-offs)
Financial prospectuses are dense with capitalized legal terms that confuse standard NER models. Managing the Precision vs. Recall trade-off was the primary challenge.
Optimizing Recall (False Negatives): Initially, the standard NLP model missed key personnel and loosely formatted phone numbers. By upgrading to a Transformer model and injecting the permissive regex, Recall was maximized.
Optimizing Precision (Explicit Choices on False Positives): The Transformer model initially over-redacted capitalized legal jargon .
 Explicit Choice: I treated standard financial terminology as non-sensitive to maintain the document's readability. To enforce this, I built a post-analysis filter leveraging a financial_jargon_allowlist that removes these false positives from Presidio's results before the replacement phase occurs.
3. Evaluation Approach
Because evaluating 340,000 characters manually is prone to human error, I evaluated the pipeline using a two-pronged approach:
Automated Validation: I wrote a post-redaction Python script that scanned the final output using strict Regular Expressions for high-risk Indian PII (Aadhaar, PAN, Passport). The script confirmed 0 leaks of these structures.
High-Density Qualitative Review: I isolated the first 2,000 characters of the document and ran a line-by-line visual diff to ensure perfect contextual replacement without structural formatting destruction.
4. Code Extensibility
The code was structured for modular scalability. Extending this pipeline to detect a new custom PII type requires zero changes to the core logic. You simply:
Define the regex Pattern.
Wrap it in a PatternRecognizer defining the new supported_entity (e.g., TICKET_NUM).
Add it to the Presidio RecognizerRegistry.
Add a single elif statement in the get_fake_value generator to dictate how Faker should replace it.

