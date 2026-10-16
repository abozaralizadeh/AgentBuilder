# CLAUDE.md

Guidance for Claude Code (and humans) working in this repository.

## What this is

AgentBuilder is a single-file Python demo of the [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)
against Azure OpenAI, following Microsoft's "Use Azure OpenAI and APIM with the OpenAI Agents SDK"
article (linked in README.md). `main.py` builds an `AsyncAzureOpenAI` client from `.env`, makes it
the SDK's default client, and runs a triage agent that hands off to a math tutor or a history tutor
for two hard-coded prompts. There is no package, CLI or server.

## Commands

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env      # then fill in the AZURE_OPENAI_* values
python main.py            # makes real calls to Azure OpenAI
```

There are no tests, no linter configuration and no CI. The only check is running `main.py` against
a real deployment.

## Layout

```
main.py            the whole demo: client setup, agents, guardrail function, two example runs
requirements.txt   openai, openai-agents, dotenv, asyncio, pydantic (all unpinned)
.env.example       template for .env (Azure OpenAI and Azure APIM variables)
README.md          setup and run instructions
```

## Configuration

`main.py` loads `.env` from the script's own folder (`os.path.dirname(__file__)`), not from the
current directory, and reads four variables with `os.getenv`:

- `AZURE_OPENAI_API_KEY`
- `AZURE_OPENAI_API_VERSION`
- `AZURE_OPENAI_ENDPOINT`
- `AZURE_OPENAI_DEPLOYMENT`

The `AZURE_APIM_OPENAI_*` variables in `.env.example` are not read anywhere: there is no APIM code
path yet. `.env` is git-ignored; real values belong only there.

The Agents SDK also reads its own variables: recent versions take the model name for agents without
a `model` from `OPENAI_DEFAULT_MODEL`, and trace export uses `OPENAI_API_KEY`.

## Conventions / gotchas

- At startup `main.py` prints the API version, endpoint and deployment, and for the key only
  `AZURE_OPENAI_API_KEY set` or `missing`; never print the key itself. The client is built before
  these lines, so with no key (or an empty one) the openai client raises "Missing credentials"
  first and nothing is printed (checked with openai 3.31).
- `model = OpenAIResponsesModel(model="gpt-5.2", ...)` is created, but every `model=model` line on
  the agents is commented out. The agents therefore use the SDK's default model name (which changes
  between SDK releases) through the default client and the Responses API.
- On Azure, the Responses call goes to `{endpoint}/openai/responses?api-version=...` and the
  deployment is chosen by the `model` name in the request body; `azure_deployment` does not enter
  that URL (checked with openai 3.31 and openai-agents 0.24). Whatever model name the agents send
  must be the Azure deployment name.
- Azure documents the Responses API from `api-version=2025-03-01-preview` on, so the
  `2024-08-01-preview` example value in `.env.example` may be too old for it.
- `homework_guardrail` and `guardrail_agent` are defined, but the `input_guardrails=[...]` block on
  `triage_agent` is commented out, so no input is blocked and the `InputGuardrailTripwireTriggered`
  handlers never fire. Uncomment that block to see the guardrail.
- `set_default_openai_client(openai_client, use_for_tracing=False)` keeps the Azure key out of
  trace export. Tracing itself stays on; without `OPENAI_API_KEY` the SDK logs that it skips the
  export.
- A commented-out `OpenAIChatCompletionsModel` sits next to the Responses model, for trying Chat
  Completions instead.
- `asyncio` in requirements.txt is part of the standard library; the PyPI package of that name is an
  obsolete backport and isn't needed. `dotenv` on PyPI is a shim that installs `python-dotenv`.
- Requirements are unpinned, so SDK behaviour (default model, signatures) can change between installs.
