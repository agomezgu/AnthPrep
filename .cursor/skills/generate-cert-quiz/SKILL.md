---
name: generate-cert-quiz
description: >-
  Generates a new 15-question multiple-choice HTML practice quiz from markdown
  notes and Jupyter notebooks under courses/. Use when the user asks for a quiz,
  practice exam, certification questions, cert prep, exam-style questions, or a
  new set of study questions from course content.
disable-model-invocation: false
---

# Generate Cert Quiz

Create a **new** self-contained HTML quiz from `courses/` markdown **and** notebooks. Do this every time the skill runs. Never overwrite an old quiz.

## Output

- Folder: `practice_quizzes/` (create if missing)
- File: `practice_quizzes/quiz_YYYYMMDD_HHMMSS.html`
- Always copy structure from [template.html](template.html). Replace `__QUESTIONS_JSON__` and `__GENERATED_AT__` only. Do not restyle the page.

After writing, tell the user the file path, how many questions came from each course, and how many of those were from notebooks. Do not open a browser unless they ask.

## Workflow

Copy and complete:

```
Quiz progress:
- [ ] Discover eligible notes and notebooks
- [ ] Read recent quizzes (avoid repeats)
- [ ] Allocate 15 questions across courses
- [ ] Read the chosen source files
- [ ] Write 15 grounded MCQs
- [ ] Fill template and save
```

### 1. Discover sources

Course = each **direct child folder** of `courses/`.

Eligible sources (same pool — a notebook counts as one file, same as a `.md`):

- `courses/<course>/**/*.md` (case-insensitive `.md`)
- `courses/<course>/**/*.ipynb`

**Skip:**

- `README.md` (setup docs, not lesson content)
- paths containing `.ipynb_checkpoints`
- generated files that are not `.md` / `.ipynb` (html, json dumps, etc.)

Include notebooks even when they live under `projects/` or `notebooks/`. Include OpenAI-named notebooks as well as Anthropic ones.

Skip a course if it has no eligible files.

Do not hardcode folder names. New courses, lessons, and notebooks are included automatically.

### 2. Avoid repeats

If `practice_quizzes/` has existing `quiz_*.html` files, read the **3 most recent**. Collect their `"stem"` values. Do not reuse a stem (case-insensitive, ignore extra whitespace).

### 3. Allocate 15 questions

Let `n` = number of courses with eligible files.

- If `n == 0`: stop. Tell the user there are no `.md` or `.ipynb` files under `courses/`.
- If `n >= 15`: 1 question each from the 15 courses that have the most files.
- Else: `base = 15 // n`, `rem = 15 % n`. Each course gets `base`. Give the extra `rem` slots to the courses with the **most** eligible files.

**Cap:** a course with `k` files may get at most `max(2, k)` questions. Move leftover slots to larger uncapped courses.

Prefer spreading across **different files** inside a course, including a mix of `.md` and `.ipynb` when both exist. If a file is too thin for a fair question, pick another file in that course.

**Difficulty mix (whole quiz, not per course):**

- 5 recall
- 6 application
- 4 scenario

Do not show difficulty in the HTML.

### 4. Read sources

Read the allocated files. Questions must come from those files, not generic Anthropic/OpenAI knowledge.

For `.ipynb`:

- Use markdown cells and code cells (API calls, parameters, schemas, prompt patterns).
- Ignore cell outputs, execution counts, secrets, `.env` keys, and boilerplate helper names unless the helper *is* the concept.
- Do not dump the whole notebook into a question; pull one teaching point per item.

### 5. Write questions

Exactly **15** items. Each item:

```json
{
  "stem": "Question text ending with a question mark",
  "choices": ["A option", "B option", "C option", "D option"],
  "correctIndex": 0,
  "explanation": "One or two sentences from the notes.",
  "source": "02_BuildingWithTheClaudeAPI/05_Tools.md"
}
```

Rules:

- Four distinct choices; exactly one correct (`correctIndex` 0–3).
- Shuffle so the correct answer is not always the same index.
- Distractors should be plausible and drawn from nearby concepts in the notes (wrong D, related API feature, similar-sounding term).
- No "all of the above", "none of the above", or trick wording.
- No questions about screenshots, image URLs, cell indexes, filenames, or helper-function trivia (`add_user_message`, etc.).
- Notebook items should be API/concept questions: parameters, tool schemas, prompting patterns, what a call or setting does.
- `source` is the path relative to `courses/` using forward slashes (`.md` or `.ipynb`).
- `explanation` must be accurate to that file and mention the key idea, not only "see the notes".

**Recall:** definition or listed fact.  
**Application:** which setting/technique to use.  
**Scenario:** short "what should you do / what happens next" situation.

**Good:** "Claude cannot fetch live weather by default. What does tool use add?"  
**Good (notebook):** "In the tools notebook, why are tool schemas passed into the API request?"  
**Bad:** "What is the filename of the weather screenshot in the tools lesson?"  
**Bad:** "What is the helper function named in cell 2 of 06_Tools.ipynb?"

### 6. Fill template and save

1. Read [template.html](template.html).
2. Replace `__QUESTIONS_JSON__` with a JSON **array** of the 15 objects (valid JSON, not markdown).
3. Replace `__GENERATED_AT__` with a local timestamp string.
4. Write the HTML file. Confirm it contains 15 questions.

## User overrides

Honor these if the user states them in the same request:

- One course or named lessons/notebooks only
- Different question count (still use the same HTML behavior)
- Open the file when done

If they do not override, use the defaults above.

## HTML behavior (already in the template)

Do not change this. The page is intentionally simple:

- All 15 questions visible
- Click one option to mark it; that question locks
- Immediate correct/incorrect, plus explanation and source
- Running tally at the top
