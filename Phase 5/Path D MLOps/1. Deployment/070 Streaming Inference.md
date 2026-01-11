# 070 Streaming Inference

Canonical documentation for 070 Streaming Inference. This document defines concepts, terminology, and standard usage.

## Purpose
070 Streaming Inference addresses the inherent latency challenges associated with Large Language Models (LLMs) and generative AI systems. In traditional synchronous inference, the client must wait for the entire output to be generated before receiving a response, leading to high Time to First Token (TTFT) and poor user experience. 

Streaming inference enables the incremental delivery of model outputs as they are produced. This allows consumers to process or display partial results in real-time, significantly reducing perceived latency and enabling interactive workflows where the system appears to "type" its response.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Mechanisms for incremental data transmission during inference.
* Architectural requirements for stateful and stateless streaming.
* Performance metrics specific to streaming (TTFT, Inter-token latency).
* Data framing and serialization strategies for partial outputs.

**Out of scope:**
* Specific vendor API specifications (e.g., OpenAI, Anthropic, Vertex AI).
* Client-side UI framework implementations (e.g., React hooks for streaming).
* Hardware-level optimizations for inference kernels.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Token** | The fundamental unit of text or data processed by the model; the atomic element of a stream. |
| **Chunk** | A discrete packet of data sent over the network containing one or more tokens or metadata updates. |
| **Time to First Token (TTFT)** | The duration between the initial request and the arrival of the first usable data chunk. |
| **Inter-token Latency** | The average time elapsed between the delivery of consecutive tokens in a stream. |
| **Delta** | A representation of the change or addition to the previous state of the output. |
| **End of Sequence (EOS)** | A specific signal or token indicating the model has completed its generation. |
| **Backpressure** | A signal from the consumer to the producer to slow down data transmission when the consumer cannot keep up. |

## Core Concepts

### Incremental Delivery
Unlike batch inference, streaming inference treats the output as a continuous flow. The inference engine pushes data to the transport layer as soon as a token is sampled, rather than waiting for a terminal state.

### Transport Protocols
Streaming inference typically relies on protocols that support long-lived connections and server-initiated data pushes:
*   **Server-Sent Events (SSE):** A unidirectional, text-based protocol ideal for web-based streaming.
*   **WebSockets:** A bidirectional protocol used when the client needs to send mid-inference signals (e.g., stop sequences).
*   **gRPC/HTTP/2:** Binary protocols that use multiplexed streams for high-performance backend-to-backend communication.

### State Management
Streaming can be **stateless** (where the server retains no memory of the stream after the connection closes) or **stateful** (where the server maintains a session context to allow for interruptions and resumes).

## Standard Model

The standard model for 070 Streaming Inference follows a **Producer-Consumer-Observer** pattern:

1.  **Request Initiation:** The client sends a request with a `stream: true` flag (or equivalent) and establishes a persistent connection.
2.  **Token Generation Loop:** The inference engine enters a loop where it predicts the next token based on the prompt and the previously generated tokens.
3.  **Serialization & Framing:** Each token is wrapped in a standardized frame (often JSON) containing the delta text, index, and optional metadata (e.g., logprobs).
4.  **Transmission:** The frame is flushed immediately to the network buffer.
5.  **Termination:** The stream is closed with a final frame containing completion statistics (e.g., total tokens, finish reason) and an EOS signal.

## Common Patterns

### The Delta Pattern
Instead of sending the entire accumulated string, the server sends only the "delta"—the new characters or tokens generated since the last chunk. This minimizes bandwidth and allows the client to append data efficiently.

### Heartbeat/Keep-Alive
To prevent intermediate proxies or load balancers from closing "idle" connections during long generation pauses, the server may emit empty comment blocks or heartbeat packets.

### Early Termination (Client-Side)
The client may close the connection or send a "stop" signal if the desired information has been received or if the model begins generating undesirable content.

## Anti-Patterns

*   **Buffering Proxies:** Configuring intermediate layers (like NGINX or Cloudflare) to buffer responses, which negates the benefits of streaming by holding chunks until the response is complete.
*   **Over-chunking:** Sending every individual character as a separate network packet, which increases overhead. Optimal streaming usually groups tokens into small, logical chunks.
*   **Lack of Error Handling mid-stream:** Assuming that if the stream starts, it will finish successfully. Errors can occur 50% through a generation; the protocol must support mid-stream error frames.
*   **JSON-in-JSON Escaping:** Improperly escaping delta content, leading to broken JSON parsers on the client side when the model generates special characters.

## Edge Cases

*   **Safety Filter Triggers:** If a content safety filter triggers halfway through a generation, the stream must be terminated immediately with a specific "filtered" status code or metadata flag.
*   **Token Limit Exhaustion:** When a model hits its maximum context window mid-sentence, the stream ends abruptly. The client must handle this "length" finish reason gracefully.
*   **Empty Streams:** Scenarios where the model returns an EOS token immediately without any content.
*   **Network Jitter:** Variations in packet arrival times can make the "typing" effect appear stuttered. Clients often implement a small jitter buffer to smooth out the display.

## Related Topics
*   **071 Inference Optimization:** Techniques like KV-caching that enable faster streaming.
*   **042 Tokenization:** The process of converting text to the units used in streaming.
*   **088 API Design:** Standards for structuring request/response payloads.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |