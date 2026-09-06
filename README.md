# Cell Image Segmentation — YOLO with a Deployment Pipeline

Instance segmentation of cells in microscopy images, wrapped in a modular training
pipeline and shipped as a container to Azure Web App. The emphasis is the DLOps
path around the model: staged training, Docker packaging, and CI deployment.

[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![YOLO](https://img.shields.io/badge/YOLO-Ultralytics-00FFFF)](https://docs.ultralytics.com/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Azure Web App](https://img.shields.io/badge/Azure-Web_App-0078D4?logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/en-us/products/app-service/web)
[![License: MIT](https://img.shields.io/badge/License-MIT-22C55E.svg)](LICENSE)

> **Related repo.** [`Image-segmentation-using-YoloV11`](https://github.com/rk-chavali/Image-segmentation-using-YoloV11)
> is the same segmentation pipeline built against YOLOv11 specifically. **This
> repo is the canonical one** — it carries the Flask serving app, the Docker
> image, and the Azure CI workflow. Start here unless you specifically want the
> v11 training variant.

## What it does

| Route | Method | Does |
|-------|--------|------|
| `/` | GET | Upload UI |
| `/train` | GET | Runs the full training pipeline |
| `/predict` | POST | Takes a base64 image, returns the segmented result |

Images arrive base64-encoded, are decoded to disk, run through the model, and the
annotated output is encoded back to base64 for the response.

## Pipeline stages

```
data ingestion  ->  data validation  ->  model trainer
  pull dataset       check required        YOLO training
  unzip to disk      files present         + export weights
```

Each stage is a module under `cellSegmentation/components/`, configured by
dataclasses in `cellSegmentation/entity/`, and wired together by
`cellSegmentation/pipeline/training_pipeline.py`. Failures raise a shared
`AppException` carrying the originating stage, and logs go through
`cellSegmentation/logger/`.

## Layout

| Path | Purpose |
|------|---------|
| `cellSegmentation/components/` | Ingestion, validation, and training stages |
| `cellSegmentation/entity/` | Config and artifact dataclasses |
| `cellSegmentation/pipeline/` | Stage orchestration |
| `app.py` | Flask serving app with CORS |
| `Dockerfile` | Container image |
| `.github/workflows/main_cellseg.yml` | Build, push to ACR, deploy to Azure Web App |
| `flowcharts/`, `reseach/` | Design notes and experiment notebooks |

## Run it locally

```bash
pip install -r requirements.txt
python app.py
```

Serves on the host and port set in `cellSegmentation/constant/application.py`.

Train from a clean state:

```bash
curl http://localhost:8080/train
```

## Container

```bash
docker build -t cellseg .
docker run -p 8080:8080 cellseg
```

## Deployment

`main_cellseg.yml` builds the image on every push to `main`, pushes it to Azure
Container Registry, and deploys to the `cellseg` Web App. It expects the registry
username and password to be set as repository secrets.

## Credits

The project structure follows a public MLOps course template by Boktiar Ahmed
Bappy. The pipeline wiring, model work, and deployment configuration in this repo
are my own.

## License

MIT, see [LICENSE](LICENSE).
