# Fabric setup and costs

## Where everything lives

| Item | Location |
|---|---|
| Workspace | `walmart-m5` (on capacity `fabrictest1`) |
| Lakehouse | `walmart_m5_lakehouse` (tables + `Files/` with the M5 CSVs and `models/walmart_m5_lgbm.txt`) |
| Notebook | `walmart_m5_forecast` |
| MLflow experiment | `walmart_m5_forecast` (runs `lgbm_validation`, `lgbm_final`) |
| Registered model | `walmart_m5_lgbm`, version 1 |
| Azure capacity | `fabrictest1`, F2, Sweden Central, resource group `Fabric_New` |

## Costs: read this before using Fabric again

* The account is not eligible for the free Fabric trial. The project runs on the **paid F2 capacity `fabrictest1`**.
* F2 is billed **per second while it is running**: roughly $0.36–0.40 per hour, about $260+ per month if left on.
* **While paused** there is no compute charge. Only the small OneLake storage cost remains (cents).
* Fabric cannot open the workspace, run the notebook or show a report while the capacity is paused.

### Resume before working

Azure portal → search "Microsoft Fabric" → `fabrictest1` → **Resume**. Wait for Status: Active.

### Pause when done (do not forget)

Azure portal → `fabrictest1` → **Pause** → Yes. Status should show Paused.

Tip: in Azure, Cost Management → Budgets → create a €5 monthly budget with an email alert as a safety net.

## Re-running the pipeline

1. Resume the capacity.
2. Open the notebook `walmart_m5_forecast` → **Run all** (about 10 minutes for one store).
3. A new model version is registered each run (v2, v3, …).
4. Stop the session (square button in the toolbar) and pause the capacity.
