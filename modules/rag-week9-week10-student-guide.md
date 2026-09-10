# RAG Pipeline — Hands-On Guide
### Week 9 & 10 · Foundations of Responsible AI Engineering

---

> **What you will build:** A working AI system that reads your course documents, finds the most relevant parts, and answers your questions — with citations showing exactly where the answer came from.

---

## Before You Start

You need three free accounts. Create them in this order:

| Account | Website | Time needed |
|---------|---------|-------------|
| Pinecone | https://pinecone.io | 2 minutes |
| Cohere | https://dashboard.cohere.com | 2 minutes |
| n8n Cloud | https://n8n.io | 2 minutes |

No credit card needed for any of them.

---

## Part 1 — Setting Up Pinecone (Your Vector Database)

Pinecone is where your document chunks are stored as numbers (vectors) so they can be searched by meaning rather than keywords.

### Step 1.1 — Create your account

Go to **https://pinecone.io** and click Sign Up. Verify your email and log in.

![Pinecone signup page showing the Sign Up button]

### Step 1.2 — Create a vector index

Once logged in, look at the left sidebar and click **Database**, then **Indexes**.

![Pinecone left sidebar with Database and Indexes highlighted]

Click the blue **"Create index"** button in the top right corner.

![Pinecone Indexes page with Create Index button highlighted]

### Step 1.3 — Configure your index

Give your index a name — for example `rag-docs`.

Check the **"Custom settings"** box in the top right of the Configuration section.

![Pinecone Create Index page with Custom Settings checkbox highlighted]

Set these values exactly:

| Setting | Value |
|---------|-------|
| Dimensions | **1024** |
| Metric | **cosine** |
| Capacity mode | **Serverless** |

> **Why 1024?** The embedding model we use (Cohere embed-english-v3.0) converts text into lists of 1024 numbers. Your index must match this exactly.

Click **"Create index"**. Wait for the green dot — it is ready.

![Pinecone index showing green status dot and Dimension: 1024]

### Step 1.4 — Get your API key

In the left sidebar click **API keys** (under the MANAGE section). Copy your key — it starts with `pcsk_`.

![Pinecone API keys page with the key value highlighted]

> Save this key. You will paste it into n8n in a later step.

---

## Part 2 — Setting Up Cohere (Your AI Model)

Cohere provides two things: the embedding model that converts text to numbers, and the language model that writes the final answer.

### Step 2.1 — Create your account

Go to **https://dashboard.cohere.com** and sign up with your email.

### Step 2.2 — Get your trial API key

Click **API Keys** in the left sidebar. Under **Trial keys**, click the eye icon to reveal your key.

![Cohere dashboard API Keys page with Trial keys section and eye icon highlighted]

Copy the key. The trial is completely free and works for this course.

---

## Part 3 — Setting Up n8n (Your Workflow Builder)

n8n connects everything together — it is the engine that runs your RAG pipeline without any coding.

### Step 3.1 — Create your account

Go to **https://n8n.io** and click "Get started free". Sign up and verify your email.

Your workspace will be at `yourname.app.n8n.cloud`.

![n8n cloud dashboard after first login showing empty workflow canvas]

### Step 3.2 — Add your credentials

Go to **Personal → Credentials** tab and click **"Create credential"**.

![n8n Credentials page showing the Create credential button]

**Add Cohere:**
- Search for `Cohere`
- Paste your Cohere trial API key
- Click Save

**Add Pinecone:**
- Click Create credential again
- Search for `Pinecone`
- Paste your Pinecone API key (`pcsk_...`)
- Click Save

![n8n Credentials page showing both Cohere and Pinecone credentials saved]

---

## Part 4 — Building the Ingestion Pipeline

This pipeline loads your course document into Pinecone. You only run this once (or when your content changes).

### Step 4.1 — Create a new workflow

Click the **+** button next to the n8n logo and select New Workflow. Name it **"RAG Ingestion"**.

### Step 4.2 — Add the trigger

Click **"Add first step..."**. The Manual Trigger node appears automatically.

![n8n canvas showing the Manual Trigger node added]

### Step 4.3 — Add the HTTP Request node

Click the **+** on the connecting line. Search for **"HTTP Request"** and select it.

Configure it:

| Setting | Value |
|---------|-------|
| Method | GET |
| URL | Your raw GitHub document URL |
| Response Format | Text (under Options → Add option) |

**How to get the raw URL from GitHub:**

Change: `https://github.com/username/repo/blob/branch/file.md`

To: `https://raw.githubusercontent.com/username/repo/branch/file.md`

Remove `/blob/` and change `github.com` to `raw.githubusercontent.com`.

![HTTP Request node configuration showing the URL field and Response Format set to Text]

### Step 4.4 — Add Pinecone Vector Store

Click **+** after the HTTP Request node. Search for **"Pinecone Vector Store"** and select **"Add documents to vector store"**.

