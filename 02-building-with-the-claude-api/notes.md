# Building with the Claude API

Course notes and progress for the Anthropic Academy "Building with the Claude API" course.

## Lesson 1 — Accessing Claude with the API

### Accessing the API

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287726

#### What you'll learn

*Estimated time: 5 minutes 18 seconds (video)*

By the end of this lesson you'll be able to:

- Describe the five-step request lifecycle for every Claude API interaction
- Explain why API calls must always be routed through a server, never from client-side code
- Identify the required fields in every API request
- Understand the four stages of Claude's internal processing (tokenize, embed, contextualize, generate)
- Interpret the structured API response and its stop reasons

---

#### Video Summary *(5 min 18 sec)*

When building applications with Claude, understanding the complete request lifecycle helps you make better architectural decisions and debug issues more effectively. This lesson walks through what happens from the moment a user clicks "send" in your chat interface to when Claude's response appears on screen.

---

#### The Five-Step Request Flow

Every interaction with Claude follows a predictable pattern with five distinct phases:

1. **Request to server** — your client app sends the user's message to your own backend.
2. **Request to Anthropic API** — your server forwards the request to Anthropic, authenticated with the securely stored API key.
3. **Model processing** — Claude processes the input through tokenization, embedding, contextualization, and generation.
4. **Response to server** — the Anthropic API returns a structured response to your server.
5. **Response to client** — your server forwards the generated text back to the user interface.

---

#### Why You Need a Server

You should never make requests to the Anthropic API directly from client-side code. Here's why:

- API requests require a secret API key for authentication.
- Exposing this key in client code creates a serious security vulnerability.
- Anyone could extract the key and make unauthorized requests.

Instead, your web or mobile app sends requests to your own server, which then communicates with the Anthropic API using the securely stored key.

---

#### Making API Requests

When your server contacts the Anthropic API, you can use either an official SDK or make plain HTTP requests. Anthropic provides SDKs for Python, TypeScript, JavaScript, Go, and Ruby.

Every request must include these essential fields:

- **API Key** — identifies your request to Anthropic.
- **Model** — name of the model to use (e.g. `claude-3-sonnet`).
- **Messages** — list containing the user's input text.
- **Max Tokens** — limit for how many tokens Claude can generate.

---

#### Inside Claude's Processing

Once Anthropic receives your request, Claude processes it through four main stages:

1. **Tokenization** — Claude breaks your input text into smaller chunks called tokens. These can be whole words, parts of words, spaces, or symbols.
2. **Embedding** — each token gets converted into an embedding — a long list of numbers that represents all possible meanings of that word. Think of embeddings as numerical definitions that capture semantic relationships.
3. **Contextualization** — Claude refines each embedding based on surrounding words to determine the most likely meaning in context (e.g. "quantum" in physics vs. computing). This adjusts the numerical representations to highlight the appropriate definition.
4. **Generation** — the contextualized embeddings pass through an output layer that calculates probabilities for each possible next word. Claude doesn't always pick the highest-probability word — it uses a mix of probability and controlled randomness to create natural, varied responses. After selecting each word, Claude adds it to the sequence and repeats the entire process for the next token.

---

#### When Claude Stops Generating

After each token, Claude checks several conditions to decide whether to continue:

- **Max tokens reached** — has it hit the limit you specified?
- **Natural ending** — did it generate an end-of-sequence token?
- **Stop sequence** — did it encounter a predefined stop phrase?

---

#### The API Response

When generation completes, the API sends back a structured response containing:

- **Message** — the generated text.
- **Usage** — count of input and output tokens.
- **Stop Reason** — why generation ended.

Your server receives this response and forwards the generated text back to your client application.

---

#### Key Takeaways

Understanding this flow helps you:

- Design secure architectures that protect your API keys.
- Set appropriate token limits for your use case.
- Handle different stop reasons in your application logic.
- Debug issues by understanding where they might occur in the pipeline.

---

### Getting an API key

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/296766

#### What you'll learn

By the end of this lesson you'll be able to:

- Navigate the Anthropic Console to create and manage API keys
- Safely copy and store an API key for use in subsequent lessons

---

#### Overview

This is a written step-by-step guide for generating your Anthropic API key before making your first request. The key is only shown once, so follow carefully.

---

#### Step-by-Step: Creating an API Key

**Step 1 — Navigate to the Anthropic API Console**

