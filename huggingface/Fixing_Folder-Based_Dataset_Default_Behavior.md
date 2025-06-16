# 🧠 Contribution to Hugging Face Datasets: Fixing Folder-Based Dataset Default Behavior

This repository documents my contribution to the [🤗 Hugging Face Datasets](https://github.com/huggingface/datasets) open-source project — particularly fixing the behavior of **folder-based datasets** like `audiofolder` when no `data_dir` or `data_files` is specified.

---

## 🧩 Problem Background

### ❗ The Bug (Issue #6152)

When using:

```python
load_dataset("audiofolder")
```

...without providing a `data_dir` or `data_files`, the dataset loader **silently defaults to the current directory**.

This leads to:

* Long load times,
* Unintended scanning of local files,
* Confusing and unintended behavior for the user.

The expected behavior was to **raise a clear error** instead.

---

## 🛠️ What I Contributed

### ✅ Objective

Fix this by adding an **early validation check** in the dataset module loading system, specifically for folder-based datasets.

---

## 📁 File Modified

* **Path:** `src/datasets/load.py`
* **Function:** `get_module` in `PackagedDatasetModuleFactory`

### ✏️ Code Added

We inserted the following logic to raise an error **before** it tries scanning the filesystem:

```python
from datasets.packaged_modules.folder_based_builder.folder_based_builder import FolderBasedBuilderConfig

# ✅ Early validation for folder-based datasets like "audiofolder"
if self.builder_cls.BUILDER_CONFIG_CLASS == FolderBasedBuilderConfig:
    if not self.data_dir and not self.data_files:
        raise ValueError(
            "Folder-based datasets require either `data_dir` or `data_files` to be specified. "
            "Neither was provided."
        )
```

This ensures that the loader stops immediately with a clear message if required inputs are missing.

---

## 🚦 PR Summary

* **Pull Request:** [#7618](https://github.com/huggingface/datasets/pull/7618)
* **Issue Resolved:** [#6152](https://github.com/huggingface/datasets/issues/6152)
* **Change Type:** `bug-fix`, `area/datasets`, `area/load`
* **Tested:** Manually tested using `load_dataset("audiofolder")` without arguments — raised expected error.

---

## 🧪 Why It Matters

This fix:

* Prevents accidental scanning of local files.
* Improves user experience with clearer feedback.
* Avoids unnecessary delays in loading datasets.

---

## 🙋‍♂️ Maintainers Pinged

During the PR process, I reached out to relevant maintainers involved in the original discussion:
`@aniyagnik`, `@npuichigo`, `@sahillihas`, `@lhoestq`, `@mariosasko`.

---

## 🔗 Connect with Me

👤 **Arjun Jagdale**
🌐 [LinkedIn Profile](https://www.linkedin.com/in/arjun-jagdale/)
💻 Aspiring Open Source Contributor in AI/ML

---
