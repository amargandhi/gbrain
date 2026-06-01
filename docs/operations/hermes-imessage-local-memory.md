# Hermes iMessage Local Memory Notes

This note captures a conservative local deployment pattern for running GBrain as the memory layer behind a personal Hermes agent reachable through iMessage on a Mac.

## Target shape

- Mac mini with Apple silicon and 16 GB unified memory.
- Dedicated standard macOS user for the agent.
- Messages.app signed in with the agent Apple Account.
- BlueBubbles as the iMessage bridge.
- Hermes as the agent gateway.
- GBrain as the local-first memory store.
- Local embeddings and reranking for retrieval privacy.

## Recommended first pass

Keep the first version intentionally small:

1. Keep System Integrity Protection enabled.
2. Use BlueBubbles only for basic text and media.
3. Require both Apple Screen Time communication limits and Hermes/BlueBubbles allow-listing.
4. Use GBrain PGLite locally with `gbrain serve` over stdio for the agent.
5. Use a local embedding model such as Qwen3-Embedding-0.6B.
6. Prefer GBrain's native reranker provider path before adding an external retrieval shim.
7. Keep Graphiti or another temporal graph sidecar optional until temporal recall tests show a real gap.

## iMessage safety gates

Apple Screen Time communication limits are useful, but they should not be the only boundary. Also configure the Hermes or BlueBubbles allow-list and test these cases before enabling autonomy:

- An unknown sender messages the agent account.
- A group chat includes one allowed sender and one unknown sender.
- A known sender asks the agent to message a new contact.
- The reserved stop phrase is sent during an active task.

For a SIP-on deployment, avoid features that require the BlueBubbles Private API, including read receipts, tapbacks, typing indicators, and creating new chats by address.

## Retrieval notes

For small personal brains, PGLite is a good starting point. Keep the brain repo as the source of truth and let GBrain index it for retrieval.

When using Qwen3 embedding models, keep vector dimensions compatible with the active pgvector index type. Qwen3-Embedding-0.6B defaults to up to 1024 dimensions, which fits standard HNSW vector indexes. Larger Qwen3 embedding models can exceed standard pgvector HNSW vector limits unless configured to emit fewer dimensions or stored through a compatible type.

Use native GBrain reranking first. Current GBrain versions have reranker integration points, including a local llama.cpp-based path, so a separate rerank shim should be a fallback rather than the default architecture.

## Rollout checklist

- iMessage round trip works for both approved humans.
- Unknown senders are ignored.
- The stop phrase halts or disables the agent.
- GBrain recalls a known fact across sessions.
- Retrieval quality is tested against a small hand-written question set.
- The agent survives a reboot with launchd-managed services.
- Logs and brain git history are reviewed before widening tool permissions.