Go to [https://console.anthropic.com/](https://console.anthropic.com/) and log in to your Anthropic account.

**Step 2 — Click the "Get API Keys" button**

Found towards the top right of the main dashboard page.

**Step 3 — Click the "Create Key" button**

Located at the top right of the API Keys page.

**Step 4 — Enter a workspace and name for your key**

Create the key in the `Default` workspace and give it a recognisable name (e.g. `Anthropic Course`). The name is only used to help you identify keys you generate.

**Step 5 — Copy the key**

Your API key will be displayed in a pop-up window. Copy it immediately and store it securely — **this key is only shown once**. If you accidentally close the window, delete the old key and generate a new one.

---

### Making a request

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287725

#### What you'll learn

*Estimated time: 6 minutes 2 seconds (video)*

By the end of this lesson you'll be able to:

- Set up a Python environment with the Anthropic SDK and `python-dotenv`
- Store and load an API key securely using a `.env` file
- Initialise an `Anthropic` client and call `client.messages.create()`
- Understand the three required parameters: `model`, `max_tokens`, and `messages`
- Extract the generated text from the response object

---

#### Video Summary *(6 min 2 sec)*

Making your first request to the Anthropic API is straightforward once you understand the basic setup and structure. This lesson walks through the essential steps to get Claude responding to your prompts using Python.

---

#### Setting Up Your Environment

Before making any API calls, install the required packages in your Jupyter notebook:

```python
%pip install anthropic python-dotenv
```

Create a `.env` file in the same directory as your notebook to store your API key securely:

```
ANTHROPIC_API_KEY="your-api-key-here"
```

This keeps your API key out of your code and prevents accidentally committing it to version control. Always add `.env` to your `.gitignore` file.

Then load the environment variables and create your API client:

```python
from dotenv import load_dotenv
load_dotenv()

from anthropic import Anthropic

client = Anthropic()
model = "claude-sonnet-4-0"
```

---

#### The `create` Function

The core of making API requests is `client.messages.create()`. It requires three parameters:

- **`model`** — the name of the Claude model to use.
- **`max_tokens`** — a safety limit on response length, not a target. If set to `1000`, Claude stops after 1,000 tokens even if it has more to say. Claude does not try to reach this limit — it writes what it considers appropriate and stops if it hits the maximum.
- **`messages`** — the conversation history sent to Claude (see below).

---

#### Understanding Messages

Messages represent the conversation between you and Claude, similar to a chat application. There are two types:

- **User messages** — content you send to Claude.
- **Assistant messages** — responses Claude has generated.

Each message is a dictionary with a `role` (`"user"` or `"assistant"`) and `content` (the actual text).

---

#### Making Your First Request

```python
message = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=[
        {
            "role": "user",
            "content": "What is quantum computing? Answer in one sentence"
        }
    ]
)
```

Claude processes the request and returns a response object containing the generated text along with metadata about the request.

---

#### Extracting the Response

The response object contains a lot of information, but you usually just want the generated text. Access it with:

```python
message.content[0].text
```

This returns clean, readable output — for example: *"Quantum computing is a type of computation that leverages quantum mechanics principles like superposition and entanglement to process information using quantum bits (qubits), potentially solving certain complex problems exponentially faster than classical computers."*

With these basics in place, you can start experimenting with different prompts and building more complex interactions with Claude.

---

### Multi-Turn conversations

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287735

#### What you'll learn

*Estimated time: 8 minutes 54 seconds (video)*

By the end of this lesson you'll be able to:

- Explain why Claude is stateless and what that means for conversation management
- Maintain conversation history manually by building and sending a complete message list
- Implement three helper functions (`add_user_message`, `add_assistant_message`, `chat`) to manage multi-turn exchanges

---

#### Video Summary *(8 min 54 sec)*

When working with the Anthropic API, there is a crucial concept you need to understand: Claude does not store any conversation history. Each request is completely independent, with no memory of previous exchanges. If you want multi-turn conversations where Claude remembers earlier context, you need to handle conversation state yourself.

---

#### The Problem with Stateless Conversations

If you ask Claude "What is quantum computing?" and get a response, then follow up with "Write another sentence" — Claude has no idea what you're referring to. It will produce a sentence about something completely random because it has no memory of the quantum computing discussion.

---

#### How Multi-Turn Conversations Work

To maintain conversation context, you need to do two things:

1. **Manually maintain a list of all messages** in your code.
2. **Send the complete message history with every request.**

The flow that works:

1. Send your initial user message to Claude.
2. Take Claude's response and add it to your message list as an assistant message.
3. Add your follow-up question as another user message.
4. Send the entire conversation history to Claude.

---

#### Building Helper Functions

Three helper functions make conversation management straightforward:

```python
def add_user_message(messages, text):
    user_message = {"role": "user", "content": text}
    messages.append(user_message)

def add_assistant_message(messages, text):
    assistant_message = {"role": "assistant", "content": text}
    messages.append(assistant_message)

def chat(messages):
    message = client.messages.create(
        model=model,
        max_tokens=1000,
        messages=messages,
    )
    return message.content[0].text
```

---

#### Putting It All Together

```python
# Start with an empty message list
messages = []

# Add the initial user question
add_user_message(messages, "Define quantum computing in one sentence")

# Get Claude's response
answer = chat(messages)

# Add Claude's response to the conversation history
add_assistant_message(messages, answer)

# Add a follow-up question
add_user_message(messages, "Write another sentence")

# Get the follow-up response with full context
final_answer = chat(messages)
```

Now Claude understands that "Write another sentence" refers to expanding on the quantum computing definition, because the complete conversation context is provided with each request.

These helper functions will be useful throughout your work with Claude, making it much easier to build applications that can maintain meaningful conversations over multiple exchanges.

---

### System prompts

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287733

#### What you'll learn

*Estimated time: 6 minutes 20 seconds (video)*

By the end of this lesson you'll be able to:

- Explain what system prompts are and why they matter for application behaviour
- Pass a system prompt into `client.messages.create()` via the `system` parameter
- Build a flexible `chat()` function that conditionally includes a system prompt
- Contrast Claude's output with and without a system prompt

---

#### Video Summary *(6 min 20 sec)*

System prompts are a powerful way to customise how Claude responds to user input. Instead of generic answers, you can shape Claude's tone, style, and approach to match your specific use case. This lesson uses a math tutor chatbot as a concrete example to show the dramatic difference a system prompt makes.

---

#### Why System Prompts Matter

Consider building a math tutor chatbot. When a student asks "How do I solve 5x + 2 = 3 for x?", you want Claude to act like a real tutor, not just give the answer. A good math tutor should:

- Initially give hints rather than complete solutions.
- Patiently walk students through problems step by step.
- Show solutions for similar problems as examples.

You do not want Claude to immediately give direct answers or tell students to just use a calculator.

---

#### How System Prompts Work

System prompts provide Claude with guidance on how to respond. You define them as plain strings and pass them into `client.messages.create()` via the `system` parameter. Key benefits:

- Claude will try to respond in the same way someone in the specified role would respond.
- Helps keep Claude on task and consistent across turns.

Basic structure:

```python
system_prompt = """
You are a patient math tutor.
Do not directly answer a student's questions.
Guide them to a solution step by step.
"""

client.messages.create(
    model=model,
    messages=messages,
    max_tokens=1000,
    system=system_prompt
)
```

---

#### Seeing the Difference

Without a system prompt, Claude gives a complete step-by-step solution immediately — helpful, but it doesn't encourage the student to think through the problem.

With the math tutor system prompt, Claude's response changes dramatically. Instead of the full solution, it asks guiding questions like: *"What do you think would be a good first step to isolate x? Consider what operation we might need to perform on both sides to start moving terms around."*

---

#### Building a Flexible Chat Function

Rather than hard-coding system prompts, make your `chat()` function more reusable by accepting them as an optional parameter:

```python
def chat(messages, system=None):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
    }

    if system:
        params["system"] = system

    message = client.messages.create(**params)
    return message.content[0].text
```

The `system=None` check is important: Claude's API does not accept `system=None`, so the parameter must be conditionally included only when a value is provided.

Usage:

```python
# Without system prompt
answer = chat(messages)

# With system prompt
system = """
You are a patient math tutor.
Do not directly answer a student's questions.
Guide them to a solution step by step.
"""
answer = chat(messages, system=system)
```

System prompts are essential for creating AI applications that behave consistently and appropriately for their intended purpose. They transform generic AI responses into specialised, role-appropriate interactions.

---

### Temperature

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287728

#### What you'll learn

*Estimated time: 6 minutes 7 seconds (video)*

By the end of this lesson you'll be able to:

- Explain how Claude's token sampling process works
- Describe what temperature controls and the effect of values near 0 vs. near 1
- Select an appropriate temperature range for a given task type
- Add the `temperature` parameter to your `chat()` function

---

#### Video Summary *(6 min 7 sec)*

Temperature is a powerful parameter that controls how predictable or creative Claude's responses will be. Understanding how to use it effectively can dramatically improve your AI applications.

---

#### How Claude Generates Text

When you send Claude a prompt, it goes through three key steps:

1. **Tokenization** — breaking your input into smaller chunks.
2. **Prediction** — calculating probabilities for possible next words (e.g. "about" 30%, "would" 20%, "of" 10%, …).
3. **Sampling** — choosing a token based on those probabilities, then repeating the process to build a complete response.

---

#### What Temperature Does

Temperature is a decimal value between 0 and 1 that directly influences the sampling probabilities — think of it as a "creativity dial".

- **Low temperature (near 0)** — Claude becomes very deterministic, almost always picking the highest-probability token. At `0.0`, a single token can receive 100% probability.
- **High temperature (near 1)** — probabilities spread more evenly across options, leading to more varied and creative outputs.

---

#### Choosing the Right Temperature

| Range | Best for |
|---|---|
| **0.0 – 0.3** (Low) | Factual responses, coding assistance, data extraction, content moderation |
| **0.4 – 0.7** (Medium) | Summarisation, educational content, problem-solving, creative writing with constraints |
| **0.8 – 1.0** (High) | Brainstorming, creative writing, marketing content, joke generation |

---

#### Implementing Temperature in Code

Add `temperature` as an optional parameter to your existing `chat()` function:

```python
def chat(messages, system=None, temperature=1.0):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature
    }

    if system:
        params["system"] = system

    message = client.messages.create(**params)
    return message.content[0].text
```

The two changes from the previous version: add `temperature=1.0` as a parameter and include `"temperature": temperature` in the `params` dictionary.

---

#### Testing Temperature Effects

To see temperature in action, try generating movie ideas at both extremes:

```python
# Low temperature - more predictable
answer = chat(messages, temperature=0.0)

# High temperature - more creative
answer = chat(messages, temperature=1.0)
```

At `0.0` you might consistently get *"A time-traveling archaeologist must prevent ancient artifacts from being stolen."* At `1.0` you will see much more variety in themes, characters, and plot elements.

---

#### Key Takeaways

Temperature does not guarantee different outputs — it changes the *probability* of getting them. Even at high temperatures, Claude might occasionally produce similar responses. Match your temperature choice to your use case:

- Need consistent, factual responses? Use low temperature.
- Want creative brainstorming? Dial up to high temperature.
- General tasks? Medium temperatures work well for most cases.

---

### Response streaming

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287734

#### What you'll learn

*Estimated time: 8 minutes 23 seconds (video)*

By the end of this lesson you'll be able to:

- Explain the UX problem streaming solves and how it works at the event level
- Identify the five stream event types Claude sends
- Implement basic streaming with `stream=True` on `client.messages.create()`
- Use the simplified `client.messages.stream()` interface to display text chunks
- Retrieve the complete assembled message after streaming with `stream.get_final_message()`

---

#### Video Summary *(8 min 23 sec)*

When building chat applications with Claude, responses can take 10–30 seconds to generate, leaving users staring at a loading spinner. Response streaming lets users see text appear chunk by chunk as Claude generates it, creating a much more responsive feel.

---

#### The Problem with Standard Responses

In a typical chat setup, your server sends a user message to Claude and waits for the complete response before sending anything back to the client. This creates an awkward delay where users have no feedback that anything is happening.

---

#### How Streaming Works

With streaming enabled, Claude immediately sends back an initial response indicating it has received your request and is starting to generate text. You then receive a series of events, each containing a small piece of the overall response. Your server can forward these text chunks to the client as they arrive, allowing users to see the response building word by word — all within a single request to Claude.

---

#### Understanding Stream Events

When streaming is enabled, Claude sends back five types of events:

- **`MessageStart`** — a new message is being sent.
- **`ContentBlockStart`** — start of a new block containing text, tool use, or other content.
- **`ContentBlockDelta`** — chunks of the actual generated text (these are the ones to display to users).
- **`ContentBlockStop`** — the current content block has been completed.
- **`MessageDelta`** / **`MessageStop`** — the current message is complete and all information has been sent.

---

#### Basic Streaming Implementation

Add `stream=True` to your `messages.create()` call to enable streaming:

```python
messages = []
add_user_message(messages, "Write a 1 sentence description of a fake database")

stream = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    stream=True
)

for event in stream:
    print(event)
```

---

#### Simplified Text Streaming

Rather than manually parsing events, use the SDK's `client.messages.stream()` interface, which filters out everything except the actual text content:

```python
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="")
```

This is usually the right approach for displaying responses to users.

---

#### Getting the Complete Message

While streaming individual chunks improves UX, you often also need the complete assembled message for storage or further processing. Retrieve it after streaming completes:

```python
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages
) as stream:
    for text in stream.text_stream:
        # Send each chunk to your client
        pass

    # Get the complete message for database storage
    final_message = stream.get_final_message()
```

This gives you the best of both worlds: real-time streaming for users and a complete message object for your application logic.

---

### Structured data

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287732

#### What you'll learn

*Estimated time: 5 minutes 59 seconds (video)*

By the end of this lesson you'll be able to:

- Identify the core problem with Claude's default structured data output
- Use assistant message prefilling to control the start of Claude's response
- Add `stop_sequences` to halt generation at a precise point
- Combine both techniques to extract clean JSON (or other structured formats) without extra commentary
- Apply the same approach to Python code snippets, bulleted lists, CSV, and other formatted content

---

#### Video Summary *(5 min 59 sec)*

When you need Claude to generate structured data like JSON, Python code, or bulleted lists, you frequently run into the same problem: Claude wants to be helpful and add explanatory text around your content. While this is great in conversational contexts, it creates friction when you need raw, machine-readable output that can be used directly in an application.

---

#### The Problem with Default Responses

Consider a web app that generates AWS EventBridge rules. Users enter a description, click generate, and expect clean JSON they can immediately copy and use. By default, Claude wraps the JSON in markdown code fences and appends a prose explanation:

````
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running"]
  }
}
```

This rule captures EC2 instance state changes when instances start running.
````

The JSON is correct, but users cannot copy the entire response — they have to manually select just the JSON portion. This is unacceptable UX for a generation tool.

---

#### The Solution: Assistant Message Prefilling + Stop Sequences

Combine **assistant message prefilling** with **stop sequences** to extract exactly the content you want with nothing else.

```python
messages = []

add_user_message(messages, "Generate a very short event bridge rule as json")
add_assistant_message(messages, "```json")

text = chat(messages, stop_sequences=["```"])
```

The technique works in four steps:

1. The user message tells Claude what to generate.
2. The prefilled assistant message makes Claude believe it already started a markdown code block.
3. Claude continues by writing just the JSON content (since the opening fence is already "there").
4. When Claude tries to close the block with ` ``` `, the stop sequence triggers and halts generation immediately.

The result is clean JSON with no extra formatting or commentary.

---

#### Updating the `chat()` Function

To support stop sequences, add a `stop_sequences` parameter to your existing `chat()` function:

```python
def chat(messages, system=None, temperature=1.0, stop_sequences=None):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature,
    }

    if system:
        params["system"] = system

    if stop_sequences:
        params["stop_sequences"] = stop_sequences

    message = client.messages.create(**params)
    return message.content[0].text
```

---

#### Processing the Response

The returned text may contain leading/trailing newline characters. Strip them before parsing:

```python
import json

# Clean up and parse the JSON
clean_json = json.loads(text.strip())
```

---

#### Beyond JSON

This technique is not limited to JSON. Use it any time you need structured output without commentary:

- Python code snippets (prefill with ` ```python `, stop on ` ``` `)
- Bulleted lists
- CSV data
- Any formatted content where you want just the content, not explanations

