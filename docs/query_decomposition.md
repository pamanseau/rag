You can enable query decomposition in both Docker and Helm deployments:

1. **Docker Deployment** – Use Docker to deploy RAG on your hardware
2. **Helm Deployment** – Deploys as a Helm chart on a Kubernetes cluster (local deployment only)

## Option 1: Docker Deployment (Default)

### Step 1: Deploy all the dependent services

### Step 2: Enable query decomposition

Set the environment variable to enable query decomposition:

```bash
export ENABLE_QUERY_DECOMPOSITION=true
export MAX_RECURSION_DEPTH=3
```

After setting these environment variables, you must restart the RAG server for `ENABLE_QUERY_DECOMPOSITION` to take effect:

```bash
docker compose -f deploy/compose/docker-compose-rag-server.yaml up -d
```

## Option 2: Helm Deployment (Local Deployment Only)


### Update RAG Blueprint Deployment with Query Decomposition

Use the Helm upgrade command below to enable query decomposition in RAG Blueprint by setting `ENABLE_QUERY_DECOMPOSITION` and `MAX_RECURSION_DEPTH`:

```bash
  --username '$oauthtoken' \
  --password "${NGC_API_KEY}" \
  --set imagePullSecret.password=${NGC_API_KEY} \
  --set ngcApiSecret.password=${NGC_API_KEY} \
  --set envVars.ENABLE_QUERY_DECOMPOSITION="true" \
  --set envVars.MAX_RECURSION_DEPTH="3"
```

## When to Use Query Decomposition

Query decomposition is especially valuable for multi-hop or complex queries that involve multiple steps or aspects. By breaking down a complex question into smaller, focused subqueries, the system can generate intermediate answers for each part. These intermediate responses are then combined to produce a comprehensive and accurate final answer.

```
"If my future wife has the same first name as the 15th first lady of the United States' mother
and her surname is the same as the second assassinated president's mother's maiden name,
what is my future wife's name?"

This query requires multiple interconnected steps:
1. Identify the 15th first lady of the United States
2. Find the 15th first lady's mother's first name
3. Identify the second assassinated president
4. Find that president's mother's maiden name
5. Combine the two names to form the final answer
```

```
"Define machine learning"
(Direct definition, no sub-aspects needed)
```

```
"What's the current stock price of NVIDIA?"
(Real-time data, decomposition adds latency)
```

```
"What's the syntax for creating a Python dictionary?"
(Specific syntax, no multiple perspectives needed)
```

## How Query Decomposition Works
To visualize how query decomposition works, see the diagram below:

![Query Decomposition Flow](assets/query_decomposition.jpeg)

**Figure:** *Query Decomposition Flow — The system breaks down a complex query into subqueries, processes each iteratively, and synthesizes a comprehensive answer.*


### Core Algorithm

1. **Subquery Generation**: Break down into focused questions
2. **Iterative Processing**: For each recursion depth:
   - Process each subquery independently
   - Rewrite queries with accumulated context
   - Retrieve and rank relevant documents
   - Generate focused answers
   - Collect contexts for synthesis
3. **Follow-up Generation**: Create follow-up questions for information missing from original query
4. **Termination Check**: Stop if no follow-up needed or max depth reached
5. **Final Synthesis**: Generate comprehensive response from all contexts and conversation

