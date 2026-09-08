# RAG Architecture — Retrieve, Augment, Generate

---

## Learning Objectives

By the end of this topic, you should be able to:

- Describe what happens at each of the three RAG stages in plain English
- Build a working RAG chatbot using Pinecone Assistant without writing any code
- Upload your course notes and get cited answers from them
- Verify that answers trace back to the exact chunk they came from
- Identify what can go wrong at each stage and how to fix it

---

## Overview

The RAG Fundamentals topic introduced RAG as a pattern — Retrieve, Augment, Generate — and explained why it exists. This topic builds the actual pipeline using **Pinecone Assistant** — a free tool that handles the complete RAG pipeline automatically.

You upload your documents. Pinecone handles chunking, embedding, and vector storage. You get a chatbot that answers questions with cited sources — no code required.

Why Pinecone? In the previous module you used ChromaDB — a local vector store you set up in Python. Pinecone is the cloud-hosted, production-grade version of the same concept. The underlying idea is identical — store vectors, retrieve by similarity, generate with an LLM. Pinecone wraps all of that into a managed service with a built-in chat interface.

---

## What Happens at Each Stage

```text
STAGE 1 — RETRIEVE
Student asks: "What is the difference between a while loop and a for loop?"
        ↓
Pinecone converts the question to a vector.
Compares it against all stored document vectors.
Returns the most similar chunks from the uploaded files.

STAGE 2 — AUGMENT
Those chunks are combined with the question
into one complete prompt sent to Claude:
  "Notes: [Source 4] A while loop repeats as long as a condition stays True...
   Question: What is the difference between a while loop and a for loop?"

STAGE 3 — GENERATE
Claude reads the notes and question together.
Writes an answer — citing [Source 4] after each claim.
        ↓
"A while loop repeats as long as a condition stays True [Source 4].
A for loop repeats once for each item in something iterable [Source 4]."
```

---

## Building the RAG Pipeline — Step by Step

### Step 1 — Sign Up for Pinecone

