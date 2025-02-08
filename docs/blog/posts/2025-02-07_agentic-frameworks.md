---
date:
    created: 2025-02-07
readtime: 30
links:
  - "Github: jonchun/agenty": https://github.com/jonchun/agenty
---
# Why use an Agentic Framework?

## LLM's and Limitations
Working directly with Large Language Models (LLMs) can feel like having a brilliant assistant who only responds when spoken to. They're great for answering direct questions but not ideal for handling complex, multi-step tasks.

One major limitation most people run into immediately is lack of memory and context retention. LLMs don’t naturally remember past interactions, meaning you have to repeatedly provide context, which can be frustrating and inefficient. 

Another common issue is lack of initiative. LLMs respond to prompts but don’t proactively take action or follow up on tasks unless explicitly told to. This makes them more like a search engine than an intelligent assistant that truly "gets things done."

There’s also the challenge of orchestration. If a task involves multiple steps -- like researching, summarizing, making decisions, and executing actions, LLM's struggle to autonomously manage the workflow. They rely entirely on the user to break tasks into chunks, provide structured inputs, and manually guide the process.

They also lack built-in tooling and the ability to act. While they can generate code, suggest solutions, or analyze data, they can’t directly interact with the internet, with API's, with code, etc. without additional layers of infrastructure.

The possibilities are immense. If we could iron out these core limitations, we could take these systems from powerful but constrained, to truly transformative.

## What's an Agent?
We've all been there. You're chatting with ChatGPT, and it seems to remember your entire conversation, almost magically tracking the context of your discussion until it suddenly doesn't know what you're talking about.

Here's the catch -- under the hood, these models are actually severely limited. They literally don't naturally remember anything. All the model itself knows how to do is predict the next token (a rough proxy for the next word) in the conversation given the previous tokens. So how do they pull off that seamless conversation?

This is where agents come into play. They are the secret sauce that turns a raw AI model into a smart, adaptive assistant. Think of an agent like a personal assistant wrapped around an LLM. It's not just about responding to questions; it's about understanding context, managing memory, and actually getting things done. These agents are like translators, interpreters, and problem-solvers all rolled into one.

- **Conversation History:** This is a fundamental mechanism in agent systems where all messages exchanged between a user and an AI model are sequentially tracked. When a new message arrives, the agent adds it to the existing conversation log and then submits the *complete* history to the model for generating a contextually informed response. This is important! Every single time you send another message in a long conversation, the model is rereading your **entire** conversation! 

- **Tool Integration:** Agents aren't just confined to generating text. They can connect to web searches, execute code, interact with APIs, and perform complex tasks. Imagine having an assistant that doesn't just talk about writing a script, but can actually write and run it for you.

- **Proactive Problem Solving:** Instead of waiting for explicit instructions, advanced agents can break down complex tasks, identify necessary steps, and even adjust their approach if something doesn't work.

- **Memory:** They can store and retrieve previous conversation details, context, and even learn from past interactions. It's like giving an agent a notebook and the ability to flip back through its pages. 

The result? An AI that feels less like a fancy autocomplete and more like a genuine assistant -- one that understands your intent, remembers your context, and can actually help you accomplish real-world tasks. It's not just about answering questions or predicting the next output anymore. It's about creating an intelligent system that can reason, plan, and execute -- bringing us closer to the dream of a true AI companion.

## What Are Agentic Frameworks?
Agentic frameworks (or Agent frameworks) are the structured systems and architectures that enable AI agents to operate intelligently, flexibly, and efficiently. These frameworks provide the necessary infrastructure for managing conversations, integrating tools, handling memory, and executing tasks autonomously. Think of them as the backbone that transforms a basic AI model into a capable, adaptive assistant.

At their core, agentic frameworks define how an AI agent:

