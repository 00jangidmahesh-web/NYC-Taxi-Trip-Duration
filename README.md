# NYC Taxi Trip Duration Prediction

This project estimates how long a New York City taxi trip will take using trip
details such as pickup and drop-off coordinates, pickup time, passenger count,
and vendor. It includes a data and model-training pipeline, saved inference
artifacts, and a FastAPI service.

## What it does

- Cleans trip data and creates pickup-time features.
- Calculates Haversine, Euclidean, and Manhattan distance features.
- Preprocesses the features and trains an XGBoost regression model.
- Serves trip-duration estimates through a REST API.

The prediction is returned in minutes. The API uses the pre-trained model and
transformers in `container_models/`.

## Run the API locally

Use Python 3.11 or a compatible version. From the project root, create and
activate a virtual environment, then install the project dependencies:

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

macOS / Linux:

```bash
source .venv/bin/activate
```

Install dependencies and start the API:

```bash
python -m pip install -r requirements.txt
python app.py
```

The service listens at `http://localhost:8000`. Interactive API documentation
is available at `http://localhost:8000/docs`.

## Make a prediction

Send a `POST` request to `/predictions` with the following fields:

```json
{
  "vendor_id": 1,
  "passenger_count": 1,
  "pickup_longitude": -73.985,
  "pickup_latitude": 40.748,
  "dropoff_longitude": -73.977,
  "dropoff_latitude": 40.752,
  "pickup_hour": 10,
  "pickup_date": 15,
  "pickup_month": 5,
  "pickup_day": 1,
  "is_weekend": 0,
  "haversine_distance": 0.8,
  "euclidean_distance": 0.009,
  "manhattan_distance": 0.012
}
```

Example request:

```bash
curl -X POST http://localhost:8000/predictions \
  -H "Content-Type: application/json" \
  -d '{"vendor_id":1,"passenger_count":1,"pickup_longitude":-73.985,"pickup_latitude":40.748,"dropoff_longitude":-73.977,"dropoff_latitude":40.752,"pickup_hour":10,"pickup_date":15,"pickup_month":5,"pickup_day":1,"is_weekend":0,"haversine_distance":0.8,"euclidean_distance":0.009,"manhattan_distance":0.012}'
```

The response body is a JSON string, for example:

```json
"Trip duration for the trip is 12.34 minutes"
```

## Run with Docker

With Docker Desktop (or Docker Engine and the Compose plugin) installed, run:

```bash
docker compose up --build
```

The API will be available at `http://localhost:8000`. Stop the service with
`Ctrl+C`, or run `docker compose down` in another terminal.

## Reproduce the training pipeline

The DVC pipeline is defined in `dvc.yaml`. The raw taxi archive is not included
in this repository; `data/raw/zipped.dvc` is a DVC pointer to it. Configure
access to the DVC remote that stores the data, then pull the data and reproduce
the pipeline:

```bash
python -m pip install -r requirements.txt -r requirements-dev.txt
python -m pip install -e .
dvc pull
dvc repro
```

The training scripts write generated models and transformers under `models/`.
The API currently loads the serving artifacts from `container_models/`.

## Project layout

```text
app.py                  FastAPI application
data_models.py          Prediction request schema
container_models/       Model and preprocessing artifacts used by the API
data/                   Dataset locations and DVC tracking files
dvc.yaml                Data and model pipeline
notebooks/              Exploratory analysis
src/data/               Data extraction and dataset preparation
src/features/           Feature engineering and preprocessing
src/models/             Model training and prediction scripts
src/visualization/      Visualization scripts
```

## Tech stack

Python, pandas, NumPy, scikit-learn, XGBoost, FastAPI, Uvicorn, Docker, and DVC.

## License

This project is distributed under the MIT License. See [LICENSE](LICENSE).