Go to **[pinecone.io](https://www.pinecone.io)** and click **Sign Up Free**.

**Screen 1 — Choose your plan:**
Select **I'm building a small or personal project** and click **Start for free**

![Pinecone plan selection screen — select small or personal project](images/pinecone_01a_plan.png)

Sign up with your Google account or email. Pinecone will then ask you three questions to customise your setup.

**Screen 2 — First two questions:**
- **What are you building or exploring?** → **RAG / Agents**
- **What kind of data do you have?** → **Raw files (PDFs, docs, images)**

![Pinecone onboarding — RAG/Agents and Raw files selected](images/pinecone_01b_onboarding.png)

**Screen 3 — Third question:**
- **How will you build your solution?** → **No-code / low-code**

![Pinecone onboarding — No-code/low-code option](images/pinecone_01c_build.png)

**Screen 4 — Fourth question:**
- **What no-code / low-code platform are you using?** → **Other**
- In the text box that appears, type: **Pinecone Assistant**

![Pinecone onboarding — Other selected with Pinecone Assistant typed in the text box](images/pinecone_01d_platform.png)

Click **Get Started**.

---

### Step 2 — Save Your API Key

A popup appears immediately after setup: **"API key generated"**

![Pinecone console showing API key generated popup with copy button](images/pinecone_02_console.png)

**This is critical** — Pinecone will not show this key again after you close the popup.

1. Click the **copy icon** next to the masked key
2. Save it in your `.env` file:
```text
PINECONE_API_KEY=your-key-here
```
3. Click **Close**

You are now in the Pinecone Console. You can see **Assistant** in the left panel — this is where the RAG chatbot lives.

---

### Step 3 — Create Your Assistant

1. Click **Assistant** → **Assistants** in the left panel
2. Click **Create an assistant**
3. A popup appears — **"Set up your new assistant"**

![Pinecone Create Assistant popup with name field and Region dropdown](images/pinecone_03_create.png)

Fill in:
- **Name your assistant:** `exam-prep-chatbot`
- **Region:** United States (leave as default)

Click **Create assistant**

---

### Step 4 — Select Claude as the Model

Your assistant opens in the **Assistant playground**. On the right panel you will see **Chat model** set to GPT-4o by default.

![Pinecone playground showing Chat model dropdown open with Claude Sonnet 4.5 visible](images/pinecone_05_model.png)

Click the **Chat model** dropdown and select **Claude Sonnet 4.5 — Anthropic**

> This is the same AI provider used throughout this course. Claude reads your uploaded notes and generates cited answers.

---

### Step 5 — Add Assistant Instructions

In the right panel, find the **Assistant instructions** box. Click inside it and type:

```text
You are an exam preparation assistant for a BTech AI programme.
Answer the student's question using only the lecture notes provided.
After each factual claim cite the source in square brackets — for example [Source 1].
If the notes do not contain enough information to answer the question,
say so clearly rather than guessing.
```

This is the system prompt — it tells Claude exactly how to behave. The instruction to cite sources is what produces the `[Source 4]` references you will see in every answer.

---

### Step 6 — Upload Your Course Notes

Click the **folder icon** 📁 at the top of the playground to open the files panel.

Click **click to upload** and select your course note files — PDF, Word, or Markdown all work.

![Pinecone files panel showing uploaded markdown files with Ready status](images/pinecone_06_files.png)

Pinecone automatically:
- Reads each document
- Splits it into chunks (the Retrieve stage setup)
- Converts each chunk to an embedding vector
- Stores all vectors in the index

Wait until each file shows no error status before testing.

---

### Step 7 — Test the Chatbot

Click back to the playground and type a question about your uploaded notes in the **Ask a question...** box at the bottom.

For the Python files uploaded here:
*"What is the difference between a while loop and a for loop?"*

![Pinecone playground showing two questions — one correctly answered with citations, one correctly refused](images/pinecone_07_answer.png)

Look carefully at what happened with two questions:

**Question 1 — "What is the role of the learning rate in gradient descent?"**
The assistant said: *"I don't have enough information in the provided lecture notes to answer your question about the role of the learning rate in gradient descent."*

This is RAG working correctly. The uploaded files are Python basics notes — gradient descent is not in them. Instead of guessing, the assistant refused and told the student what the notes do cover. This is **grounding in action**.

**Question 2 — "What is the difference between a while loop and a for loop?"**
The assistant gave a detailed answer with `[4]` citations on every claim. Every sentence traces back to the uploaded Loops file.

---

### Step 8 — Verify a Citation

Click on any citation number in the answer — for example `[4]`.

![Pinecone citation popup showing the exact chunk from 1-5.Conditionals.md that was retrieved](images/pinecone_08_citation.png)

A popup shows:
- **The source file name** — `1-5.Conditionals.md`
- **A View file link** — to open the full document
- **The exact chunk of text** that was retrieved and used as context

This is the **Retrieve stage made visible**. You can see exactly which piece of the document Pinecone found and passed to Claude. This is what makes RAG trustworthy — every claim in the answer can be traced back to the exact text it came from.

---

## What Each Step Did — Mapped to RAG Stages

| Step | RAG Stage | What happened |
|---|---|---|
| Upload files | Retrieve (setup) | Pinecone chunked documents and stored vectors |
| Ask a question | Retrieve | Pinecone found the most similar chunks for the question |
| Answer appears with citations | Augment | Retrieved chunks + question combined into one prompt |
| Claude writes the answer | Generate | Claude answered using only the retrieved chunks |
| Click a citation number | Grounding | You verified the claim traced back to the exact chunk |

---

## Two Important Things This Demo Showed

**1. RAG refuses to hallucinate when the answer is not in the notes**
When asked about gradient descent — which was not in the uploaded files — the assistant correctly said it did not have enough information. A plain LLM would have answered from general knowledge. RAG constrained the answer to the uploaded documents.

**2. Every claim is verifiable**
Every `[4]` in the answer points to a specific chunk. You clicked on it and saw the exact text. A student can verify every claim against their own notes.

---

## What Can Go Wrong — and the Fix

| Problem | What it means | Fix |
|---|---|---|
| Answer does not cite sources | Instructions not saved | Add citation instruction to Assistant instructions and save |
| Answer uses general knowledge not from notes | Instructions too weak | Strengthen: "answer using ONLY the provided notes, do not use general knowledge" |
| No answer at all | Topic not in uploaded files | Upload the relevant course notes for that topic |
| Wrong chunk cited | Pinecone retrieved a related but not exact chunk | Upload more specific notes or ask a more specific question |

---

## Best Practices

- Always add the Assistant instructions before testing — without them Claude answers freely from general knowledge
- Test with a question that is NOT in your notes — verify the assistant says it cannot answer rather than guessing
- Click citations to verify claims before sharing answers with students
- Upload all lecture notes for a topic together — Pinecone searches all uploaded files simultaneously

## Common Beginner Mistakes

- **Testing before the instruction to cite sources is added** — answers appear without `[Source N]` references. Add the instructions first.
- **Uploading files and testing immediately** — give Pinecone a moment to process. If answers seem off, wait 30 seconds and try again.
- **Expecting the assistant to know things not in the notes** — if a topic is not uploaded, the assistant should say so. This is correct behaviour, not a bug.

---

## Key Takeaways

- Pinecone Assistant implements the full RAG pipeline — chunking, embedding, storage, retrieval, and generation — without writing code
- The three stages are visible: Upload (Retrieve setup) → Question (Retrieve + Augment) → Cited answer (Generate)
- RAG correctly refuses to answer when the topic is not in the uploaded notes — this is grounding working as intended
- Clicking a citation number shows the exact chunk that was retrieved — this is how you verify grounding
- Pinecone is the cloud-hosted equivalent of ChromaDB — the concepts are identical, the interface is different

> **Interview tip:** If asked "how would you build a RAG pipeline?" — describe the three stages, mention that tools like Pinecone Assistant implement them without code, and explain what each stage does. Then say you understand the underlying components — ChromaDB, sentence-transformers, API calls — from building them in Python. Showing you know both the concept and the implementation at two levels sets you apart from candidates who only know one.

---

## Reference Links

- 📎 [Pinecone Assistant — Get Started](https://www.pinecone.io)
- 📎 [Pinecone Assistant — Documentation](https://docs.pinecone.io/guides/assistant/quickstart/sdk-quickstart)
- 📎 [Pinecone Assistant — Upload Files](https://docs.pinecone.io/guides/assistant/upload-files)
- 📎 [Anthropic — RAG Guide](https://docs.anthropic.com/en/docs/build-with-claude/retrieval-augmented-generation)
