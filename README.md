
# Automating Flask App Development with GitHub Actions

This repository demonstrates how to automate the development, testing, and deployment of a Flask application using GitHub Actions. The automation process involves creating a CI/CD pipeline that builds the application, runs tests, creates a Docker image, and pushes it to Docker Hub.

## Table of Contents

- [Project Setup](#project-setup)
- [GitHub Actions Workflow](#github-actions-workflow)
  - [Build and Test Job](#build-and-test-job)
  - [Deploy Job](#deploy-job)
- [Automation in Action](#automation-in-action)
- [Benefits of Automation](#benefits-of-automation)
- [Additional Insights](#additional-insights)
- [Conclusion](#conclusion)

## Project Setup

To get started with the project, follow these steps:

1. **Create a GitHub Repository**  
   Start by creating a new repository on GitHub.

2. **Initialize Your Project**  
   - Create a `requirements.txt` file listing the necessary dependencies (e.g., Flask, Pytest).
   - Add a `app.py` file containing the Flask application code.
   - Create a `test_app.py` file for unit tests.
   - Add a `Dockerfile` to build the Docker image for your app.

Example of **requirements.txt**:
```
Flask==2.2.3
Pytest==7.2.0
```

Example of **app.py**:
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

Example of **test_app.py**:
```python
from app import app
import pytest

@pytest.fixture
def client():
    with app.test_client() as client:
        yield client

def test_hello_world(client):
    response = client.get('/')
    assert response.data == b'Hello, World!'
```

Example of **Dockerfile**:
```Dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

CMD ["python", "app.py"]
```

## GitHub Actions Workflow

In this project, we will create a GitHub Actions workflow to automate the CI/CD pipeline.

### Create the Workflow File

Create a file at `.github/workflows/ci-cd.yml` to define the GitHub Actions workflow.

### Build and Test Job

This job automates the following steps:
- **Checkout Code**: Pulls the latest code from the repository.
- **Set up Python Environment**: Configures Python and installs dependencies.
- **Run Tests**: Executes unit tests with Pytest.
- **Build Docker Image**: Builds the Docker image using the Dockerfile.

Example of `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        run: |
          pytest test_app.py

      - name: Build Docker image
        run: |
          docker build -t flask-app .
          
  deploy:
    runs-on: ubuntu-latest
    needs: build-and-test
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Push Docker image
        run: |
          docker tag flask-app ${{ secrets.DOCKER_USERNAME }}/flask-app:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/flask-app:latest
```

### Deploy Job

This job automates the deployment process:
- **Configure Docker Hub Credentials**: Set Docker Hub credentials as GitHub secrets (`DOCKER_USERNAME` and `DOCKER_PASSWORD`).
- **Push Docker Image to Docker Hub**: If the tests pass, the Docker image is tagged and pushed to Docker Hub.

## Automation in Action

Once set up, GitHub Actions will automatically trigger every time code is pushed to the `main` branch:
- The workflow builds the Flask application, runs the unit tests, and creates a Docker image.
- If the tests pass, the Docker image is pushed to Docker Hub.

## Benefits of Automation

- **Efficiency**: Automates repetitive tasks such as building, testing, and deploying.
- **Quality**: Ensures code quality with automated testing using Pytest.
- **Consistency**: Maintains consistent and reliable deployment processes.
- **Faster Time to Market**: Speeds up the deployment process.

## Additional Insights

- The workflow integrates both **Continuous Integration (CI)** and **Continuous Deployment (CD)**.
- Each step of the process is automated and requires minimal manual intervention.
- GitHub Actions allows you to automate not only testing but also deployment, making it easier to manage your production workflows.

## Conclusion

By following the steps outlined in this guide, you can easily set up an automated CI/CD pipeline for your Flask application using GitHub Actions. This ensures your code is continuously tested, built, and deployed efficiently and reliably.

---

### How to Contribute

1. Fork the repository.
2. Create a new branch (`git checkout -b feature-branch`).
3. Make your changes and commit them (`git commit -am 'Add new feature'`).
4. Push to the branch (`git push origin feature-branch`).
5. Create a new Pull Request.

