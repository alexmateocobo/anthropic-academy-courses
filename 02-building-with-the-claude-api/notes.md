# Building with the Claude API

Course notes and progress for the Anthropic Academy "Building with the Claude API" course.

## Lesson 1 — Accessing the API

**Source:** https://anthropic.skilljar.com/claude-with-the-anthropic-api/287726

### What you'll learn

*Estimated time: 5 minutes 18 seconds (video)*

By the end of this lesson you'll be able to:

- Describe the five-step request lifecycle for every Claude API interaction
- Explain why API calls must always be routed through a server, never from client-side code
- Identify the required fields in every API request
- Understand the four stages of Claude's internal processing (tokenize, embed, contextualize, generate)
- Interpret the structured API response and its stop reasons

---

### Video Summary *(5 min 18 sec)*

When building applications with Claude, understanding the complete request lifecycle helps you make better architectural decisions and debug issues more effectively. This lesson walks through what happens from the moment a user clicks "send" in your chat interface to when Claude's response appears on screen.

---

### The Five-Step Request Flow

Every interaction with Claude follows a predictable pattern with five distinct phases:

1. **Request to server** — your client app sends the user's message to your own backend.
2. **Request to Anthropic API** — your server forwards the request to Anthropic, authenticated with the securely stored API key.
3. **Model processing** — Claude processes the input through tokenization, embedding, contextualization, and generation.
4. **Response to server** — the Anthropic API returns a structured response to your server.
5. **Response to client** — your server forwards the generated text back to the user interface.

---

### Why You Need a Server

You should never make requests to the Anthropic API directly from client-side code. Here's why:

- API requests require a secret API key for authentication.
- Exposing this key in client code creates a serious security vulnerability.
- Anyone could extract the key and make unauthorized requests.

Instead, your web or mobile app sends requests to your own server, which then communicates with the Anthropic API using the securely stored key.

---

### Making API Requests

When your server contacts the Anthropic API, you can use either an official SDK or make plain HTTP requests. Anthropic provides SDKs for Python, TypeScript, JavaScript, Go, and Ruby.

Every request must include these essential fields:

- **API Key** — identifies your request to Anthropic.
- **Model** — name of the model to use (e.g. `claude-3-sonnet`).
- **Messages** — list containing the user's input text.
- **Max Tokens** — limit for how many tokens Claude can generate.

---

### Inside Claude's Processing

Once Anthropic receives your request, Claude processes it through four main stages:

1. **Tokenization** — Claude breaks your input text into smaller chunks called tokens. These can be whole words, parts of words, spaces, or symbols.
2. **Embedding** — each token gets converted into an embedding — a long list of numbers that represents all possible meanings of that word. Think of embeddings as numerical definitions that capture semantic relationships.
3. **Contextualization** — Claude refines each embedding based on surrounding words to determine the most likely meaning in context (e.g. "quantum" in physics vs. computing). This adjusts the numerical representations to highlight the appropriate definition.
4. **Generation** — the contextualized embeddings pass through an output layer that calculates probabilities for each possible next word. Claude doesn't always pick the highest-probability word — it uses a mix of probability and controlled randomness to create natural, varied responses. After selecting each word, Claude adds it to the sequence and repeats the entire process for the next token.

---

### When Claude Stops Generating

After each token, Claude checks several conditions to decide whether to continue:

- **Max tokens reached** — has it hit the limit you specified?
- **Natural ending** — did it generate an end-of-sequence token?
- **Stop sequence** — did it encounter a predefined stop phrase?

---

### The API Response

When generation completes, the API sends back a structured response containing:

- **Message** — the generated text.
- **Usage** — count of input and output tokens.
- **Stop Reason** — why generation ended.

Your server receives this response and forwards the generated text back to your client application.

---

### Key Takeaways

Understanding this flow helps you:

- Design secure architectures that protect your API keys.
- Set appropriate token limits for your use case.
- Handle different stop reasons in your application logic.
- Debug issues by understanding where they might occur in the pipeline.

