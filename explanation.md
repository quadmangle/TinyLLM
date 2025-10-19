# TinyLLM Project Explanation

This document provides a comprehensive overview of the TinyLLM project, explaining what it is, how to use it, and how to set it up.

## What is TinyLLM?

TinyLLM is a project designed to help you run a large language model (LLM) on your own computer, even with consumer-grade hardware. It provides a web interface similar to ChatGPT, allowing you to interact with the LLM in a conversational way.

### Key Features

*   **Local LLM Server:** It helps you set up a server on your machine to run various open-source large language models.
*   **Multiple Backend Options:** You can choose from different LLM serving backends like Ollama, vLLM, or llama-cpp-python. Each has its pros and cons regarding performance and hardware requirements.
*   **ChatGPT-like Interface:** It includes a web-based chatbot application that connects to your local LLM server, allowing you to interact with the model in a conversational way.
*   **RAG Functionality:** The chatbot has some Retrieval-Augmented Generation (RAG) features, which means it can access external information. For example, it can summarize websites, get the latest news, check stock prices, and get weather information.

## How to use the TinyLLM Chatbot

Once you have it set up, you can use the chatbot by:

1.  **Opening the Chatbot:** Accessing the web interface in your browser (usually at `http://localhost:5000`).
2.  **Chatting with the LLM:** Typing messages and getting responses from the language model.
3.  **Using Special Commands:**
    *   `/news`: To get a summary of the top 10 news headlines.
    *   `/stock <company>`: To get the current stock price for a company.
    *   `/weather <location>`: To get the current weather conditions.
    *   Pasting a URL: To have the chatbot read and summarize the content of a webpage.

## How to set up TinyLLM

The setup involves two main parts: running an LLM inference server and running the chatbot application.

### 1. Clone the Project

First, you need to get the project files onto your computer.

```bash
git clone https://github.com/jasonacox/TinyLLM.git
cd TinyLLM
```

### 2. Set up an LLM Inference Server

You need to choose one of the following options to serve the LLM.

*   **Option 1: Ollama (Easiest)**
    Ollama is great for beginners and works on various systems (macOS, Linux, Windows).

    ```bash
    # Install and run Ollama server
    docker run -d --gpus=all \
        -v $PWD/ollama:/root/.ollama \
        -p 11434:11434 \
        -p 8000:11434 \
        --restart unless-stopped \
        --name ollama \
        ollama/ollama

    # Download and test run the llama3 model
    docker exec -it ollama ollama run llama3
    ```

*   **Option 2: vLLM (For powerful GPUs)**
    vLLM is a good choice if you have a powerful NVIDIA GPU with plenty of VRAM. It's fast and can handle multiple users at once.

    ```bash
    # Build Container
    cd vllm
    ./build.sh

    # Make a Directory to store Models
    mkdir models

    # Run the Container - This will download the model on the first run
    ./run.sh
    ```

*   **Option 3: llama-cpp-python (CPU or less powerful GPUs)**
    This option is good for running quantized models that are optimized to run on CPUs or GPUs with less VRAM.

    ```bash
    # Install dependencies
    pip3 install 'llama-cpp-python[server]'

    # Download a model
    cd llmserver/models
    wget https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.1-GGUF/resolve/main/mistral-7b-instruct-v0.1.Q5_K_M.gguf

    # Run the server
    python3 -m llama_cpp.server \
        --model ./models/mistral-7b-instruct-v0.1.Q5_K_M.gguf \
        --host localhost
    ```

### 3. Run the Chatbot

After setting up the LLM server, you need to run the chatbot application which will connect to it.

```bash
# Move to the chatbot folder
cd chatbot # (If you are not already there)

# Run the chatbot using Docker (recommended)
docker run \
    -d \
    -p 5000:5000 \
    -e PORT=5000 \
    -e OPENAI_API_BASE="http://localhost:8000/v1" \
    -e LLM_MODEL="tinyllm" \
    -v $PWD/.tinyllm:/app/.tinyllm \
    --name chatbot \
    --restart unless-stopped \
    jasonacox/chatbot
```

Once the chatbot is running, you can open your web browser and go to `http://localhost:5000` to start chatting with your own local LLM.
---

## Detailed Setup Guide for Google Colab

This guide provides step-by-step instructions for setting up and running the TinyLLM project on Google Colab, which offers a free environment with GPU access.

### Step 1: Initial Colab Setup

