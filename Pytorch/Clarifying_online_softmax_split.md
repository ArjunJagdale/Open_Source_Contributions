# Contribution Summary: Clarifying Online Softmax Split Reduction Limitation in PyTorch

## 1. What was the issue?

The PyTorch repository had an open issue ([#153241](https://github.com/pytorch/pytorch/issues/153241)) regarding the warning message and documentation for "online softmax" in the Inductor backend. When Inductor decides to split the reduction during softmax computation, it disables the more efficient online softmax implementation and falls back to a less efficient path. The warning message and comments were not very clear, and users/contributors were not directly guided to the relevant GitHub issue for more information or contribution.

## 2. What we did

- **Updated the docstring** for the `prepare_softmax_online` lowering in `torch/_inductor/lowering.py` to explicitly mention the limitation and reference the tracking issue.
- **Improved inline comments** to explain the technical challenge, clarify the current state, and invite community contributions.
- **Enhanced the warning message** to be more actionable, encouraging users to upvote or comment on the GitHub issue if they need this feature, and making it clear that contributions are welcome.
- **Added a direct link** to the relevant GitHub issue (#153241) wherever appropriate.

### Here is the code that was updated:

```python name=torch/_inductor/lowering.py
@register_lowering(inductor_prims.prepare_softmax_online, type_promotion_kind=None)
def prepare_softmax_online(x, dim):
    """
    Lowering inductor_prims.prepare_softmax_online to compute max/sum in one pass if no split is needed.

    Note:
        Currently, split reductions for online softmax are not supported. If a split is required,
        Inductor will fall back to a less efficient path and emit a warning. See
        https://github.com/pytorch/pytorch/issues/153241 for more details or to contribute.
    """
    kwargs = _make_reduction_inner(
        x, axis=dim, keepdims=True, dtype=None, override_return_dtype=None
    )

    reduction_ranges = kwargs["reduction_ranges"]
    rnumel = V.graph.sizevars.simplify(sympy_product(reduction_ranges))
    hint, num_split = ir.Reduction.num_splits(
        **kwargs,
        reduction_type="online_softmax_reduce",  # type: ignore[arg-type]
        reduction_numel=rnumel,
    )

    if (
        num_split == 1
        and V.graph.sizevars.size_hint(rnumel) >= config.unroll_reductions_threshold
    ):
        max_tensor, sum_tensor = OnlineSoftmaxReduction.create(
            input_node=x, num_output=2, reduction_hint=hint, **kwargs
        )
        return max_tensor, sum_tensor
    else:
        # Note: [Split online_softmax_reduce]
        # Split reductions for online softmax are currently not implemented.
        # Supporting this feature would require significant changes as split reductions
        # require two inputs instead of one. During training, online_softmax_reduce typically
        # does not require a split due to large batch*sequence sizes.
        # If you have an inference use case that needs this, please comment on
        # https://github.com/pytorch/pytorch/issues/153241.

        # TODO: [Contributions Welcome] Support split reduction for online_softmax_reduce.
        # See https://github.com/pytorch/pytorch/issues/153241 for discussion.

        warnings.warn(
            textwrap.dedent(
                """
                Online softmax is currently disabled when Inductor decides to split the reduction.
                If you need this feature for your workload, please upvote or comment on
                https://github.com/pytorch/pytorch/issues/153241 to help prioritize support.
                Contributions are welcome!
                """
            )
        )
        amax = reduce_amax(x, dim, keepdims=True)
        exp = lowerings[aten.exp](sub(x, amax))
        xsum = sum_(exp, dim, keepdims=True)
        return amax, xsum
```

We then submitted these changes to a new branch (`patch-1`) and created a pull request (PR #156556) against the upstream repository.

## 3. What results we expected?

- Improved clarity for users encountering the online softmax warning, helping them understand what is happening and how they can help.
- Encourage community participation by directly linking to the relevant issue and inviting contributions.
- Make it easier for future maintainers to locate and understand the current state and limitations of online softmax split reduction in the Inductor backend.
- Allow PyTorch maintainers to quickly review and merge the documentation/commentation improvement without requiring local setup or testing.

## 4. References

- **Pull Request:** [#156556](https://github.com/pytorch/pytorch/pull/156556)
- **Relevant Issue:** [#153241](https://github.com/pytorch/pytorch/issues/153241)