The key is identifying what Claude naturally wants to wrap your content in, then using that opening marker as your prefill and the closing marker as your stop sequence.

---

#### Key Takeaways

- **Prefilling** steers Claude into a specific output format by providing the opening of its response.
- **Stop sequences** cut generation at exactly the right point, preventing closing markers and trailing commentary from appearing.
- Together, these two parameters give you precise, programmatic control over Claude's output — essential for integrating AI-generated content into applications that consume structured data.

---

## Lesson 2 — Prompt Evaluation

### Prompt evaluation

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287731

#### What you'll learn

*Estimated time: 1 minute 48 seconds (video)*

By the end of this lesson you'll be able to:

- Distinguish prompt engineering from prompt evaluation and explain why both are necessary
- Recognise the three paths available after writing a prompt and the risks of each
- Articulate why engineers systematically under-test prompts in practice
- Explain the evaluation-first approach and the advantages it provides

---

#### Video Summary *(1 min 48 sec)*

Writing a good prompt is just the beginning. To build reliable AI applications you need two complementary disciplines: **prompt engineering** (techniques for crafting effective prompts) and **prompt evaluation** (systematically measuring how well those prompts perform).

---

#### Prompt Engineering vs. Prompt Evaluation

**Prompt engineering** is the toolkit for writing effective prompts. It includes techniques such as multishot prompting, structuring inputs with XML tags, and other best practices that help Claude understand exactly what you are asking for and how you want it to respond.

**Prompt evaluation** takes a different angle. Instead of focusing on *how* to write prompts, it focuses on *measuring* their effectiveness through automated testing — running prompts against expected answers, comparing versions, and reviewing outputs for errors.

---

#### Three Paths After Writing a Prompt

Once a prompt is drafted, there are three options for what to do next:

1. **Test once and ship** — decide it is good enough after a single run. High risk: the prompt will break in production when users provide unexpected inputs.
2. **Test a few times and tweak** — handle one or two corner cases manually. Better, but users will still encounter edge cases that were never considered.
3. **Run an evaluation pipeline** — score the prompt against objective metrics, then iterate based on the data. Requires more upfront investment in time and infrastructure, but provides significantly greater confidence in reliability.

