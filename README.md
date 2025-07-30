## 🧠 Open Source Contributions — Hugging Face

Contributed to Hugging Face's open-source ecosystem (`datasets` and `dataset-viewer`) with production-level code enhancements, usability improvements, and critical bug fixes. All contributions were reviewed and merged by core maintainers, impacting widely used tooling in the ML community.

---

### ✅ Merged Pull Requests

**1. Improve Documentation for Dataset APIs**

[Fix misleading `add_column()` usage example in docstring (#7648)](https://github.com/huggingface/datasets/pull/7648)

🔹 Identified and corrected misleading docstrings for core methods like `add_column`, `select`, and `filter`, which return new datasets instead of modifying them in-place.

🔹 Prevented potential bugs caused by misunderstanding dataset immutability by updating five+ key method examples.

✅ Improves developer onboarding and API correctness across Hugging Face datasets.

🧠 Reviewed by [@lhoestq](https://github.com/lhoestq)

---

**2. Raise Explicit Error in Folder-Based Dataset Loader**

[Raise error in FolderBasedBuilder when `data_dir` and `data_files` are missing (#7623)](https://github.com/huggingface/datasets/pull/7623)

🔹 Solved a critical UX issue where calling `load_dataset("audiofolder")` without specifying `data_dir` led to unexpected behavior and long load times.

🔹 Added explicit error validation in `FolderBasedBuilder._info()` to enforce correct usage.

📂 Affects all folder-based loaders like `audiofolder`, `imagefolder`, reducing silent failures and improving robustness.

🧠 Reviewed by [@lhoestq](https://github.com/lhoestq)

---

**3. Refactor Config Defaults to Avoid Shared Mutable State**

[Replace `get_empty_str_list` with `.copy()` constants in config (#3219)](https://github.com/huggingface/dataset-viewer/pull/3219)

🔹 Refactored internal `ParquetAndInfoConfig` logic to eliminate shared mutable defaults in dataclass fields.

🔹 Ensured safe and predictable behavior in config initialization across multiple jobs in `orchestrator.py`.

🧼 Passed all CI checks, Ruff formatting, and Hugging Face’s `poetry` build standards.

🧠 Reviewed by [@severo](https://github.com/severo)

---

**4. Add Unit Tests for Core Caching Utility**

[Add unit tests for `get_previous_step_or_raise` (#3218)](https://github.com/huggingface/dataset-viewer/pull/3218)

🔹 Designed and implemented comprehensive tests for cache retrieval and error handling in `get_previous_step_or_raise`.

🔹 Covered scenarios including cache hits, misses, and corrupted responses using internal testing utilities (`upsert_response`, etc).

🧪 Strengthened the dataset caching layer’s reliability in production workflows.

🧠 Reviewed by [@severo](https://github.com/severo)

---

**5. Simplify Gated Dataset Setup using `HfApi.update_repo_settings`**

[Refactor tests to use `update_repo_settings` from Hub client (#3206)](https://github.com/huggingface/dataset-viewer/pull/3206)

🔹 Replaced legacy helper functions with Hugging Face Hub's official API for enabling gated dataset settings.

🔹 Removed over 220+ lines of redundant code across 3 test modules while preserving test integrity.

✅ Resulted in cleaner, more maintainable tests aligned with the latest SDK capabilities.

🧠 Reviewed by [@severo](https://github.com/severo)

---

**6. Add support for Date features in Croissant schema**

[Add support for Date features in Croissant schema #3223](https://github.com/huggingface/dataset-viewer/pull/3223)

🔹 This PR adds support for datasets.features.Date in the Croissant schema generation.

🔹 Fields like Date, UTCDate, and UTCTime (e.g., in Lichess/standard-chess-games) will now be correctly included in the generated spec.

🧠 Reviewed by [@severo](https://github.com/severo)

---


### 📁 GitHub Profile

See full contribution history → [github.com/ArjunJagdale](https://github.com/ArjunJagdale)
