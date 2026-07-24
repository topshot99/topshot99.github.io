---
layout: post
title: "Building Video Q&A Locally: A Simple Architecture"
date: 2026-07-24 02:35:00 +0000
categories: [video-ai, multimodal-ai, machine-learning]
tags: [video-ai, video-qa, multimodal-ai, llm, rag, open-source, architecture]
author: Manik
excerpt: "A simple look at the high-level architecture behind Video Q&A and how we can build a smaller open-source version locally."
---

# Building Video Q&A Locally: A Simple Architecture

In the [first post](/2026/07/18/building-open-source-video-ai-locally/), I started exploring whether we can build a local, open-source Video Q&A system.

Before writing code, I studied the public architecture and APIs of [TwelveLabs](https://www.twelvelabs.io/). Their actual implementation is proprietary, so this is only a simple high-level understanding of how such a system works.

## The Big Picture

```text
Video
  |
  v
Split into scenes and understand audio
  |
  v
Create searchable video embeddings
  |
  v
Find the most relevant moments
  |
  v
LLM watches those moments and answers
```

### What is each component doing?

- **Video processing:** Reads the video and divides it into smaller scenes or clips. This makes long videos easier to handle.
- **Multimodal understanding:** Looks at frames, movement, speech, music, and other sounds. Video information is not only visual.
- **Embeddings and index:** Converts each clip into numbers that represent its meaning and stores them in a searchable index.
- **Temporal retrieval:** Finds not only the correct video, but also the exact time range where the answer may be present.
- **LLM:** Receives the question and only the relevant clips. It then prepares a natural-language answer, ideally with timestamps.

TwelveLabs provides similar capabilities through **Marengo** for video search and embeddings, and **Pegasus** for video understanding and generation.

## Our Open-Source Version

We can build a smaller version with available open-source tools:

```text
                    +-> PySceneDetect -> Keyframes --+
Video -> FFmpeg ----+                                |
                    +-> WhisperX -> Transcript -------+
                                                     v
                                    SigLIP2 embeddings and captions
                                                     |
                                                     v
                                          PostgreSQL + pgvector
                                                     |
Question -> Search -> BGE reranker -> Relevant clips |
                                                     v
                                             Qwen2.5-VL
                                                     |
                                                     v
                                      Answer with timestamps
```

### Why these choices?

- **FFmpeg** is mature and handles almost every common video format.
- **PySceneDetect** is simple, runs on CPU, and gives us scene boundaries.
- **WhisperX** provides speech transcription with word-level timestamps, which is very useful for finding exact moments.
- **SigLIP2** gives us strong image-and-text embeddings for semantic search.
- **PostgreSQL with pgvector** keeps metadata and vectors in one place. It is easier for a local first version than managing many databases.
- **BGE reranker** helps reorder search results so that the best clips go to the model.
- **Qwen2.5-VL 7B** can understand images and video, supports temporal reasoning, and is small enough to experiment with on a consumer GPU.

This may not match a production platform like TwelveLabs, but it gives us a practical pipeline that we can understand, build, and improve one part at a time.

## What Comes Next?

We will start with the **LLM component** in the next post: what model to use, how to run it locally, and how it can answer questions from selected video frames.