Configure it:

| Setting | Value |
|---------|-------|
| Credential | Pinecone account |
| Operation Mode | Insert Documents |
| Pinecone Index | rag-docs (select from dropdown) |

![Pinecone Vector Store node showing Insert Documents mode and index selected]

### Step 4.5 — Add the Embedding model

At the bottom of the Pinecone node click the **Embedding +** button. Search for **"Embeddings Cohere"** and select it.

Set Model to **Embed-English-v3.0 (1024 Dimensions)**.

![Embeddings Cohere node showing model set to Embed-English-v3.0]

### Step 4.6 — Add the Document Loader

Back in the Pinecone node, click the **Document +** button. Search for **"Default Data Loader"** and select it.

Set:
- Type of Data: **JSON**
- Text Splitting: **Custom**

![Default Data Loader node showing JSON type and Custom text splitting]

### Step 4.7 — Add the Text Splitter

Inside the Default Data Loader, click the **Text Splitter +** button. Search for **"Recursive Character Text Splitter"** and select it.

Set:
- Chunk Size: **500**
- Chunk Overlap: **50**

![Recursive Character Text Splitter node showing Chunk Size 500 and Overlap 50]

> **What are chunks?** Your document gets cut into small pieces (chunks) of about 500 words each, with 50 words shared between neighbouring chunks so nothing gets cut off mid-thought.

### Step 4.8 — Run the ingestion

Click **"Execute workflow"**. All nodes should turn green.

![n8n canvas showing all ingestion nodes with green checkmarks]

Go to your Pinecone dashboard → Database → your index → Browser tab. You should see vectors appearing with your document content stored as metadata.

![Pinecone Browser tab showing records with pageContent and metadata visible]

---

## Part 5 — Building the Query Pipeline (with Citations)

This pipeline runs every time a student asks a question. It finds the relevant chunks, sends them to the AI, and returns a grounded answer with citations.

### Step 5.1 — Create a second workflow

Click **+** and create a new workflow. Name it **"RAG Query with Citations"**.

### Step 5.2 — Add the Chat Trigger

Click **"Add first step..."**. Search for **"Chat"** and select **"On new Chat event"**.

Inside the node, toggle **"Make Chat Publicly Available"** to ON. Note the Chat URL — this is the link you share with students.

![Chat Trigger node showing Make Chat Publicly Available toggle turned ON and the chat URL]

### Step 5.3 — Add Pinecone Vector Store (retrieval)

Click **+** after the Chat Trigger. Search for **"Pinecone Vector Store"** and select **"Get ranked documents from vector store"**.

Configure it:

| Setting | Value |
|---------|-------|
| Credential | Pinecone account |
| Pinecone Index | rag-docs |
| Prompt | `{{ $json.chatInput }}` (click fx to switch to expression mode) |
| Limit | 4 |
| Include Metadata | ON |

Click the **Embedding +** button and add **Embeddings Cohere** with model **Embed-English-v3.0**.

![Pinecone Vector Store node in Get Many mode with Include Metadata toggled ON]

### Step 5.4 — Add the Edit Fields node (citation builder)

Click **+** after Pinecone. Search for **"Edit Fields"** and select it.

Click **"Add Field"** and set:
- Name: `prompt`
- Type: String
- Switch to Expression mode (click the fx button)

Paste this expression:

```
Answer this question using ONLY the context below. Be concise.

Question: {{ $('When chat message received').item.json.chatInput }}

Context:
[1] Lines {{ $('Pinecone Vector Store').all()[0].json.document.metadata.loc.lines.from }}-{{ $('Pinecone Vector Store').all()[0].json.document.metadata.loc.lines.to }}: {{ $('Pinecone Vector Store').all()[0].json.document.pageContent }}

[2] Lines {{ $('Pinecone Vector Store').all()[1].json.document.metadata.loc.lines.from }}-{{ $('Pinecone Vector Store').all()[1].json.document.metadata.loc.lines.to }}: {{ $('Pinecone Vector Store').all()[1].json.document.pageContent }}

[3] Lines {{ $('Pinecone Vector Store').all()[2].json.document.metadata.loc.lines.from }}-{{ $('Pinecone Vector Store').all()[2].json.document.metadata.loc.lines.to }}: {{ $('Pinecone Vector Store').all()[2].json.document.pageContent }}

At the end of your answer write exactly:
📄 Source: lines [X]-[Y] from the course document
```

Toggle **"Include Other Input Fields"** to ON.

![Edit Fields node showing the prompt field with expression and Include Other Input Fields ON]

### Step 5.5 — Add the Basic LLM Chain

Click **+** after Edit Fields. Search for **"Basic LLM Chain"** and select it.

Change **"Source for Prompt"** to **"Define below"**.

