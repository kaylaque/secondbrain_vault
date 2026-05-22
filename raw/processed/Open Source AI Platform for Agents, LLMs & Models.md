---
title: "Open Source AI Platform for Agents, LLMs & Models"
source: "https://mlflow.org/docs/latest/genai/eval-monitor/"
author:
published:
created: 2026-05-20
description: "Systematically measure, improve, and monitor the quality of your agent and LLM applications with MLflow's built-in and custom scorers."
tags:
  - "clippings"
---
MLflow's evaluation and monitoring capabilities help you systematically measure, improve, and maintain the quality of your LLM applications and AI agents throughout their lifecycle from development through production.

<video src="https://mlflow.org/docs/latest/images/mlflow-3/eval-monitor/evaluation-result-video.mp4" controls="" aria-label="Prompt Evaluation"></video>  

> [!-info] -info
> **Try the MLflow LLMs and Agents Demo**
> 
> The quickest way to learn about MLflow for LLMs and AI Agents is to try the demo. **Click to launch the demo ↓**

A core tenet of MLflow's evaluation capabilities is **Evaluation-Driven Development**. This is an emerging practice to tackle the challenge of building high-quality LLM/Agentic applications. MLflow is an open source AI engineering platform that is designed to support this practice and help you quickly build production-quality AI agents and LLM applications.

![Evaluation Driven Development](https://mlflow.org/docs/latest/images/mlflow-3/eval-monitor/evaluation-driven-development.png)

## Key Capabilities

## Running an Evaluation

Each evaluation is defined by three components:

| Component | Example |
| --- | --- |
| **Dataset**   Inputs & expectations (and optionally pre-generated outputs and traces) | ```markdown [   {"inputs": {"question": "2+2"}, "expectations": {"answer": "4"}},   {"inputs": {"question": "2+3"}, "expectations": {"answer": "5"}} ] ``` |
| **Scorer**   Evaluation criteria | ```markdown @scorer def exact_match(expectations, outputs):     return expectations == outputs ``` |
| **Predict Function**   Generates outputs for the dataset | ```markdown def predict_fn(question: str) -> str:     response = client.chat.completions.create(         model="gpt-4o-mini",         messages=[{"role": "user", "content": question}]     )     return response.choices[0].message.content ``` |

The following example shows a simple evaluation of a dataset of questions and expected answers.

```python
import os
import openai
import mlflow
from mlflow.genai.scorers import Correctness, Guidelines

client = openai.OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# 1. Define a simple QA dataset
dataset = [
    {
        "inputs": {"question": "Can MLflow manage prompts?"},
        "expectations": {"expected_response": "Yes!"},
    },
    {
        "inputs": {"question": "Can MLflow create a taco for my lunch?"},
        "expectations": {"expected_response": "No, unfortunately, MLflow is not a taco maker."},
    },
]

# 2. Define a prediction function to generate responses
def predict_fn(question: str) -> str:
    response = client.chat.completions.create(
        model="gpt-4o-mini", messages=[{"role": "user", "content": question}]
    )
    return response.choices[0].message.content

# 3.Run the evaluation
results = mlflow.genai.evaluate(
    data=dataset,
    predict_fn=predict_fn,
    scorers=[
        # Built-in LLM judge
        Correctness(),
        # Custom criteria using LLM judge
        Guidelines(name="is_english", guidelines="The answer must be in English"),
    ],
)
```

## Review the results

Open the MLflow UI to review the evaluation results. You can use the following command to start the UI:

```bash
mlflow server --port 5000
```

You should see a new evaluation run is created under the "Runs" tab. Click on the run name to view the evaluation results.

![Evaluation Results](https://mlflow.org/docs/latest/images/mlflow-3/eval-monitor/quickstart-eval-hero.png)

## Next Steps

### [Quickstart](https://mlflow.org/docs/latest/genai/eval-monitor/quickstart/)

Learn MLflow's evaluation workflow in action.

Start evaluating →

### [Evaluate Agents](https://mlflow.org/docs/latest/genai/eval-monitor/running-evaluation/agents/)

Evaluate AI agents with specialized techniques and custom scorers.

Evaluate agents →

### [Building Scorers](https://mlflow.org/docs/latest/genai/eval-monitor/scorers/)

Get started with MLflow's powerful scorers for evaluating qualities.

Learn about scorers →