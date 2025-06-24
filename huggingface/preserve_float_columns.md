# Fix: Preserve Float Columns in JSON Loader

## 1. What Was the Problem?

The JSON loader in Hugging Face's `datasets` library was incorrectly coercing float values that are integer-like (e.g., `0.0`, `1.0`, `2.0`) to integers. This behavior resulted in:

* **Incorrect type inference**: Columns meant to be floats were labeled as `int`.
* **Downstream issues**: Certain computations (like statistics calculations) expected the column to be of type `float`, but failed since the values had been converted to `int`.

This bug was identified in [Issue #6937](https://github.com/huggingface/datasets/issues/6937).

---

## 2. What We Did – With Code

### **File Modified:**

`datasets/src/datasets/packaged_modules/json/json.py`

### **Modified Function:**

`_generate_tables()`

### **Key Change:**

Inside the pandas fallback path (used when Arrow's native JSON reader fails), we injected a check right before converting the DataFrame to an Arrow table. The goal was to enforce that columns containing floats remain as floats—even when their values are integer-like.

### **Code Snippet:**

```python
# After loading the DataFrame from JSON using pandas:
if df.columns.tolist() == [0]:
    df.columns = list(self.config.features) if self.config.features else ["text"]

# ✅ FIX: Coerce float-looking ints (like 0.0, 1.0) back to float64
for col in df.columns:
    col_data = df[col].dropna()
    if col_data.apply(lambda x: isinstance(x, float)).all():
        if col_data.apply(lambda x: x.is_integer()).all():
            df[col] = df[col].astype("float64")

try:
    pa_table = pa.Table.from_pandas(df, preserve_index=False)
except pa.ArrowInvalid as e:
    logger.error(
        f"Failed to convert pandas DataFrame to Arrow Table from file '{file}' with error {type(e)}: {e}"
    )
    raise ValueError(
        f"Failed to convert pandas DataFrame to Arrow Table from file {file}."
    ) from None
```

This snippet is integrated into the fallback branch of the `_generate_tables()` method, ensuring that if a DataFrame column consists entirely of float values (even when integer-like), it is explicitly cast to `"float64"`.

---

## 3. What It Will Do

* **Preserve Data Types:** The fix ensures that columns containing floats remain as `float64` even if all the values are integer-like.
* **Correct Statistics:** Downstream processes (like computing dataset statistics or rendering the dataset viewer) will correctly recognize these columns as floats.
* **Robust JSON Loading:** The overall functionality of the JSON loader remains intact, but with an added safeguard for type consistency when falling back to the pandas approach.

---

## 4. Issue and PR Details

* **Issue:** [#6937](https://github.com/huggingface/datasets/issues/6937) – "JSON loader implicitly coerces floats to integers"
* **Pull Request:** [#7635](https://github.com/huggingface/datasets/pull/7635)

---