The Prompt field now shows `{{ $json.prompt }}` — this picks up the citation prompt you built in the previous node.

![Basic LLM Chain node showing Source for Prompt set to Define below and prompt field showing $json.prompt]

### Step 5.6 — Add Cohere Chat Model

At the bottom of the Basic LLM Chain, click the **Model +** button. Search for **"Cohere Chat Model"** and select it.

Set:
- Credential: Cohere account
- Model: **command-r-plus-08-2024**

![Cohere Chat Model node showing command-r-plus-08-2024 selected]

### Step 5.7 — Publish and test

Click **"Publish"** in the top right. Then click **"Open chat"** inside the Chat Trigger node.

Ask: **"What is computational thinking?"**

You should see an answer followed by:

```
📄 Source: lines 1–3 from the course document
```

![Chat interface showing a question answered with a citation at the bottom showing line numbers]

---

## What You Just Built — Explained Simply

```
Student types a question
        ↓
n8n converts the question to a vector (a list of numbers)
        ↓
Pinecone searches for the chunks with the closest matching numbers
        ↓
n8n assembles the chunks into a prompt with line numbers visible
        ↓
Cohere reads the chunks and writes a grounded answer
        ↓
Student sees the answer + exactly which lines it came from
```

---

## Week 9 Concepts — Live in Your Pipeline

| Theory concept | Where you can see it |
|----------------|----------------------|
| RAG architecture — retrieve, augment, generate | The three stages are visible as separate nodes on the n8n canvas |
| Why RAG reduces hallucination | Ask a question about something NOT in your document — the AI will say it cannot find the answer rather than making one up |
| Chunking strategies | Open the Text Splitter node — change Chunk Size from 500 to 200 and watch how retrieval changes |
| Recall vs precision trade-off | Change the Limit in the Pinecone node from 4 to 1 (high precision, less recall) or 10 (high recall, less precision) |
| Citation and grounding | Every answer ends with the exact line numbers from the source document |

---

## Week 10 Concepts — Deliberately Break Your Pipeline

These exercises show failure modes so you can diagnose them in real systems.

### Exercise 1 — Bad chunking

Open the Text Splitter node. Change Chunk Size to **2000**. Re-run the ingestion workflow. Ask a specific question.

What you will see: the retrieved chunks are too large and contain too much unrelated information, making the answer vague.

**Fix:** Set Chunk Size back to 500.

### Exercise 2 — Wrong embedding model (dimension mismatch)

Open the Embeddings Cohere node in the ingestion pipeline. Change the model to **Embed-English-v2.0 (4096 Dimensions)**.

What you will see: an error because your Pinecone index expects 1024 dimensions but the model outputs 4096.

**Fix:** Change the model back to Embed-English-v3.0.

### Exercise 3 — Hallucination test

Ask a question about a topic that is definitely not in your ingested document — for example "What is the capital of France?" when your document is about computational thinking.

What you will see: the AI either says it cannot find the answer in the context (correct behaviour) or generates a confident wrong answer from its training data (hallucination).

**Fix:** Add "If the answer is not in the context, say: I cannot find this in the course material." to the prompt in the Edit Fields node.

### Exercise 4 — When to refuse vs when to guess

In the Edit Fields node, change the last instruction from:

```
At the end of your answer write exactly:
📄 Source: lines [X]-[Y] from the course document
```

To:

```
If you are not confident in your answer based on the context, start your response with:
⚠️ Low confidence answer.
At the end write: 📄 Source: lines [X]-[Y] from the course document
```

Ask an ambiguous question. Watch how the model signals its own uncertainty.

---

## Troubleshooting

| Problem | What to check |
|---------|---------------|
| 404 error on chat URL | Toggle "Make Chat Publicly Available" ON in the Chat Trigger node, then re-publish |
| Answer not from your document | Check Pinecone dashboard — are vectors showing in the Browser tab? |
| Rate limit error (429) | Wait 60 seconds — Cohere free tier allows 100,000 tokens per minute |
| Dimension mismatch error | Delete your Pinecone index and recreate it with exactly 1024 dimensions |
| Answer says "Sources used:" but nothing after | Check that Include Other Input Fields is ON in the Edit Fields node |

---

## Key Settings to Remember

| Setting | Correct value | Why it matters |
|---------|--------------|----------------|
| Pinecone index dimensions | 1024 | Must match Cohere embed-english-v3.0 |
| Pinecone metric | cosine | Best for comparing text meaning |
| Embedding model | embed-english-v3.0 | Must be the same in both ingestion and query |
| Chat model | command-r-plus-08-2024 | Best Cohere model for grounded Q&A |
| Chunk size | 500 | Balance between context and precision |
| Chunk overlap | 50 | Prevents important content being cut off |

---

*Built with n8n Cloud · Pinecone Serverless · Cohere Trial API — all free for non-commercial learning use*
