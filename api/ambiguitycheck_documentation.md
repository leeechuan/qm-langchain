# Ambiguity Check API Documentation

This document explains how to run the `api/ambiguitycheck.py` script and details the functionality of each step within the code.

## Overview

The `ambiguitycheck.py` script is a Flask application that provides an API endpoint to analyze business requirements. It uses OpenAI's GPT models via the LangChain library to determine if a given set of requirements is ambiguous or if it's sufficient to create user stories.

## Prerequisites

Before running the script, ensure you have the following:

1.  **Python**: Installed on your system.
2.  **OpenAI API Key**: You need a valid API key from OpenAI.
3.  **Dependencies**: The required Python packages listed in `requirements.txt`.

## Setup and Installation

1.  **Install Dependencies**:
    Navigate to the project root directory and install the required packages:
    ```bash
    pip install -r requirements.txt
    ```

2.  **Environment Configuration**:
    Create a `.env` file in the `api` directory (or root, depending on where you run it from) and add your OpenAI API key:
    ```env
    OPENAI_API_KEY=your_api_key_here
    ```

## How to Run

To start the Flask server, run the following command from the project root:

```bash
python api/ambiguitycheck.py
```

*Note: The server will start in debug mode on `http://127.0.0.1:5000` by default.*

## Code Breakdown

### 1. Imports
```python
from flask import Flask, request, jsonify
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.chains import SequentialChain, LLMChain
from dotenv import load_dotenv
import os
```
-   **Flask**: Web framework to create the API.
-   **LangChain**: Libraries to interact with LLMs (Large Language Models).
-   **dotenv**: Loads environment variables from a `.env` file.

### 2. App Initialization
```python
app = Flask(__name__)
```
-   Initializes the Flask application instance.

### 3. API Route (`/langchain/ambiguitycheck`)
```python
@app.route('/langchain/ambiguitycheck', methods=['POST'])
def check_ambiguity():
    load_dotenv()
    try:
        data = request.json
        result = process_data(data)
        return jsonify({"result": result})
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```
-   **Route**: Defines a POST endpoint at `/langchain/ambiguitycheck`.
-   **load_dotenv()**: Loads the API key from the environment.
-   **Logic**: Accepts JSON input, calls `process_data`, and returns the result. Handles errors gracefully.

### 4. Core Logic (`process_data`)
```python
def process_data(data):
    llm = ChatOpenAI(openai_api_key = os.getenv("OPENAI_API_KEY"))
    
    template1 = "..." # Prompt for the Business Analyst persona
    prompt1 = ChatPromptTemplate.from_template(template1)
    
    chain_1 = LLMChain(llm=llm, prompt=prompt1, output_key="review_ambiguity")
    
    seq_chain = SequentialChain(chains=[chain_1],
                        input_variables=['statement'],
                        output_variables=['review_ambiguity'],
                        verbose=True)

    results = seq_chain(data)
    return results['review_ambiguity']
```
-   **LLM Setup**: Initializes `ChatOpenAI` with the key.
-   **Prompt**: Defines a prompt instructing the AI to act as a Business Analyst. It asks the AI to check if the input `statement` has enough information or is ambiguous.
-   **Chain**: Creates a `SequentialChain` (currently with one step) to process the input `statement` through the LLM.
-   **Execution**: Runs the chain with the input data and returns the text response.

### 5. Server Startup
```python
app.run(debug=True)
```
-   Starts the Flask application.

## Usage Example

Once the server is running, you can test it using `curl`:

```bash
curl -X POST http://127.0.0.1:5000/langchain/ambiguitycheck \
     -H "Content-Type: application/json" \
     -d '{"statement": "The user should be able to log in."}'
```

**Expected Response**:
The JSON response will contain the AI's analysis of the ambiguity of the statement "The user should be able to log in."
