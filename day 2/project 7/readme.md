# P7 – Advanced Agent with RAG
In this project, we'll enhance our AI Agent with access to internal data sources. You can [review the documents here](https://github.com/tobiaszwingmann/n8n-ai-bootcamp/tree/main/day%202/project%208/documents).

## Part 1: Document ingestion

**Purpose:** Load internal data sources into a vector data store so the AI Agent can perform a semantic search and retrieve relevant pieces of information during the conversation.

### 1. Create New Workflow
- n8n home --> New workflow
- Name the workflow `Document Ingestion`

### 2. Add `Trigger manually` node

### 3. Add `Github List File` Node

**Resource**: `file`

**Operation**: `list`

**Owner**: `your-github-username`

**Repository**: `n8n-ai-bootcamp` (forked)

**Path** 
```
day 2/project 7/documents/
```

### 3. Add `Github Get File` Node

**Resource**: `file`

**Operation**: `Get`

**Owner**: `your-github-username`

**Repository**: `n8n-ai-bootcamp` (forked)

**File Path** 
```
{{ $json.path }}
```

**As Binary Property**: `True`

**Put Output File in Field**: `data`

### 4. Add `Simple Vector Store` Node
- Operation Mode: Insert Documents
- Leave default options
- Clear Store: `True`

> Alternative: [Pinecone](https://www.pinecone.io/) – easy setup and good integration with n8n 

#### 4.1 Subnodes
Model
- **Embeddings**: Add `Embeddings OpenAI` node
- **Model**: `test-embedding-3-small`

Data Loader
- Add **Default Data Loader** Node
- **Type of Data**: `Binary`
- **Mode**: `Load All Input Data`
- **Data Format**: `Automatically Detect by Mime Type`
- **Text Splitting**: `Simple`
Options:
- **Split Pages in PDF**: `True`
- **Metadata**
   - Name: `file_name`
   - Value: `{{ $json.name }}`

### 5. Try it out!
- Run the workflow
- Inspect the created embeddings (Open the embeddings model node)

---

## Part 2: Document retrieval

**Purpose:** Perform a semantic search on the vector store and fetch relevant documents with metadata.

### 1. Create New Workflow
- n8n home --> New workflow
- Name the workflow `Document Search`

### 2. Add Trigger Node `When Executed by Another Workflow`
#### Input data mode: `Define using fields below`

| Parameter | Type   |
| ----- | ---------- |
| query | String` |

**Pin Test Data**:
- query: `Email templates aren’t displaying dynamic fields.`

### 3. Add `Simple Vector Store` ->  `Get ranked documents from vector store` Node

- **Operation**: `Get Many`
- **Prompt**: `{{ $json.query }}`
- **Limit**: 5
- **Include Metadata**: `true`

#### Add Subnode `Embedding`
- Choose **OpenAI Embeddings Model**
- **Model**: `text-embedding-3-small`

### 4. Add `Edit Fields` Node 
**Manual Mapping**
 Name | Type | Value |
| --- | ---- | ----- |
| Content | String | `{{ $json.document.pageContent }}` |
| Source File | String | `{{ $json.document.metadata.file_name }}` |
| Page | String | `{{ $json.document.metadata.loc.pageNumber }}` |

### 5. Add `Aggregate` Node 
- **Aggregate**: `All Item Data (Into a Single List)`
- **Put Output in Field**: `data`
- **Include**: `All Fields`

### 6. Add `Edit Fields` Node 
**Manual Mapping**
 Name | Type | Value |
| --- | ---- | ----- |
| response | String | `{{ $json.toJsonString() }}` |

### 6. Try it out!
- Run a test execution
- Try different search queries and test the result

### 7. Publish
- Everything works?
- Click **Publish** to publish your workflow

---

## Part 3: Add the Custom Tool to your AI Agent

Let's give your AI Agent access to the custom knowledge retrieval tool!

### 1. Open the workflow from P6 (or duplicate)
- Rename: `P7 – Advanced Agent with RAG`

### 2. Update Agent Instructions
- Open the **AI Agent Node**
- Replace the **ROLE** definition with the following:
```
### **Role**

You are an **IT Support Chatbot** for a corporate helpdesk.
You assist employees with the **Nova CRM**, **Windows** and **Microsoft Office** issues, and manage **support tickets**.

You can:

* Troubleshoot common technical issues.
* Answer questions about **Nova CRM** using the search tool
* Decide when to **escalate** to a human IT technician.
* **Create**, **update**, and **retrieve** tickets using your tools.
```
- Leave the rest of the system prompt the same!

### 3. Add the Custom Search Tool
- Click the `+` icon under the AI Agent Node
- Select `Call n8n Workflow Tool`
  
#### Custom Tool Settings
**Description**
```
Call this tool to search the user manual of the Nova CRM software.
```

**Source**: `Database`

**Workflow**: `From List` --> `Document Search`

#### Workflow Inputs
- **query**: *Defined automatically by the model*
- **Description**
```
Search query for Nova CRM manuals.
```

### 4. Try it out!
- Ask the chatbot some questions about Nova CRM

Examples:

```
Can multiple users work in the same Nova account?
```

```
I did not receive the Nova password reset email
```

```
What is the URL to log into Nova?
```
