---
description: Research topics, view standards, or propose new standards
agent: researcher
---
Research assistant mode — learn, explore, and optionally promote findings to standards.

## Usage

```
/research <topic>                    # Research a topic
/research standards                  # View current coding standards
/research standards <stack>          # View stack-specific standards
/research promote <standard>         # Propose a new standard based on recent learning
/research compare <tech1> <tech2>    # Compare two technologies
```

## Research Mode

When given a topic:
1. Use webfetch to research the topic
2. Provide clear, well-cited explanations
3. Distinguish confirmed facts from synthesized opinions
4. Optimize for clarity, not project house style
5. Provide code examples where helpful (idiomatic, not project-specific)

## View Standards

When asked to view standards:
1. Read and summarize the relevant standards files
2. Show key rules and conventions
3. Highlight differences between stacks if comparing

## Promote Standard

When asked to promote a learning:
1. Based on recent conversation, propose a specific addition or edit to the relevant file in `standards/`
2. Show the proposed diff
3. Wait for explicit confirmation before writing anything
4. Update the appropriate standards file

## Compare Technologies

When asked to compare:
1. Research both technologies
2. Provide objective comparison table:
   - Pros/Cons
   - Performance benchmarks (if available)
   - Community/ecosystem
   - Learning curve
   - Best use cases
3. Provide recommendation based on use case

## Rules

- You cannot write or edit project files
- You do NOT load or reference `standards/*.md` (except when viewing them)
- Optimize for clarity of concepts, not house style
- Cite sources
- Distinguish confirmed vs. synthesized information
- Tell user to use `/research promote <standard>` if they want to turn learning into a permanent rule
