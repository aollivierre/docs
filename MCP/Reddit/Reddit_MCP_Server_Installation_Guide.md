# Reddit MCP Server Installation Guide for Claude Code

## Overview
This guide provides step-by-step instructions for installing and configuring the Reddit MCP (Model Context Protocol) server for use with Claude Code. This enables Claude to fetch Reddit posts, comments, and subreddit information through the public Reddit API.

## Prerequisites
- Claude Code installed and configured
- macOS or Linux system with terminal access
- Python 3.11 or higher
- `uv` package manager installed

---

## 🎯 Key Insights and Lessons Learned

### Critical Success Factors
1. **Correct Command Syntax**: Must use `uvx --from mcp-server-reddit python -m mcp_server_reddit`, not `uvx mcp-server-reddit`
2. **Wrapper Script Required**: Claude Code's `mcp add` command doesn't support uvx flags, requiring a wrapper script
3. **Session Restart Mandatory**: Claude Code requires a complete restart after adding new MCP servers
4. **Permissions Configuration**: Ensure proper permissions are set in Claude Code settings

### Major Pitfalls to Avoid
- ❌ Using `uvx mcp-server-reddit` directly (incorrect syntax)
- ❌ Attempting to use pip install (package not available in standard PyPI)
- ❌ Expecting immediate availability after configuration (restart required)
- ❌ Using uvx flags directly in Claude mcp add command

---

## 📋 Step-by-Step Installation Guide

### Step 1: Install Prerequisites

#### 1.1 Install uv (if not already installed)
```bash
brew install uv
```

#### 1.2 Verify uv installation
```bash
uvx --help
```
**Expected Output**: Should show uvx help menu

### Step 2: Validate Reddit MCP Server Access

#### 2.1 Test Reddit MCP server availability
```bash
uvx --from mcp-server-reddit python -m mcp_server_reddit --help
```
**Expected Output**:
```
usage: __main__.py [-h]

give a model the ability to access Reddit public API

options:
  -h, --help  show this help message and exit
```

### Step 3: Create Wrapper Script

#### 3.1 Create the wrapper script
```bash
cat > ~/.local/bin/reddit-mcp << 'EOF'
#!/bin/bash
uvx --from mcp-server-reddit python -m mcp_server_reddit "$@"
EOF
```

#### 3.2 Make wrapper script executable
```bash
chmod +x ~/.local/bin/reddit-mcp
```

#### 3.3 Test wrapper script
```bash
~/.local/bin/reddit-mcp --help
```
**Expected Output**: Same as Step 2.1

### Step 4: Configure Claude Code

#### 4.1 Add Reddit MCP server to Claude Code
```bash
claude mcp add reddit ~/.local/bin/reddit-mcp
```
**Expected Output**: `Added stdio MCP server reddit with command: /Users/[username]/.local/bin/reddit-mcp to local config`

#### 4.2 Verify configuration
```bash
claude mcp list
```
**Expected Output**: Should include `reddit: /Users/[username]/.local/bin/reddit-mcp`

#### 4.3 Check detailed configuration
```bash
claude mcp get reddit
```

### Step 5: Restart Claude Code

#### 5.1 Exit current Claude Code session
```bash
exit
```

#### 5.2 Start new Claude Code session
```bash
claude
```

### Step 6: Verification and Testing

#### 6.1 Verify Reddit MCP server is available
In the new Claude Code session, the Reddit server should appear in available MCP servers and provide these tools:
- `mcp__reddit__get_frontpage_posts`
- `mcp__reddit__get_subreddit_info`
- `mcp__reddit__get_subreddit_hot_posts`
- `mcp__reddit__get_subreddit_new_posts`
- `mcp__reddit__get_subreddit_top_posts`
- `mcp__reddit__get_post_content`

---

## 🚨 Troubleshooting Guide: What Failed and Why

### Failure 1: Direct uvx Command
**What was tried**:
```bash
claude mcp add reddit uvx mcp-server-reddit
```

**Why it failed**: 
- Incorrect module invocation syntax
- MCP protocol communication errors
- Missing `--from` flag and proper module specification

**Error symptoms**:
```
{"jsonrpc":"2.0","id":1,"error":{"code":-32602,"message":"Invalid request parameters"}}
WARNING:root:Failed to validate request: Received request before initialization was complete
```

### Failure 2: Using uvx Flags in Claude mcp add
**What was tried**:
```bash
claude mcp add reddit uvx --from mcp-server-reddit python -m mcp_server_reddit
```

**Why it failed**: 
- Claude's `mcp add` command doesn't support complex command arguments with flags
- Command parsing limitations in Claude Code

