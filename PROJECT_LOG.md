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

## Step 12 — Run data ingestion + data transformation

**Command (run from the project root):**
```powershell
python src/components/data_ingestion.py
```

**Status:** ❌ Failed → ✅ Fixed (three errors, each found only after the previous one was fixed)

**Error (as first reported):**
```
File "...\src\components\data_ingestion.py", line 57, in <module>
    datatransformation.initiate_data_transformation(train_data,test_data)
AttributeError: 'DataTransformation' object has no attribute 'initiate_data_transformation'
```

### 1. What the error says and about which command
Data ingestion finished (it wrote `train.csv`, `test.csv`, `data.csv`). The script then called `DataTransformation().initiate_data_transformation(...)`, but Python found no method with that name on the `DataTransformation` class.

### 2. Resolution & prevention
**Root cause found:** In `src/components/data_transformation.py`, `def initiate_data_transformation(self, ...)` started at column 0, not indented under `class DataTransformation:`. Python therefore treated it as a separate module-level function, not a method of the class.

**Resolution:** Indented the whole `initiate_data_transformation` block by 4 spaces so it is inside the class.

**Second error (hit on the next run):**
```
ValueError: Cannot specify both 'axis' and 'index'/'columns'
```
`train_df.drop(columns=[target_column_name], axis=1)` passes the axis twice: `columns=` already means "drop columns", and pandas does not accept it together with `axis=`. Fix: `train_df.drop(columns=[target_column_name])` (same for `test_df`).

**Third error (hit on the run after that):**
```
NameError: name 'pickle' is not defined
```
`save_object()` in `src/utils.py` called `pickle.dump(...)`, but the file imports `dill`, not `pickle`. Fix: changed it to `dill.dump(obj, file_obj)`.

**Result:** `python src/components/data_ingestion.py` exits without errors and creates `artifacts/train.csv`, `artifacts/test.csv`, `artifacts/data.csv` and `artifacts/preprocessor.pkl`.

**How to avoid this in future:**
- An `AttributeError: '<Class>' object has no attribute '<method>'` for a method you did write usually means indentation. Check that the `def` is indented inside the class.
- Use either `df.drop(columns=[...])` or `df.drop([...], axis=1)`, never both.
- Check that every module used in a file (`pickle`, `dill`, etc.) is actually imported. Python only reports these errors when the line runs.
- Note: plain `python` here resolves to the Miniconda base interpreter (`C:\Miniconda3`), not `venv\python.exe`. To use the project's venv, run `.\venv\python.exe -m src.components.data_ingestion` from the project root (see Step 13; running the file path directly with the venv fails with `No module named 'src'`).

---

## Step 13 — Run the full pipeline with the new model trainer

**Command (as run):**
```powershell
python src/components/data_ingestion.py
```

**Status:** ❌ Failed → ✅ Fixed (import error, four bugs in the new model trainer code, then missing packages in the venv)

**Error:**
```
File "...\src\components\data_ingestion.py", line 14, in <module>
    from src.components.model_trainer import ModelTrainerConfig, ModelTrainer
File "...\src\components\model_trainer.py", line 5, in <module>
    from dataclass import dataclass
ModuleNotFoundError: No module named 'dataclass'
```

### 1. What the error says and about which command
`data_ingestion.py` now imports `src/components/model_trainer.py`. The first line of that file to fail was `from dataclass import dataclass`. Python has no module called `dataclass`; the standard-library module is `dataclasses` (with an **s**). The script stopped while importing, before any data was read, so that run's log file in `logs/` is empty.

### 2. Resolution & prevention
**Fixes in `src/components/model_trainer.py`:**

