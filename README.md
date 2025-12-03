
# Instagram Content Generator

A small but practical AI-powered tool that generates Instagram-ready post captions and hashtags based on a user-provided topic.

## 1. Project Overview

The **Instagram Content Generator** takes a short text input (for example: a product, niche, theme, or idea) and returns:

- A concise Instagram-ready caption with a modern, engaging tone.
- A list of relevant and trending hashtags.

The goal is to automate repetitive content-writing tasks for Instagram, making it easier and faster to create social media posts for brands, creators, and small businesses.

## 2. Key Features

- Accepts a simple text topic as input.
- Generates:
  - A well-structured Instagram caption.
  - A set of 15–20 relevant hashtags.
- Uses OpenRouter as the LLM gateway.
- Built with a visual flow tool, making logic easy to understand and extend.
- Adaptable for other social media platforms.

## 3. Architecture

1. **User Input** – User provides a topic.
2. **Prompt Template** – Topic inserted into a structured content-generation prompt.
3. **OpenRouter LLM Call** – Model generates caption and hashtags.
4. **Output Formatting** – Returned to the user.
5. **Optional Integrations** – Scheduling tools, databases, automation workflows.

## 4. Tech Stack

- **LLM Gateway:** OpenRouter  
- **Models:** e.g., `google/gemini-2.0-flash-lite-preview-02-05`, `mistral-small-latest`
- **Orchestration/UI:** Visual flow builder such as Langflow

## 5. Prerequisites

- OpenRouter account and token  
- Visual AI flow builder environment  
- Basic understanding of node connections  

## 6. Flow Design

1. **Input Node** – Captures user topic.  
2. **Prompt Template Node** – Inserts `{user_input}` into caption/hashtag prompt.  
3. **OpenRouter Node** – Calls selected model.  
4. **Output Node** – Displays the generated caption and hashtags.

## 7. Configuration Steps

- Set OpenRouter API key  
- Select provider and model  
- Map variables inside the prompt  
- Run and test with different topics  

## 8. Usage Example

Input:
```
Healthy breakfast ideas for busy software engineers
```

Output:
- Caption with structured lines  
- 15–20 hashtags related to fitness, health, breakfast, productivity  

## 9. Customization

- Change tone/style  
- Add additional platforms  
- Add multi-language support  
- Include Reels scripts or carousel copy  
- Add automation integrations  

## 10. Limitations

- Output quality depends on model  
- Hashtags are inferred, not guaranteed trending  
- Free-tier models may have rate limits  

## 11. Future Improvements

- Web UI  
- Saving outputs to database  
- Analytics and category tagging  
- Social media API integrations  
