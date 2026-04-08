---
title: "elasticsearch-esql"
type: docs
weight: 2
description: >
  Execute ES|QL queries.
---

## About

Execute ES|QL queries.

This tool allows you to execute ES|QL queries against your Elasticsearch
cluster. You can use this to perform complex searches and aggregations.

See the [official
documentation](https://www.elastic.co/docs/reference/query-languages/esql/esql-getting-started)
for more information.

## Compatible Sources

{{< compatible-sources >}}

## Parameters

| **name**   |                **type**                 | **required** | **description**                                                                                                                                     |
|------------|:---------------------------------------:|:------------:|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| query      |                 string                  |    false     | The ES\|QL query to run. Can also be passed by parameters.                                                                                          |
| format     |                 string                  |    false     | The format of the query. Default is json. Valid values are csv, json, tsv, txt, yaml, cbor, smile, or arrow.                                        |
| timeout    |                 integer                 |    false     | The timeout for the query in seconds. Default is 60 (1 minute).                                                                                     |
| parameters | [parameters](../#specifying-parameters) |    false     | List of [parameters](../#specifying-parameters) that will be used with the ES\|QL query.<br/>Only supports “string”, “integer”, “float”, “boolean”. |

## Example

```yaml
kind: tool
name: query_my_index
type: elasticsearch-esql
source: elasticsearch-source
description: Use this tool to execute ES|QL queries.
query: |
  FROM my-index
  | KEEP *
  | LIMIT ?limit
parameters:
  - name: limit
    type: integer
    description: Limit the number of results.
    required: true
```

### Example with Vector Search

You can perform vector-based semantic searches using the `COSINE_SIMILARITY` function in ES|QL. By combining this with the embeddedBy parameter property, you can automatically convert text queries into vector embeddings before executing the search.

#### Vector Ingestion
```yaml
kind: tool
name: ingest_vector_doc
type: elasticsearch-esql
source: elasticsearch-source
description: Prepares a document with its vector embedding for ingestion.
query: |
  FROM my-index
  | WHERE name == ?name
  | LIMIT 0
parameters:
  - name: name
    type: string
    description: The name of the document.
  - name: content
    type: string
    description: The text content to be embedded.
  - name: content_vector
    type: string
    description: The generated vector for the content.
    embeddedBy: my-embedding-model
    valueFromParam: content
```

#### Vector Search
```yaml
kind: tool
name: semantic_search
type: elasticsearch-esql
source: elasticsearch-source
description: Finds documents most relevant to the user query.
query: |
  FROM my-index
  | WHERE embedding IS NOT NULL
  | EVAL score = COSINE_SIMILARITY(embedding, ?)
  | SORT score DESC
  | LIMIT 1
  | KEEP id, content, score
parameters:
  - name: query_text
    type: string
    description: The text to search for.
    embeddedBy: my-embedding-model
```

