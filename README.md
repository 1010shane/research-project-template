# Reproducible Research Project Management

A lightweight project-management architecture for reproducible data
analysis and research on macOS using **Git/GitHub**, **VS Code**, **uv**
for Python, and **renv** for R.

The central principle is simple:

> **Keep the global system environment minimal. Keep project
> dependencies local, declarative, version-controlled, and
> reproducible.**

This approach provides most of the reproducibility benefits of
containerized development while keeping day-to-day development native
and straightforward.

------------------------------------------------------------------------

## Table of Contents

-   [Architecture](#architecture)
-   [Global Tools vs. Project
    Dependencies](#global-tools-vs-project-dependencies)
-   [General Project Principles](#general-project-principles)
-   [Python Projects with uv](#python-projects-with-uv)
    -   [Create a Python Project](#create-a-python-project)
    -   [Install Packages](#install-python-packages)
    -   [Run Commands](#run-python-commands)
    -   [Remove Packages](#remove-python-packages)
    -   [Recreate the Environment](#recreate-a-python-environment)
-   [R Projects with renv](#r-projects-with-renv)
    -   [Create an R Project](#create-an-r-project)
    -   [Install Packages](#install-r-packages)
    -   [Record the Environment](#record-the-r-environment)
    -   [Restore the Environment](#restore-the-r-environment)
-   [VS Code and Jupyter](#vs-code-and-jupyter)
-   [Git and GitHub](#git-and-github)
-   [Recreating a Project on Another
    Machine](#recreating-a-project-on-another-machine)
-   [Updating Dependencies](#updating-dependencies)
-   [Data and Secrets](#data-and-secrets)
-   [Suggested Project Structures](#suggested-project-structures)
-   [Recovery and Troubleshooting](#recovery-and-troubleshooting)
-   [Command Reference](#command-reference)
-   [Reproducibility Checklist](#reproducibility-checklist)

------------------------------------------------------------------------

## Architecture

The development environment is divided into two layers.

### System layer

Install only tools that are useful across many projects:

``` text
macOS
├── Homebrew
├── Git
├── VS Code
├── Python
├── R
├── uv
└── Apple Command Line Tools
```

These provide runtimes, compilers, editors, and package/environment
managers.

### Project layer

Each project manages its own dependencies:

``` text
Projects/
├── python-project/
│   ├── pyproject.toml
│   ├── uv.lock
│   ├── .python-version
│   └── .venv/              # local, disposable, not committed
│
└── r-project/
    ├── .Rprofile
    ├── renv.lock
    └── renv/
        └── library/        # local/disposable package environment
```

Git records the project files and dependency specifications. GitHub
stores the repository remotely.

The environment itself should generally be treated as **reconstructable
rather than precious**.

------------------------------------------------------------------------

## Using This Repository as a Project Template

This repository can serve as a **language-neutral research project
template**. The template should contain stable directory structure,
documentation, and generic Git configuration, while each real project
generates its own language-specific environment metadata.

A recommended template structure is:

``` text
research-project-template/
├── README.md
├── .gitignore
├── data/
│   ├── raw/.gitkeep
│   ├── processed/.gitkeep
│   └── external/.gitkeep
├── notebooks/.gitkeep
├── scripts/.gitkeep
└── outputs/
    ├── figures/.gitkeep
    ├── tables/.gitkeep
    └── reports/.gitkeep
```

Git does not track empty directories, so `.gitkeep` placeholder files
preserve the directory structure. `.gitkeep` is only a convention and
has no special behavior in Git.

### What belongs in the template

Keep generic, reusable structure in the template: `README.md`,
`.gitignore`, the standard directories above, and their `.gitkeep`
placeholders. Optional directories such as `tests/` and `docs/` can be
added when a project needs them.

Do **not** populate the template with empty or placeholder
dependency-management files. Generate these for each real project
instead.

For Python, `uv` should generate `pyproject.toml`, `uv.lock`,
`.python-version`, and the local `.venv/`. In particular, do not add an
empty `pyproject.toml` to the template.

For R, `renv` should generate `.Rprofile`, `renv.lock`, and `renv/`.

Lockfiles belong in the individual projects where they were generated.
Installed environments such as `.venv/` and `renv/library/` should not
be committed.

### Create a new project from the GitHub template

After this repository has been pushed to GitHub, enable **Template
repository** under the repository's GitHub settings.

To create a new project with the GitHub CLI, start in the directory
where projects are stored:

``` bash
cd ~/Projects
```

Then create a new private GitHub repository from the template and clone
that new repository locally:

``` bash
gh repo create PROJECT-NAME \
  --template YOUR-GITHUB-USERNAME/research-project-template \
  --private \
  --clone
```

Replace `PROJECT-NAME` with the new project's name and
`YOUR-GITHUB-USERNAME` with the GitHub account that owns the template.
Use `--public` instead of `--private` when appropriate.

This command:

1.  Creates a new GitHub repository from the template.
2.  Creates it as an independent project rather than a continuation of
    the template's Git history.
3.  Clones the new repository to the current machine.

Then enter the project and open it:

``` bash
cd PROJECT-NAME
code .
```

Do **not** use `git pull` to instantiate a new project from the
template. `git pull` updates an existing local repository from its
configured remote. Likewise, normally do not clone the template
repository itself when starting a new research project.

The intended workflow is:

``` text
GitHub template repository
          |
          |  gh repo create --template ... --clone
          v
New independent GitHub repository
          |
          v
New local project directory
          |
          +-- initialize uv for Python
          +-- initialize renv for R
          +-- initialize both for mixed projects
```

### Initialize a Python project created from the template

``` bash
uv init
uv add --dev jupyter ipykernel
mkdir -p src
```

Add actual project dependencies as required:

``` bash
uv add numpy pandas scipy matplotlib
```

Commit `pyproject.toml`, `uv.lock`, and `.python-version`. Do not commit
`.venv/`.

### Initialize an R project created from the template

``` bash
Rscript -e 'renv::init(bare = TRUE)'
mkdir -p R
```

Install dependencies as needed, then record the environment:

``` bash
Rscript -e 'renv::install("dplyr")'
Rscript -e 'renv::snapshot()'
Rscript -e 'renv::status()'
```

Commit `renv.lock` and the `renv` activation/configuration files, but
not `renv/library/`.

### Initialize a mixed R/Python project

Initialize both environment managers:

``` bash
uv init
uv add --dev jupyter ipykernel
Rscript -e 'renv::init(bare = TRUE)'
mkdir -p src R
```

`uv` manages Python while `renv` independently manages R. Commit both
sets of project-specific dependency metadata and lockfiles.

The template provides the **project skeleton**; each real repository
generates and records its own **environment specification**.

## Global Tools vs. Project Dependencies

### Appropriate global tools

Examples include:

-   Git
-   VS Code
-   Homebrew
-   Python runtime
-   R runtime
-   `uv`
-   `renv` bootstrap package
-   system compilers and development tools

### Avoid globally installing research dependencies

For example, avoid maintaining global collections of:

``` text
numpy
pandas
scipy
jupyter
dplyr
tidyverse
Seurat
Bioconductor packages
project-specific research libraries
```

Instead, install them into the individual project that requires them.

This reduces dependency conflicts and makes it possible to reproduce an
old analysis even after the global system changes.

------------------------------------------------------------------------

## General Project Principles

A reproducible project should contain:

1.  **Source code** --- scripts, notebooks, modules, reports, etc.
2.  **Dependency declarations** --- what packages the project requires.
3.  **A lockfile** --- the exact dependency versions used.
4.  **Documentation** --- enough information to reproduce the analysis.
5.  **Version control** --- Git history describing how the project
    changed.
6.  **No secrets or sensitive data in Git.**

Do not rely on remembering which packages were installed manually.

Do not commit an entire virtual environment simply to preserve it.

Instead, preserve the instructions needed to recreate it.

------------------------------------------------------------------------

# Python Projects with uv

`uv` manages Python versions, virtual environments, project
dependencies, and lockfiles.

A useful mental model is:

``` text
current directory
      ↓
uv finds the project configuration
      ↓
uv identifies the project's Python/environment
      ↓
command runs in that environment
```

For project-aware commands, `uv` searches the current directory and its
parent directories for the project.

This means manual virtual-environment activation is usually unnecessary.

## Create a Python Project

Create and enter a directory:

``` bash
mkdir my-project
cd my-project
```

Initialize it:

``` bash
uv init
```

Typical generated files include:

``` text
my-project/
├── .git/
├── .gitignore
├── .python-version
├── pyproject.toml
├── README.md
└── src/
```

A `.venv` is created when needed by commands such as `uv sync`,
`uv run`, or dependency installation.

### Important files

#### `pyproject.toml`

Describes the project and its declared dependencies.

Think of it as answering:

> What does this project require?

#### `uv.lock`

Records the fully resolved dependency graph and exact package versions.

Think of it as answering:

> What exact environment should be reproduced?

#### `.python-version`

Records the Python version expected by the project.

#### `.venv/`

Contains the installed Python environment.

It is **disposable** and normally should **not be committed to Git**.

------------------------------------------------------------------------

## Install Python Packages

Install a normal project dependency:

``` bash
uv add pandas
```

Install several:

``` bash
uv add numpy pandas scipy matplotlib
```

Install development-only tools:

``` bash
uv add --dev jupyter ipykernel
```

`uv add` is preferred over manually running `pip install` because it
records the dependency as part of the project and updates the lock
state.

### Rule of thumb

Use:

``` bash
uv add PACKAGE
```

rather than:

``` bash
pip install PACKAGE
```

for normal project dependency management.

------------------------------------------------------------------------

## Run Python Commands

Manual activation is generally unnecessary.

Run Python:

``` bash
uv run python
```

Run a script:

``` bash
uv run python analysis.py
```

Run Jupyter:

``` bash
uv run jupyter lab
```

Check the interpreter:

``` bash
uv run python -c "import sys; print(sys.executable)"
```

Because `uv` discovers the project from the current directory, changing
into another uv-managed project and using `uv run` automatically targets
that project's environment.

------------------------------------------------------------------------

## Remove Python Packages

Use:

``` bash
uv remove PACKAGE
```

For example:

``` bash
uv remove pandas
```

This keeps project metadata and the lockfile synchronized.

------------------------------------------------------------------------

## Recreate a Python Environment

The `.venv` should be considered disposable.

To prove that a project is reproducible:

``` bash
rm -rf .venv
uv sync
```

`uv` reconstructs the environment from the project's configuration and
lockfile.

Then verify:

``` bash
uv run python --version
```

and, if applicable:

``` bash
uv run jupyter --version
```

### What should be committed?

Commit:

``` text
pyproject.toml
uv.lock
.python-version
source code
notebooks
documentation
```

Do not commit:

``` text
.venv/
```

------------------------------------------------------------------------

# R Projects with renv

`renv` provides isolated R package libraries and reproducible lockfiles.

Its workflow differs slightly from `uv`.

When R starts inside an `renv` project, the project's `.Rprofile` loads
`renv/activate.R`, which activates the project-specific library
automatically.

Conceptually:

``` text
enter project directory
        ↓
start R
        ↓
.Rprofile executes
        ↓
renv activates
        ↓
project library becomes active
```

No shell-level `source .../activate` command is normally required.

------------------------------------------------------------------------

## Create an R Project

Create and enter the project:

``` bash
mkdir my-r-project
cd my-r-project
```

Initialize `renv`:

``` bash
Rscript -e 'renv::init(bare = TRUE)'
```

For a new empty project, create the initial lockfile:

``` bash
Rscript -e 'renv::snapshot()'
```

A typical project will contain:

``` text
my-r-project/
├── .Rprofile
├── renv.lock
└── renv/
    ├── activate.R
    └── library/
```

Check that the project library is active:

``` bash
Rscript -e 'print(.libPaths())'
```

The project's `renv` library should appear first.

------------------------------------------------------------------------

## Install R Packages

Install a package into the project environment:

``` r
renv::install("dplyr")
```

or from the shell:

``` bash
Rscript -e 'renv::install("dplyr")'
```

Verify:

``` bash
Rscript -e 'library(dplyr); packageVersion("dplyr")'
```

### Important difference from uv

Installing an R package does **not necessarily mean it will
automatically be retained in the lockfile**.

By default, `renv` can inspect project source files to determine which
packages the project actually uses.

For example:

``` r
library(dplyr)
```

or:

``` r
dplyr::filter(...)
```

provides evidence that the project depends on `dplyr`.

Inspect detected dependencies with:

``` bash
Rscript -e 'renv::dependencies()'
```

------------------------------------------------------------------------

## Record the R Environment

After installing or changing dependencies:

``` bash
Rscript -e 'renv::snapshot()'
```

This updates:

``` text
renv.lock
```

Check consistency:

``` bash
Rscript -e 'renv::status()'
```

A healthy project should report:

``` text
No issues found -- the project is in a consistent state.
```

### Mental model

``` text
renv::install()
      ↓
changes installed project packages

renv::snapshot()
      ↓
records desired package state in renv.lock
```

The distinction matters.

------------------------------------------------------------------------

## Restore the R Environment

The project package library should be reconstructable.

To test this deliberately:

``` bash
rm -rf renv/library
```

Then restore:

``` bash
Rscript -e 'renv::restore()'
```

`renv` reads `renv.lock` and reconstructs the required package
environment.

Verify:

``` bash
Rscript -e 'renv::status()'
```

### What should be committed?

Commit the files needed to activate and restore `renv`, especially:

``` text
.Rprofile
renv.lock
renv/activate.R
renv/settings.json
source code
documentation
```

Do **not** commit the project package library.

A standard `renv`-generated `.gitignore` should be respected.

------------------------------------------------------------------------

# VS Code and Jupyter

Open a project from its root directory:

``` bash
code .
```

This makes the project boundary explicit and helps VS Code discover the
appropriate files and environment.

## Python notebooks

For Python notebooks, select the interpreter associated with the
project's `.venv`.

Verify from a notebook:

``` python
import sys

print(sys.executable)
print(sys.version)
```

The executable should point into:

``` text
PROJECT/.venv/bin/python
```

Install Jupyter tooling as a development dependency when appropriate:

``` bash
uv add --dev jupyter ipykernel
```

This avoids relying on a globally installed Jupyter environment.

## R

Open VS Code from the R project root so that R starts with the project's
`.Rprofile` and `renv` configuration available.

Verify the active R library with:

``` r
.libPaths()
```

The project-specific `renv` library should appear before global
libraries.

------------------------------------------------------------------------

# Git and GitHub

Git records the project; GitHub provides remote storage and
collaboration.

A typical workflow is:

``` bash
git status
git add .
git commit -m "Describe the change"
git push
```

Before committing, always inspect:

``` bash
git status
```

This is particularly important for research projects containing
datasets, credentials, generated files, or protected information.

## Recommended files to version-control

Generally commit:

``` text
source code
notebooks
README.md
pyproject.toml
uv.lock
.python-version
renv.lock
.Rprofile
renv activation/configuration files
small non-sensitive configuration files
```

Generally do not commit:

``` text
.venv/
R package libraries
cache directories
temporary files
credentials
API keys
tokens
passwords
private keys
sensitive datasets
large generated outputs unless intentionally tracked
```

## GitHub CLI (`gh`)

`gh` manages github repositories, issues, and pull requests from the terminal. Use `git` to track and commit local changes, and `gh` to work with GitHub. 

``` bash
brew install gh
```

Sign in and verify your connection:

```bash
gh auth login
gh status
```

Follow the prompts to authenticate through your browser. Authentication is generally a one-time setup for each machine.

## Publish an existing local project

```bash
git status
git diff
git add README.md
git commit -m "Document project setup"
```

Stage any other intended project files explicitly before committing. For a local Git repository that does not yet have a GitHub remote:

```bash
gh repo create PROJECT_NAME --private --source=. --remote=origin --push
```

Replace `PROJECT_NAME` with your repository name. This creates a private
GitHub repository, adds the `origin` remote, and pushes the existing commits.

## Clone or open a repository

Clone an existing project:

```bash
gh repo clone OWNER/REPOSITORY
cd REPOSITORY
```

Open the current repository in your browser:

```bash
gh repo view --web
```

After cloning, recreate its dependencies with `uv sync` or `renv::restore()` as appropriate.

## Collaborate with pull requests

Create a branch for a change:

```bash
git switch -c update-analysis
```

After editing, reviewing, and committing your changes:

```bash
git push -u origin update-analysis
gh pr create
```

Follow the prompts to describe the change. Inspect pull requests with:

```bash
gh pr status
gh pr list
gh pr view --web
```

See the [GitHub CLI manual](https://cli.github.com/manual/)
[repository creation reference](https://cli.github.com/manual/gh_repo_create)
and [pull request reference](https://cli.github.com/manual/gh_pr_create)

------------------------------------------------------------------------

# Recreating a Project on Another Machine

The goal is for the repository---not the original computer---to describe
the project sufficiently.

## Python

Clone the repository:

``` bash
git clone REPOSITORY_URL
cd PROJECT
```

Install `uv` if necessary, then:

``` bash
uv sync
```

Run the project:

``` bash
uv run python
```

or:

``` bash
uv run jupyter lab
```

The local `.venv` is reconstructed rather than copied from the original
machine.

## R

Clone and enter the repository:

``` bash
git clone REPOSITORY_URL
cd PROJECT
```

Install an appropriate R version and the `renv` bootstrap package if
necessary.

Then restore:

``` bash
Rscript -e 'renv::restore()'
```

Check:

``` bash
Rscript -e 'renv::status()'
```

The local package library is reconstructed from `renv.lock`.

### System dependencies still matter

Lockfiles reproduce language-level packages, but some packages require
external system libraries, compilers, or SDKs.

Examples include:

-   Apple Command Line Tools
-   C/C++ or Fortran compilers
-   system libraries installed through Homebrew
-   graphics libraries
-   database clients

If a project depends on unusual system software, document it explicitly
in the project's README.

------------------------------------------------------------------------

# Updating Dependencies

Reproducibility does not mean dependencies can never change. It means
changes should be **intentional and recorded**.

## Python

Add or update dependencies through `uv`, then commit the resulting
changes to:

``` text
pyproject.toml
uv.lock
```

Review the changes before committing.

## R

After intentionally changing packages:

``` r
renv::snapshot()
```

Then:

``` r
renv::status()
```

Review and commit the updated:

``` text
renv.lock
```

For important analyses, consider creating Git tags or releases at major
reproducible milestones.

------------------------------------------------------------------------

# Data and Secrets

A public Git repository must never contain sensitive information.

Do not commit:

-   passwords
-   API keys
-   access tokens
-   SSH/private keys
-   credentials files
-   protected health information
-   personally identifying research data
-   confidential institutional data
-   private connection strings
-   local machine identifiers that are unnecessary to the project

Use environment variables or ignored local configuration files for
secrets.

Example:

``` text
.env
```

should generally be ignored, while a safe template such as:

``` text
.env.example
```

may be committed with placeholder values.

Example:

``` text
API_KEY=replace_with_your_key
```

Before publishing a repository, inspect both the current files **and the
Git history**. Deleting a secret from the latest commit does not
automatically remove it from earlier commits.

------------------------------------------------------------------------

# Suggested Project Structures

There is no single required directory structure, but consistency helps.

## Analysis-oriented Python project

``` text
project/
├── README.md
├── pyproject.toml
├── uv.lock
├── .python-version
├── .gitignore
├── data/
├── notebooks/
├── outputs/
├── scripts/
├── src/
└── tests/
```

## Analysis-oriented R project

``` text
project/
├── README.md
├── .Rprofile
├── renv.lock
├── renv/
├── .gitignore
├── data/
├── notebooks/
├── outputs/
├── R/
└── tests/
```

### Data directories

Whether `data/` belongs in Git depends on the project.

Small, public, immutable datasets may reasonably be version-controlled.

Large, sensitive, restricted, or frequently changing datasets generally
should not be.

Document how required data are obtained and where they should be placed.

### Outputs

Decide whether generated figures, tables, and reports are:

1.  reproducible build artifacts that should be regenerated, or
2.  final research outputs worth version-controlling.

Make the choice deliberately rather than allowing the repository to
accumulate arbitrary generated files.

------------------------------------------------------------------------

# Recovery and Troubleshooting

## Python environment appears broken

Because `.venv` is disposable:

``` bash
rm -rf .venv
uv sync
```

Then verify:

``` bash
uv run python --version
```

## Confirm which Python is being used

``` bash
uv run python -c "import sys; print(sys.executable)"
```

It should point into the project's `.venv`.

## R project is out of sync

Run:

``` bash
Rscript -e 'renv::status()'
```

Read the reported differences before choosing whether the installed
library or lockfile represents the desired state.

If the lockfile is authoritative:

``` bash
Rscript -e 'renv::restore()'
```

If the installed project state is intentionally newer and should become
authoritative:

``` bash
Rscript -e 'renv::snapshot()'
```

## An installed R package is missing from `renv.lock`

Check whether the project actually references it:

``` bash
Rscript -e 'renv::dependencies()'
```

Then snapshot:

``` bash
Rscript -e 'renv::snapshot()'
```

For an empty test project, simply installing a package may not cause the
default snapshot process to regard it as a project dependency.

## R package compilation fails

If errors involve standard C/C++ headers or compiler failures, test the
system compiler independently before modifying R.

For example:

``` bash
printf '#include <cstdlib>\nint main() { return 0; }\n' | clang++ -x c++ - -o /tmp/cpp-test
```

A failure here indicates a system toolchain problem rather than an
`renv` problem.

------------------------------------------------------------------------

# Command Reference

## New Python project

``` bash
mkdir my-project
cd my-project
uv init
uv add --dev jupyter ipykernel
code .
```

## Add Python dependency

``` bash
uv add PACKAGE
```

## Run in Python environment

``` bash
uv run COMMAND
```

Examples:

``` bash
uv run python
uv run python analysis.py
uv run jupyter lab
```

## Rebuild Python environment

``` bash
rm -rf .venv
uv sync
```

## New R project

``` bash
mkdir my-r-project
cd my-r-project
Rscript -e 'renv::init(bare = TRUE)'
Rscript -e 'renv::snapshot()'
code .
```

## Install R package

``` bash
Rscript -e 'renv::install("PACKAGE")'
```

## Inspect R dependencies

``` bash
Rscript -e 'renv::dependencies()'
```

## Save R package state

``` bash
Rscript -e 'renv::snapshot()'
```

## Check R environment

``` bash
Rscript -e 'renv::status()'
```

## Restore R environment

``` bash
Rscript -e 'renv::restore()'
```

------------------------------------------------------------------------

# Reproducibility Checklist

Before considering a project reproducible, verify:

-   [ ] The project is tracked with Git.
-   [ ] The repository has a useful README.
-   [ ] Project-specific dependencies are not installed only globally.
-   [ ] Python dependencies are represented in `pyproject.toml` and
    `uv.lock`.
-   [ ] The Python `.venv` is ignored by Git.
-   [ ] R dependencies are represented in `renv.lock`.
-   [ ] The R project library is ignored by Git.
-   [ ] Required system dependencies are documented.
-   [ ] Data acquisition/location is documented.
-   [ ] Secrets and sensitive data are excluded.
-   [ ] A fresh environment can be reconstructed from the repository.
-   [ ] Important analysis milestones are committed to Git.
-   [ ] Another machine can reproduce the project using the documented
    setup steps.

------------------------------------------------------------------------

## Philosophy

The goal is not to preserve a working computer forever.

The goal is to make the computer **replaceable**.

A well-managed project should contain enough code, metadata, dependency
information, and documentation that its environment can be destroyed and
reconstructed without relying on memory or an accumulation of globally
installed packages.
