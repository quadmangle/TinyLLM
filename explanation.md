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
