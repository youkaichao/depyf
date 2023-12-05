![Logo](imgs/logo-and-text.svg)

[![Documentation Status](https://readthedocs.org/projects/depyf/badge/?version=latest)](https://depyf.readthedocs.io/en/latest/) ![Supported Python Versions](https://img.shields.io/badge/python-%203.7%20%7C%203.8%20%7C%203.9%20%7C%203.10%20%7C%203.11%20%7C%203.12-blue) ![Python Decompilation Tests](https://github.com/thuml/depyf/actions/workflows/test_decompile.yml/badge.svg) ![PyTorch Integration Tests](https://github.com/thuml/depyf/actions/workflows/test_pytorch.yml/badge.svg) ![MIT License](https://img.shields.io/github/license/thuml/depyf)

Have you ever felt overwhelmed by the complexities of `torch.compile`? Diving into its workings can feel like black magic, with bytecode and Python internal details that many users fail to understand, hindering them from understanding and adapting to `torch.compile`.

If you also face the problem, then you might be interested in `depyf`. As the logo suggests, `depyf` is a software tool to leverage advanced Python features (the Python snake symbol) to open up internal details (the internal gears symbol) of PyTorch's compiler `torch.compile` (the PyTorch logo), so that users can understand it, adapt to it, and tune their code (the debugger symbol) to get maximum performance benefit out of it.

:warning: This project is developed under close collaborations with the PyTorch team. We frequently require new features from PyTorch to support better understanding of `torch.compile`. Therefore, **please use this project along with PyTorch nightly**. Visit the [PyTorch website](https://pytorch.org/) for how to install nightly version of PyTorch. **We recommend updating your PyTorch nightly installation every week or so**.

:warning: During development, we seek suggestions from the community quite a lot. You may find some early usage examples from some discussion forums or social media platforms. **Please follow the latest documentation for how to use this tool.**

# Installation

Stable release: `pip install depyf`

Nightly version (recommended): `pip install git+https://github.com/thuml/depyf.git`

# Usage

The main usage is quite simple: just wrap your code within a context manager:

```diff
import torch
from torch import _dynamo as torchdynamo
from typing import List

@torch.compile
def toy_example(a, b):
    x = a / (torch.abs(a) + 1)
    if b.sum() < 0:
        b = b * -1
    return x * b

def main():
    for _ in range(100):
        toy_example(torch.randn(10), torch.randn(10))

if __name__ == "__main__":
-     main()
+     import depyf
+     with depyf.prepare_debug("./dump_src_dir"):
+         main()
```

Then you can see all the details of `torch.compile` inside the directory `./dump_src_dir`. The details are organized into the following:

- `full_code_for_xxx.py` for each function using `torch.compile`
- `__transformed_code_for_xxx.py` for Python code associated with each graph.
- `__compiled_fn_xxx.py` for each computation graph and its optimization:
  - `Captured Graph`: a plain forward computation graph
  - `Joint Graph`: joint forward-backward graph from `AOTAutograd`
  - `Forward Graph`: forward graph from `AOTAutograd`
  - `Backward Graph`: backward graph from `AOTAutograd`
  - `kernel xxx`: compiled CPU/GPU kernel wrapper from Inductor.

If you want to use debugger to step through the above code, just add another context manager:

```diff
import torch
from torch import _dynamo as torchdynamo
from typing import List

@torch.compile
def toy_example(a, b):
    x = a / (torch.abs(a) + 1)
    if b.sum() < 0:
        b = b * -1
    return x * b

def main():
    for _ in range(100):
        toy_example(torch.randn(10), torch.randn(10))

if __name__ == "__main__":
    import depyf
-     with depyf.prepare_debug("./dump_src_dir"):
+     with depyf.prepare_debug("./dump_src_dir", pause=True):
        main()
+     with depyf.debug():
+         main()
```

Calling `depyf.prepare_debug` with `pause=True` will pause the program for you to set breakpoints, and then the next call to `depyf.debug()` enables debugging into these files.

# Python Version Coverage

The following python major versions are tested:

- Python 3.12
- Python 3.11
- Python 3.10
- Python 3.9
- Python 3.8
- Python 3.7

You can see the coverage report by simply running `python python_coverage.py`.

# Full Python Syntax Is Not Supported

This package is intended to understand the generated PyTorch bytecode, and does not aim to fully cover all the syntax of Python. For example, async operations like `async/await` are not supported.

All the bytecode generated by PyTorch when benchmarking timm and huggingface transformers are collected [here](https://github.com/thuml/depyf/tree/master/pytorch_bytecode). We can make several observations:

- No while loops (no jump back instructions).
- try-except-finally only has try-finally.
- No complicated conditions like `if a and b or c or (d and e)`.

Then, we can overfit the decompiler to work for the bytecode generated by PyTorch. How? Pure labor work. Implement all bytecode for all the supported Python versions, one by one. Yes, that's it.

# Contributions are welcome!

If you find any error in the decompilation, feel free to open issues or pull requests to fix it!

Hopefully, by using this package, everyone can understand `torch.compile` now!

# Contact

Any discussion/issue report/PR is welcome. Or contact youkaichao@gmail.com if you have any other problems.
