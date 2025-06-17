# 📦 Contribution 2: Folder-Based Dataset Validation in 🤗 Hugging Face Datasets

### 🧠 Context: Bug in `load_dataset("audiofolder")` without `data_dir`

When loading folder-based datasets like `audiofolder` using:

```python
load_dataset("audiofolder")
```

If the user didn’t specify either `data_dir` or `data_files`, the loader would **silently default to scanning the current directory**, which caused:

* 🐌 Long load times
* 🔍 Unintended scanning of local files
* 🧩 Confusing user experience

This issue was discussed in: [#6152](https://github.com/huggingface/datasets/issues/6152)

---

### 🎯 Objective

Prevent Hugging Face Datasets from falling back to the current directory when loading folder-based datasets **without proper input**.

---

### 🔍 Scope of Change

* **Target file:**
  `src/datasets/packaged_modules/folder_based_builder/folder_based_builder.py`

* **Class Modified:**
  `FolderBasedBuilder`

* **Method Modified:**
  `_info(self)`

---

### 🛠️ Implementation Details

#### ✅ Before

The class did not check whether `data_dir` or `data_files` was provided.

#### ✅ After

Added this validation block in `_info()`:

```python
def _info(self):
    if not self.config.data_dir and not self.config.data_files:
        raise ValueError(
            "Folder-based datasets require either `data_dir` or `data_files` to be specified. "
            "Neither was provided."
        )
    return datasets.DatasetInfo(features=self.config.features)
```

* If both `data_dir` and `data_files` are missing → raise a clear `ValueError`
* Ensures clean, early failure with an informative message

---

### 💾 Files Changed

| File                      | Change                                                                |
| ------------------------- | --------------------------------------------------------------------- |
| `folder_based_builder.py` | ✅ Added validation in `_info()`                                       |
| `load.py`                 | ✅ Removed previously added builder-specific check from `get_module()` |

---

### ✅ Outcome

* ❌ No more silent fallback to current working directory
* ✅ Clear error raised with actionable message
* ✅ Logic is now encapsulated within the dataset builder class, as suggested by maintainers

---

### 🔁 Maintainer Feedback Incorporated

* Original fix placed in `load.py → get_module()`
* Maintainer @lhoestq suggested relocating it to:

  ```
  FolderBasedBuilder._info()
  ```
* Refactored the PR accordingly and created a clean new PR

---

### 📝 Pull Request Info

| PR                                                         | Status                        |
| ---------------------------------------------------------- | ----------------------------- |
| [#7623](https://github.com/huggingface/datasets/pull/7623) | ✅ Open and under review       |
| Affects                                                    | `area/datasets`, `area/load`  |
| Release Note Label                                         | `rn/bug-fix`                  |
| Patch Inclusion                                            | ✅ Yes, safe for patch release |

---

### 🔗 Issue

* [x] Resolved: [#6152 – FolderBase Dataset automatically resolves under current directory when `data_dir` is not specified](https://github.com/huggingface/datasets/issues/6152)

---

### 🧑‍💻 Contributor Info

**Name:** Arjun Jagdale

**GitHub:** [@ArjunJagdale](https://github.com/ArjunJagdale)

**LinkedIn:** [linkedin.com/in/arjun-jagdale](https://www.linkedin.com/in/arjun-jagdale/)

**Contribution Style:** Fully GitHub-based editing (via fork + UI, no local clone)

---
