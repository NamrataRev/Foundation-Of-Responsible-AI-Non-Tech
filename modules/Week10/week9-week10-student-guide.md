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

Go to **https://pinecone.io** and click Sign Up/Start for free. Verify your email and log in.

<img src="images/01-pinecone-signup.png" alt="Pinecone signup page" width="700"/>

### Step 1.2 — Create a vector index

Once logged in, look at the left sidebar and click **Database**, then **Indexes**.

<img src="images/02-pinecone-sidebar.png" alt="Pinecone left sidebar with Database and Indexes" width="700"/>

Click the blue **"Create index"** button . 

<img src="images/04-pinecone-createindex.png" alt="Pinecone Indexes page with Create Index button" width="700"/>

### Step 1.3 — Configure your index

Give your index a name — for example `rag-docs`.

Check the **"Custom settings"** box in the top right of the Configuration section.

| Setting | Value |
|---------|-------|
| Dimensions | **1024** |
| Metric | **cosine** |
| Capacity mode | **Serverless** |

<img src="images/05-pinecone-customindexsettings.png" alt="Pinecone Create Index with Custom Settings checked" width="700"/>

Set these values exactly:



> **Why 1024?** The embedding model we use (Cohere embed-english-v3.0) converts text into lists of 1024 numbers. Your index must match this exactly.

Click **"Create index"**. Wait for the green dot — it is ready.

<img src="images/06-pinecone-index.png" alt="Pinecone index ready with green dot and 1024 dimensions" width="700"/>

### Step 1.4 — Get your API key

In the left sidebar click **API keys** (under the MANAGE section). Copy your key — it starts with `pcsk_`.

<img src="images/06-pinecone-api-key.png" alt="Pinecone API keys page" width="700"/>

> Save this key. You will paste it into n8n in a later step.

---

## Part 2 — Setting Up Cohere (Your AI Model)

Cohere provides two things: the embedding model that converts text to numbers, and the language model that writes the final answer.

### Step 2.1 — Create your account

Go to **https://dashboard.cohere.com** and sign up with your email.

### Step 2.2 — Get your trial API key

Click **API Keys** in the left sidebar. Under **Trial keys**, click the eye icon to reveal your key.

<img src="images/07-cohere-api-key.png" alt="Cohere API Keys page with trial key" width="700"/>

Copy the key. The trial is completely free and works for this course.

---

## Part 3 — Setting Up n8n (Your Workflow Builder)

n8n connects everything together — it is the engine that runs your RAG pipeline without any coding.

### Step 3.1 — Create your account

Go to **https://n8n.io** and click "Get started free". Sign up and verify your email.

Your workspace will be at `yourname.app.n8n.cloud`.

### Step 3.2 — Add your credentials

Go to **Personal → Credentials** tab and click **"Create credential"**.

<img src="images/09-n8n-credentials-empty.png" alt="n8n Credentials page with Create credential button" width="700"/>

**Add Cohere:**
- Search for `Cohere`
- Paste your Cohere trial API key that you already saved
- Click Save

**Add Pinecone:**
- Click Create credential again
- Search for `Pinecone`
- Paste your Pinecone API key (`pcsk_...`)
- Click Save

<img src="images/10-n8n-credentials-saved.png" alt="n8n Credentials page with Cohere and Pinecone saved" width="700"/>

---

## Part 4 — Building the Ingestion Pipeline

This pipeline loads your course document into Pinecone. You only run this once (or when your content changes).

### Step 4.1 — Create a new workflow

Click the **+** button next to the n8n logo and select New Workflow. Name it **"RAG Ingestion"**.

### Step 4.2 — Add the trigger

Click **"Add first step..."**. The Manual Trigger node appears automatically.

<img src="images/11-n8n-manual-trigger.png" alt="n8n canvas with Manual Trigger node" width="700"/>

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

<img src="images/12-n8n-http-request.png" alt="HTTP Request node with URL and Response Format Text" width="700"/>

### Step 4.4 — Add Pinecone Vector Store

Click **+** after the HTTP Request node. Search for **"Pinecone Vector Store"** and select **"Add documents to vector store"**.

Configure it:

| Setting | Value |
|---------|-------|
| Credential | Pinecone account |
| Operation Mode | Insert Documents |
| Pinecone Index | rag-docs (select from dropdown) |

<img src="images/13-n8n-pinecone-insert.png" alt="Pinecone Vector Store in Insert Documents mode" width="700"/>

### Step 4.5 — Add the Embedding model

At the bottom of the Pinecone node click the **Embedding +** button. Search for **"Embeddings Cohere"** and select it.

Set Model to **Embed-English-v3.0 (1024 Dimensions)**.

<img src="images/14-n8n-embeddings-cohere.png" alt="Embeddings Cohere node with Embed-English-v3.0" width="700"/>

### Step 4.6 — Add the Document Loader

Back in the Pinecone node, click the **Document +** button. Search for **"Default Data Loader"** and select it.

