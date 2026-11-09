# Project Execution Report

## Environment Configuration

- Operating system: Windows
- Project root: `C:\churnn1_12\churn-prediction (1)\churn-prediction`
- Python: 3.14.7
- Virtual environment: `.venv`
- pip: 26.3
- Docker CLI: 29.9.2
- Docker Engine: running through Docker Desktop
- FastAPI/Uvicorn application: running and tested locally
- Kubernetes CLI: installed; no active Kubernetes context

## Installed Dependencies

The Python API manifest installed successfully on Python 3.14.7. The complete project freeze also installed successfully; `pip check` reported no broken requirements. Directly relevant packages include:

- FastAPI 0.141.1
- Uvicorn 0.52.4
- Pydantic 2.13.4
- NumPy 2.4.6
- pandas 2.3.3
- scikit-learn 1.9.0
- MLflow 3.15.2
- XGBoost 3.2.0
- PyYAML 6.0.3

No package required the least-disruptive Python downgrade: both manifests resolved Python 3.14-compatible Windows wheels. pytest and pytest-asyncio were added to enable API verification.

## Experiment Checklist

| Experiment/Lab | Verified execution file | Status | Evidence |
|---|---|---|---|
| Lab 1 - Data exploration/EDA | `notebooks/churn_prediction.ipynb` | Completed | Executed notebook output saved as `notebooks/churn_prediction.executed.ipynb` |
| Lab 2 - Project implementation | `notebooks/project_implementation.ipynb` | Completed | Executed notebook output saved as `notebooks/project_implementation.executed.ipynb` |
| Lab 3 - Baseline ML pipeline | `pipelines/run_lab3_baseline.py` | Completed | Accuracy 0.7842, precision 0.5656, recall 0.7887, F1 0.6588 |
| Lab 4 - MLflow tracking | `pipelines/run_lab4_tracking.py` | Completed | MLflow run created and reproducibility report passed |
| Lab 5 - Production data pipeline | `pipelines/run_lab5_pipeline.py` | Completed | Schema, preprocessing, and output validation passed |
| Lab 6 - Registry/lifecycle | `pipelines/run_lab6_registry.py` | Completed | Production model v1 promoted from the Lab 6 registry run |
| Lab 7 - Integrated data pipeline | `src/mlops_pipeline/churn_prediction_pipeline.py --force` | Completed | All integrated data/preprocessing/training stages passed |
| Lab 8 - Evaluation and quality gate | `src/mlops_pipeline/06_model_evaluation.py`, `07_quality_gate.py` | Completed | Recall 0.7219 passed the configured 0.70 minimum |
| Lab 9 - Failure handling | `src/mlops_pipeline/10_failure_injection.py` | Completed | Three repetitions passed all expected fail/recovery scenarios |
| Lab 10 - API serving | `api/main.py`, `tests/test_api.py` | Completed | 23 API tests passed; live endpoints tested |
| Lab 11 - Containerization/deployment | `deployment/deploy_production.py`, Kubernetes manifests | Docker completed; Kubernetes blocked | Docker image and running container verified; no Kubernetes context is configured |
| Lab 12/13 - Monitoring and end-to-end trigger | `src/mlops_pipeline/12_*`, `12_monitor_trigger.py` | Completed | Drift reports generated and trigger correctly processed an already-ingested batch |

## Commands Used

```powershell
py -3.14 -m venv .venv
& .\.venv\Scripts\python.exe -m pip install --upgrade pip
& .\.venv\Scripts\python.exe -m pip install -r .\requirements.txt
& .\.venv\Scripts\python.exe .\pipelines\run_lab3_baseline.py
& .\.venv\Scripts\python.exe .\pipelines\run_lab4_tracking.py
& .\.venv\Scripts\python.exe .\pipelines\run_lab5_pipeline.py
& .\.venv\Scripts\python.exe .\pipelines\run_lab6_registry.py
& .\.venv\Scripts\python.exe .\src\mlops_pipeline\churn_prediction_pipeline.py --force
& .\.venv\Scripts\python.exe .\src\mlops_pipeline\10_failure_injection.py
& .\.venv\Scripts\python.exe -m pytest tests/test_api.py -q
docker build -t churn-prediction-api:local .
docker run -d --name churn-api-local -p 8000:8000 churn-prediction-api:local
& .\.venv\Scripts\python.exe .\src\mlops_pipeline\12_monitor_trigger.py
```

