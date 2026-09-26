# Automated Java Test Suite Generator & Coverage Analyzer

An intelligent multi-agent framework designed to analyze Java source code, evaluate test coverage gaps, discover boundary conditions, and synthesize comprehensive, reviewed JUnit test suites using local Large Language Models (LLMs).

The system pairs individual diagnostic utilities with an orchestrated LangGraph multi-agent pipeline powered by Ollama, enabling automated reasoning over control flows, branch complexity, and edge-case validation without relying on external cloud APIs.

---

## Built With / Tech Stack

- [Python](https://www.python.org/): Core programming language for tooling and agent runtime.
- [LangChain](https://www.langchain.com/): Framework for constructing prompts, handling LLM abstractions, and managing chat message schemas.
- [LangGraph](https://langchain-ai.github.io/langgraph/): Graph-based multi-agent orchestration framework for deterministic state machines and workflows.
- [langchain-ollama](https://python.langchain.com/docs/integrations/chat/ollama/): Integration package providing chat and completion bindings for locally served Ollama models.
- [Ollama](https://ollama.com/): Lightweight local LLM serving framework for executing models such as Llama 3 and Llama 3.2.
- [JUnit](https://junit.org/): Target Java testing framework used for all generated assertions and test fixtures.

---

## Repository Structure

```text
.
├── agent_analisi.py     # Interactive code inspector and test coverage evaluation tool
├── agent_parametri.py   # Parameter discovery agent for edge-case boundary identification
├── agent_test.py        # Complexity-aware standalone JUnit test generation script
├── multi_agent.py       # Orchestrated multi-agent pipeline built with LangGraph
└── README.md            # Project technical documentation
```

### Component Breakdown
- **`multi_agent.py`**: Coordinates a stateful four-stage pipeline (`stage` -> `analysis` -> `test_generation` -> `review_tests`) to produce fully reviewed JUnit tests from input source methods.
- **`agent_test.py`**: Standalone test generator that inspects branch and loop cyclomatic density via regular expressions and generates complete JUnit test cases targeting all branches.
- **`agent_parametri.py`**: Dedicated utility to extract parameter boundaries and partition inputs into regular and exception-handling scenarios.
- **`agent_analisi.py`**: Interactive CLI utility for deep diagnostic analysis of existing Java methods and associated tests, outputting reports to disk.

---

## How It Works / Architecture

The system operates across two operational modalities: standalone specialized agents for focused diagnostic tasks, and a centralized stateful multi-agent pipeline for end-to-end test synthesis.

```
                     +---------------------------------------+
                     |         Input: Java Methods           |
                     +---------------------------------------+
                                         |
                                         v
                     +---------------------------------------+
                     |       Stage 1: Parameter Agent        |
                     |  - Discovers boundary inputs          |
                     |  - Identifies exception scenarios     |
                     +---------------------------------------+
                                         |
                                         v
                     +---------------------------------------+
                     |       Stage 2: Analysis Agent         |
                     |  - Detects uncovered branches         |
                     |  - Diagnoses test suite blind spots   |
                     +---------------------------------------+
                                         |
                                         v
                     +---------------------------------------+
                     |    Stage 3: Test Generation Agent     |
                     |  - Generates full JUnit test suite    |
                     |  - Covers valid and error paths       |
                     +---------------------------------------+
                                         |
                                         v
                     +---------------------------------------+
                     |      Stage 4: Review Agent            |
                     |  - Syntax & logic verification        |
                     |  - Refines assertions & naming        |
                     +---------------------------------------+
                                         |
                                         v
                     +---------------------------------------+
                     |   Output: Refined JUnit Test Suite    |
                     +---------------------------------------+
```

### 1. The Multi-Agent Pipeline (`multi_agent.py`)
Built on LangGraph's `StateGraph`, the execution flow moves through a strictly ordered series of functional nodes sharing a centralized state dictionary:

1. **Parameter Discovery (`agent_stage`)**: Parses the target Java method and optional user context, outputting an extensive matrix of valid boundary inputs and invalid scenarios designed to trigger exceptions without generating actual assertions.
2. **Coverage Gap Analysis (`agent_analysis`)**: Evaluates the functional specification of the method against existing tests to identify missing branches, unhandled states, and coverage deficits.
3. **JUnit Synthesis (`agent_test_generation`)**: Consumes the raw method, recommended parameters, and gap analysis to write comprehensive JUnit tests with explicit assertions and descriptive naming.
4. **Test Review & Refinement (`agent_review_tests`)**: Acts as a senior QA reviewer, scanning the generated test suite for syntax errors, logical fallacies, assertion consistency, and naming conventions before returning the final test code.

### 2. Standalone Utility Agents
- **Complexity-Aware Generator (`agent_test.py`)**: Uses syntactic analysis to quantify conditional (`if`, `else`, `switch`) and loop constructs (`for`, `while`). It dynamically tunes the system prompt to enforce branch-coverage guarantees for high-complexity methods.
- **Boundary Explorer (`agent_parametri.py`)**: Focuses on input space partitioning, isolating valid execution paths from exception-handling paths.
- **Diagnostic Inspector (`agent_analisi.py`)**: Provides interactive analysis sessions for querying Java method responsibilities, validating test adequacy, and writing diagnostic summaries to `analisi_risultato.txt`.

---

## Prerequisites & Installation

### Prerequisites
- **Python**: Version `3.10` or higher
- **Ollama**: Installed and running locally
- **Ollama Models**:
  - `llama3.2` (primary model for generation and multi-agent workflow)
  - `llama3` (used by `agent_analisi.py`)

### Step-by-Step Installation

1. **Clone the repository and enter the directory**:
   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. **Create and activate a Python virtual environment**:
   - On Linux/macOS:
     ```bash
     python3 -m venv venv
     source venv/bin/activate
     ```
   - On Windows (Command Prompt):
     ```cmd
     python -m venv venv
     venv\Scripts\activate.bat
     ```

3. **Install the required dependencies**:
   ```bash
   pip install langchain langchain-ollama langgraph
   ```

4. **Pull the required models via Ollama**:
   Ensure the Ollama daemon is active, then download the model weights:
   ```bash
   ollama pull llama3.2
   ollama pull llama3
   ```

---

## Usage / Getting Started

### 1. Running the Full Multi-Agent Workflow
The multi-agent pipeline accepts a text file containing the Java method (and optionally, existing tests) and guides you through generation and review.

```bash
python multi_agent.py
```

**Workflow Prompt Example**:
1. Enter the path to your source file (e.g., `methods.txt`).
2. Specify whether to provide additional context (`s` for yes, `n` for no).
3. The pipeline will execute sequentially and print the extracted parameters, gap analysis, and the finalized, reviewed JUnit tests.

---

### 2. Standalone Complexity-Aware Test Generation
Pass the input file path directly as a command-line argument:

```bash
python agent_test.py input_method.txt
```

The script parses branches and loops, tailors the prompt constraints, and outputs ready-to-use JUnit `@Test` methods.

---

### 3. Standalone Parameter & Boundary Exploration
Run the parameter exploration tool to discover edge-case inputs without running full test synthesis:

```bash
python agent_parametri.py
```

Enter the path to the text file containing the Java code when prompted, along with any optional contextual notes.

---

### 4. Interactive Test Coverage Diagnostics
Run the diagnostic inspector to analyze method logic and inspect test coverage:

```bash
python agent_analisi.py
```

1. Paste the Java method into the terminal. Press **Enter on an empty line** to complete input.
2. Paste existing test code into the terminal. Press **Enter on an empty line** to complete input.
3. Select an option from the menu:
   - `1`: Describe the functionality of the Java method.
   - `2`: Verify test coverage completeness and identify gaps.
   - `3`: Suggest improvements for the test suite.
   - `4`: Exit.
4. Analysis outputs are displayed in the terminal and appended to `analisi_risultato.txt`.

---

## Authors

- **Simone Abate**
- **Alessandro Romeo**
- **Alessandro Nappi**
