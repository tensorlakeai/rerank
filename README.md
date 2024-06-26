# rerank-ts

[![Discord](https://dcbadge.vercel.app/api/server/VXkY7zVmTD?style=flat&compact=true)](https://discord.gg/VXkY7zVmTD)


`rerank-ts` is a lightweight TypeScript library for re-ranking search results from retreival systems. 

Adding Re-Ranking almost always improves accuracy of retrieval pipelines. If you are building a RAG application, and using semantic search or full-text search using this library to re-rank the results will improve accuracy of the application in most cases. However, re-ranking usually adds some amount of latency. We have added knobs in the LLM ReRanker to control latency. 

## Installation

Install `rerank` using npm:

```bash
npm install rerank
```

## Algorithms
- LLM Re-Rankers
- Reciprocal Rank Fusion

### LLM Re-Rankers

- [Permutation Generation with sliding windows](https://arxiv.org/pdf/2304.09542)

**Available Providers:**

- **Groq**: `ProviderGroq`
- **OpenAI**: `ProviderOpenAI`

> Model Providers are implemented with a clean interface, we welcome contributions to add support for other model providers from the community! 

**Example Usage:**

```typescript
import { LLMReranker, ProviderGroq } from "rerank";
const provider = new ProviderGroq("llama3-8b-8192", API_KEY);
const reranker = new LLMReranker(provider);

// Replace with your own list of objects to rerank.
const list = [
  { key: "bc8fe338", value: "I hate vegetables" },
  { key: "236386f2", value: "I love mangoes" },
];

const query = "I love apples";
const result = await reranker.rerank(list, "key", "value", query);
// ["236386f2", "bc8fe338"]
```

### Reciprocal Rank Fusion

Combine multiple rank lists by assigning scores based on reciprocal ranks, effectively prioritizing higher-ranked items across all lists.
[Paper](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf)

**Example Usage:**

```typescript
import { reciprocalRankFusion } from "rerank";

// Example structure of a search result for this usage example
interface SearchResult {
  url: string; // we will use this as our key identifier
  name: string;
}

// Assume you are searching with 3 different queries and fetching results
// searchIndex is just for demonstration purposes and should be replaced with your actual search implementation
const rankedLists: SearchResult[][] = await Promise.all([
  searchIndex("exampleIndex1", "person riding skateboard"),
  searchIndex("exampleIndex2", "person skating on the sidewalk"),
  searchIndex("exampleIndex3", "skateboard trick"),
]);

// Perform Reciprocal Rank Fusion (RRF) on all of the results
// The RRF function takes the ranked lists and a key identifier, in this case "url"
// It returns a map of all URLs now ranked with an RRF score
const rankedURLs = reciprocalRankFusion(rankedLists, "url");

// Build a map of results keyed by URL for easy access
const resultsMap = new Map<string, SearchResult>();
rankedLists.flat().forEach((result) => {
  resultsMap.set(result.url, result);
});

// Get a single sorted list of results based on the RRF ranking
const sortedResults = Array.from(rankedURLs.keys()).map((url) => {
  return resultsMap.get(url);
});
```
