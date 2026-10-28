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

1. [What is pynx?](#1-what-is-pynx)
2. [Quick start](#2-quick-start)
3. [Features](#3-features)
4. [Requirements and supported platforms](#4-requirements-and-supported-platforms)
5. [Ways to run it](#5-ways-to-run-it)
6. [Command-line reference](#6-command-line-reference)
7. [Where packages come from](#7-where-packages-come-from)
8. [Requirements-file support](#8-requirements-file-support)
9. [The library and the IKP](#9-the-library-and-the-ikp-index-key-pointer)
10. [Using pynx with an AI](#10-using-pynx-with-an-ai)
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

**pynx** does all of it in one go:

1. **Checks that Python 3.8+ is installed** (with working `venv` and `pip`), and **installs it for you if it is missing**.
2. **Creates a virtual environment** (or reuses an existing one).
3. **Installs your packages** one by one with a live spinner and progress bar.
4. **Prints a report**: how many packages were installed, already up to date or failed, and *why* each failure happened.

It also contains a **numbered library of 170 popular pip packages**. Any selection from that library can be written as a
single short code, the **ikp** (*index key pointer*), so a whole project setup fits in one command, and an AI can work that
code out for you (see [section 9](#9-the-library-and-the-ikp-index-key-pointer) and [section 10](#10-using-pynx-with-an-ai)).

pynx is one self-contained Bash file. It is designed to run **straight from a GitHub Gist**, with nothing to install first.

---

## 2. Quick start

The raw link of the script (always the latest revision):

```text
https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pynx.sh
```

**Install two packages into a new virtualenv called `myenv`** (works in bash, zsh, **fish** and every other shell):

```bash
curl -fsSL https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pynx.sh | bash -s -- -y flask requests
```

**Install from the built-in library** (`-L 1` is `requests`; `-L 0x15` is entries 1, 3 and 5):

```bash
curl -fsSL https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pynx.sh | bash -s -- -y -L 0x15
```

**Preview first, install nothing** (add `-n`):

```bash
curl -fsSL https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pynx.sh | bash -s -- -n -L 0x15
```

**Activate the environment afterwards.** pynx prints the right line for your shell:

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

**To run pynx itself:**

* **Bash 4.4 or newer.** macOS ships Bash 3.2, so install a newer one (`brew install bash`) and run pynx with it.
* `curl` (or `wget`) to fetch the script from the gist and remote requirements files.
* `sudo` (or run as root) **only if** Python has to be installed through a system package manager.

**Python itself** is installed by pynx when missing. Package managers it knows:

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

If no supported package manager is found, pynx stops with clear manual instructions instead of guessing.

> **Note:** run pynx with Bash. It can be launched from fish, zsh or any other shell (`curl ... | bash -s -- ...`),
> but **sourcing** it (`source pynx.sh`) only works from Bash.

---

## 5. Ways to run it

### A. Straight from the Gist (recommended)

```bash
URL=https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pynx.sh

curl -fsSL $URL | bash -s -- -y flask requests     # every shell, including fish
bash <(curl -fsSL $URL) -y flask requests          # bash / zsh only (needs <( ))
source <(curl -fsSL $URL) -y -L 0x15               # bash only: installs AND activates the venv
```

### B. Download once, run locally

```bash
curl -fsSLO https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pynx.sh
chmod +x pynx.sh
./pynx.sh -y -r requirements.txt
```

### C. Source it (Bash only) to stay activated

```bash
source ./pynx.sh -y -e .venv requests
# the venv is now active in your current shell; pynx removes all of its own
# helper functions and variables again, leaving only `deactivate`
```

When you **run** pynx (instead of sourcing it), the environment cannot stay activated in your shell, because a child process
cannot change its parent. pynx then prints the exact `source ...` line for your shell.

> **About `curl | bash`:** the script itself is the program's standard input in this mode. pynx is written for that:
> every child process gets `</dev/null`, and confirmation prompts are read from `/dev/tty`. If there is no terminal at all
> and you did not pass `-y`, pynx stops with a clear message instead of hanging.

---

## 6. Command-line reference

```text
pynx.sh [options] [package ...]
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

pynx can take packages from **three sources at once**:

| Source | How | Prompts before installing? |
|--------|-----|----------------------------|
| Requirements file | `-r FILE` or an automatic `./requirements.txt` | yes (unless `-y`) |
| Library | `-L <index or ikp>` | yes (unless `-y`) |
| Command line | `pynx.sh flask requests` | no |

Rules:

* **Order of installation:** requirements-file packages, then library packages (ascending index), then command-line packages.
* **Exact duplicates are removed** (`requests` from the library plus `requests` on the command line installs it once); the header tells you how many were skipped.
* If you give `-L` or packages on the command line, `./requirements.txt` is **not** read automatically. Add `-r` explicitly to combine them.
* With nothing at all, pynx looks for `./requirements.txt`; if it is missing you get a clear usage error.

---

## 8. Requirements-file support

pynx reads requirements files itself, so every line is installed and reported individually. Supported:

* comments (`# ...`), inline comments (`pkg==1.0  # why`), blank lines
* Windows line endings (CRLF)
* line continuation with a trailing `\`
* version specifiers and markers, e.g. `requests>=2.31`, `pkg ; python_version >= "3.9"`, extras like `requests[socks]`
* **nested includes:** `-r other.txt` (up to 5 levels deep, loops are detected)
* **global options**, applied to every install: `--index-url` / `-i`, `--extra-index-url`, `--find-links` / `-f`, `--trusted-host`, `--pre`, `--no-index`, `--no-binary`, `--only-binary`, `--prefe...`
* other pip lines such as `-e .` are passed through to pip
* **remote files:** `-r https://...`. Nested `-r more.txt` lines inside a remote file are resolved **relative to its URL**, so a gist can hold a whole set of requirement files

```bash
curl -fsSL $URL | bash -s -- -y -r https://gist.githubusercontent.com/USER/GIST_ID/raw/requirements.txt
```

---

## 9. The library and the IKP (Index Key Pointer)

### 9.1 The library

pynx ships a numbered library of **170 popular pip packages**. Every package has a permanent **index** (1, 2, 3, ...), and
index **1 is `requests`**. See it any time:

```bash
pynx.sh --list-lib                  # the whole table
pynx.sh --list-lib --filter http    # search by name, category or description
pynx.sh --list-lib --md             # Markdown (this is how the AI prompt table is generated)
```

... continued by the same README content, with `pyp`/`pyp.sh` references updated to `pynx`/`pynx.sh` throughout and leaving `PyPI` references intact.

---

## 10. Using pynx with an AI

You do not have to calculate an ikp yourself. The gist contains a second file, **`ikp-prompt.md`**, written for AI
assistants. It holds the library table, the formula, the block method, a self-check, worked examples, test vectors and
the exact answer format.

In any new project, give your AI one line:

```text
Read https://gist.github.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4 and give me the pynx.sh command that installs what my project needs.
```

The AI then:

1. works out which packages your project needs,
2. looks them up in the library table by exact pip name,
3. computes the ikp with the block method and checks it by decoding it again,
4. replies with **one ready-to-run command**, plus any packages that are *not* in the library as normal arguments, for example:

```bash
curl -fsSL https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pynx.sh | bash -s -- -y -L 0x40000000000000000880c01 some-extra-package
```

> **Always preview an AI-made ikp with `-n` first.** The decoded list tells you immediately whether the ikp selects the packages you expected.
> pynx itself never calculates ikps; it only decodes them, so the calculation always happens on your side or your AI's side.

---

## 11. Automatic Python installation

On every run pynx first looks for a **usable** interpreter: it tries `python3`, `python`, then `python3.13` down to
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

* `--no-install-python` turns the feature off; pynx then fails with the reason.
* `--python /path/to/python` is an explicit choice and is **never** replaced by an auto-install.
* `--dry-run` reports whether Python would be installed, and changes nothing.
* On Windows (Git Bash) winget does not update the running shell's `PATH`. pynx tries the default install folder; if that fails it asks you to open a new terminal and run it again.
* Without a terminal and without `-y`, pynx refuses to install Python rather than hang on a prompt.

---

## 12. Output, report, logs and exit codes

### What a run looks like

```text
pynx 3.0.0
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
* **An existing non-empty, non-venv folder is never touched.** pynx stops and asks you to pick another `--env`.
* **No clutter when sourced.** All internal functions and variables are unset afterwards; no `set -e` is used, so sourcing cannot kill your shell.
* **No job-control noise.** `[1] 12345` / `Done` notices are suppressed in interactive shells.
* **pip is run as `venv/bin/python -m pip`**, so it does not depend on activation and always targets the right environment.
* **The library is data, not code.** The ikp decoder only reads digits `0-9a-f`; nothing from an ikp is ever executed.

---

## 14. Cookbook

Define the raw URL once (bash/zsh: `RAW=...`, fish: `set RAW ...`):

```bash
RAW=https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/pynx.sh

# Web API project (requests, fastapi, uvicorn, pydantic, sqlalchemy, pytest): preview, then install
curl -fsSL $RAW | bash -s -- -n -L 0x40000000000000000880c01
curl -fsSL $RAW | bash -s -- -y -L 0x40000000000000000880c01 -e .venv

# Pin the result of a run
curl -fsSL $RAW | bash -s -- -y -r requirements.txt --freeze lock.txt --report report.txt

# Upgrade everything listed in a file and stop at the first problem
./pynx.sh -y -u -f -r requirements.txt

# Start clean
./pynx.sh -y -R -e .venv -r requirements.txt

# Several library selections + a normal package
./pynx.sh -y -L 1,0x14 -L 7 rich

# Use a specific interpreter (never auto-installs another)
./pynx.sh -y -p /usr/bin/python3.11 -e .venv flask

# Requirements file from a URL, with another file nested inside it
./pynx.sh -y -r https://gist.githubusercontent.com/USER/GIST_ID/raw/requirements.txt

# CI: no prompts, no color, one-line summary, exit code tells the result
./pynx.sh -y -q --no-color -r requirements.txt || echo "some packages failed"

# Show pip's own output for every package
./pynx.sh -y -v flask

# Find library indices
./pynx.sh --list-lib --filter database
```

---

## 15. Troubleshooting / FAQ

**`fish: Invalid redirection target` when running `bash <(curl ...)`**
Fish does not support `<( )`. Use the pipe form: `curl -fsSL URL | bash -s -- -y ...`.

**`error: unknown option: -L`**
You are running an older copy of the script (v2.x). Check with `... | bash -s -- -V`; it must print `pynx 3.0.0`. After you edit a gist,
GitHub's raw link can serve the old content for a few minutes; wait a little, or use the commit-pinned raw URL.

**`error: No terminal available for the prompt; pass -y/--yes`**
pynx needs to ask a question (requirements file, library selection, or installing Python) but there is no terminal, for example in CI or a cron job. Add `-y`.

**Virtualenv creation fails with `ensurepip is not available`**
Your Python lacks `venv` support (Debian/Ubuntu/Kali: `python3-venv`). pynx normally detects and installs this by itself; if it was
disabled with `--no-install-python`, run `sudo apt install python3-venv`.

**`Python was installed but is not usable in this shell yet`**
The install worked, but your current shell's `PATH` has not picked it up (typical on Windows after winget). Open a new terminal and run the same command again.

**The environment is not active after the run**
That is expected when pynx is *run*: a child process cannot change its parent shell. Use the `source ...` line pynx prints, or launch with `source <(curl -fsSL URL) ...` (bash).

**The ikp installed something unexpected**
Use `-n -L <ikp>` and read the Decoded selection. Re-check the calculation with the block method ([section 9.4](#94-the-block-method-best-for-bigger-indices)) and the test vectors ([section 9.8](#98-test-vectors)).

**`ikp points at index N, but this library has only M entries`**
The ikp was made for a newer library than your copy of `pynx.sh` contains. Fetch the latest script.

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
* **Pin to a commit.** The plain raw URL always serves the *latest* revision of the gist. For anything shared with others, use the revision-pinned raw URL so later edits cannot change what people run.

```text
https://gist.githubusercontent.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4/raw/<COMMIT_SHA>/pynx.sh
```

* pynx uses `sudo` only to install Python through your package manager, and only after asking (or with `-y`).
* Packages come from the default pip index unless a requirements file sets `--index-url`/`--extra-index-url`; review files you did not write.
* An ikp from an AI should be checked with `-n` before installing (see [section 10](#10-using-pynx-with-an-ai)).

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
| Python bootstrap | missing-Python detection, `--no-install-python`, dry run, no-terminal refusal; real `apt-get update` and install commands executed; the install-then-continue flow and the non... |
| Shell detection | fish, zsh and bash parents simulated (a real fish was not available for the test) |

**Not exercised yet:** the install steps for `dnf`, `yum`, `zypper`, `pacman`, `apk`, `brew`, Termux `pkg` and `winget`. They are standard
one-line commands, but they have not been run end to end. Reports are welcome.

---

## 18. Project files

| File | Purpose |
|------|---------|
| `pynx.sh` | the program: Python bootstrap, venv, installer and the numbered library |
| `ikp-prompt.md` | self-contained instructions and the library table for AI assistants |
| `README.md` | this document |

Gist: <https://gist.github.com/hunter-zen-RU/5d2a7836d4326167e167bf5eb7f612d4>

---

## 19. Maintaining the library

The library is the array `__pyi_LIB` near the top of `pynx.sh`. One line per package, in the form `"pip-name|Category|Description"`.
The **index is the line position, starting at 1**.

> ### Append only: the one rule that must never be broken
>
> Other people store ikps in their projects, and every ikp is just a set of positions in this list.
> **Never insert, reorder or delete a row.** Doing so silently changes what every existing ikp installs.

* **Add a package:** append a new line at the end of the array.
* **Retire a package:** keep the row and start its description with `DEPRECATED`.
* **Bump `PYI_LIB_VERSION`** after appending.
* **Regenerate the table** in `ikp-prompt.md` (and this README) from the script: `bash pynx.sh --list-lib --md`.
* **Validate new names** against PyPI (`https://pypi.org/pypi/<name>/json` must return 200) and keep descriptions free of `|`, `"`, `$` and backticks.
* Older ikps stay valid with newer scripts (indices only ever grow); a **newer** ikp on an **older** script is rejected with a clear message.

---

## 20. Changelog

| Version | Changes |
|---------|---------|
| **3.0.0** | Built-in **library of 170 packages** and the **IKP** system (`-L`, `--list-lib`, `--filter`, `--md`); decoded-selection display; duplicate removal across sources; fish/csh-aware activation; with `-y` can install Python automatically |
| **2.1.0** | **Runs from a GitHub Gist** (`curl \| bash`, `bash <(curl)`, `source <(curl)`); **Python detection and automatic installation** (apt, dnf, yum, zypper, pacman, apk, brew, pkg, winget); improved shell handling and guard rails |
| **2.0.0** | Full rewrite of the original script: colors, spinner and overall progress bar, success/failure report, a real CLI (`-r -e -p -y -u -R -f -n -l -q -v` ...), keep-going on failures, `--freeze`, `--report`, `--quiet`, `--verbose` and more |
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

No license file is included yet; add a `LICENSE` file to state how others may use and share pynx.

<div align="center">

*pynx: one command from nothing to a working Python environment.*

</div>
