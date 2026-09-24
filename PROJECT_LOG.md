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

## Step 6 — Run the custom exception module

**Command:**
```powershell
python src/exception.py
```

**Status:** ❌ Failed → ✅ Fixed (syntax error; a second latent error found and fixed)

**Error:**
```
File "...\src\exception.py", line 11
    return error_message
    ^^^^^^^^^^^^^^^^^^^^
SyntaxError: 'return' outside function
```

### 1. What the error says and about which command
Python refused to even start `src/exception.py`: the `return error_message` line at the end of `error_message_detail()` was not indented, so it sat at module level, where `return` is illegal.

### 2. Resolution & prevention
**Root cause found:** Missing 4-space indentation on `return error_message`, so it fell outside the function body.

**Resolution:** Indented it to sit inside `error_message_detail()`.

**Second issue caught while fixing (would have hit next):** the `__main__` block calls `logging.info(...)` but `logging` was never imported → `NameError`. Added `import logging`. Note: this plain import is not configured, so that message is not written to the log file. To write to the log file, import the configured logger instead (e.g. once the project is run as a package: `from src.logger import logging`, run with `python -m src.exception` from the project root).

**How to avoid this in future:**
- A `SyntaxError: 'return' outside function` almost always means indentation; check that the line is indented under its `def`.
- Use an editor with indentation guides / auto-format so misaligned blocks are visible.
- Re-read the whole file for undefined names (missing imports) after fixing a syntax error, since Python only reports one error at a time.

---

## Step 7 — Re-run the custom exception module (verification)

**Command:**
```powershell
python src/exception.py
```

**Status:** ✅ Working as intended (this is the expected output, not a bug)

**Output:**
```
ZeroDivisionError: division by zero

During handling of the above exception, another exception occurred:

CustomException: Error occured in python script name[...\src\exception.py] line number [26] error message[division by zero]
```

### 1. What the output means
The `__main__` block deliberately runs `1/0` to test the exception class. The `ZeroDivisionError` is caught and re-raised as `CustomException`, whose message shows the script name, the line number (26, where `1/0` happened) and the original error text. The "During handling of the above exception..." wording is Python's normal chaining of the two exceptions.

### 2. Notes
- Nothing to fix. `CustomException` correctly reports the file, line and message.
- The `logging.info("Divide by zero error")` call uses the plain, unconfigured `logging` module, so it is not written to the log file (see Step 6 for how to route it to `src/logger.py`).

---

## Step 8 — Route exception logging to the configured logger

**Change:** In `src/exception.py`, replaced `import logging` with `from src.logger import logging`, so `logging.info(...)` writes to the timestamped file in `logs/`.

**Command (run from the project root):**
```powershell
python -m src.exception
```

**Note:** `python src/exception.py` no longer works, because `src` is not on the import path when the file is run directly (`ModuleNotFoundError: No module named 'src'`). Use `python -m src.exception` from the project root instead.

---

## Step 9 — Fix "ipykernel is required" popup in the EDA notebook

**Problem:** Running the first cell of `notebook/1 . EDA STUDENT PERFORMANCE .ipynb` showed a popup saying `ipykernel` is required. The selected kernel was the project's `venv` (Python 3.8.0), which did not have `ipykernel` installed.

**Command (run from the project root):**
```powershell
.\venv\python.exe -m pip install ipykernel
```

**Status:** ✅ Installed `ipykernel 6.29.5` (plus dependencies such as `ipython 8.12.3`, `jupyter-client 8.6.3`, `debugpy`, `pyzmq`, `tornado`).

### 1. What it means
The `venv` folder is laid out like a conda environment: `python.exe` is in the folder root, and there is a `conda-meta` folder. There is no `venv/Scripts/python.exe`, so the interpreter path is `venv\python.exe`. VS Code needs `ipykernel` in the selected interpreter to start a notebook kernel.

### 2. Resolution & prevention
- Reload the VS Code window (`Ctrl+Shift+P` → "Developer: Reload Window").
- Click **Select Kernel** → **Python Environments…** → pick the venv (Python 3.8.0), then re-run the cell.
- The pip warnings about scripts not being on PATH are harmless.
- Add `ipykernel` to `requirement.txt` if the notebook should work on a fresh setup.

---

## Step 10 — `ModuleNotFoundError: No module named 'numpy'` on `import numpy as np`

**Problem:** After fixing the kernel (Step 9), running the first cell of `notebook/1 . EDA STUDENT PERFORMANCE .ipynb` (`import numpy as np`) failed because `numpy` was not installed in the `venv` interpreter.

**Command (run from the project root):**
```powershell
.\venv\python.exe -m pip install pandas numpy seaborn matplotlib
```

**Status:** ✅ Installed `numpy 1.24.4`, `pandas 2.0.3`, `seaborn 0.13.2`, `matplotlib 3.7.5` (plus dependencies: `contourpy`, `cycler`, `fonttools`, `kiwisolver`, `pillow`, `pyparsing`, `pytz`, `tzdata`, `importlib-resources`).

### 1. What it means
`requirement.txt` lists `pandas`, `numpy`, `seaborn`, `matplotlib`, `-e .`, but they had never been installed into this `venv` — only `ipykernel` (Step 9) was present. The venv was effectively empty of the project's data-science dependencies.

### 2. Resolution & prevention
- Verified with `venv\python.exe -c "import numpy, pandas, seaborn, matplotlib"` — imports succeed.
- To install everything in `requirement.txt` (including the local package via `-e .`) in one go instead of picking packages by hand:
  ```powershell
  .\venv\python.exe -m pip install -r requirement.txt
  ```
- Re-run the notebook cells after this; the kernel must still be set to the `venv` (Python 3.8.0) interpreter.

---

## Step 11 — `ModuleNotFoundError: No module named 'sklearn'` in MODEL TRAINING.ipynb

**Problem:** Running the first cell of `notebook/2. MODEL TRAINING.ipynb` failed on `from sklearn.metrics import mean_squared_error, r2_score`. The same cell also imports `catboost` and `xgboost`, neither of which is listed in `requirement.txt` (only `pandas`, `numpy`, `seaborn`, `matplotlib`, `-e .` are).

**Command (run from the project root):**
```powershell
.\venv\python.exe -m pip install --default-timeout=300 scikit-learn catboost xgboost
```

**Status:** ✅ Installed `scikit-learn 1.3.2`, `catboost 1.2.10`, `xgboost 2.1.4`.

### 1. What it means
`requirement.txt` never listed `scikit-learn`, `catboost`, or `xgboost`, so they had never been installed in this `venv`. A first plain `pip install scikit-learn catboost xgboost` attempt failed with `ReadTimeoutError: HTTPSConnectionPool(host='files.pythonhosted.org', port=443): Read timed out.` — `catboost`'s wheel is large (~100+ MB) and pip's default download timeout (15s) was too short for the connection speed at the time.

### 2. Resolution & prevention
- Retried with `--default-timeout=300` (5 minutes), which succeeded.
- Verified with `venv\python.exe -c "import sklearn, catboost, xgboost"` — imports succeed.
- `requirement.txt` should be updated to include `scikit-learn`, `catboost`, and `xgboost` so a fresh `pip install -r requirement.txt` sets up everything this project's notebooks need.
- If a future install times out again, retry with `--default-timeout=<seconds>` rather than assuming the package is broken.

---

<!-- Add new steps below in the same format: Command → Status → Error (if any) → (1) What it means (2) Resolution & prevention -->
