# Demystifying Vector Indexes: How Amazon Q Business Actually Finds Your Data

**Understanding the magic behind enterprise AI search with AWS Titan Embeddings and Claude**


![1761757402324_img](https://github.com/user-attachments/assets/feb1ab9a-dd9f-4e03-86e0-90bc47637acb)

---

If you've ever wondered how Amazon Q Business can search through millions of documents and return relevant answers in seconds, you're not alone. The secret isn't magic—it's **indexes** and **vector databases**. But what do those terms actually mean beyond the buzzwords?

Let's pull back the curtain and look at the technical architecture that makes enterprise AI search possible, using real AWS services and code examples.

## The Index Problem: Why We Can't Just "Ask the AI"

When someone says "just ask the AI," they're skipping over a critical step. Large language models like Claude don't have access to your company's data. They need context, and they need it fast.

Imagine trying to find a specific email in your inbox by reading every single message one by one. That's what happens without an index. Now imagine having a smart filing system that instantly points you to the relevant messages—that's what an index does, but for AI.

## What Actually Gets Indexed?

When Amazon Q Business connects to your data sources, here's what happens:

### 1. Content Ingestion
Connectors crawl your data sources: S3 buckets, SharePoint sites, Salesforce records, databases, and more.

### 2. The Index Components

Think of the index as having three main parts:

**Vector embeddings** - Your content transformed into mathematical representations:
```json
{
  "id": "chunk_12345",
  "text": "Amazon Q Business indexes your enterprise data",
  "embedding": [
    0.0234, -0.0452, 0.0789, 0.0123, -0.0567, 0.0891,
    ... (1018 more numbers) ...
    0.0234, -0.0123, 0.0445, 0.0667, -0.0889, 0.0321
  ],
  "metadata": {
    "source": "confluence",
    "page_id": "ABC123",
    "timestamp": "2024-01-15",
    "acl": ["group:engineering", "user:jane@company.com"]
  }
}
```

That array of 1,024 numbers? That's what Amazon Titan Embeddings produces—a mathematical fingerprint of the meaning of your text.

**Inverted index** - Traditional keyword-to-document mappings for exact matches

**Metadata store** - Document properties, access controls (ACLs), timestamps, and source information

## Visualizing Vector Space: Where Your Documents Live

Here's where it gets interesting. Those 1,024-dimensional vectors exist in a space where similar concepts cluster together. Let me show you a simplified 2D view:

```
        Vector Space
        
    query: "what is indexing?"
         ⭐ [0.5, 0.8]
        /  \
       /    \
      /      \
   📄        📄
 d002      d001
[0.48,0.82] [0.52,0.79]
              
              📄 d005
            [0.1, 0.2]
            (not similar)
```

Documents about similar topics sit close to each other. When you ask a question, your query becomes a vector too, and the system finds the nearest neighbors.

In reality, this happens in 1,024 dimensions (with Titan Embeddings), which is impossible to visualize but follows the same principle: **semantic similarity = spatial proximity**.

## The Index Structure: HNSW Graphs

To search billions of vectors efficiently, modern vector databases use clever data structures. The most common is HNSW (Hierarchical Navigable Small World):

```
Layer 2 (coarse):        [A]
                          |
Layer 1 (medium):    [B]─[A]─[C]
                      |   |   |
Layer 0 (fine):   [D][B][A][C][E][F]
                   | X | X | X | X |
                  [G][H][I][J][K][L]

Search path: Query → A → C → J (3 hops vs scanning all 12)
```

This multi-layer graph structure lets you jump quickly from coarse to fine matches, turning a potentially million-comparison search into just a handful of hops.

## The Full Architecture: Amazon Q Business

Here's how all the pieces fit together in the AWS ecosystem:

```
┌─────────────────────────────────────────────────────┐
│              DATA SOURCES                           │
│  [SharePoint] [S3] [Salesforce] [Database]          │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│           CONNECTOR LAYER                           │
│  • Authentication  • Rate limiting  • Pagination    │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│         PROCESSING PIPELINE                         │
│                                                     │
│  1. Extract Text    → "content here..."             │
│  2. Chunk           → [300-500 token chunks]        │
│  3. Generate Vector → Amazon Titan Embeddings       │
│                       (Bedrock)                     │
│  4. Extract ACL     → ["user:john", "group:eng"]    │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│              INDEX STORAGE                          │
│                                                     │
│  ┌────────────────┐  ┌─────────────────┐            │
│  │ Vector Store   │  │ Metadata Store  │            │
│  │ (OpenSearch    │  │ (DynamoDB)      │            │
│  │  Serverless)   │  │ • ACLs          │            │
│  │ • Embeddings   │  │ • Timestamps    │            │
│  │ • k-NN index   │  │ • Source refs   │            │
│  └────────────────┘  └─────────────────┘            │
│                                                     │
│  ┌────────────────┐                                 │
│  │ Keyword Index  │                                 │
│  │ (OpenSearch)   │                                 │
│  │ • Inverted idx │                                 │
│  └────────────────┘                                 │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│           QUERY ENGINE                              │
│                                                      │
│  User Query → Vector + Keyword Search → Filter ACL  │
│            → Retrieve Top K → Send to Claude        │
│                                (via Bedrock)         │
└─────────────────────────────────────────────────────┘
```

## A Real Query: Code Walkthrough

Let's trace an actual query through this system. Say a user asks: "How do I reset my password?"

```python
import boto3
import json

# Initialize AWS clients
bedrock = boto3.client('bedrock-runtime')
opensearch = boto3.client('opensearchserverless')

# 1. User query
query = "How do I reset my password?"

# 2. Convert to vector using Amazon Titan Embeddings
response = bedrock.invoke_model(
    modelId='amazon.titan-embed-text-v1',
    body=json.dumps({"inputText": query})
)
query_vector = json.loads(response['body'].read())['embedding']
# Returns: [0.34, -0.23, 0.78, ..., 0.12] (1024 dimensions)
```

Now we have a vector representation of the user's intent. Next, we search:

```python
# 3. Search OpenSearch vector index with ACL filtering
search_body = {
    "size": 5,
    "query": {
        "knn": {
            "embedding_vector": {
                "vector": query_vector,
                "k": 5
            }
        }
    },
    "filter": {
        "terms": {"acl": ["user:john@company.com", "group:engineering"]}
    }
}

results = opensearch.search(
    index='amazon-q-index',
    body=search_body
)
```

Notice the `filter` clause? This is crucial for enterprise security. The index only returns documents this specific user has permission to see.

The results look like this:

```python
# 4. Results from OpenSearch
[
  {
    "_score": 0.89,
    "_source": {
      "text": "To reset password, go to Settings > Security...",
      "source": "employee_handbook.pdf",
      "page": 23,
      "acl": ["group:all-employees"]
    }
  },
  {
    "_score": 0.76,
    "_source": {
      "text": "Password reset instructions: 1) Click forgot...",
      "source": "IT_wiki/authentication",
      "page": 1,
      "acl": ["user:john@company.com"]
    }
  }
]
```

Finally, we send this context to Claude:

```python
# 5. Send to Claude via Bedrock with retrieved context
context = "\n\n".join([hit['_source']['text'] for hit in results['hits']['hits']])

prompt = f"""Here is relevant context from our documentation:

<context>
{context}
</context>

Question: {query}

Please answer based on the context provided."""

response = bedrock.invoke_model(
    modelId='anthropic.claude-3-sonnet-20240229-v1:0',
    body=json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 1000,
        "messages": [
            {"role": "user", "content": prompt}
        ]
    })
)

answer = json.loads(response['body'].read())['content'][0]['text']
```

Claude now has relevant, permission-filtered context to generate an accurate answer.

## Under the Hood: OpenSearch Configuration

For those who want to go deeper, here's what the actual OpenSearch index configuration looks like:

```json
{
  "settings": {
    "index": {
      "knn": true,
      "knn.algo_param.ef_search": 512
    }
  },
  "mappings": {
    "properties": {
      "embedding_vector": {
        "type": "knn_vector",
        "dimension": 1024,
        "method": {
          "name": "hnsw",
          "engine": "nmslib",
          "parameters": {
            "ef_construction": 512,
            "m": 16
          }
        }
      },
      "text": {"type": "text"},
      "source": {"type": "keyword"},
      "acl": {"type": "keyword"},
      "timestamp": {"type": "date"}
    }
  }
}
```

Key parameters:
- **dimension: 1024** - Matches Titan Embeddings output
- **method: hnsw** - The graph structure we discussed
- **ef_construction: 512** - Quality of the graph (higher = better recall, slower indexing)
- **m: 16** - Number of connections per node (higher = better search, more memory)

## The AWS Stack Advantage

Using the full AWS stack provides some significant benefits:

**Amazon Titan Embeddings (1024 dimensions)**
- Optimized for English and 100+ languages
- Cost-effective compared to alternatives
- Native Bedrock integration

**OpenSearch Serverless**
- Managed k-NN vector search
- Automatic scaling
- Built-in security and ACL filtering

**Claude via Bedrock**
- 200K token context window (plenty of room for retrieved documents)
- Strong reasoning capabilities
- No data leaves AWS

**DynamoDB**
- Fast metadata and ACL lookups
- Seamless integration with OpenSearch

Everything stays within your AWS environment, which simplifies compliance, security, and operations.

## What This Means for Your Enterprise AI

When you implement Amazon Q Business or build your own RAG (Retrieval Augmented Generation) system, you're essentially:

1. **Building a semantic search engine** that understands meaning, not just keywords
2. **Creating a security layer** that respects existing permissions
3. **Giving your LLM a memory** by retrieving relevant context
4. **Making it fast** through clever indexing structures

The "index" isn't just a database—it's a multi-layered system combining vector math, graph structures, traditional search, and access control.

## Conclusion: It's Math, Not Magic

The next time someone mentions that Amazon Q "uses indexes," you'll know what's really happening:

- Text is chunked and transformed into 1,024-dimensional vectors
- Those vectors are organized in navigable graph structures
- Queries are converted to vectors and matched using cosine similarity
- Results are filtered by access controls
- The top matches provide context to Claude
- Claude generates a natural language answer

It's sophisticated engineering, but it's not magic. It's linear algebra, graph theory, and good software architecture working together.

And now you know how it works.

---

All the services mentioned (Amazon Bedrock, OpenSearch Serverless, Amazon Q Business) are available in AWS. Start with the [Amazon Q Business documentation](https://docs.aws.amazon.com/amazonq/latest/business-use-dg/what-is.html) or experiment with [Bedrock's Titan Embeddings](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html).

Important: Using AWS services can incur costs. Be informed about the service costs and cost managemeent on AWS before trying anything hand on.
