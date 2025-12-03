# System Architecture Document
## Project: Instagram Content Generator (Langflow + OpenRouter)
## Version: 1.0
## Date: 2025-12-03

---
## 1. Overview

The Instagram Content Generator is a small LLM-powered application built using Langflow and OpenRouter.  
It generates Instagram-ready captions and hashtags based on a user-provided topic.

The primary goals of this system are:
- To provide a simple interface where a user can input a topic or idea.
- To transform that input into a structured prompt.
- To call an OpenRouter-hosted language model to generate the response.
- To return a formatted caption and hashtags to the user using Langflow's chat interface.

This document describes the high-level architecture, main components, and data flow of the system.

---
## 2. High-Level Architecture

At a high level, the system consists of the following layers:

1. **Presentation Layer (Langflow UI / Playground)**
   - Where users interact with the system via a chat-style interface.
   - Provides an input box for the topic and displays the generated caption and hashtags.

2. **Orchestration Layer (Langflow Flow)**
   - A visual flow composed of nodes:
     - Chat Input
     - Prompt Template
     - OpenRouter LLM
     - Chat Output
   - Handles message passing between nodes and manages execution order.

3. **Model Layer (OpenRouter + LLM)**
   - Uses OpenRouter as an API gateway to different models.
   - A specific LLM (for example a Google Gemini or Gemma model) is selected for inference.
   - The model generates the final caption and hashtags.

4. **External Services**
   - OpenRouter API endpoint (hosted externally).
   - Optional: logging and monitoring from Langflow or infrastructure side.

The runtime is managed by Langflow, which executes the flow when the user sends a message.

---
## 3. Component Architecture

### 3.1 Langflow Flow Components

The Langflow project is stored and shared as a JSON definition file (`New Flow.json`).  
Within this flow, the main components are:

1. **Chat Input Node**
   - Type: `ChatInput`
   - Responsibility: Captures user input text from the Langflow Playground.
   - Output: A `Message` object containing the user's topic and any optional metadata.
   - Key behaviors:
     - Stores the message in history if enabled.
     - Supports optional session identifiers and attached files.

2. **Prompt Template Node**
   - Type: `Prompt Template`
   - Responsibility: Builds the final prompt to be sent to the LLM.
   - Input:
     - `user_input` mapped from the Chat Input node.
   - Internal template (example):
     - Describes the role of the model as an Instagram content creator.
     - Specifies that the model should generate a caption and 15–20 hashtags.
   - Output: A `Message` object containing the fully constructed prompt text.

3. **OpenRouter LLM Node**
   - Type: `OpenRouterComponent`
   - Responsibility: Sends the prompt to the OpenRouter API and receives the model output.
   - Inputs:
     - `input_value`: the prompt message from the Prompt Template node.
     - `system_message` (optional): can be used to further control model behavior.
     - `api_key`: the OpenRouter access token.
     - `provider` and `model_name`: configured to use a selected provider (for example Google) and a specific model.
   - Outputs:
     - `text_output`: the generated response from the model as a `Message`.
   - Behavior:
     - Builds a `ChatOpenAI` compatible client configured to call OpenRouter's `/api/v1` endpoint.
     - Sends the prompt and receives completion text.
     - Handles temperature and max token settings.

4. **Chat Output Node**
   - Type: `ChatOutput`
   - Responsibility: Presents the model-generated message to the user in the Playground interface.
   - Input:
     - `input_value`: the `text_output` from the OpenRouter node.
   - Output:
     - A final `Message` instance visible in the chat history.
   - Behavior:
     - Ensures message metadata (sender, source, session) is correctly attached.
     - Optionally stores the message in history.

### 3.2 External Service: OpenRouter

- **Service:** OpenRouter API
- **Role:** Gateway to multiple LLMs, including Google Gemini / Gemma models.
- **Endpoint:** `https://openrouter.ai/api/v1`
- **Key Responsibilities:**
  - Authentication using the provided API key.
  - Routing the request to the chosen provider/model.
  - Enforcing rate limits and quotas.
  - Returning structured responses that Langflow can parse.

---
## 4. Data Flow

The typical request/response data flow is as follows:

1. **User Input**
   - The user types a topic, for example:  
     `"Healthy breakfast ideas for busy software engineers"`  
   - This text is captured by the Chat Input node as a `Message` object.

