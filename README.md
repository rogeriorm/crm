# CRM Project

This project uses Claude Code with MCP (Model Context Protocol) servers to integrate with Notion.

## Setup Instructions

### Prerequisites

- Node.js and npm installed
- Claude Code extension installed in your IDE
- Notion API key ([Get one here](https://www.notion.so/my-integrations))

### Configuration Steps

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd crm
   ```

2. **Set up environment variables**

   Copy the example environment file:
   ```bash
   # macOS/Linux
   cp .env.example .env

   # Windows
   copy .env.example .env
   ```

3. **Add your Notion API key**

   Edit `.env` and replace `your_notion_api_key_here` with your actual Notion API key:
   ```
   NOTION_API_KEY=secret_xxxxxxxxxxxxxxxxxxxx
   ```

4. **Connect Claude Code to MCP**

   The `.mcp.json` file is already configured. When you open this project in Claude Code, it will automatically:
   - Detect the MCP configuration
   - Prompt you to connect to the Notion server
   - Load the server using your environment variables

### Cross-Platform Compatibility

This configuration works on:
- **macOS** - Uses default npm/npx
- **Windows** - Uses npx from your Node.js installation
- **Linux** - Uses default npm/npx

The environment variable syntax `${NOTION_API_KEY}` is supported across all platforms.

### Usage

Once configured, you can use AI agents to automate CRM workflows:
- Analyze opportunities after client meetings
- Get recommendations for pipeline updates
- Track patterns across interactions
- Maintain governance with human validation gates

**📖 See [docs/QUICK-START.md](docs/QUICK-START.md) for complete usage guide (< 5 minutes)**

### Troubleshooting

**MCP server not connecting?**
- Verify Node.js is installed: `node --version`
- Check your `.env` file exists and contains your API key
- Restart Claude Code

**Windows-specific issues?**
- Ensure Node.js is in your PATH
- Try running `npx -y @notionhq/mcp-server-notion` manually to test

### Security Notes

- Never commit `.env` to version control (it's in `.gitignore`)
- Keep your Notion API key secure
- Share `.env.example` instead, not `.env`

---

## Documentation

- **[Quick Start Guide](docs/QUICK-START.md)** - Get started in < 5 minutes
- **[Troubleshooting](docs/TROUBLESHOOTING.md)** - Common issues and solutions
- **[Documentation Index](docs/README.md)** - Complete documentation map
- **[Principles](docs/PRINCIPLES.md)** - Design philosophy (27 core principles)
