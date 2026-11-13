# AgentsVille Trip Planner

An LLM-powered travel planner that builds a validated, weather-aware itinerary for a trip to the fictional city of AgentsVille, then iteratively improves it using evaluation tools and traveler feedback.

This is the project from the Udacity course on advanced LLM reasoning. It demonstrates four techniques:

| Technique | Where it's used |
|-----------|-----------------|
| **Role-based prompting** | `ItineraryAgent` acts as a specialist travel planner |
| **Chain-of-thought** | Step-by-step planning over weather, activities and traveler interests |
| **ReAct (Thought → Action → Observation)** | `ItineraryRevisionAgent` loops over tool calls to refine the plan |
| **Feedback loops** | Evals feed their failures back to the agent until the plan passes |

## How it works

```
VacationInfo ──► ItineraryAgent ──► TravelPlan ──► Evals ──► ItineraryRevisionAgent ──► Final TravelPlan
 (Pydantic)      (chain-of-thought)  (Pydantic)   (pass/fail)   (ReAct + tools)          (+ optional audio)
```

1. **Define the trip** – travelers, interests, dates, budget and restrictions are encoded and validated with Pydantic (`Traveler`, `VacationInfo`).
2. **Gather context** – mocked weather and activities APIs supply data for the trip dates.
3. **Generate an itinerary** – `ItineraryAgent` returns a structured `TravelPlan`.
4. **Evaluate it** – programmatic and LLM-based checks score the plan:
   - start/end dates match the trip
   - total cost is accurate and within budget
   - itinerary events match real activities
   - activities satisfy traveler interests
   - activities are compatible with the weather
   - traveler feedback has been incorporated
5. **Revise it** – `ItineraryRevisionAgent` uses a ReAct loop with these tools: a calculator, an activities lookup, an evals runner, and a final-answer tool.
6. **Narrate it (optional)** – `narrate_my_trip` turns the final itinerary into a spoken audio summary.

## Repository contents

| File | Purpose |
|------|---------|
| `project_starter.ipynb` | The main notebook: agents, evals, tools and the end-to-end walkthrough |
| `project_lib.py` | Helpers: `Interest` enum, `ChatAgent` base class, `do_chat_completion`, `print_in_box`, mocked weather/activity APIs, and `narrate_my_trip` |

## Getting started

### Prerequisites

- Python 3.11+
- Jupyter (JupyterLab, VS Code or similar)
- An OpenAI-compatible API key. The notebook points at the Udacity Vocareum proxy (`https://openai.vocareum.com/v1`); change `base_url` to use OpenAI directly.

### Setup

```bash
git clone <this-repo-url>
cd agentVille-trip-planner

python -m venv .venv
source .venv/bin/activate

pip install openai pydantic pandas numexpr json-repair ipython jupyter
```

Set your API key as an environment variable rather than hard-coding it in the notebook:

```bash
export OPENAI_API_KEY="your-api-key"
```

> The starter notebook sets `os.environ["OPENAI_API_KEY"] = "my-api-key"` in its first cells. Replace or remove that line so it doesn't override your real key, and never commit a real key.

### Run

```bash
jupyter lab project_starter.ipynb
```

Run the cells top to bottom. Default models are set near the top of the notebook (`MODEL = OpenAIModel.GPT_41_MINI`); evals use `gpt-4.1-nano` and a stronger `gpt-4.1` for checking traveler feedback.

## Notes

- Weather and activity data come from mocked APIs in `project_lib.py`, so results depend only on the LLM.
- LLM output is non-deterministic: an itinerary may need several revision cycles to pass every eval.
- Audio narration writes an MP3 to `/tmp/my_trip_narration.mp3` by default.
