Certainly. Here is the `README.md` content again, with the formatting corrected so it should render properly.

# AI Code Assistant Agent

This project is a command-line AI assistant powered by Google's Gemini family of models. It's designed as an "agent" that can intelligently solve problems by using a loop of function calls (tools) to interact with the local file system, read files, and apply changes.

It is built to be a simple, extendable, and observable agent for code-related tasks.

## Features

  * **Agentic Loop**: Uses a multi-step loop to think, act, and observe, allowing it to handle complex, multi-step tasks.
  * **Tool-Enabled**: Comes with built-in functions to interact with the file system, such as listing files, reading file content, and patching code.
  * **CLI Interface**: Interact with the agent directly from your terminal.
  * **Verbose Mode**: A `--verbose` flag to see the agent's thought process, including token counts, function calls, and tool outputs.
  * **Configurable**: Easily set the maximum number of iterations in `config.py` to prevent infinite loops.

-----

## Project Structure

```
.
├── calculator/       # Example code for the agent to work on
│   └── calculator.py
├── functions/        # Contains the implementation of tools
│   ├── __init__.py
│   ├── cat.py
│   ├── ls.py
│   └── patch.py
├── .env              # (User-created) For storing API keys
├── .gitignore
├── README.md         # This file
├── call_function.py  # Maps model's function calls to Python functions
├── config.py         # Configuration (e.g., MAX_ITERS)
├── main.py           # Main entry point for the CLI application
├── prompts.py        # Contains the main system prompt for the AI
├── requirements.txt  # Project dependencies
├── tests.py          # (Optional) Tests for the agent/tools
└── ...
```

-----

## Setup

1.  **Clone the Repository**

    ```bash
    git clone https://github.com/1920xl080/AiAgent.git
    cd AiAgent
    ```

2.  **Install Dependencies**
    This project uses `requirements.txt` to manage dependencies.

    ```bash
    pip install -r requirements.txt
    ```

    *This will install `google-generativeai`, `python-dotenv`, and other necessary packages.*

3.  **Set Up Your Environment**
    Create a file named `.env` in the root of the project directory and add your Google Gemini API key:

    ```.env
    GEMINI_API_KEY="YOUR_API_KEY_HERE"
    ```

-----

## Usage

Run the assistant by passing your prompt as a string argument in your terminal.

### Standard Usage

Pass your request to the agent in quotes. The agent will use its tools to analyze the situation and provide a final answer.

**Example: Asking the agent to fix a bug in the `calculator/` directory.**

```bash
python main.py "The calculator.py file in the calculator directory has a bug. Can you find it and fix it?"
```

**Example: Asking the agent to list files.**

```bash
python main.py "What files are in the root directory?"
```

### Verbose Mode

Use the `--verbose` flag to see the agent's full thought process, including which tools it calls, what the tool outputs are, and token usage for each step. This is highly recommended for debugging and understanding the agent's logic.

```bash
python main.py "What is in the 'functions' directory?" --verbose
```

**Example Verbose Output:**

```
User prompt: What is in the 'functions' directory?

Prompt tokens: 123
Response tokens: 45
-> function call: ls(path='functions')
-> function response: {'files': ['__init__.py', 'cat.py', 'ls.py', 'patch.py']}
Prompt tokens: 178
Response tokens: 32
Final response:
The 'functions' directory contains: __init__.py, cat.py, ls.py, and patch.py.
```

-----

## How It Works

1.  **Initialization (`main.py`)**: The script parses the user's command-line prompt and the `--verbose` flag. It loads the `GEMINI_API_KEY` from the `.env` file.
2.  **System Prompt (`prompts.py`)**: It sends the user's prompt along with a detailed `system_prompt` to the Gemini model. This system prompt instructs the AI on how to behave, what tools it has, and how to format its responses.
3.  **Function Definitions (`call_function.py` & `functions/`)**: The `available_functions` list tells the model what tools it can use (e.g., `ls`, `cat`, `patch`).
4.  **Agent Loop (`main.py`)**:
      * The model receives the prompt and, instead of answering directly, it may issue a `function_call` (e.g., `ls(path='.')`).
      * The `call_function.py` script intercepts this call, maps it to the actual Python function (e.g., `functions.ls.ls_files`), and executes it.
      * The *output* of that function (e.g., a list of files) is sent back to the model.
      * The model "observes" this new information and decides what to do next: either call another function (e.g., `cat(path='calculator/calculator.py')`) or, once it has enough information, generate the `final_response`.
5.  **Iteration Limit (`config.py`)**: The `MAX_ITERS` variable ensures the agent doesn't get stuck in an infinite loop, exiting after a set number of steps.
