# Engineering Project

> Claude Code configuration for Language AI R&D Engineering projects.

## Quick Facts

- **Stack**: Python, PyTorch, Hugging Face Transformers, Jupyter
- **Python Version**: 3.10+
- **Package Manager**: `uv` (preferred), `pip`, or `conda`
- **Test Command**: `pytest`
- **Lint Command**: `ruff check .`
- **Format Command**: `ruff format .`
- **Type Check Command**: `pyright` or `mypy .`

## Key Directories

- `src/` - Core library code (models, data, training, evaluation)
- `experiments/` - Experiment configs and launch scripts
- `notebooks/` - Jupyter notebooks (name as `01_jd_exploration.ipynb`)
- `scripts/` - Standalone scripts (preprocessing, evaluation, plotting)
- `tests/` - Test files
- `configs/` - YAML/JSON config files (model, training, data)
- `data/` - Local data (gitignored; use DVC for large datasets)
  - `raw/` - Immutable original data (never modify after download)
  - `interim/` - Intermediate transformations (can be regenerated)
  - `processed/` - Final datasets ready for training/evaluation
  - `external/` - Third-party data and references
- `results/` - Experiment outputs, logs, and artifacts (gitignored)

## Git Conventions

- **Branch naming**: `{initials}/{experiment-or-feature}` (e.g., `jd/ablation-context-length`, `jd/fix-tokenizer-oom`)
- **Commit format**: Conventional Commits (`feat:`, `fix:`, `exp:`, `data:`, `docs:`, etc.)
- **PR titles**: Same as commit format
- **Experiment branches**: prefix with `exp/` for throwaway experiment work (e.g., `exp/lr-sweep-0.001-0.1`)

## Coding Behaviour 

### Code Style

- Type hints on all function signatures
- Use `pathlib.Path` over `os.path`
- Prefer dataclasses or Pydantic for config objects over raw dicts
- Use `logging` module, not `print()`, for experiment output
- Use early returns, avoid deep nesting
- Keep functions focused — one function, one responsibility
- Imports: stdlib, then third-party, then local (enforced by ruff isort)

### Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- No em dashes, smart quotes, or decorative Unicode symbols.
- Plain hyphens and straight quotes only.
- Natural language characters (accented letters, CJK, etc.) are fine when the content requires them.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

NOTE: User's instructions should always override this file

### Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" -> "Write tests for invalid inputs, then make them pass"
- "Fix the bug" -> "Write a test that reproduces it, then make it pass"
- "Refactor X" -> "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
3. [Step] -> verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## Critical Rules

### Reproducibility
- ALWAYS set and log random seeds (Python, NumPy, PyTorch, CUDA)
- ALWAYS log the full config used for every experiment run
- NEVER hardcode hyperparameters — use config files or CLI args
- Pin dependency versions in requirements.txt or pyproject.toml
- Record git commit hash in experiment metadata

### Data Handling
- NEVER commit large data files to git — use DVC, symlinks, or cloud storage
- ALWAYS validate data shapes and types at pipeline boundaries
- Log dataset statistics (size, distribution, splits) at the start of training
- Use deterministic data loading (set `worker_init_fn` seeds, `shuffle=False` for eval)

### Experiment Tracking
- Log all hyperparameters, metrics, and artifacts to the experiment tracker (W&B, MLflow, or TensorBoard)
- Use descriptive run names: `{experiment}_{variant}_{date}` (e.g., `ablation_heads_20260623`)
- Save checkpoints with enough metadata to resume training
- Never overwrite previous experiment results
- Log compute resources: GPU type, count, wall-clock time, peak memory

### Error Handling
- NEVER silently catch exceptions during training — let them crash with a full traceback
- Log GPU memory usage and gradient norms to catch instability early
- Validate config values at startup, not mid-training

## Testing

- Write tests for data processing functions and custom model components
- Use `pytest` with fixtures for reproducible test data
- Use `pytest.mark.slow` for GPU-dependent or long-running tests
- Test numerical correctness with `torch.testing.assert_close` (not `==`)
- Run `pytest -x --tb=short` before committing

## Skill Activation

Before implementing ANY task, check if relevant skills apply:

- Designing experiments -> `experiment-design` skill
- Data loading/processing -> `data-pipeline` skill
- Building/modifying models -> `model-development` skill
- Evaluating or benchmarking -> `evaluation-metrics` skill
- Ensuring reproducibility -> `reproducibility` skill
- Debugging training issues -> `systematic-debugging` skill

## Common Commands

```bash
# Environment
uv sync                    # Install dependencies
uv run python ...          # Run within the managed env

# Development
pytest                     # Run tests
pytest -x --tb=short       # Run tests, stop on first failure
ruff check .               # Lint
ruff format .              # Format
pyright                    # Type check

# Experiments
python -m src.train --config configs/base.yaml        # Run training
python -m src.evaluate --checkpoint results/model.pt   # Run evaluation
jupyter lab                                             # Launch notebooks
```
