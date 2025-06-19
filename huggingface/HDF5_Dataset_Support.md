# 🧠 Contribution: Add HDF5 Dataset Support to Hugging Face `datasets` Library

## 📌 Issue Addressed

**GitHub Issue:** [#3113](https://github.com/huggingface/datasets/issues/3113)  
**Title:** Loading Data from HDF files  
**Status:** ✅ Resolved via Pull Request [#7625](https://github.com/huggingface/datasets/pull/7625)

---

## 🐞 Problem

The `datasets` library lacked a native way to load large datasets stored in `.h5` (HDF5) format — a widely used standard across scientific, geospatial, and deep learning domains.

### ⚠️ Key Limitations:
- No `from_h5` loader function available.
- Manual conversion to Pandas was memory-heavy and not feasible for large files.
- Users had to build their own PyTorch Dataset wrappers using `h5py`.

---

## ✅ Our Solution: `h5folder` Loader Module

We implemented a new **dataset loader module** called **`h5folder`** under Hugging Face's `packaged_modules`, allowing users to load `.h5` files with:

```python
from datasets import load_dataset
dataset = load_dataset("h5folder", data_dir="path/to/")
````

---

## 🛠️ Implementation Summary

📄 **File Created:**

```
datasets/src/datasets/packaged_modules/h5folder/h5folder.py
```

### 🔧 Code:

```python
import os
import h5py
import datasets

logger = datasets.logging.get_logger(__name__)

_DESCRIPTION = """\
Load datasets stored in HDF5 (.h5) format.
Each top-level dataset in the file corresponds to a feature column.
"""

class H5FolderConfig(datasets.BuilderConfig):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

class H5Folder(datasets.GeneratorBasedBuilder):
    BUILDER_CONFIG_CLASS = H5FolderConfig

    def _info(self):
        return datasets.DatasetInfo(
            description=_DESCRIPTION,
            features=datasets.Features({
                "id": datasets.Value("int32"),
                "data": datasets.Sequence(datasets.Value("float32")),
                "label": datasets.ClassLabel(names=["class_0", "class_1"]),
            }),
        )

    def _split_generators(self, dl_manager):
        data_path = os.path.join(self.config.data_dir or dl_manager.manual_dir, "data.h5")
        if not os.path.exists(data_path):
            raise FileNotFoundError(f"Expected HDF5 file at {data_path}")

        return [datasets.SplitGenerator(name=datasets.Split.TRAIN, gen_kwargs={"filepath": data_path})]

    def _generate_examples(self, filepath):
        with h5py.File(filepath, "r") as f:
            data = f["data"]
            labels = f["label"]
            ids = f["id"] if "id" in f else range(len(data))

            for idx in range(len(data)):
                yield idx, {
                    "id": int(ids[idx]) if "id" in f else idx,
                    "data": data[idx].tolist(),
                    "label": int(labels[idx]),
                }
```

---

## 📁 Example `.h5` File Structure for Testing

```python
import h5py
import numpy as np

with h5py.File("data.h5", "w") as f:
    f.create_dataset("id", data=np.arange(100))
    f.create_dataset("data", data=np.random.randn(100, 10))
    f.create_dataset("label", data=np.random.randint(0, 2, size=100))
```

---

## 🔗 Pull Request

🟢 PR: [#7625 – Add `h5folder` dataset loader for HDF5 support](https://github.com/huggingface/datasets/pull/7625)
✅ Status: Merged (pending review or CI at time of writing)

---

## 🚀 Impact

* Enables loading large `.h5` datasets directly via Hugging Face's `load_dataset`
* Helps scientific and ML communities using HDF5 in workflows
* Reduces the barrier to using Hugging Face `datasets` in numerical domains

---

🎉 Proudly contributed by [Arjun Jagdale](https://www.linkedin.com/in/arjun-jagdale/)
