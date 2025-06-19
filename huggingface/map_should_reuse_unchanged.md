### ✅ 1. What Was the Problem

**Issue: [#6013](https://github.com/huggingface/datasets/issues/6013)**
Title: *"\[FR] map should reuse unchanged columns from the previous dataset to avoid disk usage"*

#### Problem Summary:

When using `dataset.map()` with the `input_columns` parameter, only the specified columns are processed—but **other untouched columns are unnecessarily dropped** and **re-added from scratch** (or lost unless manually handled).
This led to:

* Higher **disk usage** (due to copying or duplication)
* **Wasted I/O operations**
* Extra **memory & time** costs

---

### 🛠️ 2. What We Did (PR [#7626](https://github.com/huggingface/datasets/pull/7626))

**Pull Request:** [feat(map): reuse unchanged columns when input\_columns specified to reduce disk usage (#6013)](https://github.com/huggingface/datasets/pull/7626)

#### ✅ Summary of Changes:

We added logic at the **end of the `Dataset.map()` function** in `src/datasets/arrow_dataset.py` to automatically **reuse untouched columns** from the original dataset when:

* `input_columns` is specified
* and the untouched columns are **not** in `remove_columns`

#### 🔧 Code Logic (in simple form):

At the end of the `map()` method:

```python
if input_columns is not None:
    untouched_columns = set(self.column_names) - set(input_columns or []) - set(remove_columns or [])
    if untouched_columns:
        untouched_dataset = self.remove_columns([col for col in self.column_names if col not in untouched_columns])
        untouched_table = untouched_dataset._data
        result._data = pa.concat_tables([result._data, untouched_table], promote=True)
```

* `untouched_columns`: finds columns that were not transformed or removed
* `untouched_dataset`: filters original dataset to keep only those untouched columns
* Then `pyarrow.concat_tables()` combines the untouched part with the transformed result

---

### 🚀 3. How It Helps

#### Technical Impact:

* ⚡ **Performance Boost**: Reuses untouched columns directly without reprocessing
* 💾 **Less Disk I/O**: No need to re-save or re-read unchanged data
* 🧠 **Smarter Behavior**: Honors user intent when `input_columns` is explicitly set
* 🛡️ **Safe**: Only applies the logic when `input_columns` is used, preserving legacy behavior otherwise

#### In Simple Words:

If you're updating only one column of a dataset, the other columns should stay as-is.
Before this fix, Hugging Face datasets would **drop those other columns**, making you lose them or forcing you to manually handle them.
Now, they are **automatically reused and preserved**—saving time, memory, and effort.

---

### 👤 4. Author Details

**Contributor:** Arjun Jagdale
**GitHub:** [@ArjunJagdale](https://github.com/ArjunJagdale)
**LinkedIn:** [linkedin.com/in/arjunjagdale](https://www.linkedin.com/in/arjunjagdale)
