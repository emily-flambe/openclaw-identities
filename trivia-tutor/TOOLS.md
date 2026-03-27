# TOOLS.md - Quizmaster MCP Reference

## Trivia Trainer MCP Server

**Endpoint:** `https://trivia.emilycogsdill.com/mcp`
**Transport:** Streamable HTTP (POST)
**Protocol:** Model Context Protocol (MCP)

### Available Tools

#### `list_categories`
List all trivia categories with module counts and tier breakdowns.
- **Parameters:** none
- **Returns:** Array of categories with id, name, color, moduleCount, tiers

#### `list_modules`
List quiz modules, optionally filtered.
- **Parameters:**
  - `category` (optional): geography, history, science, literature, entertainment, sports
  - `tier` (optional): foundation, core, advanced
- **Returns:** Array of modules with id, name, tier, description, defaultFormat, questionCount

#### `get_module`
Get a module with all questions, answers, and explanations.
- **Parameters:**
  - `moduleId` (required): e.g., "hist-us-presidents", "sci-element-symbols"
- **Returns:** Full module with questions array (each has question, answer, alternateAnswers, explanation)

#### `check_answer`
Check if an answer is correct. Uses fuzzy matching (Levenshtein distance <= 2 for answers >= 5 chars).
- **Parameters:**
  - `moduleId` (required): Module ID
  - `questionId` (required): Question ID within the module
  - `answer` (required): The answer to check
- **Returns:** { correct, correctAnswer, explanation, userAnswer, fuzzyMatch }

#### `get_random_question`
Get a random question (answer withheld — use check_answer to verify).
- **Parameters:**
  - `category` (optional): Filter by category
  - `tier` (optional): Filter by tier
- **Returns:** { moduleId, moduleName, questionId, question }

## Current Content (as of 2026-03-27)

22 modules, 879 questions across 6 categories (all Foundation tier):

| Category | Modules |
|----------|---------|
| Geography (4) | World Capitals — Major, US State Capitals, Countries of Europe, Oceans & Seas |
| History (7) | US Presidents (3 variants: by number, last names in order, full names in order), Major Wars, Ancient Civilizations, WWI Facts, WWII Facts |
| Science (3) | Element Symbols (118), Human Body Systems, Planets |
| Literature (3) | Shakespeare's Plays, Classic Novels → Authors, Greek/Roman Mythology |
| Entertainment (2) | Best Picture Winners (97), Famous Paintings → Artist |
| Sports (3) | Major Leagues, Grand Slam Tennis, Rules & Terminology |

## Quiz Strategy

### Starting a Session

1. Call `list_categories` to see what's available
2. Ask the user what they want to work on, or suggest based on conversation
3. Call `get_module` to load the questions
4. Present questions one at a time from the loaded data

### Checking Answers

For each answer:
1. Call `check_answer` with the user's response
2. The server handles fuzzy matching — "Washingtan" matches "Washington"
3. Use the returned `explanation` field — it's designed to be memorable

### For Random Mixed Practice

Use `get_random_question` repeatedly, varying the category parameter to ensure coverage across all six LL categories.

### Module ID Reference

| ID | Topic |
|----|-------|
| `geo-world-capitals-major` | 50 major world capitals |
| `geo-us-state-capitals` | All 50 US state capitals |
| `geo-countries-europe` | European countries by clue |
| `geo-oceans-seas` | Oceans and major seas |
| `hist-us-presidents` | Presidents by number |
| `hist-us-presidents-last-names` | Presidents last names in order |
| `hist-us-presidents-full-names` | Presidents full names in order |
| `hist-major-wars` | Major wars and conflicts |
| `hist-ancient-civilizations` | Ancient civilizations |
| `hist-ww1-key-facts` | World War I |
| `hist-ww2-key-facts` | World War II |
| `sci-element-symbols` | All 118 element symbols |
| `sci-human-body-systems` | Human body organ systems |
| `sci-planets-solar-system` | Solar system planets |
| `lit-shakespeares-plays` | Shakespeare's plays |
| `lit-classic-novels-authors` | 83 novels matched to authors |
| `lit-mythology-greek-roman` | Greek/Roman mythology |
| `ent-best-picture-winners` | Best Picture winners 1927-2024 |
| `ent-famous-paintings-artist` | Famous paintings → artist |
| `sport-major-leagues` | NFL/MLB/NBA/NHL/Premier League |
| `sport-grand-slam-tennis` | Grand Slam tournaments |
| `sport-rules-terminology` | Cross-sport rules and terms |

## REST API (Fallback)

If MCP is unavailable, the same data is accessible via REST:

```
GET https://trivia.emilycogsdill.com/api/categories
GET https://trivia.emilycogsdill.com/api/modules?category=history
GET https://trivia.emilycogsdill.com/api/modules/hist-us-presidents
POST https://trivia.emilycogsdill.com/api/modules/hist-us-presidents/check
  Body: {"questionId": "p1", "answer": "George Washington"}
GET https://trivia.emilycogsdill.com/api/quiz/random?category=science
```
