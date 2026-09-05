# Chapter 15: Provider Setup

## Core Idea
Zoo Code supports a wide ecosystem of AI providers—from direct API access (Anthropic, OpenAI, Google Gemini) to local models (Ollama, LM Studio) and unified gateways (OpenRouter, LiteLLM)—each with specific configuration patterns and trade-offs.

## Key Concepts
- **API Provider**: The AI service that powers Zoo Code's capabilities. Selected in Settings → API Provider dropdown, requiring an API key and model selection.
- **Native Tool Calling**: Zoo Code's exclusive tool protocol—models must support OpenAI-compatible tool calling to function. No XML-based fallback exists.
- **Prompt Caching**: Provider-specific feature that caches repeated prompts for cost/latency reduction. Supported by Anthropic, OpenAI, OpenRouter, Requesty, and others.
- **OpenAI-Compatible Providers**: Any provider offering an OpenAI-standard API endpoint (Perplexity, Together AI, Anyscale, custom endpoints), configured via Base URL + API Key + Model ID.
- **Unified Gateway**: Services like OpenRouter, Requesty, LiteLLM, and Vercel AI Gateway that provide access to 100+ models from multiple providers through a single API.
- **Local Model Provider**: Ollama and LM Studio run models on your machine for privacy, offline access, and cost savings, but require more setup and powerful hardware.
- **OAuth Providers**: Some providers (ChatGPT Plus/Pro, Kimi Code, Qwen Code) support OAuth device-flow sign-in, eliminating the need for manual API key management.
- **Application Default Credentials (ADC)**: GCP Vertex AI authentication method using `gcloud auth application-default login` for seamless cloud integration.

## Mental Models
- **Provider selection matrix**: Choose based on your priority—cost (local models), capability (Anthropic/OpenAI), privacy (Ollama), or flexibility (OpenRouter/LiteLLM).
- **API key lifecycle**: Create → Copy immediately → Store securely → Never share. Most providers show the key only once.
- **Gateway pattern**: Unified gateways abstract away provider differences, letting you switch models without reconfiguring Zoo Code.
- **Model discovery**: Many providers auto-fetch available models via API endpoints, keeping your model list current without extension updates.

## Anti-patterns
- **Using model name instead of model ID for Bedrock**: AWS Bedrock requires the model ID (e.g., `anthropic.claude-3-sonnet-20240229-v1:0`), not the friendly name.
- **Ignoring tool-calling compatibility**: Not all models on OpenAI-compatible providers support native tool calling—verify before selecting.
- **Hardcoding base URLs**: Some providers (Z AI, MiniMax) have region-specific endpoints; use the provider's region selector instead of manual URL entry.
- **Not preloading Ollama models**: Cold starts can cause OOM errors—preload models with `ollama run <model>` before Zoo Code requests.

## Code Examples
```
# Anthropic configuration
Provider: Anthropic
API Key: <your-key>
Model: claude-sonnet-4-20250514
# Supports: prompt caching, 200K context, extended thinking

# OpenAI configuration with reasoning effort
Provider: OpenAI
API Key: <your-key>
Model: gpt-5
Reasoning Effort: medium  # minimal/low/medium/high
Verbosity: medium  # low/medium/high

# AWS Bedrock configuration
Provider: Bedrock
Auth: AWS Credentials or AWS Profile
Region: us-east-1
Model ID: anthropic.claude-3-sonnet-20240229-v1:0
# Use model ID, not friendly name

# LiteLLM local proxy setup
pip install 'litellm[proxy]'
litellm --config config.yaml
# Zoo Code Base URL: http://localhost:4000
# Provider: LiteLLM or OpenAI Compatible

# Ollama local setup
ollama pull qwen2.5-coder:32b
ollama run qwen2.5-coder:32b  # preload to avoid OOM
# Zoo Code Provider: Ollama
# Model: qwen2.5-coder:32b

# Azure OpenAI (via OpenAI Compatible)
Provider: OpenAI Compatible
Base URL: https://<resource>.openai.azure.com/openai
API Key: <azure-key>
Model: <deployment-name>  # e.g., "my-gpt4o-deployment"
Use Azure: checked
```

## Worked Example
**Scenario**: You want to use Claude models through AWS Bedrock for enterprise compliance.

1. Request Bedrock access and model access for Anthropic Claude in AWS console.
2. Configure AWS credentials: `aws configure` or create an IAM user with `bedrock:InvokeModel` permission.
3. In Zoo Code settings: Provider → Bedrock, Auth → AWS Credentials, Region → us-east-1.
4. Select model using the model ID (not the friendly name): `anthropic.claude-3-sonnet-20240229-v1:0`.
5. Enable reasoning budget for extended thinking if needed.
6. (Optional) Configure VPC endpoint for enterprise environments requiring private network traffic.

## Key Takeaways
1. Most providers follow the same pattern: Settings → Select Provider → Enter API Key → Select Model.
2. Native tool calling is mandatory—verify model support before selecting from OpenAI-compatible providers.
3. Unified gateways (OpenRouter, Requesty, LiteLLM) simplify multi-model management and experimentation.
4. Local models (Ollama, LM Studio) require preloading and context window configuration to avoid OOM errors.

## Connects To
- **Ch 14**: Provider choice directly impacts cost—compare pricing across providers for your workload.
- **Ch 13**: Local models and prompt caching are advanced usage patterns tied to specific providers.
- **Ch 16**: Many troubleshooting issues relate to provider configuration (API keys, model IDs, base URLs).
