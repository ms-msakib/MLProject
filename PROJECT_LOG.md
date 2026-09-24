# Project Log — Section 48 ML Project End to End

This document tracks every command run in this project and, for any error encountered, records what the error meant and how it was resolved (so the same issue isn't repeated later).

---

## Step 1 — Create a conda virtual environment

**Command:**
```powershell
conda create -p venv python==3.8 -y
```

**Status:** ❌ Failed

**Error:**
```
conda : The term 'conda' is not recognized as the name of a cmdlet, function, script file, or operable program.
Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
CategoryInfo          : ObjectNotFound: (conda:String) [], CommandNotFoundException
FullyQualifiedErrorId : CommandNotFoundException
```

### 1. What the error says and about which command
PowerShell could not find an executable/cmdlet named `conda` for the `conda create -p venv python==3.8 -y` command. This means the shell has no program called `conda` reachable via the `PATH` environment variable in the current session.

### 2. Resolution & prevention
**Root cause found:** Conda (Miniconda/Anaconda) is not actually installed on this machine. There were stale `PATH` entries pointing to `C:\Users\Midnight Sun\miniconda3\...`, but that folder does not exist on disk — leftover configuration from a prior/removed install, not a working one.

**Resolution:**
1. Install Miniconda from https://docs.conda.io/en/latest/miniconda.html (Windows 64-bit installer).
2. During install, on "Advanced Options", check **"Add Miniconda3 to PATH environment variable"**.
3. Close and reopen the terminal (PATH changes require a new shell session).
4. Verify with `conda --version`.
5. If PowerShell still doesn't recognize it, run once: `conda init powershell`, then restart the terminal.

**How to avoid this in future:**
- Always verify a tool is installed and on PATH (`conda --version`, `python --version`, etc.) before running project setup commands in a new environment/machine.
- After installing any tool that modifies PATH, always open a **new** terminal window — existing sessions won't see the update.
- Prefer the "Anaconda Prompt" / "Anaconda PowerShell Prompt" shortcuts if PATH issues persist, since those initialize the environment automatically without relying on system PATH.

---

## Step 2 — Installing Miniconda (GUI installer)

**Action:** Running the Miniconda3 Windows installer, choosing install location.

**Status:** ⚠️ Warning encountered (caught before completing install)

**Warning shown by installer:**
```
Warning: 'Destination Folder' contains 1 space.
This can cause problems with several conda packages.
Please consider removing the space.
```

### 1. What the error says and about which command
The installer's default destination was `C:\Users\Midnight Sun\miniconda3`. Because the Windows username is "Midnight Sun" (contains a space), the full install path contains a space. Several conda/pip packages — especially ones with compiled/native (C/C++) components — can fail to build or install correctly when there is a space anywhere in the install path.

### 2. Resolution & prevention
**Resolution:** Changed the installer's destination folder (via "Browse...") to a space-free path at the root of the drive:
```
C:\Miniconda3
```

**How to avoid this in future:**
- On Windows, always install conda/Python/dev tools to a path with no spaces (avoid installing under a username that contains a space, e.g. prefer `C:\Miniconda3`, `C:\Python3xx`, etc.).
- When any installer warns about spaces or special characters in the path, don't dismiss it by default — change the path instead, since these warnings map to real, hard-to-debug package build failures later.

---

## Step 3 — Verify the Miniconda installation

**Status:** ✅ Verified (conda 26.7.1, installed at `C:\Miniconda3`)

**Commands (run in a NEW PowerShell window, so the updated PATH is loaded):**
```powershell
conda --version          # expected: conda 26.7.1
conda info --base        # expected: C:\Miniconda3
Get-Command conda        # shows where conda is resolved from
python --version         # expected: Python 3.14.x (Miniconda base)
Get-Command python       # expected: C:\Miniconda3\python.exe (not the WindowsApps stub)
Test-Path C:\Miniconda3\Scripts\conda.exe   # expected: True
```

**Results observed:**
| Check | Result |
|---|---|
| `conda --version` | `conda 26.7.1` |
| `conda info --base` | `C:\Miniconda3` |
| `Get-Command conda` | `C:\Miniconda3\Library\bin\conda.bat` |
| `python --version` | `Python 3.14.7` |
| `Get-Command python` | `C:\Miniconda3\python.exe` |
| `conda.exe` exists | `True` |

**Note:** An already-open terminal will not see the new PATH until it is closed and reopened. If `conda` is "not recognized" again, open a new terminal; if still failing, run `conda init powershell` and reopen it.

---

## Step 4 — Push to GitHub with a different account

**Command:**
```powershell
git push -u origin main
```

**Status:** ❌ Failed

**Error:**
```
remote: Permission to ms-msakib/MLProject.git denied to Doon-Paints-and-Hardware.
fatal: unable to access 'https://github.com/ms-msakib/MLProject.git/': The requested URL returned error: 403
```

### 1. What the error says and about which command
`git push -u origin main` reached GitHub, but GitHub authenticated the push as the user `Doon-Paints-and-Hardware`, which has no write permission on the repo `ms-msakib/MLProject`. HTTP 403 = authenticated but not authorized. The repo owner is `ms-msakib`, so the wrong account was used.

### 2. Resolution & prevention
**Root cause found:** Windows Credential Manager has a saved GitHub login (`git:https://github.com`, user `Doon-Paints-and-Hardware`). Git reuses it automatically for every github.com HTTPS push, so it never asks for the `ms-msakib` login. (The global git `user.name`/`user.email` are already `ms-msakib`, but those only label commits; they do not control login.)

**Resolution:**
1. Remove the cached credential (either way):
   - PowerShell: `cmdkey /delete:LegacyGeneric:target=git:https://github.com`
   - or GUI: Start menu → *Credential Manager* → *Windows Credentials* → delete `git:https://github.com`.
2. Run `git push -u origin main` again. A GitHub sign-in window/prompt appears; sign in as `ms-msakib` (a browser login, or a Personal Access Token as the password if it asks for one).
3. Git saves the new credential, so later pushes work without prompting.

**How to avoid this in future:**
- When switching GitHub accounts on HTTPS, clear the old `git:https://github.com` credential first.
- To keep two accounts side by side, embed the username in the remote URL so each repo picks its own login: `git remote set-url origin https://ms-msakib@github.com/ms-msakib/MLProject.git`
- Make sure the account you sign in with is the repo owner or a collaborator.

---

## Step 5 — Run the logger module

**Command:**
```powershell
python src/logger.py
```

**Status:** ❌ Failed → ✅ Fixed

**Error:**
```
FileNotFoundError: [Errno 2] No such file or directory: 'C:\\Projects\\ML Project\\Section 48 ML Project end to end\\logs\\09-24-2026_13-31-28.log\\09-24-2026_13-31-28.log'
```

### 1. What the error says and about which command
`logging.basicConfig(filename=LOG_FILE_PATH, ...)` in `src/logger.py` tried to open a log file, but the path was `logs\<timestamp>.log\<timestamp>.log`, i.e. the log file name appeared twice, and the parent "directory" `<timestamp>.log` did not exist.

### 2. Resolution & prevention
**Root cause found:** `logs_path` was built as `os.path.join(os.getcwd(), "logs", LOG_FILE)` (already includes the file name), then `LOG_FILE_PATH = os.path.join(logs_path, LOG_FILE)` appended the file name again. `os.makedirs(os.path.dirname(logs_path))` only created `logs\`, not the `<timestamp>.log` folder the final path required.

**Resolution:** Make `logs_path` the directory only and create it:
```python
logs_path = os.path.join(os.getcwd(), "logs")
os.makedirs(logs_path, exist_ok=True)
LOG_FILE_PATH = os.path.join(logs_path, LOG_FILE)
```

**How to avoid this in future:**
- Keep "directory" and "file path" variables separate; only join the file name once.
- `os.makedirs` should be called on the directory itself, not on a path that includes the file name.
- Note: the log goes under `os.getcwd()`, so `logs/` is created wherever the script is run from (run from the project root for consistency).

---

<!-- Add new steps below in the same format: Command → Status → Error (if any) → (1) What it means (2) Resolution & prevention -->
