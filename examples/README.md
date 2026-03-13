# Examples

This directory contains real-world examples showing the project rules generator in action. Each example demonstrates the full input/output contract: a project description goes in, a complete rules system comes out.

## Example Structure

Every example follows this layout:

```
examples/
└── {example-name}/
    ├── project-description.md    # Input: the project description fed to the generator
    └── generated/                # Output: the complete generated rules system
        ├── .github/
        │   ├── instructions/     # 5 instruction files
        │   └── prompts/          # 4 reusable prompt files
        └── docs/
            ├── logic/            # Feature domain documentation
            ├── task/             # Task tracking (todo, lessons, logs, planning)
            ├── reports/          # Generated reports directory
            ├── server/           # Server & deployment docs
            └── agent-observations/  # Observation logs (critical, recommendations, anomalies)
```

## Available Examples

| Example | Stack | Description |
|---------|-------|-------------|
| [`glitch-payment-gateway`](glitch-payment-gateway/) | PHP · WordPress · WooCommerce | A bespoke payment gateway plugin for a South African payment provider, supporting cards, EFT, digital wallets, and bank transfers. |

## Guidelines for Examples

### What belongs in an example

- **`project-description.md`** — The input that the generator consumes. Should be realistic and descriptive enough to produce meaningful output.
- **`generated/`** — The full output produced by running the generator prompt against the project description. Should include all instruction files, prompts, feature docs, task tracking files, and observation logs.

### Safety rules

- **No secrets or credentials.** API keys, passwords, tokens, and webhook secrets must be placeholder values or omitted entirely.
- **No private client information.** Company names, real merchant IDs, internal URLs, or proprietary business logic must be fictional or anonymized.
- **No environment-specific paths.** Absolute filesystem paths should be replaced with relative paths or generic placeholders.
- **Realistic but safe.** Examples should feel like real projects without exposing anything sensitive.

### Adding a new example

1. Create a new directory under `examples/` with a descriptive kebab-case name.
2. Add `project-description.md` with a thorough project description.
3. Run the generator prompt against the description and capture the output.
4. Place all generated files under `generated/`.
5. Review for secrets, credentials, and private information — remove all of them.
6. Add a row to the "Available Examples" table in this README.
