# Receipt Splitter Agent (LangGraph)

A ReAct agent built with **LangGraph** and **Gemini** that reads a real restaurant receipt from an image and splits the bill between people, including a tip. The agent decides on its own which tools to call: it reads the receipt first, then calls a calculation tool, then answers.

Module 9 homework: Frameworks for AI Agents (LangChain, LangGraph).

## How it works

```
START → assistant → (tool call?) → tools → assistant → … → END
```

- **State**: `input_file` (receipt image path) and `messages` (full conversation history)
- **assistant node**: the LLM with bound tools decides to answer or call a tool
- **tools node** (`ToolNode`): runs the requested tool and returns the result
- **tools_condition**: routes to `tools` or to `END`
- **Loop**: after each tool the flow returns to the assistant (ReAct: Thought → Action → Observation)

## Tools

| Tool | What it does |
|---|---|
| `extract_text(img_path)` | Reads the receipt image with a multimodal model |
| `multiply(a, b)` / `divide(a, b)` | Basic calculations |
| `split_bill(total, people, tip_percent)` | Adds the tip and returns the amount per person |

## Tech stack

Google Colab · LangGraph · LangChain (`langchain-google-genai`) · Gemini 3.5 Flash Lite · Hugging Face Datasets · Colab Secrets

## Example run

**Question:** We are 3 people and want to leave a 10% tip. How much should each person pay?

1. `extract_text` → TOTAL 302,016 IDR
2. `split_bill(302016, 3, 10)` → 110,739.2
3. **Answer:** each person pays **IDR 110,739.20**

The extracted receipt matches the dataset ground truth.

## Lessons learned

- The model calculated the tip itself instead of using `multiply`, so a system prompt does not fully guarantee tool usage.
- Handled model deprecation (404), server overload (503) and free-tier daily quota (429) by switching to a model with a higher limit.

## Data source

[CORD v2](https://huggingface.co/datasets/naver-clova-ix/cord-v2) by NAVER CLOVA (Hugging Face), real restaurant receipts, license CC BY 4.0.

Based on the structure of the [Hugging Face Agents Course](https://huggingface.co/learn/agents-course) LangGraph example.
