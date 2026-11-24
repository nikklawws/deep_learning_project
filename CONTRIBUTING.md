# Development and Deep Learning Guidelines

There's Layers to This — Team working agreement

## 1. Development workflow

### GitHub Projects and sprints

We work in four one-week sprints. Use the Product Backlog to track outstanding work, the Sprint Board to manage the current sprint and Sprint History to review previous work.

- Create a GitHub issue using the appropriate issue form (Development Task, ML Experiment, Bug Report, or Research and Documentation).
- Describe the task and its acceptance criteria. Assign an owner, priority, estimate and sprint during planning. Leave the sprint unassigned if the task has not been scheduled.
- Use the relevant `type:` and `area:` labels. Use `blocked` when work cannot continue and explain the blocker in the issue.
- Keep the issue status up to date: `Todo` → `In Progress` → `In Review` → `Done`. Return the issue to `In Progress` if changes are requested.
- Use P0 (Critical), P1 (High), P2 (Medium) and P3 (Low) to prioritise work. Estimate effort with story points (1, 2, 3, 5, 8); split larger tasks where possible. Story points are not hours or individual performance measures.

### Branch protection

`main` is protected. The following rules apply to everyone:

- No direct pushes or force pushes to `main`. Submit changes through a pull request.
- Each PR requires at least one independent approval. You cannot approve your own PR, and the latest reviewable push must be approved by someone other than its pusher.
- New reviewable commits dismiss previous approvals. Resolve review conversations before merging.
- Use **Create a merge commit**. Do not squash or rebase merge; we retain the original commits, authorship and branch history.
- Do not bypass or disable the protection rules.

### Branches and commits

Create one branch per issue using `<type>/<issue-number>-<short-description>`:

| Type | Example |
| --- | --- |
| Feature | `feature/12-training-pipeline` |
| Bug fix | `fix/15-data-leakage` |
| Experiment | `experiment/18-learning-rate` |
| Refactor | `refactor/21-data-loader` |
| Documentation | `docs/27-setup-instructions` |
| Tests (if applicable) | `test/24-preprocessing` |

Start from an up-to-date `main`:

```bash
git switch main
git pull --ff-only origin main
git switch -c feature/12-training-pipeline
```

Make focused commits using `<type>: <short description>`. For example:

```text
feat: implement baseline classifier
fix: fit preprocessing on training data only
experiment: compare dropout configurations
refactor: separate training and evaluation
docs: record baseline evaluation results
```

Commit meaningful stages of your work rather than committing everything at the end. Avoid messages such as `update` or `final fix`.

```bash
git add src/train.py
git commit -m "feat: implement training loop"
git push -u origin feature/12-training-pipeline
```

If you need to update your branch, fetch and merge `origin/main` into it. Resolve conflicts on your own branch. Do not rewrite shared branch history without coordinating with the team.

Once the PR is merged, check the associated issue's status and delete the feature branch. Its merged commits remain in the history of `main`.

## 2. Pull requests and code review

Use the repository's pull request template. Keep each PR focused on one issue or coherent objective. Open a draft PR if the work is not ready for review.

Link the issue using `Closes #12` if merging completes it, or `Related to #12` if further work remains. Move the issue to `In Review` when requesting formal review.

### Before requesting review (author)

- Review your own diff and run the changed code. Record what you ran and the results in the PR.
- Include the commands or notebook instructions the reviewer needs, together with dependencies, data access and any required configuration or checkpoint.
- If the full run is expensive, provide a reduced configuration that exercises the same changed code. The reviewer should not have to figure out how to shorten your experiment.
- For experiments, link the experiment record, full-run logs, results and configuration. State known limitations.
- Complete the author checklist in the PR template. Leave the merge requirements unchecked until they have actually been met.

### Reviewer guidelines

Do not approve a PR based on the author's description or saved notebook outputs alone. You are responsible for checking the implementation independently.

1. Read the issue, acceptance criteria and PR description.
2. Inspect the diff. Check the logic, readability, unintended changes, secrets and large files. For ML code, check data leakage, dataset splits, evaluation methodology and reproducibility where relevant.
3. Check out the latest PR revision and run the affected functionality yourself. For a notebook, restart the kernel and run the relevant cells in order, including setup. For documentation-only changes, inspect the content and verify changed instructions where applicable.
4. Check the outputs against the acceptance criteria. For experiments, also inspect the author's full-run evidence and whether it supports their conclusions.
5. Leave a review comment recording what you ran, what happened and any limitations. Approve only once the required checks pass. Request changes when something needs correcting; distinguish required fixes from suggestions.

To check out a PR locally, use the branch name shown on GitHub (start with a clean working tree):

```bash
git fetch origin
git switch --detach origin/feature/12-training-pipeline
git rev-parse --short HEAD
```

The detached checkout lets you review the exact fetched revision without modifying the author's branch. Return to your own branch when finished (`git switch main`, for example). If the author pushes new changes, fetch again, check out the updated revision, review the new diff and repeat affected checks before approving.

