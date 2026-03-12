# Claude Skills

> A growing collection of custom skills for Claude Code that enhance its capabilities across development, security, and data domains.

---

## Tech Stack

![Claude Code](https://img.shields.io/badge/Claude_Code-CC785C?style=for-the-badge&logo=anthropic&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Ruff](https://img.shields.io/badge/Ruff-D7FF64?style=for-the-badge&logo=ruff&logoColor=black)
![Pytest](https://img.shields.io/badge/Pytest-0A9EDC?style=for-the-badge&logo=pytest&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![Markdown](https://img.shields.io/badge/Markdown-000000?style=for-the-badge&logo=markdown&logoColor=white)

---

## What are Claude Skills?

Claude Skills are custom instruction sets that guide Claude Code to perform specialized tasks with expert-level precision. Each skill is defined in a markdown file with specific triggers, workflows, and domain knowledge.

---

## Skills Catalog

### DevSkills

| Skill | Description |
|-------|-------------|
| [CyberSec](DevSkills/CyberSec.md) | Security vulnerability scanner — detects secrets, CVEs, SAST issues, and misconfigurations |
| [Lint](DevSkills/lint.md) | Lint and format Python projects using Ruff — checks for issues and optionally auto-fixes them |
| [Test](DevSkills/test.md) | Run Python tests with pytest, with optional coverage reporting and test structure scaffolding |

### DataSkills

| Skill | Description |
|-------|-------------|
| [Data Generator](DataSkills/datagenerator.md) | Generate synthetic datasets from a natural language prompt — supports CSV, JSON, Parquet, Excel, and SQL formats |

---

## Repository Structure

```
ClaudeSkills/
├── README.md
├── claude.md                  # CLAUDE.md template for new projects
├── DevSkills/
│   ├── CyberSec.md
│   ├── lint.md
│   └── test.md
├── DataSkills/
│   └── datagenerator.md
└── templates/
    ├── claude.md              # Prompt for generating CLAUDE.md on existing projects
    └── firstPrompt.md         # Example first prompt templates
```

---

## Usage

Copy skill markdown files into your project's `.claude/skills/` directory, or reference them globally via `~/.claude/`.

Trigger phrases vary by skill — see each skill file for details.

---

## Contributing

More skills will be added over time across development, DevOps, data analysis, and more.

## License

Feel free to use and adapt these skills for your own workflows.
