# token-loom

Self-hosted LLM inference platform for k8s.
Serves an open-source language model via an OpenAI-compatible REST API

## Overview

~~~
[Client] → REST API → [Service] → [Ollama Pod] → [PVC: model storage]
~~~

Ollama runs as a k8s Deployment.
The model is pulled once onto a persistent volume and loaded into RAM on startup,
where it stays resident for the lifetime of the pod.
All inference happens in RAM - disk is not involved after the initial load

## Deployment

~~~bash
helm repo add token-loom https://morooshka.github.io/token-loom
helm upgrade --install token-loom token-loom/token-loom \
  --namespace token-loom \
  --create-namespace
~~~

## Model Download via Init Container

An init container runs before the main pod on every startup.
It checks whether the model is already present on the PVC and pulls it only if not

> First deployment on an empty PVC will take time to download the model before the pod becomes ready.
> Download time depends on model size and network speed.
> This is expected behaviour

## Smoke Test

After deployment, verify end-to-end:

~~~bash
# 1. Confirm the server is up and the model is registered
curl http://<host>:<port>/api/tags

# 2. Confirm inference works
curl http://<host>:<port>/api/generate \
  -d '{
    "model": "<your-model>",
    "prompt": "Say hello",
    "stream": false
  }'
~~~

A successful response contains `"done": true`.
The first request may take additional time while the model loads into RAM
