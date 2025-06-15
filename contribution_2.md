# Contribution 2: Clarifying `mlflow.log_artifacts()` Behavior in Tracking API Docs

### 📂 **File Access & Purpose**

* **Target File:** `mlflow/docs/docs/classic-ml/tracking/tracking-api/index.mdx`
* **Purpose:** Updates the **Tracking API reference table** to more accurately describe the usage of `mlflow.log_artifacts()` in MLflow documentation.

---

### 🎯 **Problem Identified**

* The API table listed:

  ```md
  | <APILink fn="mlflow.log_artifacts" /> | Log entire directory | `mlflow.log_artifacts("./plots/")` |
  ```

* ❗ This entry was misleading, as `mlflow.log_artifacts()` **only supports directories** — **not single files**.

* This led to **confusion among users** trying to log a file (e.g., `mlflow.log_artifacts("file.txt")`), which results in **no effect or an empty list** because the function internally depends on `os.walk()`.

---

### 🛠️ **What I Did**

1. Reviewed the function definition in the source code:

   ```python
   def log_artifacts(local_dir: str, ...)
   ```

2. Verified behavior through testing and `os.walk()` output.

3. Updated the docs table to clarify the expected input and prevent incorrect usage:

   ```md
   | <APILink fn="mlflow.log_artifacts" /> | Log all files in a directory (**not for single files**) | `mlflow.log_artifacts("./plots/")` |
   ```

4. Submitted a Pull Request with the fix and filled in the standard MLflow PR template.

5. Responded to maintainer feedback and ensured the formatting followed contribution guidelines.

---

### 🧾 **Files/Technologies Involved**

* 📄 **File Modified:** `index.mdx` inside MLflow's Tracking API docs
* 🧠 **Tooling:** Docusaurus (Markdown, MDX format), GitHub PR workflow
* 🧪 **Testing:** Manual verification of rendered table and function input behavior

---

### ✅ **Pull Request Details**

* **Title:** `Clarify: log_artifacts requires a directory, not a file`
* **PR Link:** [#16256](https://github.com/mlflow/mlflow/pull/16255)
* **Labels/Areas Touched:**

  * `area/docs`
  * `rn/documentation`

---
