# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A multi-AI agent stock analysis system for Chinese A-share markets built with Python + Streamlit + DeepSeek. The system simulates an analyst team providing comprehensive stock investment analysis and trading decision support.

**Primary Language:** Chinese (UI, comments, documentation)

## Build & Run Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run locally (two options)
streamlit run app.py
# or
python run.py

# Docker deployment
docker-compose up -d
# Access at: http://localhost:8503
```

## Project Structure

```
aiagents-stock/
├── app.py                    # Main Streamlit entry point
├── run.py                    # Startup script with dependency checks
├── src/                      # Source code (organized by layer)
│   ├── ui/                   # Streamlit UI components
│   │   ├── longhubang_ui.py
│   │   ├── sector_strategy_ui.py
│   │   ├── smart_monitor_ui.py
│   │   ├── portfolio_ui.py
│   │   ├── main_force_ui.py
│   │   ├── monitor_manager.py
│   │   └── config_manager.py
│   ├── data/                 # Data fetching layer
│   │   ├── stock_data.py
│   │   ├── data_source_manager.py
│   │   └── [feature]_data.py
│   ├── ai/                   # AI agents and LLM clients
│   │   ├── deepseek_client.py
│   │   ├── ai_agents.py
│   │   └── [feature]_agents.py
│   ├── db/                   # Database models (SQLite)
│   │   ├── database.py
│   │   └── [feature]_db.py
│   ├── engines/              # Business logic engines
│   │   └── [feature]_engine.py
│   ├── services/             # Background services & schedulers
│   │   ├── monitor_service.py
│   │   ├── notification_service.py
│   │   └── [feature]_scheduler.py
│   ├── utils/                # Utilities & configuration
│   │   ├── config.py
│   │   ├── model_config.py
│   │   └── miniqmt_interface.py
│   └── pdf/                  # PDF report generators
│       └── [feature]_pdf.py
├── data/                     # SQLite database files (*.db)
├── conf/                     # Configuration files (.env, *.json)
├── extra/                    # Non-core/legacy code
├── docs/                     # Documentation (60+ files)
├── Dockerfile               # Docker image build
├── docker-compose.yml       # Docker orchestration
└── requirements.txt         # Python dependencies
```

## Import Conventions

All imports use absolute paths from project root:
```python
from src.data.stock_data import StockDataFetcher
from src.ai.deepseek_client import DeepSeekClient
from src.db.database import db
from src.services.notification_service import notification_service
from src.ui.config_manager import config_manager
```

For same-directory imports, use relative imports:
```python
from .deepseek_client import DeepSeekClient  # within src/ai/
```

## Architecture

### Core Application Flow
```
Streamlit UI (app.py)
    ↓
UI Components (src/ui/*.py)
    ↓
Engine Layer (src/engines/*.py)
    ↓
AI Agents (src/ai/*.py)
    ↓
DeepSeek Client (src/ai/deepseek_client.py)
    ↓
Data Layer (src/data/*.py)
    ↓
Database (src/db/*.py - SQLite)
```

### Module Pattern
Each feature follows a consistent structure across directories:
- `src/data/*_data.py` - Data fetching
- `src/db/*_db.py` - Database model (SQLite)
- `src/ai/*_agents.py` - AI agent definitions
- `src/engines/*_engine.py` - Business logic
- `src/ui/*_ui.py` - Streamlit UI component
- `src/services/*_scheduler.py` - Scheduled tasks (if applicable)
- `src/pdf/*_pdf.py` - PDF report generation

### Key Components
- **StockAnalysisAgents** (`src/ai/ai_agents.py`): Coordinates 4 analysis agents (technical, fundamental, fund flow, risk)
- **DeepSeekClient** (`src/ai/deepseek_client.py`): OpenAI SDK wrapper for DeepSeek API with reasoner model support
- **StockDataFetcher** (`src/data/stock_data.py`): Multi-source data with fallback (TDX → AKShare → Tushare → YFinance)
- **NotificationService** (`src/services/notification_service.py`): Email + Webhook (DingTalk/Feishu)

### Data Sources Priority
1. TDX API (local, fastest) - configurable via `TDX_ENABLED`
2. AKShare (Chinese market primary)
3. Tushare (backup)
4. YFinance (international)

## Key Environment Variables

```env
DEEPSEEK_API_KEY          # Required - AI analysis
TUSHARE_TOKEN             # Optional - backup data source
TDX_ENABLED               # Optional - local TDX data
MINIQMT_ENABLED           # Optional - trading execution
EMAIL_ENABLED             # Optional - email notifications
WEBHOOK_ENABLED           # Optional - DingTalk/Feishu webhooks
```

## Feature Modules

| Module | Purpose | Key Files |
|--------|---------|-----------|
| Stock Analysis | Multi-agent stock analysis with PDF export | `app.py`, `src/ai/ai_agents.py` |
| 主力选股 (Main Force) | Institutional capital flow analysis | `src/ui/main_force_ui.py`, `src/engines/main_force_analysis.py` |
| 智瞰龙虎 (Longhubang) | Dragon Tiger List analysis | `src/ui/longhubang_ui.py`, `src/engines/longhubang_engine.py` |
| 智策 (Sector Strategy) | Sector rotation strategy | `src/ui/sector_strategy_ui.py`, `src/engines/sector_strategy_engine.py` |
| AI盯盘 (Smart Monitor) | Real-time AI trading decisions | `src/ui/smart_monitor_ui.py`, `src/engines/smart_monitor_engine.py` |
| 持仓分析 (Portfolio) | Portfolio batch analysis with scheduling | `src/ui/portfolio_ui.py`, `src/services/portfolio_manager.py` |
| 实时监测 (Monitor) | Price alert monitoring | `src/ui/monitor_manager.py`, `src/services/monitor_service.py` |

## Code Conventions

- Snake_case for files and functions
- CamelCase for class names
- Private methods prefixed with `_`
- Chinese comments for domain-specific logic
- JSON serialization for database storage
- Pandas DataFrames for data processing
- Plotly for interactive charts

## A-Share Market Rules

The system implements T+1 trading rules:
- Stocks bought today cannot be sold until next trading day
- Trading hours: 9:30-11:30, 13:00-15:00
- `SmartMonitorEngine` enforces these constraints

## Documentation

Extensive documentation in `docs/` folder (60+ markdown files). Key docs:
- `docs/QUICK_START.md` - Getting started
- `docs/DOCKER_DEPLOYMENT.md` - Container deployment
- `docs/PORTFOLIO_USAGE.md` - Portfolio analysis
- Feature-specific guides for each module