| Line | Was | Now | Why it would fail |
|---|---|---|---|
| 5 | `from dataclass import dataclass` | `from dataclasses import dataclass` | Module name is `dataclasses` → `ModuleNotFoundError` |
| 27 | `path.join("artifacts", "model.pkl")` | `os.path.join(...)` | `path` is not defined → `NameError` |
| 54 | `evaluate_model(...)` | `evaluate_models(...)` | Function in `src/utils.py` is named `evaluate_models` → `NameError` |
| 34 | `initiate_model_trainer(self, train_array, test_array, preprocessor_path)` | `initiate_model_trainer(self, train_array, test_array)` | `data_ingestion.py` calls it with 2 arguments and the parameter was unused → `TypeError` |
| 79 | `r2_score = r2_score(y_test, predicted)` | `r2_square = r2_score(y_test, predicted)` | Assigning to `r2_score` inside the function makes it a local variable, so the call happens before it has a value → `UnboundLocalError` |

**Environment issues found while re-running:**
- `ModuleNotFoundError: No module named 'catboost'` → plain `python` is the Miniconda base interpreter, which does not have `catboost`. `catboost` was installed into `venv` in Step 11, so run the project with the venv instead.
- The venv was missing `dill` (used by `save_object` in `src/utils.py`). Installed with `.\venv\python.exe -m pip install dill` (`dill 0.4.0`).
- `.\venv\python.exe src/components/data_ingestion.py` → `ModuleNotFoundError: No module named 'src'`. Plain `python` found `src` only because the project is installed with `-e .` in Miniconda; the venv does not have that. Running it as a module from the project root fixes this (same approach as Step 8).
- Added `catboost` to `requirements.txt`, since `model_trainer.py` imports it.

**Command to use from now on (from the project root):**
```powershell
.\venv\python.exe -m src.components.data_ingestion
```

**Result:** ✅ Pipeline ran end to end. Output: `0.8795158595242263` (R² on the test set). Best model: `LinearRegression()`. Created `artifacts/model.pkl` alongside `train.csv`, `test.csv`, `data.csv`, `preprocessor.pkl`.

**Harmless warning in the output:** `UserWarning: Could not find the number of physical cores ... Returning the number of logical cores instead.` (from joblib, used by scikit-learn). It is only a warning, and the run still exits successfully. To silence it: `$env:LOKY_MAX_CPU_COUNT = "4"` before running.

**How to avoid this in future:**
- Standard-library module names must be exact: `dataclasses`, not `dataclass`.
- Never reuse an imported function's name as a variable (`r2_score = r2_score(...)`); pick a different name.
- When calling a helper from another file, check its exact name and parameters (`evaluate_models` in `utils.py`), and keep a method's parameters in step with the places that call it.
- Always run with the same interpreter the packages were installed into (`.\venv\python.exe`), and add every new import (`catboost`, `dill`, …) to `requirements.txt`.
- Known remaining issue: `raise CustomException("No best model found")` (line 67) is missing the `sys` argument, so it would itself error if no model scored ≥ 0.6. It does not trigger now (best score 0.88).

---

## Step 14 — `No module named 'catboost'` with plain `python` (Miniconda base)

**Command:**
```powershell
python src/components/data_ingestion.py
```

**Status:** ❌ Failed → ✅ Fixed

**Error:**
```
File "...\src\components\model_trainer.py", line 7, in <module>
    from catboost import CatBoostRegressor
ModuleNotFoundError: No module named 'catboost'
```

### 1. What the error says and about which command
Plain `python` is the Miniconda base interpreter (`C:\Miniconda3\python.exe`, Python 3.14.7), not the project's `venv`. `catboost` was installed only in the `venv` (Step 11), so importing `model_trainer.py` failed under Miniconda base. The code itself was fine: the venv command from Step 13 worked.

### 2. Resolution & prevention
**Resolution:** Installed `catboost` into Miniconda base:
```powershell
python -m pip install --default-timeout=300 catboost
```
Installed `catboost 1.2.10` (plus `plotly 7.1.0`, `graphviz 0.21`).

**Result:** ✅ `python src/components/data_ingestion.py` now runs end to end (R² = `0.8804332983749565`).

**Notes:**
- Both commands now work: `python src/components/data_ingestion.py` (Miniconda base) and `.\venv\python.exe -m src.components.data_ingestion` (venv).
- The two environments have different package versions (Python 3.14 vs 3.8), so the R² score differs slightly between them (0.8804 vs 0.8795). Pick one environment and use it consistently.
- CatBoost creates a `catboost_info/` folder in the project root with its training logs. It is safe to ignore, or add it to `.gitignore`.

