# Paperless-ngx with Paperless-AI and a Local Ollama Instance

This repository provides a `docker-compose` setup to run [Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx) alongside [Paperless-AI](https://github.com/clusterzx/paperless-ai), using an Ollama instance running directly on the host machine for AI-powered features like OCR and document analysis.

The primary goal of this configuration is to provide a stable and working solution that bypasses current compatibility issues with Paperless-AI's built-in Docker Model Runner.

## Docker Model Runner Compatibility

The issue with using Paperless-AI and Docker Desktop's integrated **Model Runner** is a fundamental **API incompatibility**, not a network problem. Although the Paperless-AI interface may show a "connection error," network connectivity is functional. The root cause is that Docker's Model Runner does not fully adhere to the OpenAI API specification that the Paperless-AI client requires. As detailed in **[GitHub Issue #719](https://github.com/clusterzx/paperless-ai/issues/719)**, debugging shows two key failures:

1. The `/v1/models` endpoint returns a plain text response instead of the expected JSON data, causing the initial connection test to fail.
2. The essential `/v1/chat/completions` endpoint is not implemented and returns a `404 Not Found` error, making the service unusable for AI tasks.

Until official support for the Docker Model Runner is added to Paperless-AI, the host-based Ollama setup described in this guide is the recommended workaround.

## Host-Based Ollama

This setup offers a robust workaround by leveraging Docker's networking capabilities to connect a containerized application to a service running on the host machine.

* **Paperless-ngx & Paperless-AI:** Both applications run as services within a Docker environment, isolated from the host but able to communicate with each other over a dedicated Docker network (`paperless-net`).
* **Ollama:** Ollama is installed and runs directly on your host operating system (Windows, macOS, or Linux). This gives it direct access to system resources, including GPUs for faster inference.
* **The Connection:** The Paperless-AI container connects to the host's Ollama instance using the special DNS name `host.docker.internal`. This name resolves to the internal IP address of the host machine, allowing the container to access services listening on the host's network interfaces, like the Ollama API on port `11434`.

This architecture provides the stability of a native Ollama installation while maintaining the convenience and isolation of Docker for the Paperless services.

## Prerequisites

Before you begin, ensure you have the following installed on your host machine:
* [Docker](https://docs.docker.com/get-docker/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* [Ollama](https://ollama.com/)

## Installation & Configuration Guide

Follow these steps to get the entire stack up and running.

### Step 1: Install and Run Ollama on the Host
First, install Ollama on your host machine by following the official instructions on their website. Once installed, open a terminal or command prompt and pull the vision model you intend to use. For this guide, we will use `llama3.2-vision`.

```bash
# This command will download and run the model.
# The first run may take some time depending on your internet connection.
ollama run llama3.2-vision
```
After the model is downloaded, Ollama will be running in the background, serving the model via an API on `http://localhost:11434`.

### Step 2: Prepare the Docker Environment
Clone this repository or create the `docker-compose.yml` file provided below. Then, create the dedicated bridge network that will allow the containers to communicate.

```bash
# Create the Docker network
docker network create -d bridge paperless-net
```

### Step 3: Launch the Docker Containers
With the network created, you can now pull the required container images and start the services.

```bash
# Pull the latest versions of the images
docker compose pull

# Start all services in detached mode
docker compose up -d
```

### Step 4: Configure Paperless-ngx
Navigate to the Paperless-ngx web interface in your browser, typically at **`http://localhost:8000`**.

1.  Complete the initial setup by creating a superuser.
2.  Log in with your new credentials.
3.  Click on your username in the top right corner and go to **Profile**.
4.  Under the **API Tokens** section, create a new token. Give it a descriptive name (e.g., `paperless-ai-token`) and copy the generated token to your clipboard.

### Step 5: Configure Paperless-AI
Now, navigate to the Paperless-AI web interface, typically at **`http://localhost:3000`**. You will be presented with a setup screen. Fill in the details as follows:

* **Paperless-ngx URL:** `http://webserver:8000`
    * **Explanation:** This is the internal service name for the Paperless-ngx container as defined in the `docker-compose.yml` file. The containers communicate over the `paperless-net` network, not via `localhost` 
* **Username:** The username of the Paperless-ngx user associated with the API token.
* **API Token:** Paste the token you created in the previous step.
* **AI Provider:** Select **Ollama**.
* **API URL:** `http://host.docker.internal:11434`
    * **Explanation:** This is the key to this setup. `host.docker.internal` is a special DNS name that Docker resolves to the IP address of the host machine, allowing the Paperless-AI container to reach the Ollama service running outside of Docker.
* **Model Name:** `llama3.2-vision`
    * **Explanation:** This must exactly match the name of the model you pulled with the `ollama run` command.

Click **Submit** to save the configuration. Paperless-AI is now connected to both your Paperless-ngx instance and your host's Ollama instance.