### Manual validation (smoke checks)

We use manual smoke checks for routine reviews. Writing automated unit tests or maintaining a test suite is not required. Run enough of the code to exercise the actual changes, aiming for 5–10 minutes of execution where feasible (excluding environment and data setup).

| Change | What to run and check |
| --- | --- |
| Data loading or preprocessing | Process a small representative sample. Inspect records, labels, shapes, missing values and splits as relevant. |
| Model architecture | Instantiate the model and run a forward pass. Verify output shapes and finite values. Run a training step if training behaviour changes. |
| Training pipeline | Run a few batches or a short epoch. Check that the loss is finite, backpropagation works and trainable parameters update. |
| Evaluation or metrics | Evaluate a small sample, known predictions or an existing checkpoint. Check the metric calculation and evaluation split. |
| ML experiment | Run the changed workflow with a reduced configuration. Inspect the full-run logs, results and conclusions separately. |
| Bug fix | Reproduce the original issue where possible, then verify the fix and check nearby functionality for regressions. |
| Notebook | Restart the kernel and run the affected workflow in order, with a reduced configuration if needed. |
| Documentation or configuration | Check accuracy, file paths, commands and affected settings. Purely textual changes do not require code execution. |

#### Example: reviewing a training notebook

Suppose the author's full training run takes four hours. The reviewer can run the same pipeline with the following reduced configuration:

| Parameter | Full experiment | Reviewer smoke check |
| --- | ---: | ---: |
| Training samples | 50,000 | 256 |
| Epochs | 30 | 1 |
| Batch size | 32 | 32 |
| Hardware | GPU | CPU or GPU |
| Objective | Scientific evaluation | Functional validation |

The author can expose these settings near the beginning of the notebook, for example:

```python
SMOKE_CHECK = True
NUM_SAMPLES = 256 if SMOKE_CHECK else 50_000
NUM_EPOCHS = 1 if SMOKE_CHECK else 30
BATCH_SIZE = 32
```

Use a reproducible subset of the training data and wire these settings into the actual data loader and training loop. Use a validation subset for evaluation, not the held-out test set. Choose sample sizes and hardware that are practical for the model; the numbers above are an example, not a fixed requirement.

The reviewer checks that data loading, the forward pass, loss calculation, backpropagation and parameter updates work as intended. The reduced run does not need to achieve the same metrics as the full experiment. It verifies that the implementation works on a small scale, not that the author's final scientific results have been independently reproduced.

Full retraining is not a routine review requirement. Independently reproduce key final results where compute permits, or arrange a deeper check if the reduced run cannot validate a critical change.

If data, dependencies or hardware are unavailable, explain the blocker in the PR. Ask the author for a runnable reduced configuration or request a reviewer with access. Do not approve an executable change you could not verify. Any unavoidable exception requires team agreement, documented alternative evidence and a clear statement of what remains unverified.

### Review comment

Use the following format when submitting your review. Keep it short but specific.

```markdown
**Validation performed:**
- Revision/commit:
- Command, notebook or cells executed:
- Configuration or sample used:
- Observed result:

**Findings / limitations:**
- Required changes or optional suggestions:
- Checks not performed and why:

**Decision:** Approve / Request changes / Comment (review blocked)
```

Approval does not replace the author's own validation. Address feedback before resolving the review conversations.

## 3. Code and repository standards

### Python and notebooks

- Follow PEP 8. Use descriptive English names, `snake_case` for functions and variables, and `PascalCase` for classes.
- Keep data loading, preprocessing, model definitions, training and evaluation separate where practical. Move reusable notebook code into Python modules.
- Document non-trivial functions, tensor shapes and assumptions where they are not obvious. Use type hints where useful.
- Use relative paths and configurable parameters rather than machine-specific paths, credentials or hardcoded experiment settings.
- Remove unnecessary notebook outputs before committing. Shared notebooks must run in order from a fresh kernel. Preserve evidence for reported experiments separately.
- Avoid unrelated refactoring and unnecessary duplication.

### Repository layout

Use the following layout as needed; do not create empty directories just to match it.

```text
project/
├── src/                  # Reusable Python code
│   ├── data.py
│   ├── model.py
│   ├── train.py
│   └── evaluate.py
├── configs/              # Experiment configurations
├── notebooks/            # Exploration and analysis
├── experiments/          # Experiment records and results
├── reports/              # Figures and academic findings
├── .github/              # Issue forms and PR template
├── README.md
├── CONTRIBUTING.md
├── requirements.txt
└── .gitignore
```

Keep `README.md` current with environment setup, dataset access and the commands for training and evaluation. Use a shared Python version and dependency specification (e.g. `requirements.txt`, `pyproject.toml` or `environment.yml`). Record dependency versions, hardware and CUDA requirements for reported experiments. Include dependency changes in the corresponding PR.

### Files and secrets

Keep virtual environments, caches, credentials, raw or restricted data, model checkpoints and large generated files out of Git. Use shared external storage for data and models; version lightweight configurations and result summaries in Git.

