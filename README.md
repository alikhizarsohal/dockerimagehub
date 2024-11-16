
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
Flask
```

Example of **app.py**:
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, Flask!"

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000)

```

Example of **test_app.py**:
```python
from app import app

def test_home():
    # Test the home route
    response = app.test_client().get("/")
    
    assert response.status_code==200  # Check for HTTP 200 status
    assert response.data==b"Hello, Flask!"  # Check the response content

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
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    # Step 1: Check out the code
    - name: Checkout Code
      uses: actions/checkout@v3

    # Step 2: Set up Python
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'

    # Step 3: Install dependencies
    - name: Install Dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    # Step 4: Run tests
    - name: Run Tests
      run: python test.py

  docker-build:
    runs-on: ubuntu-latest
    needs: test

    steps:
    # Step 1: Check out the code
    - name: Checkout Code
      uses: actions/checkout@v3

    # Step 2: Set up Docker Buildx (required for multi-platform builds and pushing)
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    # Step 3: Log in to Docker Hub (optional)
    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    # Step 4: Build and push Docker image
    - name: Build and Push Docker Image
      id: docker_build  # Assign an ID to this step
      uses: docker/build-push-action@v4
      with:
        context: .  # Use the current directory for the context
        file: DockerFile  # Ensure it is named exactly 'Dockerfile'
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/flask-app:latest

    # Step 5: Echo Docker Image Digest
    - name: Output Docker Image Digest
      run: |
        echo "Docker Image Digest: ${{ steps.docker_build.outputs.digest }}"
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