Set:
- Type of Data: **JSON**
- Text Splitting: **Custom**

<img src="images/15-n8n-data-loader.png" alt="Default Data Loader node with JSON and Custom splitting" width="700"/>

### Step 4.7 — Add the Text Splitter

Inside the Default Data Loader, click the **Text Splitter +** button. Search for **"Recursive Character Text Splitter"** and select it.

Set:
- Chunk Size: **500**
- Chunk Overlap: **50**

<img src="images/16-n8n-text-splitter.png" alt="Recursive Character Text Splitter with Chunk Size 500 Overlap 50" width="700"/>

> **What are chunks?** Your document gets cut into small pieces (chunks) of about 500 words each, with 50 words shared between neighbouring chunks so nothing gets cut off mid-thought.

### Step 4.8 — Run the ingestion

Click **"Execute workflow"**. All nodes should turn green.

<img src="images/17-n8n-ingestion-success.png" alt="n8n ingestion canvas with all green nodes" width="700"/>

Go to your Pinecone dashboard → Database → your index → Browser tab. You should see vectors appearing with your document content stored as metadata.

<img src="images/18-pinecone-vectors-stored.png" alt="Pinecone Browser tab with stored vectors and metadata" width="700"/>

### Step 4.9 — If you get a 429 Rate Limit error from Cohere

The Cohere free trial allows 100,000 tokens per minute. If your document produces many chunks, you may hit this limit. Here is how to fix it with three additions to your workflow.

#### Step 4.9a — Add a Code node to clean the HTML (optional but recommended)

If you are fetching from a GitHub page URL (not the raw URL), the response will contain HTML tags, SVG icons, and navigation markup mixed in with your actual content. The Code node strips all of that out before it reaches Pinecone.

Click **+** after the HTTP Request node. Search for **"Code"** and select **"Code in JavaScript"**.

Paste this code into the editor:

```javascript
const text = $input.first().json.data;
// Remove HTML tags
const clean = text.replace(/<[^>]*>/g, ' ').replace(/\s+/g, ' ').trim();
return [{ json: { text: clean } }];
```

<img src="images/17b-n8n-code-node.png" alt="Code in JavaScript node with HTML stripping code" width="700"/>

> **What this does:** The first `.replace()` removes everything between `<` and `>` (all HTML tags). The second `.replace()` collapses multiple spaces into one. The result is plain readable text — no HTML noise.

> **When to skip this step:** If you are already using the `raw.githubusercontent.com` URL, your content is already plain text and you do not need this node.

#### Step 4.9b — Add Loop Over Items + Wait to control the rate

1. Click **+** after the Code node (or after HTTP Request if you skipped 4.9a)
2. Search for **"Loop Over Items"** and add it — set **Batch Size to 5**
3. Click **+** on the **"loop"** output of the Loop node
4. Search for **"Wait"** and add it — set to **3 seconds**
5. Connect the Wait node's output to the **Pinecone Vector Store** node
6. The **"done"** output of Loop Over Items should go to **nothing** (leave it unconnected)

Your full ingestion flow should now look like this:

```
Execute Workflow
      ↓
HTTP Request
      ↓
Code in JavaScript  ← strips HTML
      ↓
Loop Over Items (batch: 5)
      ↓ [loop]
    Wait (3s)
      ↓
Pinecone Vector Store
  ├── Embeddings Cohere
  ├── Default Data Loader
  └── Recursive Character Text Splitter
```

<img src="images/17c-n8n-loop-wait-pinecone.png" alt="n8n canvas showing Code, Loop Over Items, Wait, and Pinecone connected correctly" width="700"/>

> **Why this works:** Instead of sending all chunks to Cohere at once, you now send 5 at a time, pause 3 seconds, then send the next 5 — staying well within the 100,000 token per minute limit.

> **Still hitting the limit?** Increase the Wait time to 6 seconds, or reduce the batch size to 3.

> **Common wiring mistake:** Make sure the Wait node connects to Pinecone, NOT the "done" output of Loop. The "done" output just signals the loop has finished — it should be left unconnected.

### ⚠️ Important — Use clean text, not raw HTML

If you are fetching content from a GitHub page URL (e.g. `https://github.com/username/repo`), n8n downloads the full HTML of the page including navigation menus, SVG icons, and code. This garbage data gets embedded into Pinecone and pollutes your search results.

**The best fix is to use the raw content URL:**

Change: `https://github.com/username/repo/blob/branch/file.md`

To: `https://raw.githubusercontent.com/username/repo/branch/file.md`

Remove `/blob/` and change `github.com` to `raw.githubusercontent.com`. This gives you plain text with no HTML noise.

**If you must use the page URL**, add the Code node from Step 4.9a to strip the HTML before it reaches Pinecone.

> **Signs your data is dirty:** If you open the Embeddings Cohere node and see chunks full of `<path d="M1.5 3.25...">` or `data-view-component="true"` — you are embedding HTML, not content. Delete your Pinecone index, fix the URL, and re-run ingestion.

---

## Part 4.10 — Managing Your Pinecone Index Data

**Pinecone never deletes old data automatically.** Every time you run the ingestion workflow, it adds new vectors on top of whatever is already there. This means:

- If you re-ingest after fixing a bug, old bad vectors remain alongside new good ones
- If you change your document, both old and new chunks coexist
- Query results will mix old and new data

**How to clear your index before re-ingesting:**

1. Go to **https://pinecone.io** → Database → Indexes → your index
2. Click the **Namespaces** tab
3. Click **"Delete all"** to wipe all vectors
4. Re-run your ingestion workflow with clean data

**Best practice — use Namespaces to separate datasets:**

If you plan to ingest multiple documents or versions, use Pinecone namespaces so they don't mix:

In your Pinecone Vector Store node (both ingestion and query), set:
- Namespace: `week9-docs` for one dataset
- Namespace: `week10-docs` for another

Then in your query pipeline, set the same namespace so you only search the right data.

---

## Part 5 — Building the Query Pipeline (with Citations)

This pipeline runs every time a student asks a question. It finds the relevant chunks, sends them to the AI, and returns a grounded answer with citations.

### Step 5.1 — Create a second workflow

Click **+** and create a new workflow. Name it **"RAG Query with Citations"**.

### Step 5.2 — Add the Chat Trigger

Click **"Add first step..."**. Search for **"Chat"** and select **"On new Chat event"**.

Inside the node, toggle **"Make Chat Publicly Available"** to ON. Note the Chat URL — this is the link you share with students.

<img src="images/19-n8n-chat-trigger.png" alt="Chat Trigger node with Make Chat Publicly Available ON" width="700"/>

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

<img src="images/20-n8n-pinecone-retrieve.png" alt="Pinecone Vector Store in Get Many mode with metadata ON" width="700"/><img src="images/20-n8n-pinecone-retrieve2.png" alt="Pinecone Vector Store in Get Many mode with metadata ON" width="700"/>

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

<img src="images/21-n8n-edit-fields.png" alt="Edit Fields node with prompt expression and Include Other Input Fields ON" width="700"/>

### Step 5.5 — Add the Basic LLM Chain

Click **+** after Edit Fields. Search for **"Basic LLM Chain"** and select it.

Change **"Source for Prompt"** to **"Define below"**.

The Prompt field now shows `{{ $json.prompt }}` — this picks up the citation prompt you built in the previous node.

<img src="images/22-n8n-basic-llm-chain.png" alt="Basic LLM Chain with Define below and $json.prompt" width="700"/>

> **Important — stop the LLM answering from general knowledge:** By default the Cohere model will answer any question using its own training data, even if that topic is not in your document. To force it to answer only from your retrieved chunks, click **"Add prompt"** inside the Basic LLM Chain and add a **System** prompt with this text:
>
> ```
> You are a helpful assistant. Answer ONLY based on the context provided.
> If the answer is not in the context, say "I don't have that information in my knowledge base."
> Do NOT use your general knowledge.
> ```
>
> Test it by asking a question you know is NOT in your document (e.g. "What is the capital of France?"). It should now say it cannot find the answer rather than answering from general knowledge.

### Step 5.6 — Add Cohere Chat Model

At the bottom of the Basic LLM Chain, click the **Model +** button. Search for **"Cohere Chat Model"** and select it.

Set:
- Credential: Cohere account
- Model: **command-r-plus-08-2024**

<img src="images/23-n8n-cohere-chat-model.png" alt="Cohere Chat Model with command-r-plus-08-2024" width="700"/>

### Step 5.7 — Publish and test

Click **"Publish"** in the top right. Then click **"Open chat"** inside the Chat Trigger node.

Ask: **"What is computational thinking?"**

You should see an answer followed by:

```
📄 Source: lines 1–3 from the course document
```

<img src="images/24-chat-with-citation.png" alt="Chat interface showing answer with citation line numbers" width="700"/>

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
| Rate limit error (429) | See the full fix in Part 4.9 below — Cohere free tier allows 100,000 tokens per minute |
| Dimension mismatch error | Delete your Pinecone index and recreate it with exactly 1024 dimensions |
| Answer says "Sources used:" but nothing after | Check that Include Other Input Fields is ON in the Edit Fields node |
| RAG answers questions not in your document | The LLM is using its own general knowledge — add a System prompt in the Basic LLM Chain to restrict it to context only (see Step 5.5) |
| Chunks contain HTML tags like `<path>` or `<svg>` | You are scraping a GitHub page URL — switch to the raw.githubusercontent.com URL, or add the Code node from Step 4.9a to strip HTML |
| Re-ingesting adds duplicates | Pinecone keeps all old vectors — delete all vectors in the Pinecone dashboard before re-ingesting (see Part 4.10) |
| 429 error even with small files | Your chunks may be very large — reduce Chunk Size to 200 in the Text Splitter node, and add a Loop Over Items + Wait node (see Step 4.9) |

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
