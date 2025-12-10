# CovScript Handbook - AI Coding Guidelines

## Project Overview

This is the **CovScript Handbook** (`handbook`) - a comprehensive technical documentation project for CovScript, a cross-platform dynamic programming language. The entire codebase is 100% Markdown documentation, organized as a progressive multilevel learning resource with no compilation or build processes.

**Key Facts:**
- **Subject Matter**: CovScript (dynamic scripting language)
- **Project Type**: Educational documentation (Markdown only, no code to build/test)
- **Target Audience**: CovScript learners, developers, and language enthusiasts
- **Multilingual Support**: Currently documented in Simplified Chinese (中文), with planned expansion to other languages
- **Documentation Standards**: Based on CSC2511XX (CovScript 3) and ECS2106XX (CovScript 4)
- **Active Maintenance**: Regularly updated to reflect official language specifications

## Architecture & Organization

The handbook is organized in a **strict progressive learning sequence**, where each section builds upon previous knowledge. The current structure reflects the organization as of 2025-12-10, but **may evolve over time**. Always refer to the actual directory structure and `README.md` files within each section for the authoritative source of organization.

**Current Structure (Reference Only):**

The handbook typically organizes content into major chapters:
- **Chapter 1 (intro/)**: Getting started - Installation, setup, first program
- **Chapter 2 (syntax/)**: Language fundamentals - Syntax basics through async/coroutines
- **Chapter 3 (ecosystem/)**: Extension libraries - Third-party integrations and specialized tools
- **Cross-cutting references**: FAQ.md for common questions, CHANGELOG.md for update history

Each chapter contains sequentially-numbered files (01-*.md, 02-*.md, etc.) following a strict progression where later files assume knowledge of earlier ones.

**Prerequisites & Dependencies:**
- Generally: intro/ → syntax/ → ecosystem/
- Within syntax/: strict sequential dependency (2.1→2.2→...→2.14)
- Within ecosystem/: each library is mostly independent but assumes syntax/ mastery
- FAQ.md: serves as a cross-cutting reference for all sections

**Important for Agents**: 
- Don't hardcode file paths or assumptions about project structure
- When referencing documentation organization, check the actual file structure in the workspace
- Relative paths like `../syntax/README.md` should be verified before use
- Consult section README.md files for up-to-date chapter listings

## Key Conventions & Patterns

### 1. File Naming & Structure
- **Markdown files only** - No code compilation, just documentation
- **Sequential numbering**: `01-*.md`, `02-*.md` indicates reading order
- **Bilingual code blocks**: CovScript examples with English-language explanations (current standard for Chinese locale)
- **Cross-section references**: Use relative paths like `../syntax/README.md`, `../ecosystem/05-cspkg.md`
- **Language-aware structure**: Current implementation uses Chinese; multilingual versions may use language-specific directories (e.g., `/en/`, `/zh/`, `/ja/`) in the future

### 2. Content Sections (Standard Template)
Each chapter follows this pattern:
```markdown
# 2.X Chapter Title

Brief intro paragraph with context/prerequisites.

## Subsection 1: Topic
- Explanation with **bold** for key terms
- CovScript code examples in ` ```covscript ` blocks
- Emphasis on "why" not just "how"

## Subsection 2: Another Topic
```

Example: See `syntax/01-basic.md` for notes about **variable declarations**, **constants**, and deprecation warnings.

### 3. CovScript-Specific Documentation Patterns

**Version Awareness**: CovScript has two active versions with significant differences:
- **CSC (CovScript 3)**: Stable, released 2018-12, current mainstream version
- **ECS (CovScript 4)**: Newer, began public testing 2022 mid-year, has language improvements

Always document version-specific behavior:
```markdown
# Topic Name

Basic explanation here...

**Note:** This feature differs between versions:
- **CSC (CovScript 3)**: Original behavior/syntax
- **ECS (CovScript 4)**: New behavior/syntax

Example code showing both if significant.
```

Refer to official standards when applicable: `CSC2511XX`, `ECS2106XX`

**Code Examples**: Every explanation should include working code:
```covscript
# Comment explaining what this demonstrates
var x = 10
system.out.println(x)  # Output: 10

