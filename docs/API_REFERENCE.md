# API Reference

## Main Workflow (MCP-Powered)

The primary interface is the MCP workflow in `src/main_workflow.js`.

### runWorkflow(query, options)

Runs the MCP workflow to research a topic (LLM if configured; heuristic fallback otherwise).

**Parameters:**
- `query` (string): The search query/research topic
- `options` (object, optional):
  - `maxResults` (number): Maximum number of sources to analyze (default: 10)

**Returns:**
```javascript
{
  query: string,              // Original query
  result: string,             // Comprehensive research report
  toolUsage: object,          // Tools used and frequency
  executionTime: number,      // Time in seconds
  sourcesAnalyzed: number     // Number of pages successfully scraped
}
```

**Example:**
```javascript
import { runWorkflow } from './src/main_workflow.js';

const result = await runWorkflow('quantum computing 2025', {
  maxResults: 10
});

console.log(result.result);        // Research report
console.log(result.toolUsage);     // { search_engine: 1, scrape_as_markdown: 10 }
console.log(result.executionTime); // 45 (seconds)
console.log(result.sourcesAnalyzed); // 10
```

---

## How the Workflow Works

The workflow performs a straightforward pipeline:

1. Search using Bright Data MCP `search_engine`
2. Select URLs with domain diversity
3. Scrape with `scrape_as_markdown`
4. Synthesize a comprehensive report with citations
   - LLM (OpenAI) if configured
   - Heuristic fallback if not

The workflow has access to these Bright Data tools:
- `search_engine`: Search Google/Bing/Yandex
- `scrape_as_markdown`: Extract clean content from any URL
- `web_data_*`: Structured extractors for major platforms

---

## Environment Configuration

Required variables in `.env`:

```bash
# Required
BRIGHTDATA_API_KEY=your_api_key_here

# Optional (recommended for better synthesis)
OPENAI_API_KEY=your_openai_key_here
OPENAI_MODEL=gpt-4o-mini
```

**Getting your keys:**
- Bright Data API Key (required): https://brightdata.com/cp/settings
- OpenAI API Key (optional): https://platform.openai.com/api-keys

---

## Alternative Demos (For Reference)

These demos are available for learning purposes but not used by the main workflow:

### Direct API Demo

See [src/brightdata_api_demo.js](../src/brightdata_api_demo.js) for raw HTTP requests to Bright Data. Run with `npm run demo:api`.

### ReAct Agent Demo

See [src/brightdata_mcp_demo.js](../src/brightdata_mcp_demo.js) for MCP integration with ReAct agent. Run with `npm run demo:mcp`.

---

## Common Patterns

### Basic Research
```javascript
const result = await runWorkflow('AI developments 2025');
console.log(result.result);
```

### Advanced Usage with Statistics
```javascript
const result = await runWorkflow('quantum computing', { maxResults: 10 });

console.log('Research Report:');
console.log(result.result);

console.log('\nStatistics:');
console.log(`- Execution time: ${result.executionTime}s`);
console.log(`- Tools used: ${Object.keys(result.toolUsage).length}`);
Object.entries(result.toolUsage).forEach(([tool, count]) => {
  console.log(`  - ${tool}: ${count}x`);
});
```

### Integration with RAG Pipeline
```javascript
async function enhancedRAG(userQuery) {
  // Get comprehensive web research
  const webResearch = await runWorkflow(userQuery, { maxResults: 10 });

  // Use synthesized research in your RAG pipeline
  return {
    context: webResearch.result,
    metadata: {
      sources: webResearch.toolUsage,
      executionTime: webResearch.executionTime
    }
  };
}
```

---

## Error Handling

Always use try-catch:

```javascript
try {
  const result = await runWorkflow(query);
  // Use result
} catch (error) {
  console.error('Research failed:', error.message);
  // Handle error appropriately
}
```

Common issues:
- "Missing: BRIGHTDATA_API_KEY" ? Check `.env` file
- "Missing: OPENAI_API_KEY" ? LLM not configured; the workflow will use the heuristic fallback
- Authentication errors ? Verify keys are valid (for demos/LLM)

---

## Performance Tips

**Execution Time**: The workflow analyzes up to 10 sources. Typical execution: 30-90 seconds depending on query complexity and sources selected.

**Cost Optimization**:
- Uses gpt-4o-mini by default (cost-effective model)
- Bright Data charges based on data transferred
- Set OPENAI_MODEL=gpt-4o for higher quality at higher cost

**Quality vs Speed**:
- More sources (`maxResults: 10`) = more comprehensive but slower
- Fewer sources (`maxResults: 5`) = faster but less comprehensive

---

## Need More?

- **LangChain Docs**: [js.langchain.com](https://js.langchain.com)
- **Bright Data API**: [docs.brightdata.com](https://docs.brightdata.com)
- **MCP Protocol**: [modelcontextprotocol.io](https://modelcontextprotocol.io)

---

## Notes

- Main workflow: MCP search + scrape, then synthesis. Uses OpenAI if configured; otherwise falls back to heuristic summarization with citations.
- ReAct agent demo: The `demo:mcp` script demonstrates a ReAct-style agent and requires `OPENAI_API_KEY`.
- Environment: Only `BRIGHTDATA_API_KEY` is required to run the main workflow. `OPENAI_API_KEY` is optional (improves synthesis quality). `SERP_ZONE` and `UNLOCKER_ZONE` are only needed for the demo scripts.