**How to avoid this in future:**
- `ModuleNotFoundError` for a package you already installed usually means a different Python is running. Check with `Get-Command python` / `python -c "import sys; print(sys.executable)"`.
- Install packages with `python -m pip install ...` using the same `python` you run the project with, so they go into that interpreter.

---

## Step 15 — Hyperparameter tuning (GridSearchCV) added to the model trainer

**Command:**
```powershell
python src/components/data_ingestion.py
```

**Status:** ❌ Failed → ✅ Fixed (one error reported, two more bugs found and fixed, plus one warning cleaned up)

**Error:**
```
src.exception.CustomException: Error occured in python script name[...\src\components\model_trainer.py] line number [95]
error message[Error occured in python script name[...\src\utils.py] line number [31] error message[name 'GridSearchCV' is not defined]]
```

### 1. What the error says and about which command
Ingestion and transformation finished; the failure came during model training. `evaluate_models()` in `src/utils.py` (line 31) now calls `GridSearchCV(...)` to tune each model, but `GridSearchCV` was never imported, so Python did not know the name. The error message is nested because `CustomException` from `utils.py` was caught and wrapped again by `model_trainer.py` (line 95, the `evaluate_models(...)` call).

### 2. Resolution & prevention
**Fixes:**

| File / line | Was | Now | Why it would fail |
|---|---|---|---|
| `src/utils.py` imports | *(missing)* | `from sklearn.model_selection import GridSearchCV` | The reported error → `NameError` |
| `src/utils.py` line 32 | `GridSearchCV(model, para, cv=3)` | `GridSearchCV(model, param, cv=3)` | The variable is named `param`; `para` does not exist → `NameError` (would hit next) |
| `src/components/model_trainer.py` `models` dict | `"Liner Regression"`, `"K-Neighbour Classifier"`, `"XGBClassifier"`, `"CatBoosting Classifier"`, `"AdaBoost Classifier"` | `"Linear Regression"`, `"K-Neighbour Regressor"`, `"XGBRegressor"`, `"CatBoosting Regressor"`, `"AdaBoost Regressor"` | `evaluate_models` looks up `params[<model name>]`, and these names did not match the keys in the new `params` dict → `KeyError: 'Liner Regression'` (would hit next) |

**Warning cleaned up after the fixes:**
```
3 fits failed ... InvalidParameterError: The 'criterion' parameter of DecisionTreeRegressor must be a str among {'absolute_error', 'squared_error', 'poisson'}. Got 'friedman_mse' instead.
UserWarning: One or more of the test scores are non-finite: [0.69659899 nan 0.6973352 0.71249762]
```
Miniconda base has scikit-learn 1.9.1, which no longer accepts `'friedman_mse'` for `DecisionTreeRegressor` (the venv's 1.3.2 still does). The run did not crash, but those grid-search fits were wasted. Removed `'friedman_mse'` from the Decision Tree `criterion` list; it now works in both environments.

**Result:** ✅ Both commands run end to end with no errors or warnings:
- `python src/components/data_ingestion.py` → R² `0.8804332983749565`
- `.\venv\python.exe -m src.components.data_ingestion` → R² `0.8795158595242263`
- Best model is still `LinearRegression()`. Training now takes ~40 s (the grid search tries every parameter combination with 3-fold cross-validation).

**How to avoid this in future:**
- When you use a new class (`GridSearchCV`, etc.), add its import at the top of the file at the same time.
- A `NameError` right after a rename usually means one place still uses the old name (`para` vs `param`).
- When two dicts are matched by key (`models` and `params`), the keys must be spelled exactly the same. Copy them from one dict to the other instead of retyping.
- Parameter values accepted by scikit-learn change between versions. If a grid search prints "fits failed", read the `InvalidParameterError` and remove the rejected value.

---

<!-- Add new steps below in the same format: Command → Status → Error (if any) → (1) What it means (2) Resolution & prevention -->