Include the relevant patterns in `.gitignore`:

```gitignore
.venv/
venv/
__pycache__/
*.py[cod]
.pytest_cache/
.ipynb_checkpoints/
.env
.env.*
!.env.example
/data/raw/
/data/processed/
/checkpoints/
/outputs/
/runs/
*.pt
*.pth
*.ckpt
*.safetensors
```

Ignoring a file does not remove it from existing Git history. If credentials are committed, notify the team and revoke or rotate them immediately.

### Validation and automation

Manual validation is required before requesting review, and independent smoke checks are required for executable changes. Automated test writing is optional. There is no mandatory coverage threshold or requirement to implement a test suite. Do not enable required GitHub status checks until the corresponding workflows exist and run reliably.

## 4. Deep learning standards

### Data and evaluation protocol

Record the agreed dataset source, version, access instructions, licence, inclusion criteria, preprocessing, data splits, baseline and primary evaluation metric in the README or methodology document before comparing models.

- Use the same reproducible train/validation/test splits across approaches. Record the seed and split identifiers or manifest where permitted.
- Check for duplicates or related observations across splits; use group-aware or temporal splitting where appropriate.
- Fit data-dependent preprocessing on training data only. Apply the fitted transformations to validation and test data. Apply training augmentation only to the training partition.
- Use validation data (or the agreed cross-validation procedure) for hyperparameters, early stopping, model selection and threshold selection. Reserve test data for final evaluation.
- Discuss and document changes to shared data, preprocessing, splits or metrics before implementing them. Do not compare incompatible results as if they were obtained under the same conditions.
- Do not commit restricted or personally identifiable data.

### Baselines and model development

Establish a working baseline before optimising more complex architectures. Record the motivation or hypothesis for each substantive model change. Isolate changes where practical, and compare against the baseline using the same evaluation protocol.

For the paper replication, our own modification and the frontier/local model comparison, keep evaluation conditions comparable and document differences in data access, model configuration, tuning, and computational budget. Do not describe unequal evaluation setups as a fair direct comparison without explaining the limitations.

### Experiment tracking

Record every substantive experiment, including unsuccessful or interrupted runs, in the shared experiment register (Markdown or CSV). Use a unique experiment ID. Small debugging runs only need recording if they inform a decision or are reported as evidence.

```markdown
### EXP-001 — Baseline model
- Owner / date:
- Related GitHub issue:
- Git commit / configuration path:
- Objective / hypothesis:
- Dataset version / split ID:
- Model architecture / pretrained checkpoint:
- Preprocessing / augmentation:
- Random seed(s):
- Hyperparameters (optimiser, learning rate, batch size, epochs, etc.):
- Checkpoint selection / early stopping / schedule:
- Environment / hardware:
- Training duration / computational budget:
- Validation results (primary and secondary metrics):
- Checkpoint / log location (if retained):
- Conclusion / limitations / next step:
```

Run reported experiments from an identifiable committed revision. If that is not possible, record any uncommitted changes needed to reproduce the run. Do not report interrupted runs as completed experiments.

### Reporting and reproducibility

- Report the agreed primary and relevant secondary metrics. For classification, account for class balance when selecting metrics; accuracy alone may be misleading.
- Use the same metric implementation and evaluation data for model comparisons. Report differences in training or tuning budgets.
- Where compute permits, run important comparisons with multiple seeds and report variability (e.g. mean and standard deviation). If only one run is feasible, say so.
- Inspect representative errors and document failure modes, negative results and limitations. Report relevant efficiency measures with their measurement conditions.
- Record random seeds, configuration, dependency versions and execution instructions. Note known nondeterminism; identical seeds do not guarantee identical outputs across environments.
- Keep the final model checkpoint where licensing and storage permit. Record its experiment ID, location and selection criteria. Do not commit large weights to ordinary Git history.
- Document how to reproduce final training and evaluation, including the dataset, code revision, environment, model configuration and commands.

## 5. Definition of Done

An implementation issue can move to `Done` when all applicable items are complete:

- [ ] The issue's acceptance criteria are met.
- [ ] Code follows the repository conventions.
- [ ] Author validation and independent reviewer smoke checks are documented.
- [ ] No credentials, restricted data or unnecessary large files are committed.
- [ ] Documentation, dependencies and configuration are up to date.
- [ ] Experiment configurations, results and limitations are recorded where applicable.
- [ ] The PR is linked to the issue, independently approved and all conversations are resolved.
- [ ] Any configured required checks pass.
- [ ] The PR is merged into `main` using a merge commit and the issue status is updated.

For research or planning tasks without code changes, link the agreed deliverable to the issue and have it reviewed before marking the task `Done`.

## 6. Team communication

Attend sprint planning, provide short asynchronous progress updates on working days and participate in the weekly review and retrospective. Keep the board current. If you are blocked or expect to miss a deadline, inform the team early.

Discuss consequential changes to shared interfaces, methodology or project conventions before making them. Record decisions in the relevant issue or PR. Changes to this agreement also go through a reviewed PR.