## Code Modifications

- [src/evaluate.py](src/evaluate.py): created the missing `outputs` directory before writing baseline false-positive and false-negative CSV files.
- [src/validate_outputs.py](src/validate_outputs.py): corrected a malformed dictionary declaration.
- [src/mlops_pipeline/11_prepare_deployment.py](src/mlops_pipeline/11_prepare_deployment.py): replaced Windows-only Unicode tree output with ASCII output, and preserves existing deployment files when refreshing artifacts.
- [notebooks/project_implementation.ipynb](notebooks/project_implementation.ipynb): corrected the misplaced ROC-AUC import and normalized the incorrectly typed markdown cell.

## Metrics

### Baseline Lab 3

- Accuracy: 0.7842
- Precision: 0.5656
- Recall: 0.7887
- F1: 0.6588

### Integrated Lab 7/8 evaluation

- Test rows: 1,409
- Accuracy: 0.7672
- Precision: 0.5466
- Recall: 0.7219
- F1: 0.6221
- ROC-AUC: 0.8452
- Quality gate: passed (0.7219 >= 0.70)

### Drift analysis

- Data drift: MonthlyCharges
- Probability-distribution prediction drift: detected (KS 0.229241)
- Degraded concept metrics: accuracy, recall, F1, ROC-AUC

## Generated Output Locations

- Models: `models/`
- Deployment bundle: `deployment/model.pkl`, `deployment/preprocessor.pkl`, `deployment/model_metadata.json`
- Evaluation: `reports/evaluation_report.json`
- Quality gate: `reports/quality_gate_report.json`
- Lifecycle decision: `reports/lifecycle_report.json`
- Failure handling: `reports/failure_report.json`
- Drift reports: `outputs/drift/`
- Monitoring decision: `outputs/monitoring/monitor_trigger_report.json`
- Pipeline log: `logs/churn_prediction_pipeline_run_log.json`
- Executed notebooks: `notebooks/churn_prediction.executed.ipynb`, `notebooks/project_implementation.executed.ipynb`

## Docker and Container Status

- Existing Dockerfile built successfully.
- Image: `churn-prediction-api:local`
- Image size: approximately 716 MB.
- Container: `churn-api-local`
- Container status: running and exposing port 8000.
- Container startup and `/health` model loading were verified.
- Deployment script also built:
  - `churn-prediction-api:model-v2`
  - `churn-prediction-api:latest`

## API Testing Results

- Root `/`: passed
- Health `/health`: passed
- Metadata `/metadata`: passed; production model version 2 loaded from the deployment bundle
- Individual `/predict`: passed, example probability 0.3580405962744439
- Batch `/predict/batch`: passed for two customers with count 2
- Pytest suite: 23 passed

Container and host API ports share `8000`, so the container must be stopped before starting a separate host Uvicorn server, or the container port must be mapped to another host port.

## Kubernetes Status

Kubernetes manifests are present under `k8s/`. Docker Desktop is running, but `kubectl config get-contexts` showed no configured/current cluster context, so deployment could not be validated. The deployment automation now reaches Kubernetes correctly and stops with the explicit no-cluster error rather than claiming success.

Enable Docker Desktop Kubernetes in Docker Desktop settings, wait until the cluster is ready, then run:

```powershell
kubectl config get-contexts
kubectl config use-context docker-desktop
& .\.venv\Scripts\python.exe .\deployment\deploy_production.py
kubectl rollout status deployment/churn-api --timeout=180s
kubectl get pods
kubectl get service churn-api
```
