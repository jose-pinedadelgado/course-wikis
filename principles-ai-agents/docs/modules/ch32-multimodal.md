# Chapter 32 — Multimodal

## Beyond Text

Modern LLMs can process multiple modalities:

| Modality | Input | Output | Examples |
|----------|-------|--------|----------|
| **Text** | ✅ | ✅ | Chat, generation, analysis |
| **Images** | ✅ | ✅ (some models) | Vision, image generation |
| **Audio** | ✅ | ✅ | Transcription, TTS, voice agents |
| **Video** | ✅ (some) | ❌ (mostly) | Video understanding |
| **Code** | ✅ | ✅ | Code generation, review |

## Key Use Cases

### Vision
- Document understanding (receipts, forms, screenshots)
- UI analysis for testing
- Medical imaging assistance

### Audio
- Voice-based agents (phone support, assistants)
- Meeting transcription and summarization
- Real-time translation

### Image Generation
- Product mockups
- Marketing content
- Data visualization

!!! note "Multimodal Agents"
    The most powerful agents combine modalities — e.g., a support agent that can **see** a screenshot of the user's problem, **read** their description, and **generate** a visual guide for the fix.

??? question "Discussion: Multimodal UX"
    How does adding non-text modalities change the user experience of an agent? What new failure modes appear?
