# MCP Server Setup Guide: Florence2 & PDF Reader

## Overview

This document covers the complete setup process for two MCP (Model Context Protocol) servers: **florence2** (computer vision) and **mcp_pdf_reader** (PDF processing with OCR). This guide documents all failures encountered, solutions implemented, and provides a comprehensive reproduction guide.

## Table of Contents

1. [Initial Problem](#initial-problem)
2. [MCP Server Analysis](#mcp-server-analysis)
3. [Failure Analysis & Solutions](#failure-analysis--solutions)
4. [Final Working Configuration](#final-working-configuration)
5. [Step-by-Step Reproduction Guide](#step-by-step-reproduction-guide)
6. [Tech Stack Analysis](#tech-stack-analysis)
7. [Documentation Alignment Assessment](#documentation-alignment-assessment)
8. [Key Insights & Best Practices](#key-insights--best-practices)

## Initial Problem

**Starting State**: Two MCP servers failing to connect
```
florence2           ✘ failed
mcp_pdf_reader      ✘ failed
```

**Error Pattern**: `spawn https://github.com/... ENOENT`

**Root Cause**: Claude MCP was trying to execute GitHub URLs directly as commands instead of proper executable paths.

## MCP Server Analysis

### Florence2 MCP Server
- **Purpose**: Computer vision and image analysis using Florence-2 model
- **Type**: CLI-based stdio MCP server
- **Tech Stack**: Python + PyTorch + Transformers + UV package manager
- **Repository**: https://github.com/jkawamoto/mcp-florence2
- **Communication**: stdio (stdin/stdout)
- **Dependencies**: 
  - Python 3.13+
  - PyTorch
  - Transformers >=4.49
  - Pillow >=11.1
  - MCP framework

### MCP PDF Reader Server
- **Purpose**: PDF text extraction, image extraction, and OCR capabilities
- **Type**: CLI-based stdio MCP server  
- **Tech Stack**: Python + FastMCP + PyMuPDF + Tesseract OCR + UV package manager
- **Repository**: https://github.com/labeveryday/mcp_pdf_reader
- **Communication**: stdio (stdin/stdout)
- **Dependencies**:
  - Python 3.12+
  - FastMCP >=0.2.0
  - PyMuPDF >=1.23.0
  - pytesseract >=0.3.10
  - Pillow >=10.0.0
  - **System dependency**: Tesseract OCR engine

## Failure Analysis & Solutions

### Failure 1: Invalid Command Execution
**Error**: `spawn https://github.com/jkawamoto/mcp-florence2 ENOENT`

**Cause**: MCP configuration was pointing to GitHub URLs instead of executable commands

**Solution**: Remove broken servers and set up proper local installations
```bash
claude mcp remove florence2
claude mcp remove mcp_pdf_reader
```

### Failure 2: Missing Local Repositories
**Error**: No local installations available

**Solution**: Clone repositories to local directory
```bash
mkdir -p mcp-servers
git clone https://github.com/jkawamoto/mcp-florence2 mcp-servers/mcp-florence2
git clone https://github.com/labeveryday/mcp_pdf_reader mcp-servers/mcp_pdf_reader
```

### Failure 3: Dependency Installation Issues
**Error**: `npm` commands failing, package.json not found

**Cause**: Both projects use Python + UV, not Node.js

**Solution**: Use UV package manager instead of npm
```bash
cd mcp-servers/mcp-florence2 && uv sync
cd mcp-servers/mcp_pdf_reader && uv sync
```

### Failure 4: Incorrect Module Paths
**Error**: `No module named mcp_florence2` / `No module named pdf_reader_server`

**Cause**: Wrong entry point specifications

**Solutions**:
- **Florence2**: Use `-m mcp_florence2` with virtual environment Python
- **PDF Reader**: Use direct path to `src/server.py`

### Failure 5: Command Argument Parsing Issues
**Error**: `unknown option '-m'` when adding MCP servers

**Solution**: Use `--` separator for command arguments
```bash
claude mcp add florence2 /path/to/python -- -m mcp_florence2
```

### Failure 6: MCP Connection Closes Immediately
**Error**: `MCP error -32000: Connection closed`

**Cause**: Missing system dependency (Tesseract OCR) for PDF reader

**Solution**: Install Tesseract OCR
```bash
brew install tesseract  # macOS
```

### Failure 7: Working Directory Issues
**Error**: PDF reader still failing after tesseract installation

**Cause**: UV needs to run from correct working directory

**Solution**: Create wrapper script to handle directory changes and environment setup

## Final Working Configuration

### Florence2
```bash
claude mcp add florence2 ~/mcp-servers/mcp-florence2/.venv/bin/python -- -m mcp_florence2
```

### MCP PDF Reader
```bash
# Wrapper script approach
chmod +x ~/mcp-servers/mcp_pdf_reader/run_server.sh
claude mcp add mcp_pdf_reader ~/mcp-servers/mcp_pdf_reader/run_server.sh
```

**Wrapper Script Content**:
```bash
#!/bin/bash
cd ~/mcp-servers/mcp_pdf_reader
export PATH="$HOME/.local/bin:$PATH"
uv run python src/server.py
```

## Step-by-Step Reproduction Guide

### Prerequisites
1. **System Requirements**:
   - macOS/Linux environment
   - Python 3.12+ available
   - Git installed
   - Homebrew (macOS) or equivalent package manager

2. **Install UV Package Manager**:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   source $HOME/.local/bin/env
   ```

3. **Install System Dependencies**:
   ```bash
   # macOS
   brew install tesseract
   
   # Ubuntu/Debian
   sudo apt update && sudo apt install tesseract-ocr tesseract-ocr-eng
   ```

### Setup Process

#### Step 1: Remove Any Existing Broken Servers
```bash
claude mcp remove florence2 2>/dev/null || true
claude mcp remove mcp_pdf_reader 2>/dev/null || true
```

#### Step 2: Create Project Directory
```bash
mkdir -p ~/mcp-servers
cd ~/mcp-servers
```

#### Step 3: Clone Repositories
```bash
git clone https://github.com/jkawamoto/mcp-florence2
git clone https://github.com/labeveryday/mcp_pdf_reader
```

#### Step 4: Install Dependencies
```bash
# Florence2
cd mcp-florence2
uv sync
cd ..

# PDF Reader
cd mcp_pdf_reader
uv sync
cd ..
```

#### Step 5: Create PDF Reader Wrapper Script
```bash
cat > mcp_pdf_reader/run_server.sh << 'EOF'
#!/bin/bash
cd "$(dirname "$0")"
export PATH="$HOME/.local/bin:$PATH"
uv run python src/server.py
EOF

chmod +x mcp_pdf_reader/run_server.sh
```

#### Step 6: Add MCP Servers
```bash
# Get absolute paths
FLORENCE_PATH="$(pwd)/mcp-florence2/.venv/bin/python"
PDF_SCRIPT_PATH="$(pwd)/mcp_pdf_reader/run_server.sh"

# Add servers
claude mcp add florence2 "$FLORENCE_PATH" -- -m mcp_florence2
claude mcp add mcp_pdf_reader "$PDF_SCRIPT_PATH"
```

#### Step 7: Verify Installation
```bash
claude mcp list
```

Expected output should show both servers listed.

#### Step 8: Test Connection
Restart Claude Code and verify both servers show as "✔ connected" in the MCP management interface.

## Tech Stack Analysis

### Florence2
- **Core Framework**: MCP (Model Context Protocol)
- **Language**: Python 3.13
- **ML Framework**: PyTorch + HuggingFace Transformers
- **Model**: Microsoft Florence-2 vision model
- **Package Manager**: UV (modern Python package manager)
- **Interface Type**: stdio-based CLI
- **Communication Protocol**: JSON-RPC over stdio

### MCP PDF Reader
- **Core Framework**: FastMCP (MCP framework wrapper)
- **Language**: Python 3.12+
- **PDF Processing**: PyMuPDF (fast PDF parsing)
- **OCR Engine**: Tesseract (system dependency)
- **Image Processing**: Pillow
- **Package Manager**: UV
- **Interface Type**: stdio-based CLI
- **Communication Protocol**: JSON-RPC over stdio

### Common Patterns
- Both use **stdio communication** (not HTTP/SSE)
- Both use **UV package manager** (not pip/conda)
- Both are **Python-based** (not Node.js despite .js references)
- Both follow **MCP specification v2024-11-05**

## Documentation Alignment Assessment

### Florence2 Repository
**Documentation Quality**: ⭐⭐⭐⭐⭐ (Excellent)

**Alignment with Reality**: ✅ **Highly Aligned**
- Clear pyproject.toml configuration
- Proper UV setup instructions
- Correct module entry point (`-m mcp_florence2`)
- Dependencies properly specified
- Working examples provided

**Discrepancies**: None significant

### MCP PDF Reader Repository
**Documentation Quality**: ⭐⭐⭐⭐⭐ (Excellent)

**Alignment with Reality**: ⚠️ **Mostly Aligned with Caveats**

**Accurate Documentation**:
- Clear system dependency requirements (Tesseract)
- Proper UV setup instructions
- Correct dependency specifications
- Good usage examples

**Issues Found**:
1. **Entry Point Confusion**: 
   - `pyproject.toml` specifies `pdf_reader_server:main` 
   - But actual working path is `src/server.py`
   - `main.py` in root is just a placeholder

2. **Working Directory Sensitivity**:
   - Documentation doesn't mention UV needs correct working directory
   - Required wrapper script approach not documented

3. **MCP Integration**:
   - Missing specific Claude Code integration examples
   - Command-line argument handling not clearly documented

## Key Insights & Best Practices

### Critical Dependencies
1. **System-level dependencies matter**: Tesseract OCR was critical for PDF reader
2. **Package managers**: Both projects use UV, not traditional pip
3. **Python version specificity**: Each project has specific Python version requirements
4. **Working directories**: UV-based projects are sensitive to working directory

### MCP Server Configuration Patterns
1. **Use absolute paths**: Relative paths cause issues in MCP configuration
2. **Argument separation**: Use `--` to separate command from arguments
3. **Environment handling**: Some servers need environment variables or working directory setup
4. **Wrapper scripts**: Often necessary for complex startup requirements

### Troubleshooting Approach
1. **Test manually first**: Always verify servers work standalone before MCP integration
2. **Check logs systematically**: MCP logs provide detailed error information
3. **Verify system dependencies**: Many failures are due to missing system packages
4. **Isolate issues**: Remove and re-add servers to isolate configuration problems

### Common Failure Patterns
1. **GitHub URLs as commands**: Initial misconfiguration pattern
2. **Package manager mismatches**: Assuming npm when projects use UV/pip
3. **Missing system dependencies**: OCR, image processing libraries
4. **Working directory issues**: UV and path-sensitive tools
5. **Argument parsing**: Command-line argument handling in MCP

### Best Practices for Future Setups
1. **Always check system dependencies first**
2. **Use UV for Python projects when specified**
3. **Create wrapper scripts for complex startup sequences**
4. **Test servers manually before MCP integration**
5. **Document working configurations for future reference**
6. **Use absolute paths in MCP configurations**

## Conclusion

This setup process revealed that while MCP servers can be powerful tools, their integration requires careful attention to:
- System dependencies
- Package manager requirements  
- Working directory sensitivities
- Command-line argument handling

The documentation for both projects was generally accurate, but some practical integration details were missing. The key to success was systematic troubleshooting and understanding that these are sophisticated Python applications with specific runtime requirements, not simple command-line utilities.

**Final Status**: ✅ Both servers successfully configured and operational

- **florence2**: ✔ Connected (Computer vision/image analysis)
- **mcp_pdf_reader**: ✔ Connected (PDF processing with OCR support)