# Getting Started

## Installation

```bash
git clone https://github.com/brightdata/brightdata-workshop-calhack.git
cd brightdata-workshop-calhack
npm install
```

## Setup

### 1. Sign Up
Create account at [brightdata.com](https://brightdata.com)

### 2. Get Bright Data API Key
Go to [brightdata.com/cp/settings](https://brightdata.com/cp/settings)
- Navigate to "API tokens"
- Generate new token
- Copy API key

### 3. (Optional) Get OpenAI API Key
Go to [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
- Create new API key
- Copy key (optional; enables higher-quality synthesis)

### 4. Configure
```bash
cp .env.example .env
```

Edit `.env`:
```bash
BRIGHTDATA_API_KEY=your_api_key
# Optional: enables LLM synthesis (heuristic fallback is used if omitted)
OPENAI_API_KEY=your_openai_key
```

**Note**: Zone configuration (SERP_ZONE, UNLOCKER_ZONE) is optional. The main workflow uses MCP and doesn't need zones. Zones are only required for the `demo:api` script to demonstrate direct HTTP API calls.

## Run

```bash
# Main workflow (analyzes up to 10 sources)
npm start "your search query"

# Alternative demos (for learning)
npm run demo:api            # Direct HTTP API approach
npm run demo:mcp            # ReAct agent with MCP
```

## What's Happening?

The main workflow uses Bright Data MCP tools to search, scrape, and then synthesize results:

1. **Initialize MCP Client**: Connects to Bright Data MCP server
2. **Search**: Uses `search_engine` tool to find relevant sources
3. **Select URLs**: Chooses up to 10 URLs with domain diversity
4. **Scrape**: Uses `scrape_as_markdown` to extract clean content
5. **Synthesize**: Combines information (LLM or heuristic) with citations

Bright Data automatically:
- Rotates IPs
- Solves CAPTCHAs
- Renders JavaScript
- Handles retries

## Architecture

```
┌─────────────────────────────────────────────┐
│         Your Query                          │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  Main Workflow (src/main_workflow.js)      │
│  • Search with domain diversity            │
│  • Scrape selected sources                 │
│  • Synthesize findings                     │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  Bright Data MCP Tools                      │
│  • search_engine (Google/Bing/Yandex)       │
│  • scrape_as_markdown (any URL)             │
│  • web_data_* (structured extractors)       │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  Comprehensive Research Report              │
│  • Key findings                             │
│  • Source citations                         │
│  • Tool usage statistics                    │
└─────────────────────────────────────────────┘
```

## Next Steps

### Integrate with Your Application

```javascript
import { runWorkflow } from './src/main_workflow.js';

// Get comprehensive research
const result = await runWorkflow('your query', { maxResults: 10 });

// Use in your application
console.log(result.result);       // Research report
console.log(result.toolUsage);    // Tool statistics
console.log(result.executionTime); // Performance metrics
```

### Use in RAG Pipeline

```javascript
async function enhancedRAG(userQuery) {
  const webResearch = await runWorkflow(userQuery, { maxResults: 10 });

  // The workflow has already synthesized information
  return {
    context: webResearch.result,
    metadata: webResearch.toolUsage
  };
}
```

### Build Custom Workflow

Modify the system prompt in [src/main_workflow.js:182](src/main_workflow.js#L182) to specialize the workflow for your use case:

```javascript
{ role: 'system', content: 'You are a specialized [DOMAIN] research assistant...' }
```

## Troubleshooting

**Missing environment variables**
- Check `.env` file exists
- Verify BRIGHTDATA_API_KEY is set (required)
- OPENAI_API_KEY is optional

**Authentication failed**
- Verify Bright Data API key at brightdata.com/cp/settings
- Verify OpenAI API key at platform.openai.com/api-keys

**Workflow takes too long**
- Normal for comprehensive research (30-90s for 10 sources)
- Reduce `maxResults` for faster testing

**Workflow output is brief**
- Try a more specific query
- Consider adding OPENAI_API_KEY for better synthesis

## Python Developers

If you prefer working in Python, Bright Data offers a [Python SDK](https://github.com/luminati-io/bright-data-sdk-python) that provides high-level abstractions and type safety for production applications.

## Support

- Docs: [docs.brightdata.com](https://docs.brightdata.com)
- LangChain: [js.langchain.com](https://js.langchain.com)
- Tutorial: [TUTORIAL.md](../TUTORIAL.md)
- API Reference: [API_REFERENCE.md](API_REFERENCE.md)

---

## No-LLM Offline Mode

- You can run the main workflow without an OpenAI key. In that case, it uses a heuristic summarizer (headings, first sentences, keywords, citations).
- For the best results, set `OPENAI_API_KEY` in `.env`.
- The `demo:mcp` script uses OpenAI for the ReAct agent and requires the key to run.
