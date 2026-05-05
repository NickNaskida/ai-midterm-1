# Academic Transfer Planner — Documentation

> **Student:** Nikoloz  
> **Current Program:** Computer Science  
> **Institution:** Ilia State University  
> **Assignment:** Introduction to Artificial Intelligence — Midterm Exam

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Features](#2-features)
3. [How to Use](#3-how-to-use)
4. [Technical Structure](#4-technical-structure)
5. [Equivalency Algorithm](#5-equivalency-algorithm)
6. [Improvement Plan — AI-Powered Automatic Syllabus Comparison](#6-improvement-plan--ai-powered-automatic-syllabus-comparison)

---

## 1. Project Overview

The **Academic Transfer Planner** is a single-file web application that helps a Computer Science student at GTU plan a transfer to another academic program. It automatically shows which of the student's completed courses correspond to courses in the target program, generating a visual equivalency table on demand.

The application is built with pure **HTML, CSS, and JavaScript** — no frameworks, no external files, no build tools.

---

## 2. Features

| Feature | Description |
|---|---|
| Course List | Auto-displayed table of all CS courses with pass status |
| Program Display | Current program name shown prominently |
| Program Dropdown | Select from 3 target programs |
| Transfer Button | Generates equivalency table instantly |
| Stats Summary | Shows pass count, matches, N/A count, and coverage % |

---

## 3. How to Use

1. **Open** `transfer_planner.html` in any modern web browser.
2. The course list loads automatically on page load.
3. Select a **target program** from the dropdown menu:
   - Computer Engineering
   - Civil Engineering
   - Electrical and Electronics Engineering
4. Click the **⚡ Transfer** button.
5. An equivalency table appears showing all passed courses matched (or not) to the target program.

---

## 4. Technical Structure

The entire application lives in one `.html` file with three embedded sections:

```
transfer_planner.html
├── <style>   — All CSS (variables, layout, animations, responsive design)
├── <body>    — HTML structure (header, cards, tables, controls)
└── <script>  — JavaScript (data, rendering, equivalency logic)
```

### Data Model

```javascript
const myCourses = [
  { name: "Calculus I",  passed: true  },
  { name: "Physics II",  passed: false },
  // ...
];
```

### Equivalency Map Structure

```javascript
const equivalencies = {
  "Computer Engineering": {
    "Calculus I": "Calculus I",
    "Machine Learning": "N/A",
    // ...
  },
  "Civil Engineering": { /* ... */ },
  "Electrical and Electronics Engineering": { /* ... */ }
};
```

---

## 5. Equivalency Algorithm

### Core Logic

The equivalency system uses a **rule-based lookup table** approach. Each possible target program has a pre-defined mapping dictionary that links every Computer Science course to either:

- A **named equivalent course** in the target program, or
- `"N/A"` if no reasonable equivalent exists.

The algorithm runs as follows:

```
1. Filter myCourses → keep only courses where passed = true
2. For each passed course:
   a. Look up its name in equivalencies[targetProgram]
   b. If a match exists and is not "N/A" → record as equivalency
   c. If the value is "N/A" → record as no equivalent
3. Render results table + statistics
```

### Equivalency Decision Rules

The mappings were built using the following reasoning principles:

**Rule 1 — Identical Name Match**  
If a course name is identical (or near-identical) across programs, it is a direct equivalency. Examples:
- `Calculus I` → `Calculus I` (in Computer Engineering, EEE)
- `English Language I` → `English Language I` (all programs)
- `Linear Algebra` → `Linear Algebra` (all programs)

**Rule 2 — Functional Equivalency**  
If a course covers the same core material under a different name, it is mapped as equivalent:
- `Calculus I` (CS) → `Mathematics I` (Civil Engineering) — same content, renamed
- `Object-Oriented Programming` (CS) → `Programming for Engineers` (EEE) — same fundamentals
- `Probability and Statistics` (CS) → `Statistics and Probability` (Civil Eng.) — reordered title, same subject

**Rule 3 — Partial Overlap → N/A**  
If a course only partially overlaps with anything in the target program, `N/A` is assigned to avoid misrepresentation. It is better to flag a course as unmatched than to force an inaccurate equivalency.

**Rule 4 — CS-Specific Courses**  
Highly specialized CS courses — `Machine Learning`, `Artificial Intelligence`, `Theory of Computation`, `Compiler Design`, `Human-Computer Interaction` — have no equivalents in Civil Engineering or EEE because those programs have no courses covering those topics. They are all marked `N/A` for those targets.

**Rule 5 — Common Foundation Courses**  
Physics, Mathematics, English, and History courses are almost universally shared across all four programs at GTU and are mapped as direct equivalencies wherever they appear.

### Limitations of the Current Approach

- Mappings are **hardcoded** — any program curriculum change requires manual updates.
- Equivalency is **binary**: a course is either matched or not; there is no partial credit.
- The algorithm has **no access to actual syllabi** — it relies entirely on course names and human judgment.

---

## 6. Improvement Plan — AI-Powered Automatic Syllabus Comparison

> This chapter describes a future system where equivalency decisions are made automatically by an AI, based on the actual content of course syllabi — not course names alone.

### 6.1 Motivation

The current hardcoded approach does not scale. Universities regularly update their curricula, add new courses, or rename existing ones. A name-based lookup table breaks the moment a course is renamed or a new program is introduced. More importantly, two courses with completely different names might cover the same material — and two courses with the same name might teach entirely different things. The only reliable way to determine equivalency is to read what is actually taught: the **syllabus**.

### 6.2 System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI Transfer Planner v2                       │
│                                                                 │
│  [Syllabus Loader] → [Text Extractor] → [Embedding Engine]      │
│                                              │                  │
│                                    [Similarity Matcher]         │
│                                              │                  │
│                                    [LLM Decision Layer]         │
│                                              │                  │
│                                    [Equivalency Table Output]   │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 Step-by-Step Algorithm

#### Step 1 — Syllabus Ingestion

The system automatically fetches syllabus documents for all courses in both the source and target programs. Sources could include:

- University course catalog pages (scraped via HTTP)
- PDF documents uploaded by the student or administrator
- A structured university API (if one exists)

Each syllabus is parsed to extract:

```
{
  course_name: "Data Structures and Algorithms",
  program: "Computer Science",
  topics: ["arrays", "linked lists", "trees", "graphs", "sorting", "complexity"],
  learning_outcomes: ["implement common data structures", "analyze time complexity"],
  prerequisites: ["Introduction to Programming"],
  credits: 6
}
```

#### Step 2 — Embedding Generation

Every syllabus is converted into a **dense vector embedding** using a pre-trained language model (e.g., `text-embedding-ada-002` from OpenAI, or a local model like `all-MiniLM-L6-v2`). The embedding captures the *semantic meaning* of the course content — not just keywords.

```python
# Pseudocode
for course in all_courses:
    text = course.name + " " + " ".join(course.topics) + " " + " ".join(course.outcomes)
    course.embedding = embedding_model.encode(text)
```

#### Step 3 — Similarity Matching

For each passed course in the source program, the system computes **cosine similarity** against every course in the target program:

```
similarity(A, B) = cos(embedding_A, embedding_B)
                 = (A · B) / (|A| × |B|)
```

Courses are ranked by similarity. The top candidate is proposed as the equivalent. A similarity threshold (e.g., `0.75`) determines whether a match is "strong enough" to count as an equivalency.

```
Calculus I (CS)  →  similarity scores:
  Mathematics I (Civil)        → 0.94  ✓ Match
  Physics I (Civil)            → 0.61
  Structural Analysis (Civil)  → 0.21
```

#### Step 4 — LLM Verification Layer

For borderline cases (similarity between `0.55` and `0.75`), a **Large Language Model** is called to make the final decision. It receives both syllabi as input and is prompted:

```
System: You are an academic advisor determining course equivalency for university transfer.

User: 
  Course A — "Object-Oriented Programming" (Computer Science):
  Topics: classes, inheritance, polymorphism, design patterns, Java
  Outcomes: design and implement OOP systems

  Course B — "Programming for Engineers" (Electrical Engineering):
  Topics: procedural programming, basic OOP, C++, embedded systems
  Outcomes: write structured programs for engineering applications

  Are these courses equivalent? Answer: YES / PARTIAL / NO
  Reason: [one sentence]
```

The LLM output (`YES`, `PARTIAL`, `NO`) + its reasoning is shown to the student so they can understand and verify the decision.

#### Step 5 — Output

The system generates a final equivalency table with confidence levels:

| Your Course | Equivalent | Confidence | Basis |
|---|---|---|---|
| Calculus I | Mathematics I | 94% | Syllabus similarity |
| OOP | Programming for Engineers | 71% | LLM verified |
| Machine Learning | N/A | — | No match above threshold |

### 6.4 Why AI Helps

| Task | Manual Approach | AI Approach |
|---|---|---|
| Reading syllabi | Human reads dozens of PDFs | Automated extraction in seconds |
| Finding matches | Name-based guessing | Semantic understanding of actual content |
| Borderline decisions | Subjective human judgment | Explainable LLM reasoning |
| Handling renames | Breaks immediately | Unaffected — based on content, not names |
| Scaling to new programs | Manual update of all tables | Automatic — just load new syllabi |

### 6.5 Technologies Involved

- **PDF/HTML parsing:** `PyMuPDF`, `BeautifulSoup`
- **Embedding model:** `sentence-transformers` (local) or OpenAI API
- **Vector similarity:** `numpy`, `scikit-learn`, or a vector database like `ChromaDB`
- **LLM for verification:** `Claude`, `GPT-4`, or a local model via `Ollama`
- **Frontend:** Updated version of the current single-file HTML app, calling a lightweight Python backend (Flask/FastAPI)

### 6.6 Example Workflow (User Perspective)

1. Student opens the improved Transfer Planner.
2. Selects their current program and the target program.
3. Clicks **"Auto-Compare Syllabi"**.
4. The system fetches syllabi, runs embeddings, computes similarity.
5. Within ~10 seconds, a complete equivalency table appears with confidence scores and AI reasoning.
6. Student can click any row to see a side-by-side syllabus comparison.
7. The student exports the result as a PDF to submit to the academic office.

This approach transforms course transfer planning from a slow, bureaucratic process into an instant, evidence-based, AI-assisted workflow.

---

*Documentation written for the Introduction to Artificial Intelligence midterm exam.*  
*All code in `transfer_planner.html`. Repository contains exactly two files: `transfer_planner.html` and `README.md`.*
