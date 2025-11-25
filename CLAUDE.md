# CLAUDE.md - AI Assistant Guide for AI Hedge Fund

## Project Overview

This is an **educational AI-powered hedge fund simulator** that uses multiple AI agents mimicking famous investors to generate trading signals. It does NOT execute real trades - it's designed for learning about quantitative investing and multi-agent AI systems.

## Quick Start Commands

```bash
# Install dependencies
poetry install

# Run real-time analysis
poetry run python src/main.py --ticker AAPL,MSFT,NVDA

# Run with reasoning output
poetry run python src/main.py --ticker AAPL --show-reasoning

# Run backtester
poetry run python src/backtester.py

# Format code
poetry run black src/

# Run tests
poetry run pytest
```

## Architecture

```
START → [Analyst Agents (parallel)] → Risk Manager → Portfolio Manager → END
```

The system uses **LangGraph** for workflow orchestration:
1. Selected analyst agents run in parallel
2. Results aggregate to Risk Manager (position limits)
3. Portfolio Manager makes final trading decisions

## Directory Structure

```
src/
├── main.py              # Entry point - interactive CLI
├── backtester.py        # Historical backtesting engine
├── agents/              # All AI agents
│   ├── ben_graham.py        # Value investing
│   ├── warren_buffett.py    # Quality at fair price
│   ├── charlie_munger.py    # Quality businesses
│   ├── bill_ackman.py       # Activist investing
│   ├── cathie_wood.py       # Disruptive innovation
│   ├── stanley_druckenmiller.py  # Macro trading
│   ├── fundamentals.py      # Financial health analysis
│   ├── technicals.py        # Technical indicators
│   ├── valuation.py         # DCF, P/E analysis
│   ├── sentiment.py         # News + insider trades
│   ├── risk_manager.py      # Position limits
│   └── portfolio_manager.py # Final decisions
├── tools/api.py         # Financial Datasets API wrapper
├── data/
│   ├── models.py        # Pydantic data models
│   └── cache.py         # In-memory caching
├── graph/state.py       # LangGraph state definition
├── llm/models.py        # LLM provider configurations
└── utils/
    ├── llm.py           # LLM calling utility
    ├── analysts.py      # Agent configuration registry
    ├── display.py       # Terminal output formatting
    ├── progress.py      # Progress tracking
    └── visualize.py     # Graph visualization
```

## Key Concepts

### Agent State (`src/graph/state.py`)
All agents share state via `AgentState` TypedDict:
- `messages`: LangChain message history
- `data`: Contains tickers, portfolio, dates, analyst_signals
- `metadata`: show_reasoning, model_name, model_provider

### Adding a New Analyst Agent
1. Create `src/agents/your_agent.py`
2. Define analysis logic and signal generation
3. Register in `src/utils/analysts.py`
4. Agent will be available in CLI selection

### Signal Format
Each agent outputs:
```python
{
    "signal": "bullish" | "bearish" | "neutral",
    "confidence": 0-100,
    "reasoning": "explanation string"
}
```

## Environment Variables

Required in `.env`:
```bash
# LLM Providers (at least one required)
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
GROQ_API_KEY=
DEEPSEEK_API_KEY=
GOOGLE_API_KEY=

# Financial Data
FINANCIAL_DATASETS_API_KEY=
```

Free data available for: AAPL, GOOGL, MSFT, NVDA, TSLA

## Supported LLM Models

| Provider | Models |
|----------|--------|
| OpenAI | GPT-4.5, GPT-4o, o1, o3-mini |
| Anthropic | Claude 3.5 Sonnet, 3.5 Haiku, 3.7 Sonnet |
| DeepSeek | DeepSeek R1, V3 |
| Groq | Llama 3.3 70B |
| Google | Gemini 2.0 Flash, Pro |

## Code Style

- **Line length**: 420 characters (configured in pyproject.toml)
- **Formatter**: Black
- **Import sorting**: isort
- **Type hints**: Use Pydantic models for data validation
- **Python version**: 3.9+

## Common Patterns

### Calling LLMs
```python
from utils.llm import call_llm

result = call_llm(
    prompt="Your prompt",
    model_name="gpt-4o",
    model_provider="OpenAI",
    pydantic_model=YourResponseModel,  # Optional
    temperature=0.7
)
```

### Fetching Financial Data
```python
from tools.api import get_prices, get_financial_metrics, get_insider_trades

prices = get_prices(ticker="AAPL", start_date="2024-01-01", end_date="2024-12-31")
metrics = get_financial_metrics(ticker="AAPL", limit=10)
```

### Agent Function Signature
```python
def your_agent(state: AgentState) -> dict:
    """Agent that analyzes stocks based on specific criteria."""
    data = state["data"]
    tickers = data["tickers"]
    # ... analysis logic
    return {"messages": [HumanMessage(content=json.dumps(result))]}
```

## Testing

When modifying agents or adding features:
1. Test with free tickers first (AAPL, GOOGL, MSFT, NVDA, TSLA)
2. Use `--show-reasoning` to verify logic
3. Run backtester to validate strategy performance

## Important Notes

- This is for **educational purposes only** - no real trading
- Financial data API has rate limits - caching helps reduce calls
- Each agent should be independent and produce consistent signal format
- Risk Manager enforces 20% max position per stock
- Portfolio starts with $100,000 default cash