---

#### Why Engineers Fall Into Testing Traps

Options 1 and 2 are common traps. It is natural to write a prompt for a serious application and not test it thoroughly enough — we systematically underestimate how many edge cases real users will encounter. What seemed like a solid prompt during limited testing can break down quickly when exposed to the full variety of real-world inputs.

---

#### The Evaluation-First Approach

Option 3 represents a systematic, data-driven approach to prompt development. Running prompts through an evaluation pipeline allows you to:

- Identify weaknesses before they become production issues
- Compare different prompt versions objectively
- Iterate with confidence based on measurable improvements
- Build more reliable AI applications overall

The upfront cost in testing infrastructure pays dividends in the robustness of the final application. The goal is to catch problems during development — not after users encounter them.

---

### A typical eval workflow

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287736

#### What you'll learn

By the end of this lesson you'll be able to:

- Describe the five-step eval loop and what happens at each stage
- Construct a minimal eval dataset and explain how to scale it
- Explain how a grader scores Claude's outputs and why numerical scoring matters
- Iterate on a prompt using objective scores as the decision criterion

---

#### Overview

A typical prompt evaluation workflow follows five repeating steps that let you improve prompts through objective measurement. The same core process applies regardless of whether you use open-source tooling or a paid platform — understanding it lets you start small and scale up as needed.

---

#### Step 1 — Draft a Prompt

Write an initial prompt to use as a baseline. Keep it simple at first; the goal is to establish a reference score you can beat.

```python
prompt = f"""
Please answer the user's question:

{question}
"""
```

---

#### Step 2 — Create an Eval Dataset

The eval dataset contains sample inputs representing the types of requests your prompt will handle in production. Each item is interpolated into the prompt template at runtime.

Example dataset (three questions):

- `"What's 2+2?"`
- `"How do I make oatmeal?"`
- `"How far away is the Moon?"`

In real-world evaluations the dataset may contain tens, hundreds, or thousands of records. You can assemble these by hand or use Claude to generate them for you.

---

#### Step 3 — Feed Through Claude

Merge each dataset item with the prompt template to produce a complete prompt, then send each one to Claude and collect the responses.

Example — the math question becomes:

```
Please answer the user's question:

What's 2+2?
```

Claude might return `"2 + 2 = 4"` for the math question, oatmeal cooking instructions for the second, and the distance to the Moon for the third.

---

#### Step 4 — Feed Through a Grader

The grader evaluates Claude's responses by examining both the original question and Claude's answer, producing a numerical score — typically 1 to 10, where 10 is a perfect answer.

Example scores:

| Question | Score | Reason |
|---|---|---|
| Math (`2+2`) | 10 | Perfect answer |
| Oatmeal | 4 | Needs improvement |
| Moon distance | 9 | Very good answer |

Average: (10 + 4 + 9) ÷ 3 = **7.66**

---

#### Step 5 — Change Prompt and Repeat

With a baseline score in hand, modify the prompt and run the full pipeline again to see whether the change improves performance.

Example improvement — adding an instruction for depth:

```python
prompt = f"""
Please answer the user's question:

{question}

Answer the question with ample detail
"""
```

After re-running the eval, this revised prompt might score **8.7** — a measurable improvement over the baseline 7.66.

---

#### Key Takeaways

- **Numerical scores** let you compare prompt versions objectively rather than relying on gut feel.
- **Dataset scale matters** — a handful of test cases misses the edge cases real users will hit; aim for breadth and diversity.
- **Iteration is the point** — the workflow is a loop: draft → evaluate → score → revise → repeat until performance meets your bar.
- This approach removes guesswork from prompt engineering and gives you confidence that changes are genuine improvements, not just different variations.

---

### Generating test datasets

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287739

#### What you'll learn

By the end of this lesson you'll be able to:

- Define a concrete prompt with structured output requirements suitable for evaluation
- Decide between hand-curating and auto-generating an eval dataset
- Use Claude (Haiku) with prefilling and stop sequences to generate a JSON eval dataset programmatically
- Parse and persist the dataset to `dataset.json` for reuse across eval runs

---

#### Overview

Building a prompt evaluation system starts with two things: a well-defined prompt and a dataset of test inputs. This lesson walks through both, using a running example of a prompt that helps users write AWS-specific code.

---

#### Defining the Goal Prompt

The target prompt must return clean, structured output — no explanatory text, headers, or footers — in one of three formats depending on the task:

- Python code
- JSON configuration files
- Regular expressions

**Prompt v1 (baseline):**

```python
prompt = f"""
Please provide a solution to the following task:
{task}
"""
```

This minimal prompt establishes the baseline score that subsequent iterations must beat.

---

#### What an Eval Dataset Looks Like

An eval dataset is an array of JSON objects, each with a `"task"` property describing what Claude needs to accomplish. Example:

```json
[
  { "task": "Description of a Python task for AWS" },
  { "task": "Description of a JSON config task for AWS" },
  { "task": "Description of a regex task for AWS" }
]
```

The dataset can be assembled by hand or generated automatically using Claude. For test data generation, a faster, cheaper model like **Haiku** is preferred.

---

#### Generating the Dataset with Code

**Helper functions** (reused from earlier lessons):

```python
def add_user_message(messages, text):
    user_message = {"role": "user", "content": text}
    messages.append(user_message)

def add_assistant_message(messages, text):
    assistant_message = {"role": "assistant", "content": text}
    messages.append(assistant_message)

def chat(messages, system=None, temperature=1.0, stop_sequences=[]):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature
    }
    if system:
        params["system"] = system
    if stop_sequences:
        params["stop_sequences"] = stop_sequences
    response = client.messages.create(**params)
    return response.content[0].text
```

**Dataset generation function:**

