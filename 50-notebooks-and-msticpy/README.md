<div align="center">

# 🏹 Step 50 · Notebooks & MSTICPy

### *Run a hunting notebook against your workspace*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~50 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-compute if you use Azure ML-E3B341?style=flat-square)](#)

</div>

---

## 🎯 Goal

You've authenticated a Jupyter notebook to the workspace with MSTICPy, run a query, and produced one
enrichment/visualisation you couldn't do in the Logs blade.

## 🧠 Why this step

Notebooks are for hunts that need real code: multi-step pivoting, external enrichment at scale,
statistical/ML analysis, timeline reconstruction, and repeatable investigation playbooks. MSTICPy is
Microsoft's hunting library that wires KQL + threat intel + visual tools together.

## ✅ Prerequisites

- [Step 05](../05-rbac-and-roles/README.md) — Sentinel Reader/Responder on the workspace
- Python 3.10+ locally, **or** an Azure ML compute instance (has a cost — a small instance,
  stopped when idle)

## 🧭 Where to run it

| Option | Cost | Notes |
|---|---|---|
| **Local Jupyter / VS Code** | free | Best for the lab; `pip install msticpy` |
| **Azure ML compute instance** | per-hour compute | Sentinel's **Notebooks** blade launches here; stop it when done |
| **Sentinel Notebooks blade templates** | (uses Azure ML) | Ready-made: "Getting Started", "Entity Explorer", "Guided Investigation" |

## 🖱️ Do it — local

```bash
python -m venv .venv && . .venv/Scripts/activate   # Windows; use bin/activate on Linux
pip install "msticpy[azure]" jupyterlab
jupyter lab
```

New notebook:

```python
import msticpy as mp
mp.init_notebook()

# one-time: create msticpyconfig.yaml with your workspace (no secrets — uses az login / device code)
from msticpy.context.azure import MicrosoftSentinel
# or the QueryProvider path:
qry_prov = mp.QueryProvider("MSSentinel")
qry_prov.connect(
    workspace_id="<WORKSPACE_ID>",       # keep this out of git
    tenant_id="<TENANT_ID>",
)
```

```python
# run a query
df = qry_prov.exec_query("""
SigninLogs
| where TimeGenerated > ago(7d) and ResultType != 0
| summarize Failures=count() by UserPrincipalName, IPAddress, bin(TimeGenerated, 1h)
""")
df.head()
```

```python
# something the Logs blade can't do easily: a per-user failure timeline heatmap
import pandas as pd
pivot = df.pivot_table(index="UserPrincipalName", columns="TimeGenerated",
                       values="Failures", aggfunc="sum").fillna(0)
mp.nbwidgets.  # explore; or use seaborn:
import seaborn as sns; sns.heatmap(pivot)
```

```python
# MSTICPy IP enrichment (uses your configured TI providers)
from msticpy.context.tilookup import TILookup
ti = TILookup()
ti.lookup_iocs(data=df, ioc_col="IPAddress", providers=["OTX"])  # or VirusTotal, etc.
```

## 🧪 Validate

**You should see**:
- `qry_prov.exec_query` returns a pandas DataFrame with your real sign-in failures.
- One visual (heatmap / timeline / folium map via `df.mp_plot.folium_map`) rendered from that data.
- (If TI configured) an enrichment table joining your IPs to reputation.

Save the notebook to `artifacts/notebooks/identity-failures.ipynb` — **strip the workspace/tenant
IDs** (parameterise or use a cell that reads them from env). The `.gitignore` blocks
`.ipynb_checkpoints/`.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Workspace ID / tenant ID committed in the notebook | It's in the JSON — parameterise |
| Azure ML compute left running | Bills per hour around the clock |
| `pip install msticpy` without `[azure]` | Missing the Sentinel query provider |
| Huge `exec_query` with no time bound | Pulls the workspace into memory |
| Notebook as a detection | Notebooks are investigation/analysis — detections are rules |

## 🗒️ Log your run

`LOG.md` — the DataFrame shape, the visual you produced, and where the sanitised notebook lives.

## 📚 Microsoft Learn

- [Use Jupyter notebooks to hunt for security threats](https://learn.microsoft.com/en-us/azure/sentinel/notebooks)
- [Get started with notebooks and MSTICPy](https://learn.microsoft.com/en-us/azure/sentinel/notebook-get-started)
- [MSTICPy documentation](https://msticpy.readthedocs.io/)

---

<div align="center">
<sub>

[⬅ Prev: 49 · Hunt → detection](../49-hunt-to-detection/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 51 · UEBA & entity behavior ➡](../51-ueba-and-entity-behavior/README.md)

</sub>
</div>