2. **Mapping to Prompt Template**
   - The Chat Input node output is connected to the Prompt Template node.
   - The `user_input` variable within the prompt template is populated with the message text.

3. **Prompt Construction**
   - The Prompt Template node constructs the full prompt string. For example:
     - Definition of the model role (Instagram content creator).
     - Steps to generate caption and hashtags.
     - The user topic appended in a structured way.
   - The result is a new `Message` object containing the final prompt.

4. **Model Invocation via OpenRouter**
   - The Prompt Template output is passed into the `input_value` of the OpenRouter node.
   - The OpenRouter node:
     - Builds configuration for the selected model.
     - Sends the prompt to OpenRouter’s `/chat/completions`-style endpoint.
     - Waits for the generated response.
   - The response text is wrapped into a `Message` object and exposed as `text_output`.

5. **Output to User**
   - The `text_output` from OpenRouter is connected to the Chat Output node.
   - Chat Output renders the response in the UI:
     - Caption text.
     - Hashtag list.
   - Optionally, the response is stored in the session history for future context.

---
## 5. Execution Flow

The execution order in Langflow is event-driven and flows from left to right:

1. User triggers the flow by sending a message in the Playground.
2. Chat Input node receives the message and outputs a `Message` object.
3. Prompt Template node is invoked next, using the input message as context.
4. OpenRouter node executes with the constructed prompt.
5. Chat Output node displays the final result to the end user.

Langflow internally manages execution scheduling and ensures that dependent nodes execute only after their inputs are available.

---
## 6. Non-Functional Considerations

### 6.1 Performance

- Latency is primarily dependent on the selected LLM and network round-trip to OpenRouter.
- Prompt complexity and model size affect response time.
- For faster responses, lighter or "flash" style models can be used.

### 6.2 Scalability

- Since the application is orchestrated by Langflow and uses a remote LLM, scalability is handled by:
  - The hosting environment for Langflow (for example, a container or server).
  - OpenRouter's ability to handle concurrent requests.
- Multiple users can interact with the same Langflow deployment instance, as each chat session is separated by session IDs.

### 6.3 Reliability

- Errors can occur due to:
  - Invalid or missing API key.
  - Network connectivity issues.
  - Model unavailability or rate limiting on OpenRouter’s side.
- The architecture assumes basic error reporting through Langflow logs and node status messages.

### 6.4 Security

- The OpenRouter API key must be kept secret and not committed to version control.
- Access to the Langflow instance should be controlled (for example via network rules or authentication).
- User inputs should be treated as untrusted data but, in this case, are only used for content generation and not for critical operations.

---
## 7. Deployment View

The deployment model can be summarized as:

- **Langflow Runtime**
  - Runs as a backend application (for example, in Docker, on a VM, or managed platform).
  - Exposes a web UI for the Playground.

- **Client (User Browser)**
  - Connects to Langflow UI over HTTP/HTTPS.
  - Sends topic text as chat messages.

- **External Service: OpenRouter**
  - Hosted by OpenRouter.
  - Receives API requests from Langflow.
  - Returns LLM responses over HTTPS.

No additional database or persistence layer is strictly required for this project unless session data or generated content needs to be stored long term.

---
## 8. Extension Points and Future Enhancements

This architecture allows for the following extensions without major redesign:

1. **Additional Platforms**
   - Extend the prompt template to support mode selection:
     - Instagram, LinkedIn, Twitter/X, etc.
   - Use conditional logic or additional nodes to adjust prompt wording.

2. **Multilingual Support**
   - Update the prompt template to generate captions and hashtags in different languages.
   - Optionally detect language from user input and adapt instructions.

3. **Bulk Content Generation**
   - Integrate file or list inputs.
   - Iterate over multiple topics and call the LLM for each.

4. **Persistence Layer**
   - Store generated captions and hashtags in a database.
   - Enable re-use, analytics, and versioning.

5. **API Exposure**
   - Wrap the Langflow flow behind an API endpoint (if supported by the hosting setup).
   - Allow programmatic access from external systems.

---
## 9. Summary

This system uses a simple but robust architecture:

- Langflow handles orchestration, UI, and node-based logic.
- OpenRouter provides flexible access to multiple LLMs.
- The flow consists of four core components: Chat Input, Prompt Template, OpenRouter, and Chat Output.
- The design is modular, easy to understand, and easy to extend with new features such as additional content types, languages, or integrations.

This document can be attached as the official architecture reference for the Instagram Content Generator project.