```python
def generate_dataset():
    prompt = """
Generate an evaluation dataset for a prompt evaluation. The dataset will be used to evaluate prompts that generate Python, JSON, or Regex specifically for AWS-related tasks. Generate an array of JSON objects, each representing a task that requires Python, JSON, or a Regex to complete.

Example output:
```json
[
  {
    "task": "Description of task",
  },
  ...additional
]
```

* Focus on tasks that can be solved by writing a single Python function, a single JSON object, or a single regex
* Focus on tasks that do not require writing much code

Please generate 3 objects.
"""
    messages = []
    add_user_message(messages, prompt)
    add_assistant_message(messages, "```json")
    text = chat(messages, stop_sequences=["```"])
    return json.loads(text)
```

Key points on this implementation:

- **Prefilling** with ` ```json ` forces Claude to return raw JSON immediately.
- **Stop sequence** ` ``` ` halts generation before the closing fence, leaving only the array contents.
- `json.loads(text)` parses the stripped response directly into a Python list.

---

#### Testing and Saving the Dataset

```python
dataset = generate_dataset()
print(dataset)

# Persist for reuse across eval runs
with open('dataset.json', 'w') as f:
    json.dump(dataset, f, indent=2)
```

`dataset.json` is saved in the same directory as the notebook and loaded in subsequent evaluation steps — keeping data generation separate from evaluation execution.

---

#### Key Takeaways

- Reusing the **prefilling + stop sequence** pattern from the Structured data lesson is the cleanest way to get parseable JSON out of Claude.
- Using a **cheaper model (Haiku)** for dataset generation keeps costs low — this is not the task where you need full model quality.
- Persisting the dataset ensures **reproducible evaluation runs**: the same inputs are used every time you test a new prompt version.

---

### Running the eval

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287743

#### What you'll learn

By the end of this lesson you'll be able to:

- Implement the three core functions of an eval pipeline (`run_prompt`, `run_test_case`, `run_eval`)
- Explain the responsibility of each function and how they compose
- Execute the pipeline against a persisted dataset and interpret the structured results
- Identify what the hardcoded placeholder score reveals about the next step (grading)

---

#### Overview

With the eval dataset ready, this lesson builds the core evaluation pipeline: take each test case, merge it with the prompt template, send it to Claude, grade the result, and collect structured output. The grading logic is intentionally deferred — a placeholder score of `10` keeps the pipeline testable while the focus is on the overall scaffold.

---

#### The Three Core Functions

**`run_prompt` — merges the prompt template with one test case and returns Claude's output**

```python
def run_prompt(test_case):
    """Merges the prompt and test case input, then returns the result"""
    prompt = f"""
Please solve the following task:

{test_case["task"]}
"""
    messages = []
    add_user_message(messages, prompt)
    output = chat(messages)
    return output
```

The prompt is intentionally minimal at this stage — no formatting instructions yet. Claude will return verbose output, which is expected and will be addressed during prompt iteration.

---

**`run_test_case` — runs one test case end-to-end and returns a scored result object**

```python
def run_test_case(test_case):
    """Calls run_prompt, then grades the result"""
    output = run_prompt(test_case)

    # TODO - Grading
    score = 10

    return {
        "output": output,
        "test_case": test_case,
        "score": score
    }
```

The hardcoded `score = 10` is a placeholder. It keeps the pipeline functional and the result schema stable while the grading logic is built out in subsequent lessons.

---

**`run_eval` — iterates over the full dataset and collects all results**

```python
def run_eval(dataset):
    """Loads the dataset and calls run_test_case with each case"""
    results = []
    for test_case in dataset:
        result = run_test_case(test_case)
        results.append(result)
    return results
```

---

#### Executing the Pipeline

Load the persisted dataset and run the eval:

```python
with open("dataset.json", "r") as f:
    dataset = json.load(f)

results = run_eval(dataset)
```

Even with Claude Haiku, expect ~30 seconds for a full dataset run. Performance optimisation is covered later.

---

#### Examining the Results

Each result object contains three fields:

- **`output`** — Claude's complete response to the test case
- **`test_case`** — the original input object from the dataset
- **`score`** — the evaluation score (hardcoded `10` for now)

```python
print(json.dumps(results, indent=2))
```

The verbose output at this stage is a clear signal: without explicit formatting instructions, Claude adds explanation and context around the raw structured content. This is exactly what the next prompt iteration will address.

---

#### Key Takeaways

- The three-function structure (`run_prompt` → `run_test_case` → `run_eval`) cleanly separates concerns: prompt rendering, per-case orchestration, and dataset iteration.
- The hardcoded score is intentional scaffolding — it proves the pipeline works before investing in grading logic.
- This scaffold represents the majority of what any eval pipeline does; complexity comes from better prompts, real grading, and performance optimisation.
- The next step is replacing the placeholder score with a real grader.

---

### Model based grading

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287742

#### What you'll learn

By the end of this lesson you'll be able to:

- Distinguish the three grader types (code, model, human) and select the right one for each criterion
- Define concrete evaluation criteria for a code generation prompt
- Implement a `grade_by_model` function that returns structured JSON with score and reasoning
- Explain why asking for strengths, weaknesses, and reasoning produces more reliable scores
- Wire the grader into `run_test_case` and compute an average score across the full dataset

---

#### Overview

A grader takes model output and returns a measurable quality signal — typically a number from 1 (poor) to 10 (high quality). There are three approaches, each with different trade-offs.

---

#### Types of Graders

**Code graders** — programmatic checks implemented in custom logic. Common uses:

- Checking output length
- Verifying presence or absence of specific words
- Syntax validation for JSON, Python, or regex
- Readability scores

The only requirement is that the function returns a usable numeric signal.

**Model graders** — feed the original output into a second API call for evaluation. Highly flexible; well-suited for assessing response quality, instruction following, completeness, helpfulness, and safety.

**Human graders** — manual review. Most flexible but time-consuming. Useful for evaluating general quality, comprehensiveness, depth, conciseness, and relevance.

---

#### Defining Evaluation Criteria

Before implementing any grader, define clear criteria. For a code generation prompt, three criteria apply:

| Criterion | Description | Best grader |
|---|---|---|
| **Format** | Returns only Python, JSON, or regex without explanation | Code grader |
| **Valid syntax** | Produced code has valid syntax | Code grader |
| **Task following** | Response directly addresses the task with accurate code | Model grader |

The first two are deterministic checks — ideal for code graders. Task following requires semantic judgement, which is why a model grader is more appropriate.

---

#### Implementing a Model Grader

```python
def grade_by_model(test_case, output):
    task = test_case["task"]
    solution = output

    eval_prompt = f"""
    You are an expert code reviewer. Evaluate this AI-generated solution.

    Task: {task}
    Solution: {solution}

    Provide your evaluation as a structured JSON object with:
    - "strengths": An array of 1-3 key strengths
    - "weaknesses": An array of 1-3 key areas for improvement
    - "reasoning": A concise explanation of your assessment
    - "score": A number between 1-10
    """

    messages = []
    add_user_message(messages, eval_prompt)
    add_assistant_message(messages, "```json")

    eval_text = chat(messages, stop_sequences=["```"])
    return json.loads(eval_text)
```

**Key design insight:** asking for `strengths`, `weaknesses`, and `reasoning` alongside the score prevents the model from defaulting to middling scores around 6. The structured context forces a more deliberate, calibrated assessment.

The prefilling + stop sequence pattern is reused here to extract clean, parseable JSON from the grader response.

---

#### Wiring the Grader into the Pipeline

Update `run_test_case` to call the grader and propagate `score` and `reasoning`:

```python
def run_test_case(test_case):
    output = run_prompt(test_case)

    model_grade = grade_by_model(test_case, output)
    score = model_grade["score"]
    reasoning = model_grade["reasoning"]

    return {
        "output": output,
        "test_case": test_case,
        "score": score,
        "reasoning": reasoning
    }
```

Update `run_eval` to compute and print the average score:

```python
from statistics import mean

def run_eval(dataset):
    results = []

    for test_case in dataset:
        result = run_test_case(test_case)
        results.append(result)

    average_score = mean([result["score"] for result in results])
    print(f"Average score: {average_score}")

    return results
```

The average score is the single objective metric used to compare prompt versions. While model graders can be somewhat variable, they provide a consistent enough baseline for tracking incremental improvements.

---

#### Key Takeaways

- Match grader type to criterion: deterministic checks → code grader; semantic judgement → model grader.
- Requesting structured reasoning alongside the score is critical — it stabilises grader output and prevents score compression toward the middle.
- The prefilling + stop sequence pattern appears again here, reinforcing it as the standard technique for extracting structured data from Claude.
- The average score across all test cases is the core iteration signal: run eval → tweak prompt → run eval again → compare averages.

---

## Lesson 3 — Prompt engineering techniques

### Prompt engineering

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287745

#### What you'll learn

By the end of this lesson you'll be able to:

- Describe the iterative prompt improvement cycle and when to stop iterating
- Set up a `PromptEvaluator` instance with controlled concurrency
- Generate a typed eval dataset using `evaluator.generate_dataset()` with a `prompt_inputs_spec`
- Write a naive baseline prompt and understand why low initial scores (~2–3/10) are expected
- Pass `extra_criteria` into `evaluator.run_evaluation()` to target domain-specific requirements
- Interpret HTML evaluation reports to identify where a prompt is failing

---

#### Overview

Prompt engineering is the process of taking an initial prompt and iteratively refining it to produce more reliable, higher-quality outputs. Each iteration applies a specific technique and is validated against objective evaluation scores — no changes are kept unless they demonstrably improve the average.

---

#### The Iterative Improvement Cycle

1. **Set a goal** — define what the prompt must accomplish.
2. **Write an initial prompt** — create a simple baseline; do not over-engineer it.
3. **Evaluate the prompt** — run it through the eval pipeline and record the score.
4. **Apply a prompt engineering technique** — make one targeted change.
5. **Re-evaluate** — verify the change produced a measurable improvement.
6. Repeat steps 4–5 until performance meets the target.

---

#### Running Example: Athlete Meal Plans

The lesson uses a prompt that generates one-day meal plans for athletes, given height, weight, goal, and dietary restrictions — a realistic case where output quality is easy to grade semantically.

---

#### Setting Up the Evaluator

The `PromptEvaluator` class handles dataset generation and model grading. Control concurrency with `max_concurrent_tasks` to avoid rate limit errors:

```python
evaluator = PromptEvaluator(max_concurrent_tasks=5)
```

Start at `3` and increase only if your API quota allows it.

---

#### Generating a Typed Eval Dataset

Define the prompt's required inputs via `prompt_inputs_spec` — the evaluator uses these to auto-generate realistic test cases:

```python
dataset = evaluator.generate_dataset(
    task_description="Write a compact, concise 1 day meal plan for a single athlete",
    prompt_inputs_spec={
        "height": "Athlete's height in cm",
        "weight": "Athlete's weight in kg",
        "goal": "Goal of the athlete",
        "restrictions": "Dietary restrictions of the athlete"
    },
    output_file="dataset.json",
    num_cases=3
)
```

Keep `num_cases` at 2–3 during development for fast iteration; increase for final validation.

---

#### Writing the Baseline Prompt

Start deliberately simple — the baseline exists only to establish a score floor:

```python
def run_prompt(prompt_inputs):
    prompt = f"""
What should this person eat?

- Height: {prompt_inputs["height"]}
- Weight: {prompt_inputs["weight"]}
- Goal: {prompt_inputs["goal"]}
- Dietary restrictions: {prompt_inputs["restrictions"]}
"""
    messages = []
    add_user_message(messages, prompt)
    return chat(messages)
