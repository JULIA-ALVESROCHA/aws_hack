# Markdown Documentation Consolidation Prompt

Act as a Markdown documentation specialist. Your task is to consolidate all attached `.md` files into a single Markdown document, following this exact section order:

1. Design  
2. Project  
3. History and Memory  
4. Personalization  
5. Guardrails  

## Requirements

- Preserve the original content as much as possible.
- Do not rewrite, summarize, simplify, translate, or change the technical meaning of the content.
- Preserve existing headings, lists, tables, code blocks, JSON examples, links, and formatting.
- You may adjust heading levels only when necessary to create a consistent hierarchy in the final document.
- Add a clear top-level heading for each of the five sections.
- Place the complete content of each source file under its corresponding section.
- Do not remove duplicated or conflicting content without my approval.
- Do not merge definitions in a way that changes their original meaning.
- Do not add new technical information that is not present in the source files.

## Conflicts and Duplicated Content

Before producing the final document, identify any:

- Exact duplicates
- Partially duplicated definitions
- Conflicting requirements
- Conflicting architectural decisions
- Inconsistent terminology
- Different definitions for the same concept
- Sections that appear to overlap but have different implementations

For each issue, report:

1. The conflicting or duplicated content
2. The source file
3. The section or heading where it appears
4. The corresponding content in the other file
5. Why it may be considered a conflict or duplication
6. A suggested resolution, without applying it automatically

## Expected Output

Provide two deliverables:

1. **Conflict and duplication report**  
   A structured report listing all identified issues and their exact locations.

2. **Consolidated Markdown document**  
   A single `.md` file containing all source content in the required order.

When a conflict or duplication exists, preserve both versions in the consolidated document until I decide which one should remain.
