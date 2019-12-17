# Pluralsight Tech Blog - Converting Flask to FastAPI for Serving ML Models

example code of converting a Flask API serving a machine-learning model to FastAPI.

Used for the blog post [Porting Flask to FastAPI for ML Model Serving](https://pluralsight.com/tech-blog/porting-flask-to-fastapi-for-ml-model-serving/)

## running the API

This code presents two different API services - a simple Flask app or a FastAPI conversion of the same.

### requirements

This code is entirely dockerized - you should only need Docker and `docker-compose` installed on your machine.

### installation

simply run

```bash
$ docker-compose build
```

from the root directory of the project to build all the included services (based on the `python:3.7-slim` image).

### creating the model

This app serves a simple Naive Bayes model running on the "20 newsgroups" dataset included in scikit-learn, subselected to include the `rec.sport.hockey`, `sci.space`, and `talk.politics.misc` groups for three-way classification.  To generate the serialized model, run

```bash
$ ./ml-models/generate_model.sh
```

from the project root.
This will run a one-off script from within a disposable Flask base container to download the data and train the model.
You should expect model accuracy in the neighborhood of 90%.

### launch the API

the API variants can be triggered by launching the service with one of

```bash
$ docker-compose up [-d] flask-app-[dev|prod]
$ docker-compose up [-d] fastapi-app-[dev|prod]
```

Each of these will launch the app on `http://localhost:${PORT}` for ports 5000 and 5001 respectively.
The `dev` service launches in debug/development mode, running Flask's built-in `werkzeug` server or a single direct `uvicorn` worker for Flask and FastAPI respectively.
The `prod` service runs a production-capable `gunicorn` server with appropriate settings and worker type for Flask or FastAPI.

## API Endpoints

Each API presents the following endpoints.
Note that the FastAPI app also presents autogenerated documentation on http://localhost:5001/docs and http://localhost:5001/redoc.

#### `GET /healthcheck`

Simple GET request to check that the server is running.

Response body:

```json
{
  "message": "this sentence is already halfway over, and still hasn't said anything at all"
}
```

#### `POST /predict`

for each sample, predicts the most likely class and returns its probability.

Request body:

```json
{
  "samples": [
    {
      "text": "this is a target text for prediction"
    },
    ...
  ]
}
```

Response body:

```json
{
  "predictions": [<str>],
  "probabilities": [<float>]
}
```

where each element of `predictions` is one of `rec.sport.hockey`, `sci.space`, or `talk.politics.misc` and each element of `probabilities` is a float between 0 and 1 for the predicted class probability.

#### `POST /predict/:label`

for each sample, predicts the probability for the selected class `:label`.  The route parameter `:label` must be one of `rec.sport.hockey`, `sci.space`, or `talk.politics.misc`.

Request body:

```json
{
  "samples": [
    {
      "text": "this is a target text for prediction"
    },
    ...
  ]
}
```

Response body:

```json
{
  "label": <str>,
  "probabilities": [<float>]
}
```

returning the selected class label and a list of the probabilities of that class label for each sample.

#### `GET /cat-facts`

In addition to the endpoints above, the `fastapi-app` services also presents an endpoint to asynchronously request a random Cat Fact from https://cat-fact.herokuapp.com.

Response body:

```json
{
  "text": <str>
}
```

## License

This code is licensed under the [Apache 2.0 license](https://github.com/pluralsight/tech-blog-fastapi-demo/blob/master/LICENSE).