```

A score of ~2.3/10 on a first attempt is normal. Low initial scores are useful — they give more room to demonstrate measurable improvement.

---

#### Running the Evaluation with Extra Criteria

Pass domain-specific requirements via `extra_criteria` so the grader scores against what actually matters:

```python
results = evaluator.run_evaluation(
    run_prompt_function=run_prompt,
    dataset_file="dataset.json",
    extra_criteria="""
The output should include:
- Daily caloric total
- Macronutrient breakdown
- Meals with exact foods, portions, and timing
"""
)
```

---

#### Analysing Results

The evaluator returns both a numerical average score and a detailed HTML report showing per-test-case performance and the grader's reasoning for each score. Use the report to pinpoint exactly where the prompt is failing before selecting which technique to apply next.

---

#### Key Takeaways

- Never skip the baseline — a low score is informative, not discouraging.
- Apply **one technique at a time** and re-evaluate; stacking multiple changes makes it impossible to attribute score changes to a specific improvement.
- `extra_criteria` is the lever for domain alignment — it ensures the grader cares about the same things the end user does.
- The HTML report is the primary debugging tool: read the reasoning, not just the number.

---

### Being clear and direct

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287744

#### What you'll learn

By the end of this lesson you'll be able to:

- Explain why the first line of a prompt is the most consequential
- Distinguish between clarity (what you want) and directness (how you structure the ask)
- Rewrite vague or question-based prompts as direct, action-verb-led instructions
- Quantify the impact of this technique using eval scores

---

#### Overview

The first line of a prompt sets the stage for everything that follows. Getting it right — with clarity and directness — is the highest-leverage single change you can make to a prompt.

---

#### Two Principles

**Clarity — what you want**

- Use simple language that leaves no room for ambiguity.
- State exactly what you want without circumlocution.
- Lead with a straightforward statement of the task.

Weak: *"I need to know about those things people put on their roofs that use sun — those solar panel things, I think they're called."*
Strong: *"Write three paragraphs about how solar panels work."*

**Directness — how you structure the request**

- Use instructions, not questions.
- Start with an action verb.
- Be specific about the desired output.

Weak: *"Can you tell me about some countries that use geothermal energy?"*
Strong: *"List five countries that use geothermal energy. Include generation stats for each."*

---

#### Applying to the Running Example

The baseline meal plan prompt opened with: `"What should this person eat?"` — a question with no action verb, no output specification, and no constraints stated up front.

The improved version: `"Generate a one-day meal plan for an athlete that meets their dietary restrictions."`

This single-line revision immediately communicates:

- **What action to take** — generate
- **What to create** — a meal plan
- **Key constraints** — one day, for an athlete, meeting dietary restrictions

---

#### Measured Impact

| Prompt version | Average eval score |
|---|---|
| Baseline (`"What should this person eat?"`) | 2.32 |
| After applying clear & direct | 3.92 |

A ~70% score increase from restructuring a single line.

---

#### Key Takeaway

Treat Claude like a capable assistant who needs clear direction, not someone who has to guess your intent. Lead with a direct action verb, specify the output, and state constraints up front — before adding any supporting context.

---

### Being specific

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287740

#### What you'll learn

By the end of this lesson you'll be able to:

- Explain why specificity reduces output variance and improves quality
- Distinguish between output quality guidelines and process steps
- Know when to apply each type and when to combine them
- Quantify the impact of specificity using eval scores

---

#### Overview

Without specific guidelines, Claude has to infer intent — and a single vague prompt can produce wildly different outputs in terms of length, structure, tone, and content. Being specific gives Claude a precise target, dramatically improving both consistency and quality.

---

#### Two Types of Guidelines

**Output quality guidelines** — list qualities the output must have. Control:

- Length and format
- Specific attributes or elements to include
- Tone or style requirements

Example for a story prompt: *under 1,000 words, include a clear action that reveals the character's talent, feature at least one supporting character.*

**Process steps** — provide an explicit sequence for Claude to follow before producing the final answer. Useful when you want systematic reasoning across multiple perspectives rather than an immediate response.

Example for a story prompt:
1. Brainstorm three talents that would create dramatic tension
2. Pick the most interesting talent
3. Outline a pivotal scene that reveals the talent
4. Brainstorm supporting character types that could increase the impact

---

#### When to Use Each

| Approach | When to use |
|---|---|
| **Output guidelines** | Almost every prompt — they are the baseline safety net for consistent results |
| **Process steps** | Complex troubleshooting, decision-making, critical thinking, any task requiring multiple perspectives |

Example of process steps for a business problem: analysing a sales team performance drop, you would guide Claude through market metrics, industry changes, individual performance, organisational changes, and customer feedback — rather than letting it fixate on one cause.

In professional prompting, both are frequently combined: guidelines control the output shape; process steps ensure thorough reasoning before the output is generated.

---

#### Measured Impact on the Running Example

Adding six explicit output guidelines to the meal plan prompt:

1. Include accurate daily calorie amount
2. Show protein, fat, and carb amounts
3. Specify when to eat each meal
4. Use only foods that fit restrictions
5. List all portion sizes in grams
6. Keep budget-friendly if mentioned

| Prompt version | Average eval score |
|---|---|
| After "clear & direct" | 3.92 |
| After adding specificity guidelines | 7.86 |

More than doubling the score simply by telling Claude exactly what elements to include.

---

#### Key Takeaway

Output quality guidelines belong in almost every prompt — they eliminate guesswork about format, length, and content. Process steps are the tool for tasks requiring structured reasoning. Used together, they deliver both consistent output shape and confidence that Claude has considered all relevant factors before responding.

---

## Lesson 2 — Prompt Evaluation

### Promp engineering techniques