1.  **Open Google Colab:** Go to [colab.research.google.com](https://colab.research.google.com) and create a new notebook.
2.  **Enable GPU:** In the notebook, go to **Runtime** -> **Change runtime type**. Select **T4 GPU** from the "Hardware accelerator" dropdown and click **Save**.
3.  **Clone the Repository:** Run the following command in a code cell to clone the TinyLLM project.
    ```python
    !git clone https://github.com/jasonacox/TinyLLM.git
    ```
4.  **Navigate into the Project Directory:**
    ```python
    %cd TinyLLM
    ```

### Step 2: Set Up the LLM Inference Server

Choose one of the following two options to set up the backend LLM server.

#### Option A: Ollama (Easiest)

This is the most straightforward method to get an LLM server running.

1.  **Install Ollama:** This command downloads and runs the installation script.
    ```python
    !curl -fsSL https://ollama.com/install.sh | sh
    ```
2.  **Start the Ollama Server:** We run the server as a background process so we can continue using the notebook.
    ```python
    !nohup ollama serve &
    ```
    After running, wait about 10-15 seconds for the server to initialize.
3.  **Download an LLM Model:** This command downloads the `llama3` model. You can choose other models from the [Ollama library](https://ollama.com/library).
    ```python
    !ollama pull llama3
    ```
    The Ollama server is now running in the background and is ready to be used by the chatbot.

#### Option B: `llama-cpp-python` (More Control)

This option is great for running specific GGUF-quantized models and gives you more control over the hardware usage.

1.  **Install Dependencies:** This command installs `llama-cpp-python` and compiles it to use the Colab GPU.
    ```python
    !CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install llama-cpp-python==0.2.27 --no-cache-dir
    !pip install 'llama-cpp-python[server]'
    ```
2.  **Download an LLM Model:** We will download the Mistral 7B model as an example.
    ```python
    # Navigate to the models directory
    %cd llmserver/models
    # Download the model file
    !wget https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.1-GGUF/resolve/main/mistral-7b-instruct-v0.1.Q5_K_M.gguf
    # Navigate back to the project root
    %cd ../..
    ```
3.  **Start the Server:** This command starts the server as a background process. We offload 35 layers (`--n_gpu_layers 35`) to the GPU for better performance.
    ```python
    !nohup python3 -m llama_cpp.server \
        --model ./llmserver/models/mistral-7b-instruct-v0.1.Q5_K_M.gguf \
        --host 0.0.0.0 \
        --port 8000 \
        --n_gpu_layers 35 &
    ```
    The `llama-cpp-python` server is now running and ready for the chatbot.

### Step 3: Run the Chatbot Interface

After setting up one of the LLM servers above, you can start the web-based chatbot.

1.  **Navigate to the Chatbot Directory:**
    ```python
    %cd chatbot
    ```
2.  **Install Chatbot Dependencies:**
    ```python
    !pip install fastapi uvicorn python-socketio jinja2 openai bs4 pypdf requests lxml aiohttp
    ```
3.  **Configure API Settings:** Run the appropriate cell below depending on which LLM server you started.
    *   **For Ollama:**
        ```python
        import os
        os.environ['OPENAI_API_BASE'] = 'http://localhost:11434/v1'
        os.environ['LLM_MODEL'] = 'llama3'
        ```
    *   **For `llama-cpp-python`:**
        ```python
        import os
        os.environ['OPENAI_API_BASE'] = 'http://localhost:8000/v1'
        os.environ['LLM_MODEL'] = 'tinyllm' # The server uses a default name
        ```
4.  **Expose the Chatbot with ngrok:** Colab requires a tunnel to expose the web application to the internet. We'll use `ngrok` for this.

    *   **Install `pyngrok`:**
        ```python
        !pip install pyngrok
        ```
    *   **Run the Chatbot and Get Public URL:**
        ```python
        # Run the chatbot server in the background
        !nohup python3 server.py &

        # Import pyngrok
        from pyngrok import ngrok

        # Terminate any existing tunnels
        ngrok.kill()

        # IMPORTANT: Get your authtoken from https://dashboard.ngrok.com/get-started/your-authtoken
        # and paste it below.
        NGROK_AUTH_TOKEN = "YOUR_NGROK_AUTHTOKEN"
        ngrok.set_auth_token(NGROK_AUTH_TOKEN)

        # Open a tunnel to the chatbot on port 5000 and get the public URL
        public_url = ngrok.connect(5000)
        print(f"Click this link to open the chatbot: {public_url}")
        ```
5.  **Access the Chatbot:** Click the URL printed by the final cell. It will open the TinyLLM chatbot interface, connected to the LLM server you set up. You can now start chatting!
