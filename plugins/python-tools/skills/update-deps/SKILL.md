---
name: update-deps
description: Update Python dependencies to latest stable versions in uv workspaces
context: fork
---

# Update Dependencies

## Workflow

1. **Find outdated**: Run `uv pip list --outdated`
2. **Find pyproject.toml files**: Glob `**/pyproject.toml`, exclude `.venv/`
3. **Update versions**: Edit each file to use latest stable versions from outdated list
4. **Sync and verify**: Run `uv sync`, resolve conflicts if any, run `uv sync` again
5. **Commit**: `chore: update dependencies to latest stable versions`

## Rules

- Only update packages from the `--outdated` output
- Preserve constraint style (`>=1.2.3` stays `>=`)
- Update all workspace members consistently

## Example

Before:
```toml
dependencies = [
    "dagster>=1.12.6",
    "pydantic-ai>=1.32.0",
]
```

After:
```toml
dependencies = [
    "dagster>=1.12.10",
    "pydantic-ai>=1.42.0",
]
```
