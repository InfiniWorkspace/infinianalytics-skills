# Python SDK (`infinianalytics`)

Official PyPI package wrapping the event API. Requires Python 3.7+, no external dependencies.

```bash
pip install infinianalytics
```

## Initialization

One `InfiniAnalytics` instance per execution, instantiated directly. There is no `init()` step or context manager:

```python
from infinianalytics import InfiniAnalytics
import os

execution = InfiniAnalytics(
    token=os.environ["INFINI_TOKEN"],
    automation_id="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
)
```

If `execution_id` is not passed, the SDK generates one from `datetime.now().isoformat()`. Pass your own only to correlate with an ID from the user's system:

```python
execution = InfiniAnalytics(token=..., automation_id=..., execution_id=my_run_id)
```

## Methods

Called directly on the instance:

| Method | Sends |
|---|---|
| `.start(description)` | `START`, once, at the beginning of the run |
| `.event(description)` | `EVENT`, on each progress milestone |
| `.warning(description)` | `WARNING`, for a non-critical issue, run continues |
| `.end(description)` | `END`, once, on successful completion |
| `.error(description, error_id=None, error_description=None)` | `ERROR`, from the except block; marks the run failed |

## Canonical shape

```python
execution.start("Invoice processing started")
try:
    invoices = fetch_invoices()
    execution.event(f"Fetched {len(invoices)} invoices")
    for inv in invoices:
        process(inv)
    execution.end(f"Processed {len(invoices)} invoices successfully")
except Exception as ex:
    execution.error(
        description="Invoice processing failed",
        error_id=type(ex).__name__,
        error_description=str(ex),
    )
    raise
```

## Failure behavior

The SDK never raises. If a call to the InfiniAnalytics API fails (network, bad token), the method prints a message and returns `None`. Tracking failures never break the automation, but they are silent except for stdout. Full docs: https://pypi.org/project/infinianalytics/
