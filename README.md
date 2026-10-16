# AgentBuilder – Azure OpenAI Agents Example

This repo contains a minimal Python example that mirrors the workflow from Microsoft’s guide on using the OpenAI Agents SDK with Azure OpenAI and Azure API Management. It shows how to load credentials from `.env`, build a small multi-agent orchestration, and call an Azure OpenAI deployment through the SDK’s `OpenAIResponsesModel`.

## Prerequisites
- Python 3.12+
- An Azure OpenAI resource with a deployed model (e.g., `gpt-5.2`)
- (Optional) A virtual environment tool such as `venv`

## Setup
1. **Install dependencies**
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
2. **Configure environment variables**
   Copy `.env.example` (or create `.env`) and set:
   ```env
   AZURE_OPENAI_API_KEY=<your Azure OpenAI key>
   AZURE_OPENAI_API_VERSION=<supported version, e.g., 2024-08-01-preview>
   AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
   AZURE_OPENAI_DEPLOYMENT=<deployment name, e.g., gpt-5.2>
   ```
   These values are read in `main.py` via `python-dotenv`.

## Running the demo
```bash
source .venv/bin/activate
python main.py
```
`main.py` loads the Azure credentials, initializes an `AsyncAzureOpenAI` client, and runs a triage agent that chooses between math and history tutors. A homework guardrail is included to mimic the behavior described in the Microsoft article, but it is commented out in `main.py`—uncomment the triage agent's `input_guardrails` block to block non-homework inputs and observe how the SDK surfaces guardrail results.

## Reference
- “Use Azure OpenAI and APIM with the OpenAI Agents SDK” – Microsoft Tech Community: https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/use-azure-openai-and-apim-with-the-openai-agents-sdk/4392537
- OpenAI Agents SDK Python docs: https://openai.github.io/openai-agents-python/
