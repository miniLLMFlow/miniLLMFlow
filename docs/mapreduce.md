---
layout: default
title: "Map Reduce"
parent: "Paradigm"
nav_order: 3
---

# Map Reduce

Process large inputs by splitting them into chunks using [BatchNode](./batch.md), then combining results.

### Example: Document Summarization

```python
class MapSummaries(BatchNode):
    def prep(self, shared):
        text = shared["text"]
        return [text[i:i+10000] for i in range(0, len(text), 10000)]

    def exec(self, chunk):
        return call_llm(f"Summarize this chunk: {chunk}")

    def post(self, shared, prep_res, exec_res_list):
        shared["summaries"] = exec_res_list

class ReduceSummaries(Node):
    def prep(self, shared):
        return shared["summaries"]
        
    def exec(self, summaries):
        return call_llm(f"Combine these summaries: {summaries}")
        
    def post(self, shared, prep_res, exec_res):
        shared["final_summary"] = exec_res

# Connect nodes
map_node = MapSummaries()
reduce_node = ReduceSummaries()

map_node >> reduce_node

# Create flow 
summarize_flow = Flow(start=map_node)
summarize_flow.run(shared)
```