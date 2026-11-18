# Software Requirements Specification to Maude Modeling

Prompts and artifacts for an LLM pipeline that generates executable Maude models from natural-language requirements.

<br>

## Overview

An LLM pipeline that turns natural-language requirements into an executable formal model and checks it against test cases.

- **Model generation:** SRS → IR (static / dynamic) → Maude object-oriented model
- **Test generation:** high-level test cases (HL TC) → mid-level test cases (ML TC) → Maude initial states and search commands
- **Testing:** the Maude model is executed to decide whether each test case passes
- Each stage is followed by reverse-translation and judgment steps. On failure, the pipeline goes back and regenerates the earlier stage.
- Generation steps run on GPT (Codex); judgment steps run on Claude.

<br>

## Repository Structure

```
prompt/
├── .agents/skills/    # generation steps (GPT / Codex)
└── .claude/skills/    # judgment steps (Claude)
```

<table>
  <tr>
    <th>Stage</th>
    <th>Generation (<code>.agents/skills</code>)</th>
    <th>Judgment (<code>.claude/skills</code>)</th>
  </tr>
  <tr>
    <td>SRS preprocessing</td>
    <td><code>srs-numbering</code></td>
    <td></td>
  </tr>
  <tr>
    <td>IR</td>
    <td><code>static-ir</code>, <code>dynamic-ir</code>, <code>ir-report-writing</code>, <code>ir-reverse</code></td>
    <td><code>ir-report-check</code>, <code>ir-verdict</code>, <code>ir-verdict-a/b/c</code></td>
  </tr>
  <tr>
    <td>ML TC</td>
    <td><code>ml-tc</code>, <code>ml-tc-report-writing</code>, <code>ml-tc-reverse</code></td>
    <td><code>ml-tc-report-check</code>, <code>ml-tc-verdict</code>, <code>ml-tc-verdict-a/b/c</code></td>
  </tr>
  <tr>
    <td>Maude</td>
    <td><code>maude-model</code>, <code>initial-state</code>, <code>maude-property</code>, <code>maude-testing</code>, <code>maude-reference</code></td>
    <td><code>maude-verdict</code></td>
  </tr>
  <tr>
    <td>Output rules (all steps)</td>
    <td colspan="2" align="center"><code>output-format</code></td>
  </tr>
</table>

<br>

## Requirements

- [Maude](https://maude.cs.illinois.edu/)
- [Codex CLI](https://developers.openai.com/codex)
- [Claude Code](https://claude.com/claude-code)
