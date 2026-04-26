# LANGCHAIN-SKILL.md

## Purpose

Guide AI coding agents in building correct, maintainable, and observable LangChain applications. This skill covers chain composition, prompt design, tool integration, retries, and avoiding unnecessary complexity.

---

## Scope / When to Use

Apply this skill when generating or modifying code that uses:
- `langchain-core`, `langchain`, `langchain-community`, or provider-specific packages (e.g., `langchain-openai`, `langchain-anthropic`).
- LangChain Expression Language (LCEL) chains.
- LangChain tools, agents, and retrievers.
- LangSmith tracing and observability integrations.

---

## Core Principles

- **Explicit over magic.** Prefer LCEL pipe syntax and named steps over implicit abstraction.
- **Minimal chain depth.** Do not add chain steps that do not contribute to the output.
- **Prompt boundaries are sacred.** Separate prompt templates from business logic.
- **Observability is required.** Every non-trivial chain should be traceable.
- **Fail predictably.** Handle model errors, rate limits, and parsing failures explicitly.

---

## Required Behavior

### Package and Imports
- Use `langchain-core` types (`BaseMessage`, `ChatPromptTemplate`, `RunnablePassthrough`) as the foundation.
- Import from provider-specific packages (e.g., `langchain_openai.ChatOpenAI`) rather than the legacy `langchain` top-level.
- Pin package versions in `requirements.txt` or `pyproject.toml`.

### Chain Composition (LCEL)
- Use LCEL (`|` operator) for composing chains.
- Name each step clearly when building complex chains using `RunnableMap` or dictionary inputs.
- Use `RunnablePassthrough` to forward unchanged context through chain steps.
- Prefer `RunnableLambda` for lightweight transformations over writing custom `BaseChain` subclasses.

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{question}"),
])

chain = prompt | ChatOpenAI(model="gpt-4o") | StrOutputParser()
response = chain.invoke({"question": "What is LangChain?"})
```

### Prompt Design
- Define all prompts as `ChatPromptTemplate` or `PromptTemplate` objects — never as raw f-strings.
- Keep system messages focused and under 200 tokens when possible.
- Parameterize all variable content using template variables, not string concatenation.
- Document what each template variable represents with an inline comment.

### Tool Integration
- Decorate tools with `@tool` or define them as `StructuredTool` with explicit input schemas.
- Document each tool's purpose, inputs, and expected output in the docstring.
- Validate tool outputs before passing them to the next chain step.
- Do not give agents access to tools they do not need for the task.

### Output Parsing
- Use typed output parsers (`PydanticOutputParser`, `JsonOutputParser`) for structured outputs.
- Handle `OutputParserException` explicitly with fallback or retry logic.
- Do not rely on regex parsing of free-form model output for structured data.

### Retries and Error Handling
- Use `.with_retry()` on runnables that call external models or APIs.
- Configure `stop_after_attempt` and `wait_exponential_jitter` for retry policies.
- Catch `RateLimitError`, `APIConnectionError`, and `AuthenticationError` explicitly.
- Log the error context (chain step, input summary) before raising or retrying.

### Observability
- Set `LANGCHAIN_TRACING_V2=true` and `LANGCHAIN_API_KEY` for LangSmith tracing in any non-trivial chain.
- Use `callbacks` parameter to attach custom handlers when LangSmith is not available.
- Add `run_name` metadata to long chains to make traces identifiable.

### Memory and State
- For conversational chains, use `RunnableWithMessageHistory` with an explicit session ID.
- Store message history in a persistent store (Redis, database) for production use — never in-memory only.
- Clear or summarize history when token limits may be exceeded.

---

## Things to Avoid

- Using deprecated `LLMChain`, `ConversationalRetrievalChain`, or `AgentExecutor` patterns without checking whether an LCEL equivalent exists.
- Concatenating user input directly into prompt strings — always use template variables.
- Building chains without any error handling or retry logic.
- Using `verbose=True` in production code without directing output to a logger.
- Hardcoding model names or API keys in source code.
- Over-engineering chains with unnecessary steps, routing, or branching.

---

## Output Expectations

Generated LangChain code must:
- Use LCEL syntax for chain construction.
- Import from `langchain-core` and provider-specific packages.
- Include retry logic on any external model call.
- Have prompts defined as `ChatPromptTemplate` objects.
- Include at least a comment indicating where LangSmith tracing should be enabled.
- Not contain hardcoded API keys or model names without environment variable references.

---

## Optional Checklist

- [ ] Prompts defined as `ChatPromptTemplate` or `PromptTemplate`, not raw strings.
- [ ] LCEL `|` syntax used for chain composition.
- [ ] `.with_retry()` applied to model calls.
- [ ] Structured outputs use `PydanticOutputParser` or `JsonOutputParser`.
- [ ] LangSmith tracing enabled or documented.
- [ ] No deprecated `LLMChain` or `AgentExecutor` patterns.
- [ ] API keys sourced from environment variables.