**Error symptoms**:
```
error: unknown option '--from'
```

### Failure 3: Pip Installation Attempts
**What was tried**:
```bash
pip install mcp-server-reddit
pip3 install mcp-server-reddit
```

**Why it failed**: 
- Package not available in standard pip repositories
- Outdated pip version
- Dependency resolution issues

**Error symptoms**:
```
ERROR: Could not find a version that satisfies the requirement mcp-server-reddit
```

### Failure 4: Expecting Immediate Availability
**What was tried**: 
Using MCP server immediately after configuration in same session

**Why it failed**: 
- Claude Code session limitation
- MCP servers only reload on session restart
- Internal connection management requires restart

**Error symptoms**:
```
Server "reddit" not found. Available servers: [other servers listed]
```

---

## 🔍 Validation Procedures

### Pre-Installation Validation
```bash
# Check if uv is installed
which uvx
# Should return: /Users/[username]/.local/bin/uvx

# Check Claude Code MCP functionality
claude mcp list
# Should return list of configured MCP servers
```

### Post-Installation Validation
```bash
# 1. Verify wrapper script works
~/.local/bin/reddit-mcp --help

# 2. Verify Claude Code configuration
claude mcp get reddit

# 3. Test MCP protocol communication (optional)
echo '{"jsonrpc": "2.0", "id": 1, "method": "tools/list"}' | ~/.local/bin/reddit-mcp
```

### Runtime Validation (After Restart)
In new Claude Code session, verify that Reddit MCP tools are available by attempting to use them.

---

## 📚 Available Reddit MCP Tools

Once successfully installed, these tools become available:

| Tool Name | Function | Description |
|-----------|----------|-------------|
| `get_frontpage_posts` | Fetch Reddit frontpage | Gets posts from Reddit's front page |
| `get_subreddit_info` | Subreddit metadata | Retrieves information about a specific subreddit |
| `get_subreddit_hot_posts` | Hot posts | Gets hot posts from a specific subreddit |
| `get_subreddit_new_posts` | New posts | Gets newest posts from a specific subreddit |
| `get_subreddit_top_posts` | Top posts | Gets top-rated posts from a specific subreddit |
| `get_post_content` | Full post details | Gets complete post content including comments |

---

## 🎯 Example Usage

After successful installation, you can use the Reddit MCP server like this:

```
# Fetch a specific Reddit post with comments
Use the Reddit MCP server to get the full content and comments for this post: 
https://www.reddit.com/r/ClaudeAI/comments/1lm9pfp/after_8_months_of_daily_ai_coding_i_built_a/

# Get hot posts from a subreddit
Get the top 10 hot posts from r/programming

# Get subreddit information
What information can you tell me about the r/MachineLearning subreddit?
```

---

## 🛠️ Advanced Configuration

### Permissions Management
The installation automatically adds necessary permissions to `~/.claude/settings.local.json`:
- `Bash(uvx:*)`
- `Bash(/Users/[username]/.local/bin/reddit-mcp:*)`
- `mcp__reddit__get_post_content` (and other Reddit tools)

### Custom Installation Location
If you need to install the wrapper script elsewhere:
```bash
# Create in custom location
cat > /path/to/custom/reddit-mcp << 'EOF'
#!/bin/bash
uvx --from mcp-server-reddit python -m mcp_server_reddit "$@"
EOF

# Make executable
chmod +x /path/to/custom/reddit-mcp

# Configure Claude Code
claude mcp add reddit /path/to/custom/reddit-mcp
```

---

## 📖 References and Research Validation

This installation validates the research claims about Reddit MCP integration:

✅ **Validated Claims**:
- `mcp-server-reddit` by Hawstein exists and is functional
- Supports frontpage posts, subreddit info, hot/new/top posts, and full post content
- Uses Reddit's public API via `redditwarp` (as claimed)
- Compatible with MCP clients like Claude Code

✅ **Available Tools Match Research**:
- `get_frontpage_posts` ✅
- `get_subreddit_info` ✅  
- `get_subreddit_hot_posts` ✅
- `get_subreddit_new_posts` ✅ (confirmed during testing)
- `get_subreddit_top_posts` ✅ (confirmed during testing)
- `get_post_content` ✅ (confirmed during testing)

---

## 🎉 Success Confirmation

Installation is successful when:
1. ✅ `claude mcp list` shows reddit server
2. ✅ Reddit MCP tools are available in Claude Code
3. ✅ Can successfully fetch Reddit content using MCP commands
4. ✅ No error messages during MCP server communication

---

*Last Updated: [Current Date]*  
*Installation tested on: macOS with Claude Code*  
*Reddit MCP Server Version: Latest available via uvx*