# Show variations if relevant
var name = "CovScript"
var message = "Hello, " + name  # String concatenation
```

**Cross-References**: Link to related topics to build conceptual bridges (pattern example only):
```markdown
See also: Related topic documentation for higher-order function patterns
Prerequisites: Basic Syntax, Operators
Related libraries: Network Programming
```

### 4. Ecosystem & Library Documentation

Files in `ecosystem/` follow a consistent pattern to help users and AI agents:

**File Structure Template:**
1. **Introduction** - What problem does this library solve? When to use it?
2. **Installation** - cspkg commands with specific version examples
3. **Quick Start** - Minimal working example (usually 10-20 lines)
4. **API Reference** - Key classes/functions with parameter descriptions
5. **Common Patterns** - Real-world usage scenarios
6. **FAQ Section** - Library-specific troubleshooting
7. **Links** - GitHub repo, related ecosystem libraries, official docs

**Example Structure** (see actual ecosystem files for reference):
- Each library is independent but may reference other ecosystem libraries
- Installation instructions use: `cspkg install <library-name> --latest`
- Code examples show both success cases and error handling
- Performance tips included if library has efficiency concerns

**Note on Library Documentation**: For current details on specific libraries, refer directly to the ecosystem/ directory files rather than this guide, as library APIs and best practices evolve.

## Critical Developer Workflows

### Content Contribution Workflow
1. **Identify gap**: Missing topic or outdated section in `syntax/` or `ecosystem/`
2. **Check structure**: Maintain reading order and cross-references
3. **Update CHANGELOG.md**: Log changes with date and category (e.g., "标准库文档增强")
4. **Verify links**: Ensure relative paths in markdown are correct (`../syntax/`, `./01-basic.md`)
5. **Update README.md navigation**: If adding new sections, update table of contents

### Documentation Quality Checks
- **Consistency**: Check that terminology matches across files (e.g., "CovScript 3 (CSC)" vs "CovScript 4 (ECS)")
- **Links**: Test relative links between sections (especially `../` patterns)
- **Code validity**: CovScript examples should reflect documented language features
- **FAQ coverage**: Common issues should be documented in either topic file or `FAQ.md`

## External Dependencies & Integration Points

**Official GitHub Repos** (referenced in docs):
- `covscript/covscript` - Core interpreter; test cases, official examples live here
- `covscript/ecs` - CovScript 4 compiler with new language features
- `covscript/cspkg` - Package manager (handles installation of all libraries)
- `covscript/covscript-network`, `covscript-imgui`, `covscript-gmssl`, etc. - Extension libraries
- `covscript/covanalysis` - Data analysis framework
- `covscript/covscript-sqlite`, `csdbc` - Database libraries

**Documentation Sources**:
- **Official website**: http://covscript.org.cn/
- **Online manual**: https://manual.covscript.org.cn/ (authoritative reference)
- **VSCode extension**: https://github.com/covscript/covscript-vscode (editor support)
- **Example repository**: https://github.com/covscript/covscript-example (official samples)

**Key Observation for Agents**: When documentation claims a feature exists, validate against:
1. Official manual (https://manual.covscript.org.cn/)
2. Core interpreter GitHub repo (test cases are truth source)
3. Example repository for real-world patterns

## Common Tasks for AI Agents

### Task: Add Documentation for New Library Feature

**Workflow:**
1. Verify library is not already documented in `ecosystem/`
2. Create new file: `ecosystem/NN-libname.md` (use next sequential number)
3. Follow standard template structure:
   - Add installation instructions via cspkg
   - Include working "Hello World" example (10-20 lines)
   - Document key APIs with parameter types
   - Add FAQ section with 3-5 common questions
   - Include link to official GitHub repo
4. Update `ecosystem/README.md` table of contents with new entry
5. Update main `README.md` if it's a major library (check "参考资料" section)
6. Add entry to `CHANGELOG.md` with date and description
7. Verify all relative links work correctly

**Quality Checklist:**
- [ ] Code examples are tested and runnable
- [ ] Version notes included (CSC 3 vs ECS 4 differences, if any)
- [ ] Cross-references to related libraries work
- [ ] Installation commands show specific version numbers
- [ ] Performance considerations mentioned, if relevant
- [ ] Error handling demonstrated in examples

### Task: Improve Existing Documentation

**When to improve:**
- Existing section lacks code examples
- Explanations are unclear or overly technical
- Links are broken or outdated
- Version information is outdated
- FAQ doesn't cover common user questions

**Improvement Process:**
1. Preserve existing structure and bilingual style
2. Add more practical examples if section lacks them
3. Update terminology to match official CovScript documentation
4. Verify all cross-file links still point to correct chapters
5. Update CHANGELOG.md with improvement category and date
6. Ensure examples follow CovScript best practices

**Categories for CHANGELOG:**
- `文档结构增强` - Organizational improvements
- `标准库文档增强` - Standard library additions/fixes
- `生态系统文档增强` - Ecosystem documentation
- `FAQ 更新` - FAQ additions/clarifications
- `示例代码完善` - Better code examples

### Task: Update FAQ

**Before Adding:**
- Search `FAQ.md` for similar questions to avoid duplicates
- Check if answer exists in specific topic files

**Q&A Format Guidelines:**
```markdown
### Q: Clear question phrased from learner's perspective?

**A**: Concise answer (2-4 sentences typically).

Brief code example if helpful:
\`\`\`covscript
# Example demonstrating the answer
\`\`\`

See also: Related topic documentation for deeper learning.
```

**Organization:**
- Group by category: 安装与配置, 语法与特性, 标准库, 生态系统, 性能与优化, 开发工具, 疑难杂症
- Within each category, sort by beginner → advanced difficulty
- Use consistent **Q:** and **A:** prefixes
- Link to detailed documentation for complex topics

## Files to Review First

When starting work on this codebase, prioritize these files:

1. **`README.md`** - Understand project scope, navigation structure, and official resources
2. **`CHANGELOG.md`** - See recent updates and improvement patterns
3. **`syntax/README.md`** - Grasp progressive learning structure (chapters 2.1→2.14)
4. **`ecosystem/README.md`** - Understand library ecosystem organization
5. **`FAQ.md`** - Review common question patterns and categories
6. **Relevant topic file** (e.g., `syntax/05-functions.md` for function docs)

## Language & Terminology Notes

- **Primary language**: Simplified Chinese (中文) for headings, explanations, and commentary
- **Code language**: CovScript with English-language inline comments
- **Technical terms**: Use both Chinese and English where appropriate (e.g., "面向对象编程 (OOP)")
- **Version labels**: Always clarify CSC vs. ECS context
- **Do not translate**: Programming language keywords, GitHub URLs, command-line syntax
- **Future expansion**: As multilingual versions emerge, maintain consistent terminology across languages

---

**Last Updated**: 2025-12-10
**Handbook Version**: Based on CSC2511XX and ECS2106XX standards
