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

