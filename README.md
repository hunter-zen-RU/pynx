<div align="center">

# pynx

**Python bootstrap · virtualenv · package installer · numbered package library**

One command that makes sure Python exists, creates a virtual environment and installs your packages,
with a live progress display, colored output and a final success/failure report.

`v3.0.0` · Bash · Linux · macOS · Termux · Windows (Git Bash / MSYS2)

**Developer: HUNTER ZEN RU** · GitHub [@hunter-zen-RU](https://gist.github.com/hunter-zen-RU)

</div>

---

## Table of contents

1. [What is pynx?](#1-what-is-pyp)
2. [Quick start](#2-quick-start)
3. [Features](#3-features)
4. [Requirements and supported platforms](#4-requirements-and-supported-platforms)
5. [Ways to run it](#5-ways-to-run-it)
6. [Command-line reference](#6-command-line-reference)
7. [Where packages come from](#7-where-packages-come-from)
8. [Requirements-file support](#8-requirements-file-support)
9. [The library and the IKP](#9-the-library-and-the-ikp-index-key-pointer)
10. [Using pyp with an AI](#10-using-pyp-with-an-ai)
11. [Automatic Python installation](#11-automatic-python-installation)
12. [Output, report, logs and exit codes](#12-output-report-logs-and-exit-codes)
13. [Safety and design notes](#13-safety-and-design-notes)
14. [Cookbook](#14-cookbook)
15. [Troubleshooting / FAQ](#15-troubleshooting--faq)
16. [Security](#16-security)
17. [Test status](#17-test-status)
18. [Project files](#18-project-files)
19. [Maintaining the library](#19-maintaining-the-library)
20. [Changelog](#20-changelog)
21. [Credits and license](#21-credits-and-license)

---

## 1. What is pynx?

Setting up a Python project by hand is repetitive. On Linux you often need `sudo apt install python3-venv python3-pip`
first, then `python3 -m venv`, then activate, then `pip install` package after package, and when one fails you have to
scroll back through the output to find out which.

**pyp** does all of it in one go:

1. **Checks that Python 3.8+ is installed** (with working `venv` and `pip`), and **installs it for you if it is missing**.
2. **Creates a virtual environment** (or reuses an existing one).
3. **Installs your packages** one by one with a live spinner and progress bar.
4. **Prints a report**: how many packages were installed, already up to date or failed, and *why* each failure happened.

It also contains a **numbered library of 170 popular pip packages**. Any selection from that library can be written as a
single short code, the **ikp** (*index key pointer*), so a whole project setup fits in one command, and an AI can work that
code out for you (see [section 9](#9-the-library-and-the-ikp-index-key-pointer) and [section 10](#10-using-pyp-with-an-ai)).

pyp is one self-contained Bash file. It is designed to run **straight from a GitHub Gist**, with nothing to install first.

---

## 2. Quick start

The raw link of the script (always the latest revision):

```text
https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pyp.sh
```

**Install two packages into a new virtualenv called `myenv`** (works in bash, zsh, **fish** and every other shell):

```bash
curl -fsSL https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pyp.sh | bash -s -- -y flask requests
```

**Install from the built-in library** (`-L 1` is `requests`; `-L 0x15` is entries 1, 3 and 5):

```bash
curl -fsSL https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pyp.sh | bash -s -- -y -L 0x15
```

**Preview first, install nothing** (add `-n`):

```bash
curl -fsSL https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pyp.sh | bash -s -- -n -L 0x15
```

**Activate the environment afterwards.** pyp prints the right line for your shell:

| Shell | Command |
|-------|---------|
| bash / zsh | `source myenv/bin/activate` |
| fish | `source myenv/bin/activate.fish` |
| csh / tcsh | `source myenv/bin/activate.csh` |
| Windows Git Bash | `source myenv/Scripts/activate` |

> **Using fish?** Fish has no `<( ... )` process substitution, so always use the **pipe form**
> (`curl ... | bash -s -- ...`) shown above. It works everywhere.

---

## 3. Features

| Area | What you get |
|------|--------------|
| **Python bootstrap** | Detects Python 3.8+ with `venv` and `pip`; if missing (or too old, or without `venv`) installs it with the system package manager after asking (or automatically with `-y`) |
| **Virtualenv** | Creates or reuses a venv (`myenv` by default), optional `--recreate`, optional `--upgrade-pip` |
| **Live progress** | Spinner, per-package timer and an overall progress bar; plain line-by-line output when piped or logged |
| **Color and symbols** | ✔ installed · ● already satisfied · ✘ failed; automatic ASCII fallback (`+ = x`) on non-UTF-8 terminals; honors `NO_COLOR` |
| **Final report** | Installed / up-to-date / failed / not attempted counts, total time, per-failure reason, full pip log path |
| **Keep going on errors** | A failed package never stops the others (unless you pass `--fail-fast`) |
| **Numbered library + IKP** | 170 curated packages addressed by index or by a hex *index key pointer* |
| **Requirements files** | Comments, CRLF, line continuation, nested `-r`, global pip options, and **remote `http(s)://` files** |
| **Gist-ready** | Works with `curl \| bash`, `bash <(curl ...)` and `source <(curl ...)`; prompts read from the real terminal |
| **Freeze and report files** | `--freeze lock.txt` and `--report report.txt` |
| **Safe** | Never lets child processes read your stdin, restores the cursor, handles Ctrl-C, refuses to delete non-venv folders |
| **CI friendly** | Quiet mode, meaningful exit codes, no prompts with `-y` |

---

## 4. Requirements and supported platforms

**To run pyp itself:**

* **Bash 4.4 or newer.** macOS ships Bash 3.2, so install a newer one (`brew install bash`) and run pyp with it.
* `curl` (or `wget`) to fetch the script from the gist and remote requirements files.
* `sudo` (or run as root) **only if** Python has to be installed through a system package manager.

**Python itself** is installed by pyp when missing. Package managers it knows:

| System | Package manager | What is installed |
|--------|-----------------|-------------------|
| Debian, Ubuntu, **Kali**, Mint | `apt-get` | `python3 python3-venv python3-pip` |
| Fedora, RHEL, Rocky, Alma | `dnf` / `yum` | `python3 python3-pip` |
| openSUSE | `zypper` | `python3 python3-pip` |
| Arch, Manjaro | `pacman` | `python python-pip` |
| Alpine | `apk` | `python3 py3-pip` |
| macOS | Homebrew (`brew`) | `python` |
| Android (Termux) | `pkg` | `python` |
| Windows, MSYS2 | `pacman` | `python python-pip` |
| Windows, Git Bash | `winget` | Python 3.12 |

If no supported package manager is found, pyp stops with clear manual instructions instead of guessing.

> **Note:** run pyp with Bash. It can be launched from fish, zsh or any other shell (`curl ... | bash -s -- ...`),
> but **sourcing** it (`source pyp.sh`) only works from Bash.

---

## 5. Ways to run it

### A. Straight from the Gist (recommended)

```bash
URL=https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pyp.sh

curl -fsSL $URL | bash -s -- -y flask requests     # every shell, including fish
bash <(curl -fsSL $URL) -y flask requests          # bash / zsh only (needs <( ))
source <(curl -fsSL $URL) -y -L 0x15               # bash only: installs AND activates the venv
```

### B. Download once, run locally

```bash
curl -fsSLO https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pyp.sh
chmod +x pyp.sh
./pyp.sh -y -r requirements.txt
```

### C. Source it (Bash only) to stay activated

```bash
source ./pyp.sh -y -e .venv requests
# the venv is now active in your current shell; pyp removes all of its own
# helper functions and variables again, leaving only `deactivate`
```

When you **run** pyp (instead of sourcing it), the environment cannot stay activated in your shell, because a child process
cannot change its parent. pyp then prints the exact `source ...` line for your shell.

> **About `curl | bash`:** the script itself is the program's standard input in this mode. pyp is written for that:
> every child process gets `</dev/null`, and confirmation prompts are read from `/dev/tty`. If there is no terminal at all
> and you did not pass `-y`, pyp stops with a clear message instead of hanging.

---

## 6. Command-line reference

```text
pyp.sh [options] [package ...]
```

| Option | Description |
|--------|-------------|
| `-r`, `--requirements FILE` | Requirements file **or `http(s)://` URL** (default: `requirements.txt`) |
| `-e`, `--env DIR` | Virtualenv directory (default: `myenv`) |
| `-p`, `--python EXE` | Use this interpreter instead of auto-detecting. An explicit interpreter is **never** replaced by an auto-install |
| `--no-install-python` | Never install Python; fail if it is missing |
| `-L`, `--lib IKP` | Install library entries by **index or ikp**. Repeatable; comma lists allowed |
| `--list-lib` | Print the library with index, block and weight, then exit |
| `--filter TEXT` | With `--list-lib`: only rows containing `TEXT` (case-insensitive) |
| `--md` | With `--list-lib`: Markdown table output |
| `-y`, `--yes` | Do not ask for confirmation (also allows installing Python) |
| `-u`, `--upgrade` | Pass `--upgrade` to pip for every package |
| `--upgrade-pip` | Upgrade pip inside the venv first |
| `-R`, `--recreate` | Delete and recreate the venv (only if it really is a venv) |
| `-f`, `--fail-fast` | Stop at the first failed package (default: keep going) |
| `-n`, `--dry-run` | Show Python status, the decoded library selection and the package list; change nothing |
| `-l`, `--log FILE` | Save the full pip output of every package to `FILE` |
| `--freeze FILE` | Write `pip freeze` of the venv to `FILE` when done |
| `--report FILE` | Write the summary as a plain-text report |
| `--no-activate` | Do not activate the venv (when sourced) |
| `-q`, `--quiet` | Only errors and a one-line summary |
| `-v`, `--verbose` | Show pip's output for every package |
| `--no-color` | Disable colors (`NO_COLOR` is also honored) |
| `-V`, `--version` | Print the version |
| `-h`, `--help` | Show help |

* Options with a value accept both `--env DIR` and `--env=DIR`.
* Short flags can be combined: `-yqu` = `-y -q -u`. (Options that take a value, such as `-e` or `-L`, are not combinable.)
* `--` ends option parsing; everything after it is a package.

---

## 7. Where packages come from

pyp can take packages from **three sources at once**:

| Source | How | Prompts before installing? |
|--------|-----|----------------------------|
| Requirements file | `-r FILE` or an automatic `./requirements.txt` | yes (unless `-y`) |
| Library | `-L <index or ikp>` | yes (unless `-y`) |
| Command line | `pyp.sh flask requests` | no |

Rules:

* **Order of installation:** requirements-file packages, then library packages (ascending index), then command-line packages.
* **Exact duplicates are removed** (`requests` from the library plus `requests` on the command line installs it once); the header tells you how many were skipped.
* If you give `-L` or packages on the command line, `./requirements.txt` is **not** read automatically. Add `-r` explicitly to combine them.
* With nothing at all, pyp looks for `./requirements.txt`; if it is missing you get a clear usage error.

---

## 8. Requirements-file support

pyp reads requirements files itself, so every line is installed and reported individually. Supported:

* comments (`# ...`), inline comments (`pkg==1.0  # why`), blank lines
* Windows line endings (CRLF)
* line continuation with a trailing `\`
* version specifiers and markers, e.g. `requests>=2.31`, `pkg ; python_version >= "3.9"`, extras like `requests[socks]`
* **nested includes:** `-r other.txt` (up to 5 levels deep, loops are detected)
* **global options**, applied to every install: `--index-url` / `-i`, `--extra-index-url`, `--find-links` / `-f`, `--trusted-host`, `--pre`, `--no-index`, `--no-binary`, `--only-binary`, `--prefer-binary`
* other pip lines such as `-e .` are passed through to pip
* **remote files:** `-r https://...`. Nested `-r more.txt` lines inside a remote file are resolved **relative to its URL**, so a gist can hold a whole set of requirement files

```bash
curl -fsSL $URL | bash -s -- -y -r https://gist.githubusercontent.com/USER/GIST_ID/raw/requirements.txt
```

---

## 9. The library and the IKP (Index Key Pointer)

### 9.1 The library

pyp ships a numbered library of **170 popular pip packages**. Every package has a permanent **index** (1, 2, 3, ...), and
index **1 is `requests`**. See it any time:

```bash
pyp.sh --list-lib                  # the whole table
pyp.sh --list-lib --filter http    # search by name, category or description
pyp.sh --list-lib --md             # Markdown (this is how the AI prompt table is generated)
```

| Category | Entries |
|----------|--------:|
| ML/AI | 26 |
| Utils | 23 |
| Data | 16 |
| Web | 15 |
| CLI | 11 |
| Database | 10 |
| Packaging | 9 |
| Networking | 8 |
| Testing | 8 |
| Documents | 8 |
| Quality | 7 |
| Cloud | 7 |
| Security | 6 |
| Apps | 5 |
| Scraping | 4 |
| Notebook | 4 |
| Parsing | 3 |

The full list is in [section 9.9](#99-the-full-library). It is a **curated** set, not all of PyPI (which has hundreds of thousands
of packages); anything not in the library can still be given by name, and the library can grow (see [section 19](#19-maintaining-the-library)).

### 9.2 One index: `-L N`

A **plain number** always means **one** library entry:

```bash
pyp.sh -y -L 1        # requests
pyp.sh -y -L 11       # fastapi
```

### 9.3 Several entries: the ikp

When you want more than one entry you need an **ikp**: a **hex number, written with a `0x` prefix**, in which every selected
index is switched on as one bit.

**The formula** (you or an AI calculate it; pyp only *decodes* it):

```text
ikp = "0x" + hex( sum of 2^(i-1) for every selected index i )
```

Example, entries 1, 3 and 5:

```text
2^0 + 2^2 + 2^4  =  1 + 4 + 16  =  21  =  0x15
```

```bash
pyp.sh -y -L 0x15     # installs entries [1] requests, [3] aiohttp, [5] websockets
```

### 9.4 The block method (best for bigger indices)

Big powers of two are error-prone by hand, so work in **blocks of four indices**. Each block is **one hex digit**:

| Block | Indices | Weights |
|------:|---------|---------|
| 0 | 1, 2, 3, 4 | 1, 2, 4, 8 |
| 1 | 5, 6, 7, 8 | 1, 2, 4, 8 |
| 2 | 9, 10, 11, 12 | 1, 2, 4, 8 |
| *b* | 4b+1 ... 4b+4 | 1, 2, 4, 8 |

1. For each block, **add the weights** of the selected indices in it: the sum (0 to 15) is one hex digit (10=A, 11=B, 12=C, 13=D, 14=E, 15=F).
2. Write the digits **highest block on the left, block 0 on the right**. Blocks with no selection are `0`.
3. Put `0x` in front. Leading zeros may be dropped.

The `--list-lib` table has **Block** and **Weight** columns so you can read these straight off.

**Worked example:** `rich` [80], `click` [81], `colorama` [83], `tqdm` [84], `pyyaml` [89]

| Block | Chosen (index = weight) | Sum | Hex digit |
|------:|-------------------------|----:|:---------:|
| 22 | 89 = 1 | 1 | `1` |
| 21 | none | 0 | `0` |
| 20 | 81 = 1, 83 = 4, 84 = 8 | 13 | `D` |
| 19 | 80 = 8 | 8 | `8` |
| 18 ... 0 | none | 0 | `0` (19 zeros) |

Result: `-L 0x10d80000000000000000000`

### 9.5 Singular versus combined (the one rule to remember)

| You write | Meaning |
|-----------|---------|
| `-L 3` | **one entry**: index 3 |
| `-L 0x3` | **an ikp**: entries 1 **and** 2 (bits 0 and 1) |

* A plain number is **never** an ikp; hex **must** start with `0x` (or `0X`).
* `-L` can be repeated and/or comma-separated: `-L 1,0x14 -L 7`. Everything is merged and de-duplicated.
* Upper/lower case hex digits and leading zeros are fine (`0xFF` = `0xff` = `0x00ff`).
* There is **no size limit**: pyp decodes the hex digit by digit, so an ikp for a library of any size works (the current 170-entry library already needs 170 bits).

### 9.6 Verify before you install

pyp always prints a **Decoded selection** so a wrong ikp is obvious. To look without installing, add `-n`:

```text
$ pyp.sh -n -L 0x15
pyp 3.0.0
  Library:      -L 0x15  ->  3 package(s) (library v1, 170 entries)
  Environment:  myenv
  Decoded selection:
    [1] requests - HTTP library for humans
    [3] aiohttp - Async HTTP client and server
    [5] websockets - WebSocket client and server
```

### 9.7 Errors (nothing is ever installed on a bad ikp, exit code `2`)

| Input | Message |
|-------|---------|
| `-L 0` | library indices start at 1 |
| `-L 171` | index 171 is not in this library (v1 has 170 entries) |
| `-L ff` | not valid: use a plain number for ONE index or 0x-prefixed hex for an ikp |
| `-L 0x0` | this ikp selects nothing |
| `-L 0xZZ` | not valid |
| ikp with a bit beyond the library | points at index N, but this library has only 170 entries; *your copy of pyp.sh may be older than the one this ikp was made for* |
| `-L ""` or `-L` | needs an index or an ikp |

### 9.8 Test vectors

If your calculation method reproduces every row, it is correct.

| Selected indices | ikp |
|------------------|-----|
| 1 | `0x1` |
| 2 | `0x2` |
| 3 | `0x4` |
| 4 | `0x8` |
| 5 | `0x10` |
| 1, 2 | `0x3` |
| 1, 3, 5 | `0x15` |
| 1, 2, 3, 4 | `0xf` |
| 1 to 8 | `0xff` |
| 9 | `0x100` |
| 4, 5 | `0x18` |
| 10, 20 | `0x80200` |
| 170 (last entry) | `0x2000000000000000000000000000000000000000000` |

### 9.9 The full library

The library is **append-only**: an index never changes meaning, so an ikp written today works forever.

<details>
<summary><b>Click to expand all 170 entries</b></summary>

| Index | Block | Weight | Package | Category | Description |
|------:|------:|-------:|---------|----------|-------------|
| 1 | 0 | 1 | requests | Networking | HTTP library for humans |
| 2 | 0 | 2 | httpx | Networking | Modern sync/async HTTP client |
| 3 | 0 | 4 | aiohttp | Networking | Async HTTP client and server |
| 4 | 0 | 8 | urllib3 | Networking | Low-level HTTP client |
| 5 | 1 | 1 | websockets | Networking | WebSocket client and server |
| 6 | 1 | 2 | paramiko | Networking | SSH2 protocol library |
| 7 | 1 | 4 | certifi | Networking | Mozilla CA certificate bundle |
| 8 | 1 | 8 | dnspython | Networking | DNS toolkit |
| 9 | 2 | 1 | flask | Web | Lightweight WSGI web framework |
| 10 | 2 | 2 | django | Web | Batteries-included web framework |
| 11 | 2 | 4 | fastapi | Web | Fast async API framework |
| 12 | 2 | 8 | uvicorn | Web | ASGI server |
| 13 | 3 | 1 | gunicorn | Web | WSGI HTTP server |
| 14 | 3 | 2 | starlette | Web | ASGI toolkit |
| 15 | 3 | 4 | jinja2 | Web | Template engine |
| 16 | 3 | 8 | werkzeug | Web | WSGI utility library |
| 17 | 4 | 1 | djangorestframework | Web | REST toolkit for Django |
| 18 | 4 | 2 | flask-cors | Web | CORS support for Flask |
| 19 | 4 | 4 | python-multipart | Web | Multipart form parser |
| 20 | 4 | 8 | pydantic | Web | Data validation using type hints |
| 21 | 5 | 1 | celery | Web | Distributed task queue |
| 22 | 5 | 2 | streamlit | Web | Data apps in pure Python |
| 23 | 5 | 4 | gradio | Web | Quick web UIs for ML demos |
| 24 | 5 | 8 | sqlalchemy | Database | SQL toolkit and ORM |
| 25 | 6 | 1 | alembic | Database | SQLAlchemy migrations |
| 26 | 6 | 2 | psycopg2-binary | Database | PostgreSQL adapter |
| 27 | 6 | 4 | pymysql | Database | Pure-Python MySQL client |
| 28 | 6 | 8 | pymongo | Database | MongoDB driver |
| 29 | 7 | 1 | redis | Database | Redis client |
| 30 | 7 | 2 | sqlmodel | Database | SQLAlchemy + Pydantic models |
| 31 | 7 | 4 | peewee | Database | Small ORM |
| 32 | 7 | 8 | duckdb | Database | In-process analytical database |
| 33 | 8 | 1 | elasticsearch | Database | Elasticsearch client |
| 34 | 8 | 2 | numpy | Data | N-dimensional arrays |
| 35 | 8 | 4 | pandas | Data | DataFrames and data analysis |
| 36 | 8 | 8 | scipy | Data | Scientific computing |
| 37 | 9 | 1 | matplotlib | Data | Plotting library |
| 38 | 9 | 2 | seaborn | Data | Statistical data visualization |
| 39 | 9 | 4 | plotly | Data | Interactive graphing |
| 40 | 9 | 8 | polars | Data | Fast DataFrame library |
| 41 | 10 | 1 | pyarrow | Data | Apache Arrow bindings |
| 42 | 10 | 2 | openpyxl | Data | Read/write Excel xlsx files |
| 43 | 10 | 4 | xlsxwriter | Data | Write Excel xlsx files |
| 44 | 10 | 8 | tabulate | Data | Pretty-print tables |
| 45 | 11 | 1 | statsmodels | Data | Statistical modeling |
| 46 | 11 | 2 | sympy | Data | Symbolic mathematics |
| 47 | 11 | 4 | networkx | Data | Graph and network analysis |
| 48 | 11 | 8 | dask | Data | Parallel computing |
| 49 | 12 | 1 | altair | Data | Declarative visualization |
| 50 | 12 | 2 | scikit-learn | ML/AI | Machine learning toolkit |
| 51 | 12 | 4 | scikit-image | ML/AI | Image processing |
| 52 | 12 | 8 | torch | ML/AI | PyTorch deep learning |
| 53 | 13 | 1 | torchvision | ML/AI | PyTorch vision utilities |
| 54 | 13 | 2 | torchaudio | ML/AI | PyTorch audio utilities |
| 55 | 13 | 4 | tensorflow | ML/AI | TensorFlow deep learning |
| 56 | 13 | 8 | keras | ML/AI | Keras deep learning API |
| 57 | 14 | 1 | transformers | ML/AI | Hugging Face Transformers |
| 58 | 14 | 2 | datasets | ML/AI | Hugging Face Datasets |
| 59 | 14 | 4 | tokenizers | ML/AI | Fast tokenizers |
| 60 | 14 | 8 | sentencepiece | ML/AI | Subword tokenizer |
| 61 | 15 | 1 | accelerate | ML/AI | Hugging Face training accelerator |
| 62 | 15 | 2 | xgboost | ML/AI | Gradient boosting |
| 63 | 15 | 4 | lightgbm | ML/AI | Gradient boosting |
| 64 | 15 | 8 | catboost | ML/AI | Gradient boosting |
| 65 | 16 | 1 | onnxruntime | ML/AI | ONNX model runtime |
| 66 | 16 | 2 | opencv-python | ML/AI | OpenCV computer vision |
| 67 | 16 | 4 | pillow | ML/AI | Image library (PIL) |
| 68 | 16 | 8 | openai | ML/AI | OpenAI API client |
| 69 | 17 | 1 | anthropic | ML/AI | Anthropic API client |
| 70 | 17 | 2 | langchain | ML/AI | LLM application framework |
| 71 | 17 | 4 | tiktoken | ML/AI | Fast BPE tokenizer |
| 72 | 17 | 8 | huggingface-hub | ML/AI | Hugging Face Hub client |
| 73 | 18 | 1 | nltk | ML/AI | Natural language toolkit |
| 74 | 18 | 2 | spacy | ML/AI | Industrial NLP |
| 75 | 18 | 4 | gensim | ML/AI | Topic modelling |
| 76 | 18 | 8 | jupyterlab | Notebook | JupyterLab interface |
| 77 | 19 | 1 | notebook | Notebook | Classic Jupyter Notebook |
| 78 | 19 | 2 | ipython | Notebook | Interactive Python shell |
| 79 | 19 | 4 | ipykernel | Notebook | Jupyter kernel for Python |
| 80 | 19 | 8 | rich | CLI | Rich text and formatting in terminals |
| 81 | 20 | 1 | click | CLI | Command line toolkit |
| 82 | 20 | 2 | typer | CLI | CLI apps from type hints |
| 83 | 20 | 4 | colorama | CLI | Cross-platform colored terminal text |
| 84 | 20 | 8 | tqdm | CLI | Progress bars |
| 85 | 21 | 1 | questionary | CLI | Interactive prompts |
| 86 | 21 | 2 | prompt-toolkit | CLI | Interactive command lines |
| 87 | 21 | 4 | termcolor | CLI | ANSI color formatting |
| 88 | 21 | 8 | python-dotenv | CLI | Load .env files |
| 89 | 22 | 1 | pyyaml | CLI | YAML parser/emitter |
| 90 | 22 | 2 | tomli | CLI | TOML parser |
| 91 | 22 | 4 | pytest | Testing | Testing framework |
| 92 | 22 | 8 | pytest-cov | Testing | Coverage plugin for pytest |
| 93 | 23 | 1 | pytest-asyncio | Testing | Asyncio support for pytest |
| 94 | 23 | 2 | tox | Testing | Test automation |
| 95 | 23 | 4 | coverage | Testing | Code coverage measurement |
| 96 | 23 | 8 | hypothesis | Testing | Property-based testing |
| 97 | 24 | 1 | faker | Testing | Fake data generator |
| 98 | 24 | 2 | responses | Testing | Mock the requests library |
| 99 | 24 | 4 | black | Quality | Code formatter |
| 100 | 24 | 8 | ruff | Quality | Fast linter and formatter |
| 101 | 25 | 1 | flake8 | Quality | Style guide enforcement |
| 102 | 25 | 2 | pylint | Quality | Static code analysis |
| 103 | 25 | 4 | mypy | Quality | Static type checker |
| 104 | 25 | 8 | isort | Quality | Import sorter |
| 105 | 26 | 1 | pre-commit | Quality | Git hook manager |
| 106 | 26 | 2 | build | Packaging | PEP 517 package builder |
| 107 | 26 | 4 | twine | Packaging | Upload packages to PyPI |
| 108 | 26 | 8 | setuptools | Packaging | Build system |
| 109 | 27 | 1 | wheel | Packaging | Wheel packaging |
| 110 | 27 | 2 | virtualenv | Packaging | Virtual environments |
| 111 | 27 | 4 | pipx | Packaging | Install CLI apps in isolation |
| 112 | 27 | 8 | pip-tools | Packaging | Pin and sync requirements |
| 113 | 28 | 1 | poetry | Packaging | Dependency management |
| 114 | 28 | 2 | pyinstaller | Packaging | Bundle apps as executables |
| 115 | 28 | 4 | beautifulsoup4 | Parsing | HTML/XML parsing |
| 116 | 28 | 8 | lxml | Parsing | Fast XML/HTML processing |
| 117 | 29 | 1 | html5lib | Parsing | HTML5 parser |
| 118 | 29 | 2 | selenium | Scraping | Browser automation |
| 119 | 29 | 4 | playwright | Scraping | Modern browser automation |
| 120 | 29 | 8 | scrapy | Scraping | Web crawling framework |
| 121 | 30 | 1 | yt-dlp | Scraping | Video downloader |
| 122 | 30 | 2 | python-dateutil | Utils | Date/time extensions |
| 123 | 30 | 4 | pytz | Utils | Time zone definitions |
| 124 | 30 | 8 | arrow | Utils | Friendlier dates and times |
| 125 | 31 | 1 | pendulum | Utils | Easy datetimes |
| 126 | 31 | 2 | regex | Utils | Advanced regular expressions |
| 127 | 31 | 4 | orjson | Utils | Fast JSON |
| 128 | 31 | 8 | ujson | Utils | Ultra-fast JSON |
| 129 | 32 | 1 | msgpack | Utils | MessagePack serialization |
| 130 | 32 | 2 | protobuf | Utils | Protocol Buffers |
| 131 | 32 | 4 | jsonschema | Utils | JSON Schema validation |
| 132 | 32 | 8 | marshmallow | Utils | Object serialization |
| 133 | 33 | 1 | attrs | Utils | Classes without boilerplate |
| 134 | 33 | 2 | cachetools | Utils | Caching utilities |
| 135 | 33 | 4 | tenacity | Utils | Retrying library |
| 136 | 33 | 8 | loguru | Utils | Simple logging |
| 137 | 34 | 1 | structlog | Utils | Structured logging |
| 138 | 34 | 2 | email-validator | Utils | Email address validation |
| 139 | 34 | 4 | validators | Utils | Value validators |
| 140 | 34 | 8 | psutil | Utils | System and process utilities |
| 141 | 35 | 1 | watchdog | Utils | Filesystem event monitoring |
| 142 | 35 | 2 | schedule | Utils | Job scheduling for humans |
| 143 | 35 | 4 | apscheduler | Utils | Advanced Python scheduler |
| 144 | 35 | 8 | pyperclip | Utils | Clipboard access |
| 145 | 36 | 1 | markdown | Documents | Markdown to HTML |
| 146 | 36 | 2 | pypdf | Documents | Read/write PDF files |
| 147 | 36 | 4 | reportlab | Documents | Generate PDF files |
| 148 | 36 | 8 | python-docx | Documents | Read/write Word files |
| 149 | 37 | 1 | python-pptx | Documents | Read/write PowerPoint files |
| 150 | 37 | 2 | qrcode | Documents | QR code generator |
| 151 | 37 | 4 | mkdocs | Documents | Project documentation |
| 152 | 37 | 8 | sphinx | Documents | Documentation generator |
| 153 | 38 | 1 | boto3 | Cloud | AWS SDK |
| 154 | 38 | 2 | botocore | Cloud | AWS low-level client |
| 155 | 38 | 4 | google-cloud-storage | Cloud | Google Cloud Storage client |
| 156 | 38 | 8 | azure-storage-blob | Cloud | Azure Blob Storage client |
| 157 | 39 | 1 | kubernetes | Cloud | Kubernetes API client |
| 158 | 39 | 2 | docker | Cloud | Docker Engine API client |
| 159 | 39 | 4 | fabric | Cloud | Remote execution over SSH |
| 160 | 39 | 8 | cryptography | Security | Cryptographic recipes |
| 161 | 40 | 1 | bcrypt | Security | Password hashing |
| 162 | 40 | 2 | pyjwt | Security | JSON Web Tokens |
| 163 | 40 | 4 | passlib | Security | Password hashing framework |
| 164 | 40 | 8 | pycryptodome | Security | Low-level crypto primitives |
| 165 | 41 | 1 | pyopenssl | Security | OpenSSL wrapper |
| 166 | 41 | 2 | pygame | Apps | Games and multimedia |
| 167 | 41 | 4 | pyqt6 | Apps | Qt6 GUI bindings |
| 168 | 41 | 8 | discord.py | Apps | Discord bot API |
| 169 | 42 | 1 | python-telegram-bot | Apps | Telegram bot API |
| 170 | 42 | 2 | tweepy | Apps | X/Twitter API client |

</details>

---

## 10. Using pyp with an AI

You do not have to calculate an ikp yourself. The gist contains a second file, **`ikp-prompt.md`**, written for AI
assistants. It holds the library table, the formula, the block method, a self-check, worked examples, test vectors and
the exact answer format.

In any new project, give your AI one line:

```text
Read https://gist.github.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4 and give me the pyp.sh command that installs what my project needs.
```

The AI then:

1. works out which packages your project needs,
2. looks them up in the library table by exact pip name,
3. computes the ikp with the block method and checks it by decoding it again,
4. replies with **one ready-to-run command**, plus any packages that are *not* in the library as normal arguments, for example:

```bash
curl -fsSL https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pyp.sh | bash -s -- -y -L 0x40000000000000000880c01 some-extra-package
```

> **Always preview an AI-made ikp with `-n` first.** The decoded list tells you immediately whether the ikp selects the packages you expected.
> pyp itself never calculates ikps; it only decodes them, so the calculation always happens on your side or your AI's side.

---

## 11. Automatic Python installation

On every run pyp first looks for a **usable** interpreter: it tries `python3`, `python`, then `python3.13` down to
`python3.8`, and accepts the first one that is **version 3.8 or newer** *and* has a working **`venv`** and **`pip`**
(`ensurepip`).

If none qualifies it tells you exactly why:

| Message | Meaning |
|---------|---------|
| Python is not installed | no interpreter found |
| no usable Python 3.8+ was found | only an old or broken one (for example the Windows Store stub) |
| Python is installed but its venv/pip support is missing | the classic Debian/Ubuntu case without `python3-venv` |

Then, **after** you confirmed the package installation (or with `-y`), it installs Python:

1. picks the package manager (see the table in [section 4](#4-requirements-and-supported-platforms)),
2. if root rights are needed it asks for your `sudo` password **before** the spinner starts, so the prompt is not drawn over,
3. runs the install steps with a spinner (for apt: `apt-get update`, then the install),
4. re-checks for Python and carries on straight into the virtualenv and package install.

Good to know:

* `--no-install-python` turns the feature off; pyp then fails with the reason.
* `--python /path/to/python` is an explicit choice and is **never** replaced by an auto-install.
* `--dry-run` reports whether Python would be installed, and changes nothing.
* On Windows (Git Bash) winget does not update the running shell's `PATH`. pyp tries the default install folder; if that fails it asks you to open a new terminal and run it again.
* Without a terminal and without `-y`, pyp refuses to install Python rather than hang on a prompt.

---

## 12. Output, report, logs and exit codes

### What a run looks like

```text
pyp 3.0.0
  Library:      -L 0x10d80000000000000000000  ->  5 package(s) (library v1, 170 entries)
  Environment:  testenv
  Decoded selection:
    [80] rich - Rich text and formatting in terminals
    [81] click - Command line toolkit
    [83] colorama - Cross-platform colored terminal text
    [84] tqdm - Progress bars
    [89] pyyaml - YAML parser/emitter
  Python:       Python 3.12.3 (/usr/bin/python3)

  ✔ Virtual environment created (4.9s)
Installing 5 package(s)
  ✔ rich       15.0.0       +3 deps  8.2s
  ✔ click      8.5.0        2.0s
  ✔ colorama   0.4.6        1.1s
  ✔ tqdm       4.70.1       2.5s
  ✔ pyyaml     6.0.3        1.6s

────────────────────────────────────────────
 Summary
────────────────────────────────────────────
  [██████████████████████████████] 100% succeeded

  ✔ Installed     5
  ● Up-to-date    0
  ✘ Failed        0
  Total         5 package(s) in 15.6s
```

While a package is installing, one live line shows the overall bar, the spinner and a timer:
`[█████░░░░░░░░░░░░░░░] 1/4  ⠹ Installing numpy (12s)`. pip does not report a real percentage for a single package, so the
bar shows **overall** progress across packages.

### Result symbols

| Symbol | ASCII | Meaning |
|:------:|:-----:|---------|
| ✔ | `+` | installed (shows the version and how many dependencies came along) |
| ● | `=` | already satisfied (version shown) |
| ✘ | `x` | failed (the reason is shown, and listed again in the summary) |
| `!` | `!` | warning / not attempted |

### Files

| File | How |
|------|-----|
| Full pip output | kept automatically at `/tmp/pyp-<pid>.log` **when something failed** (path is printed); always saved with `--log FILE` |
| `pip freeze` | `--freeze lock.txt` |
| Plain-text report | `--report report.txt` (environment, Python version, counts, duration, failures) |

### Exit codes

| Code | Meaning |
|-----:|---------|
| `0` | everything succeeded (or was already satisfied) |
| `1` | one or more packages failed, or a runtime error (Python or venv could not be set up) |
| `2` | bad usage: unknown option, invalid ikp, missing file, no prompt possible |
| `130` | interrupted with Ctrl-C (a partial summary is printed) |

### Environment variables and terminal handling

| Variable | Effect |
|----------|--------|
| `NO_COLOR` | disables colors |
| `TMPDIR` | where the temporary pip log is written (default `/tmp`) |
| `COLUMNS` / `TERM` | layout width; `TERM=dumb` or a non-terminal turns the animation off |
| `LANG` / `LC_ALL` | UTF-8 locales get ✔ ● ✘ and block characters; others get ASCII |

---

## 13. Safety and design notes

* **stdin is protected.** In `curl | bash` mode stdin *is* the script; any command that read it would swallow the rest of the program. Every child process runs with `</dev/null`, and prompts use `/dev/tty`.
* **Truncated downloads cannot run half a script.** All logic is inside functions and the call to `main` is the last line of the file.
* **Ctrl-C is handled.** The running pip is stopped, the cursor and job-control state are restored, temporary files are removed, a partial summary is printed and the exit code is `130`.
* **`--recreate` is guarded.** It refuses to delete `/`, `.`, `..`, `~`, `$HOME`, or any folder without a `pyvenv.cfg`.
* **An existing non-empty, non-venv folder is never touched.** pyp stops and asks you to pick another `--env`.
* **No clutter when sourced.** All internal functions and variables are unset afterwards; no `set -e` is used, so sourcing cannot kill your shell.
* **No job-control noise.** `[1] 12345` / `Done` notices are suppressed in interactive shells.
* **pip is run as `venv/bin/python -m pip`**, so it does not depend on activation and always targets the right environment.
* **The library is data, not code.** The ikp decoder only reads digits `0-9a-f`; nothing from an ikp is ever executed.

---

## 14. Cookbook

Define the raw URL once (bash/zsh: `RAW=...`, fish: `set RAW ...`):

```bash
RAW=https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pyp.sh

# Web API project (requests, fastapi, uvicorn, pydantic, sqlalchemy, pytest): preview, then install
curl -fsSL $RAW | bash -s -- -n -L 0x40000000000000000880c01
curl -fsSL $RAW | bash -s -- -y -L 0x40000000000000000880c01 -e .venv

# Pin the result of a run
curl -fsSL $RAW | bash -s -- -y -r requirements.txt --freeze lock.txt --report report.txt

# Upgrade everything listed in a file and stop at the first problem
./pyp.sh -y -u -f -r requirements.txt

# Start clean
./pyp.sh -y -R -e .venv -r requirements.txt

# Several library selections + a normal package
./pyp.sh -y -L 1,0x14 -L 7 rich

# Use a specific interpreter (never auto-installs another)
./pyp.sh -y -p /usr/bin/python3.11 -e .venv flask

# Requirements file from a URL, with another file nested inside it
./pyp.sh -y -r https://gist.githubusercontent.com/USER/GIST_ID/raw/requirements.txt

# CI: no prompts, no color, one-line summary, exit code tells the result
./pyp.sh -y -q --no-color -r requirements.txt || echo "some packages failed"

# Show pip's own output for every package
./pyp.sh -y -v flask

# Find library indices
./pyp.sh --list-lib --filter database
```

---

## 15. Troubleshooting / FAQ

**`fish: Invalid redirection target` when running `bash <(curl ...)`**
Fish does not support `<( )`. Use the pipe form: `curl -fsSL URL | bash -s -- -y ...`.

**`error: unknown option: -L`**
You are running an older copy of the script (v2.x). Check with `... | bash -s -- -V`; it must print `pyp 3.0.0`. After you edit a gist,
GitHub's raw link can serve the old content for a few minutes; wait a little, or use the commit-pinned raw URL.

**`error: No terminal available for the prompt; pass -y/--yes`**
pyp needs to ask a question (requirements file, library selection, or installing Python) but there is no terminal, for example in CI or a cron job. Add `-y`.

**Virtualenv creation fails with `ensurepip is not available`**
Your Python lacks `venv` support (Debian/Ubuntu/Kali: `python3-venv`). pyp normally detects and installs this by itself; if it was
disabled with `--no-install-python`, run `sudo apt install python3-venv`.

**`Python was installed but is not usable in this shell yet`**
The install worked, but your current shell's `PATH` has not picked it up (typical on Windows after winget). Open a new terminal and run the same command again.

**The environment is not active after the run**
That is expected when pyp is *run*: a child process cannot change its parent shell. Use the `source ...` line pyp prints, or launch with `source <(curl -fsSL URL) ...` (bash).

**The ikp installed something unexpected**
Use `-n -L <ikp>` and read the Decoded selection. Re-check the calculation with the block method ([section 9.4](#94-the-block-method-best-for-bigger-indices)) and the test vectors ([section 9.8](#98-test-vectors)).

**`ikp points at index N, but this library has only M entries`**
The ikp was made for a newer library than your copy of `pyp.sh` contains. Fetch the latest script.

**macOS: odd syntax errors**
The system Bash is 3.2. Install Bash 4.4+ (`brew install bash`) and make sure `bash` on your `PATH` is the new one.

**A package fails to install**
The summary lists the reason and the path of the full log. Common causes: a typo in the name, no wheel for your Python version, missing system libraries, or no network.
Re-run just that package with `-v` or `--log FILE` for the complete pip output.

**Can I use it with Python 3.7 or older?** No. Python 3.8+ is required; an older system Python is treated as "not usable" and a newer one is installed.

---

## 16. Security

Running a script straight from the internet means trusting its author and the connection.

* Read the script first: `curl -fsSL URL | less`.
* **Pin to a commit.** The plain raw URL always serves the *latest* revision of the gist. For anything shared with others, use the revision-pinned raw URL so later edits cannot change what people run:

```text
https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/<COMMIT_SHA>/pyp.sh
```

* pyp uses `sudo` only to install Python through your package manager, and only after asking (or with `-y`).
* Packages come from the default pip index unless a requirements file sets `--index-url`/`--extra-index-url`; review files you did not write.
* An ikp from an AI should be checked with `-n` before installing (see [section 10](#10-using-pyp-with-an-ai)).

---

## 17. Test status

What has actually been exercised (Ubuntu 24.04, Bash 5.2, Python 3.12):

| Area | Result |
|------|--------|
| Static analysis | `shellcheck` clean, `bash -n` clean |
| IKP decoder | **176 randomized and edge cases** checked against an independent encoder: single indices, every block boundary, 64/65-bit boundaries, the full 170-bit ikp: **0 failures** |
| Invalid input | all error cases in [section 9.7](#97-errors-nothing-is-ever-installed-on-a-bad-ikp-exit-code-2) exit with code 2 and install nothing |
| Real installs | `-L 1` installed `requests`; `-L 0x10d80000000000000000000` installed rich, click, colorama, tqdm and pyyaml; all verified inside the venv |
| Requirements files | CRLF, comments, nested and **remote** `-r`, failing package reported correctly |
| Run modes | `curl \| bash -s --`, `bash <(curl ...)`, `source <(curl ...)` (activation works, no leftovers), prompts via the real terminal, no-terminal error path |
| Ctrl-C | exit 130, partial summary, cursor restored, no orphaned pip process |
| Python bootstrap | missing-Python detection, `--no-install-python`, dry run, no-terminal refusal; real `apt-get update` and install commands executed; the install-then-continue flow and the non-root `sudo` path verified with stubbed `apt-get`/`sudo` |
| Shell detection | fish, zsh and bash parents simulated (a real fish was not available for the test) |

**Not exercised yet:** the install steps for `dnf`, `yum`, `zypper`, `pacman`, `apk`, `brew`, Termux `pkg` and `winget`. They are standard
one-line commands, but they have not been run end to end. Reports are welcome.

---

## 18. Project files

| File | Purpose |
|------|---------|
| `pyp.sh` | the program: Python bootstrap, venv, installer and the numbered library |
| `ikp-prompt.md` | self-contained instructions and the library table for AI assistants |
| `README.md` | this document |

Gist: <https://gist.github.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4>

---

## 19. Maintaining the library

The library is the array `__pyi_LIB` near the top of `pyp.sh`. One line per package, in the form `"pip-name|Category|Description"`.
The **index is the line position, starting at 1**.

> ### Append only: the one rule that must never be broken
>
> Other people store ikps in their projects, and every ikp is just a set of positions in this list.
> **Never insert, reorder or delete a row.** Doing so silently changes what every existing ikp installs.

* **Add a package:** append a new line at the end of the array.
* **Retire a package:** keep the row and start its description with `DEPRECATED`.
* **Bump `PYI_LIB_VERSION`** after appending.
* **Regenerate the table** in `ikp-prompt.md` (and this README) from the script: `bash pyp.sh --list-lib --md`.
* **Validate new names** against PyPI (`https://pypi.org/pypi/<name>/json` must return 200) and keep descriptions free of `|`, `"`, `$` and backticks.
* Older ikps stay valid with newer scripts (indices only ever grow); a **newer** ikp on an **older** script is rejected with a clear message.

---

## 20. Changelog

| Version | Changes |
|---------|---------|
| **3.0.0** | Built-in **library of 170 packages** and the **IKP** system (`-L`, `--list-lib`, `--filter`, `--md`); decoded-selection display; duplicate removal across sources; fish/csh-aware activation hint; `ikp-prompt.md` for AI assistants; pipe form promoted to the default run mode |
| **2.1.0** | **Runs from a GitHub Gist** (`curl \| bash`, `bash <(curl)`, `source <(curl)`); **Python detection and automatic installation** (apt, dnf, yum, zypper, pacman, apk, brew, pkg, winget); `/dev/tty` prompts; remote `-r URL` requirements |
| **2.0.0** | Full rewrite of the original script: colors, spinner and overall progress bar, success/failure report, a real CLI (`-r -e -p -y -u -R -f -n -l -q -v` ...), keep-going on failures, `--freeze`, `--report`, Ctrl-C handling, proper exit codes |
| **1.x** | Original script: virtualenv creation and sequential install with a simulated dual progress bar |

Fixes made when moving from 1.x to 2.0: the empty progress bar that printed one block, the `venv` exit code being lost behind a
`printf`, byte-based padding that misaligned the bars, requirements lines with inline comments or options being passed to pip as one bad
argument, the abort on the first failure, missing Ctrl-C handling, hard-coded `python`, an activation that could never persist, and
helper functions and variables leaking into the user's shell when sourced.

---

## 21. Credits and license

**Developer: HUNTER ZEN RU**
GitHub: [hunter-zen-RU](https://gist.github.com/hunter-zen-RU)
Project home: <https://gist.github.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4>

No license file is included yet; add a `LICENSE` file to state how others may use and share pyp.

<div align="center">

*pyp: one command from nothing to a working Python environment.*

</div>