- **Processes and Retains Context**: They implement mechanisms for tracking and managing conversation history, ensuring that the AI can recall relevant details from past interactions and maintain coherence across long discussions.
- **Uses Tools and APIs**: These frameworks allow AI agents to interact with external systems, such as databases, web searches, and computational tools, to enhance their capabilities beyond text generation.
- Plans and Executes Multi-Step Tasks: Instead of simply responding to isolated queries, agentic frameworks empower AI agents to break down complex tasks into manageable steps, prioritize actions, and adjust based on feedback.
- **Implements Memory Mechanisms**: While raw AI models don't inherently "remember," agentic frameworks enable agents to store, retrieve, and leverage past interactions, making them more personalized and context-aware over time.
- **Adapts and Improves**: Some advanced frameworks incorporate learning mechanisms, allowing agents to refine their problem-solving strategies and improve their effectiveness through repeated interactions.


## Technical Examples
Let's do a simple example of how agentic frameworks represent a significant improvement over direct LLM interactions. I'm going to use examples with [agenty](https://github.com/jonchun/agenty), an open-source framework I am developing that aims to be developer-friendly.

### Conversation Management
The examples I use here are not going to be fully runnable examples for the sake of brevity. We set a generic system prompt (a set of instructions the LLM should follow that the end-user doesn't see) to `You are a helpful assistant.`

#### OpenAI API

Let's start with a simple greeting that includes my name.

> Hello, I'm Jon. How are you?

```py 
response = openai.ChatCompletion.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello, I'm Jon. How are you?"}
    ]
)
print(response['choices'][0]['message']['content'])

# Example Output:
#
# Hello Jon! I'm just a computer program, so I don't have feelings, 
# but I'm here to help you. How can I assist you today?
```

Great! We got a response. Let's see what happens when we naively try to ask a follow-up question.

> What's my name?

```py hl_lines="10-16 23-24"
response = openai.ChatCompletion.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello, I'm Jon. How are you?"}
    ]
)
print(response['choices'][0]['message']['content'])
print("")
response2 = openai.ChatCompletion.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "What's my name?"} # (1)!
    ]
)
print(response2['choices'][0]['message']['content'])

# Example Output:
#
# Hello Jon! I'm just a computer program, so I don't have feelings, 
# but I'm here to help you. How can I assist you today?
#
# I'm sorry, but I don't have access to personal data,  
# so I can't tell you your name.
```

1. Ask a new question to the same underlying model.

Unfortunately, the LLM has no idea what my name is. I just provided it though... what's going on? I mentioned this earlier, but models have NO memory. Every single new message you send means you have to send the *entire* conversation history every time.

Below, we first capture the response to our first interaction and include it in our second as we ask our follow-up question.
```py hl_lines="8 15-16"
response = openai.ChatCompletion.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello, I'm Jon. How are you?"}
    ]
)
response_content = response2['choices'][0]['message']['content'] # (1)!

response2 = openai.ChatCompletion.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello, I'm Jon. How are you?"}
        {"role": "assistant", "content": response_content} # (2)!
        {"role": "user", "content": "What's my name?"} # (3)!
    ]
)
print(response_content)
print("")
print(response2['choices'][0]['message']['content'])

# Example Output:
#
# Hello Jon! I'm just a computer program, so I don't have feelings, 
# but I'm here to help you. How can I assist you today?
#
# You mentioned that your name is Jon. How can I assist you further?
```

1. Capture the response to `Hello, I'm Jon. How are you?`
2. We insert the captured response into the conversation history
3. Finally, we add the new question/message into the list.


We did it! The model now "remembers" my name and answers successfully.

---

#### Framework (agenty)
Now let's examine what the above workflow looks like with the help of a framework.

```python
agent = Agent(model="gpt-4o", system_prompt="You are a helpful assistant.")

response = await agent.run("Hello, I'm Jon. How are you?") # (1)!
print(response)
print("")
response2 = await agent.run("What's my name?") # (2)!
print(response2)

# Example Output:
#
# Hello Jon! I'm just a computer program, so I don't have feelings, 
# but I'm here to help you. How can I assist you today?
#
# You mentioned that your name is Jon. How can I assist you further?

```

1. Note we don't have to define the model again or the system prompt.
2. We just continue and talk to the agent without doing anything else!

Behind the scenes, the framework handles all of the conversation history for you. It also comes with an additional bonus of being able to handle more than just OpenAI. I can switch to using another model as easily as
```python
from pydantic_ai.models.anthropic import AnthropicModel
agent = Agent(
    model=AnthropicModel("claude-3-5-sonnet-latest"),
    system_prompt="You are a helpful assistant.",
)
```
If you were to try and do this manually, you would need to rewrite all of your code to confirm to the [Anthropic API](https://docs.anthropic.com/en/api/getting-started)

---
### Tools
#### OpenAI API
Let's build a simple tool that retrieves the current weather using the `get_temperature` function. This can be done through [function calling](https://platform.openai.com/docs/guides/function-calling).  

Here’s an example of how it works when making a request via the API:
```python  hl_lines="13-17 29 41"

def get_temperature() -> float:
    return 72.0

messages = [
    {"role": "system", "content": "You help users check the weather."},
    {"role": "user", "content": "What is the weather like?"},
]
response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=[
        {
            "type": "function", # (1)!
            "function": {
                "name": "get_temperature",
                "description": "Get the current temperature.",
            },
        }
    ],
)
tool_calls = None
if response.choices[0].message:
    tool_calls = response.choices[0].message.tool_calls # (2)!

if not tool_calls:
    raise Exception("no tool calls!")

for tool_call in tool_calls:
    if tool_call.function.name == "get_temperature":  # (3)!
        messages.append(
            {
                "role": "assistant",
                "content": None,
                "tool_calls": tool_calls,  # (4)!
            }
        )
        messages.append(
            {
                "role": "tool",
                "name": "get_temperature",
                "content": str(get_temperature()), # (5)!
            }
        )
final_response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages, # (6)!
)

print(final_response.choices[0].message.content) # (7)!

# Example Output:
#
# It is currently 72 degrees.
```

1. OpenAI [function definition](https://platform.openai.com/docs/guides/function-calling#defining-functions) requires a JSON Schema of your tool.
2. Store the request from the model to call a tool
3. Check if the model called the `get_temperature` tool.
4. Add the `tool_calls` request from the model to the conversation history.
5. Actually execute `get_temperature()` function
6. Send the tool response back to the model
7. Finally display a response from the model.

It's quite a long and convoluted process just to call a single tool. Imagine how much more you have to do when you're trying to call multiple tools, add error handling, have multiple different LLM's, etc.

---

#### Framework (agenty)
Here is the same functionality reproduced with the agenty framework.
```python hl_lines="5 11"
class WeatherAgent(Agent):
    model = "gpt-4o"
    system_prompt = "You help users check the weather."

    @tool # (1)!
    def get_temperature(self) -> float:
        """Get the current temperature."""
        return 72.0

weather_agent = WeatherAgent()
print(weather_agent.run("What is the weather like?")) # (2)!
```

1. Simple decorator automatically registers the function to the agent so it knows and executes the tool
2. The agent executes the tool automatically under-the-hood. None of the parsing boilerplate!

---

## Why Agentic Frameworks?
Agentic frameworks represent a significant architectural leap beyond raw LLM integration. While basic LLM calls serve well for text generation, agent frameworks provide the components for building complex AI systems through robust state management, memory persistence, and tool integration capabilities.

From an engineering perspective, the frameworks abstract away much of the boilerplate required for a sophisticated AI. Rather than repeatedly implementing context management, tool integration, and error handling patterns, developers can leverage pre-built implementations and components. This dramatically reduces technical debt while improving system reliability.

The end result is a powerful abstraction for building AI systems that can plan and execute complex tasks. Rather than treating AI as a simple text generation service, agent frameworks enable the development of autonomous systems that can reason about goals, formulate plans, and reliably execute against them. This represents the natural evolution of AI system architecture -- from simple prompt-completion patterns to robust, production-ready frameworks for building reliable AI agents.

These frameworks provide the architectural foundation necessary for the next generation of AI systems -- ones that don't just respond to prompts, but proactively reason, plan, and execute with the reliability required by real-world applications.

---

## What Framework is Best?
I think "best" always depends on what exactly you're looking to achieve. However, here is a list of some of the more popular and/or promising projects.

- https://ai.pydantic.dev/
- https://github.com/BrainBlend-AI/atomic-agents
- https://github.com/microsoft/autogen/
- https://www.langchain.com/langgraph
- https://www.crewai.com/
- https://github.com/kyegomez/swarms

I am the author of agenty which was used in the examples above. You can find my project here: https://github.com/jonchun/agenty