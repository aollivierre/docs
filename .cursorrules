# Markdown Linting Guidelines

## Common Rules and Solutions

### MD022 - Headings Must Be Surrounded by Blank Lines

- Each heading must have a blank line before and after

- Example:

```markdown
## Good Heading

Content here...

### Next Heading
```

### MD031 - Fenced Code Blocks Must Be Surrounded by Blank Lines

- Add blank lines before and after code blocks
- Example:

```markdown
Here's some code:

```python
print("Hello World")
```

Next content here...
```

### MD032 - Lists Must Be Surrounded by Blank Lines
- Add blank lines before and after lists
- Also add blank lines between different list types
- Example:
```markdown
Here's a list:

- Item 1
- Item 2
- Item 3

Next content here...
```

### MD040 - Fenced Code Blocks Should Have a Language Specified

- Always specify the language for syntax highlighting
- Use `text` for plain text or error messages
- Example:

```markdown
```python
def hello():
    print("Hello World")
```

```text
Error: Connection failed
```
```

### MD047 - Files Must End with Single Newline
- Ensure exactly one blank line at end of file
- No multiple blank lines at end
- No missing newline at end

### MD009 - No Trailing Spaces
- Remove spaces at the end of lines
- Exception: Two spaces for line breaks
- Example:
```markdown
This line has no trailing spaces
This line has two spaces for break  
Next line here
```

## Best Practices for Implementation

1. Spacing Rules:
   - One blank line before and after headings
   - One blank line before and after lists
   - One blank line before and after code blocks
   - One newline at end of file
   - No trailing spaces unless needed for line breaks (two spaces)

2. Code Blocks:
   - Always specify language
   - Use `text` for plain text/logs/errors
   - Surround with blank lines

3. Lists:
   - Surround with blank lines
   - Consistent indentation
   - Blank lines between different list types

4. File Structure:
   - Single newline at end
   - No trailing spaces
   - Consistent heading hierarchy

## Usage with Claude

When requesting markdown documentation from Claude, specify:

1. "Please follow markdown linting rules MD009, MD022, MD031, MD032, MD040, and MD047"
2. "Ensure proper spacing around headings, lists, and code blocks"
3. "Always specify language in code blocks"
4. "Remove trailing spaces and end file with single newline"
