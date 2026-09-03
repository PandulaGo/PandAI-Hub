# AI Agents Skills

A comprehensive collection of AI agent configurations, documentation templates, and automation scripts for enhanced development workflows.

## Overview

This repository provides:
- **Opencode Configuration** - Pre-configured AI coding assistant settings with specialized agents
- **AutoHotkey Scripts** - Windows automation scripts for app launching and workflow optimization
- **Agent Skills** - Documentation templates and context synchronization ledgers for AI agents

## Project Structure

```
AI Agents Skills/
├── Agents Skills/                 # AI agent documentation templates
│   ├── Architecture and Other Details.md
│   └── Context Ledger.md
├── Opencode/                      # OpenCode AI assistant configuration
│   ├── opencode.jsonc
│   └── ReadMe.md
└── autohot key scripts/           # Windows automation scripts
    ├── Helium-App-Launcher-Backend-and-Frontend.ahk
    └── ReadMe.md
```

## Installation

### Prerequisites

- **Git** - Version control system
- **Node.js** (v18+) - Runtime for OpenCode
- **AutoHotkey** (v2+) - For running .ahk scripts (Windows only)

### Clone the Repository

```bash
git clone https://github.com/yourusername/AI-Agents-Skills.git
cd AI-Agents-Skills
```

### Install OpenCode

```bash
# Install OpenCode globally via npm
npm install -g opencode

# Verify installation
opencode --version
```

### Install Dependencies

```bash
# No additional npm packages required for this repository
# OpenCode handles its own dependencies internally
```

## Configuration

### OpenCode Setup

1. **Copy the configuration file** to your OpenCode config directory:

   ```bash
   # Windows
   mkdir %USERPROFILE%\.config\opencode
   copy Opencode\opencode.jsonc %USERPROFILE%\.config\opencode\

   # macOS/Linux
   mkdir -p ~/.config/opencode
   cp Opencode/opencode.jsonc ~/.config/opencode/
   ```

2. **Update the configuration paths** in `opencode.jsonc`:

   ```jsonc
   {
     "model": "deepseek/deepseek-v4-pro",
     "small_model": "deepseek/deepseek-v4-flash",
     "instructions": [
       "C:/path/to/your/Agents Skills/Architecture and Other Details.md",
       "C:/path/to/your/Agents Skills/Context Ledger.md"
     ]
   }
   ```

### AutoHotkey Scripts

1. Install AutoHotkey v2 from [autohotkey.com](https://www.autohotkey.com/)
2. Double-click any `.ahk` file to run it
3. Optionally, add scripts to startup for auto-launch

## Application Startup

### OpenCode Service

| Property | Value |
|----------|-------|
| **Default Port** | `5000` |
| **API Base Path** | `http://localhost:5000/api/v1` |
| **Web Interface** | `http://localhost:3000` |

### Starting OpenCode

```bash
# Start the OpenCode server
opencode serve

# Or run in development mode
opencode dev
```

### Environment Variables

```bash
# Optional environment configuration
PORT=5000                    # API server port
NODE_ENV=development         # Environment mode
API_KEY=your-api-key         # API authentication key
```

## Documentation

### Agents Skills Documentation

| Document | Purpose |
|----------|---------|
| [Architecture and Other Details.md](./Agents%20Skills/Architecture%20and%20Other%20Details.md) | System architecture blueprint, database schema, and API endpoint reference |
| [Context Ledger.md](./Agents%20Skills/Context%20Synchronization.md) | Session tracking and context synchronization across machines |

### Key Features

#### Architecture Documentation
- Executive summary and component blueprint
- Tech stack mapping (Frontend, Backend, Data Layer)
- Data flow and communication lifecycle diagrams
- Database schema and entity relationships
- RESTful API endpoint reference with request/response examples

#### Context Ledger
- Machine context and environment state tracking
- Session history with file change ledgers
- State handover instructions for machine switching
- Engineering sprint progress documentation

## Usage Examples

### Running the Documentation Agent

```bash
# Start OpenCode with the doc-agent configuration
opencode --agent doc-agent

# The agent will automatically sync codebase changes to ./docs/
```

### Using AutoHotkey Scripts

```ahk
; Example: Helium App Launcher
; Press Ctrl+Alt+L to launch configured applications
^!l::
    Run, "C:\Program Files\App1\app1.exe"
    Run, "C:\Program Files\App2\app2.exe"
return
```

## API Reference

### Authentication Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/auth/register` | User registration |
| POST | `/api/v1/auth/login` | User login |

### User Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/users/:id` | Get user profile |

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- **Issues**: [GitHub Issues](https://github.com/yourusername/AI-Agents-Skills/issues)
- **Documentation**: See the `Agents Skills/` folder for detailed guides

---

*Last Updated: September 2026*
