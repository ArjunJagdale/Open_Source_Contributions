## 🎯 Hugging Face Datasets – Load Cached Dataset as Iterable (#5481)

**Repository:** [huggingface/datasets](https://github.com/huggingface/datasets)  
**My PR:** [#7628 – Add `as_iterable_dataset()` method to DatasetBuilder](https://github.com/huggingface/datasets/pull/7628)

### 🧩 Overview

This contribution implements a new method `as_iterable_dataset()` in the `DatasetBuilder` class, allowing users to **stream datasets directly from cached Arrow files** without loading the full dataset into memory.

This feature is valuable for large-scale training scenarios (e.g., C4, OSCAR) where using traditional `Dataset` objects is memory-intensive. It enables training models with datasets stored in cache using streaming-style access.

---

### 💡 Motivation

Hugging Face maintainers suggested implementing this “one level below” the high-level `load_dataset()` API:

```python
builder = load_dataset_builder("c4", "en")
builder.download_and_prepare()
ids = builder.as_iterable_dataset(split="train[:100]")
````

The goal was to provide an efficient, memory-friendly way to read preprocessed data for streaming-based training or evaluation.

---

### 🛠️ What I Did

#### ✅ Feature Implementation (`builder.py`)

* Added `as_iterable_dataset()` method to `DatasetBuilder`
* Used `ArrowReader.get_file_instructions()` to locate cached data
* Leveraged `ArrowExamplesIterable` to build a PyTorch-style `IterableDataset`
* Returned an `IterableDataset` with examples streamed from Arrow files

#### ✅ Testing (in follow-up PR)

* Wrote a test in `test_builder.py` to:

  * Load a builder (e.g., C4)
  * Run `download_and_prepare()`
  * Stream 100 examples using `as_iterable_dataset()`
  * Validate streamed examples contain expected fields

---

### 🔍 Key Files Changed

* `src/datasets/builder.py` – Core logic
* `tests/test_builder.py` – Unit test for new method (optional, in follow-up PR)

---

### 🤝 Collaborated With

* **@lhoestq** – Hugging Face maintainer who provided guidance
* Referenced discussions in #5481 and related design suggestions

---

### ✅ Outcome

* Main PR opened: [#7628](https://github.com/huggingface/datasets/pull/7628)
* Test PR opened: [#7629](https://github.com/huggingface/datasets/pull/7629)
* Test PR submitted separately to keep feature PR clean
* Followed OSS contribution best practices (modular code, small PRs, test coverage)

---

### 🔗 Related Links

* 🔍 Original Issue: [#5481 – Load a cached dataset as iterable](https://github.com/huggingface/datasets/issues/5481)
* 🚀 Feature PR: [#7628](https://github.com/huggingface/datasets/pull/7628)
