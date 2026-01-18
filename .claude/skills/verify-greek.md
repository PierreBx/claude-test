# Verify Greek Translations

Skill for verifying and enhancing Ancient Greek translations in the Lecteur Multilingue app.

## Usage

```
/verify-greek <text-id>
/verify-greek <text-id>:<verse-num>
```

Examples:
- `/verify-greek 2` - Verify all verses in text ID 2 (Iliad)
- `/verify-greek 2:1` - Verify only verse 1 of text ID 2

## Instructions

When this skill is invoked, follow these steps:

### 1. Parse Arguments

Extract the text ID and optional verse number from the arguments:
- If format is `N` → verify entire text with id=N
- If format is `N:M` → verify only verse M in text id=N

### 2. Read Current Data

Read `/home/user/claude-test/dictionnaire.html` and extract:
- The target text from `defaultTexts` array
- All sentences/verses in scope
- All tokens (words) and their linked dictionary entries from `defaultWords`

For each word, collect:
- `token.form` - the inflected form in the text
- `token.lemma` - the dictionary form
- `token.wordId` - link to dictionary entry
- `token.formInfo` - current grammatical analysis
- `word.french` - French translation
- `word.translations.greek.word` - Greek lemma
- `word.translations.greek.meaning` - definition
- `word.translations.greek.romanized` - romanization if present

### 3. Verify Using Online Resources

For each word, query these resources (prioritize Homeric resources for Homer texts):

**Primary Resources:**
1. **Logeion** (logeion.uchicago.edu) - Use WebFetch to query:
   - `https://logeion.uchicago.edu/{greek_word}`
   - Contains LSJ, Middle Liddell, and Autenrieth (Homeric dictionary)

2. **Perseus Word Study Tool** - For morphological analysis:
   - `https://www.perseus.tufts.edu/hopper/morph?l={greek_word}&la=greek`

3. **Wiktionary** - For declensions/conjugations:
   - `https://en.wiktionary.org/wiki/{greek_word}`

**What to verify:**
- **Spelling/Accentuation**: Check if the Greek word is correctly spelled and accented
- **French Translation**: Verify the French translation is accurate for the context
- **Grammatical Form (formInfo)**: Verify case, number, gender, tense, mood, voice
- **Definition (meaning)**: Check if the meaning field is complete and accurate

### 4. Group Findings by Type

Organize all proposed changes into these categories:

#### A. Spelling/Accentuation Issues
Words with incorrect Greek spelling or accentuation.

#### B. Translation Improvements
French translations that could be more accurate or complete.

#### C. Grammar Enhancements
Improvements to `formInfo` fields (case, gender, number, tense, mood, voice).

#### D. Definition Updates
Missing or incomplete `meaning` fields in dictionary entries.

#### E. Missing Data
- Words without `formInfo`
- Dictionary entries without `meaning`
- Tokens not linked to dictionary entries

### 5. Present Findings to User

For each category, present the proposed changes in a clear format:

```
## Category: [Category Name]

### Word 1: {form} ({lemma})
**Current:** [current value or "missing"]
**Proposed:** [new value]
**Source:** [which resource confirmed this]
**Confidence:** [High/Medium/Low]

### Word 2: ...
```

After presenting all findings, use AskUserQuestion to let the user choose which changes to apply:
- Accept all changes in a category
- Accept individual changes
- Reject changes

### 6. Apply Accepted Changes

For accepted changes, edit `/home/user/claude-test/dictionnaire.html`:

- Update `formInfo` in token definitions within `defaultTexts`
- Update `meaning`, `word`, or other fields in `defaultWords`
- Use the Edit tool with precise old_string/new_string matching

After applying changes:
- Increment `DICT_VERSION`
- Summarize what was changed
- Offer to commit the changes

### 7. Special Considerations for Homer

When the text author is "Homère" or the text is from the Iliad/Odyssey:
- Prioritize Autenrieth's Homeric Dictionary (available via Logeion)
- Note Homeric/Epic Greek forms vs. Attic Greek
- Flag Epic dialect features in formInfo (e.g., "forme épique", "dialecte ionien")
- Be aware of:
  - Epic genitive in -οιο
  - Uncontracted forms
  - Tmesis (separated compound verbs)
  - Digamma traces
  - Patronymic formations (-ίδης, -ιάδης)

## Output Format

Always structure your response as:

1. **Scope Summary**: What text/verses are being verified
2. **Resources Consulted**: Which online resources were queried
3. **Findings by Category**: Grouped proposed changes
4. **User Decision**: Questions about which changes to apply
5. **Applied Changes**: Summary of what was modified
6. **Next Steps**: Offer to commit or verify more
