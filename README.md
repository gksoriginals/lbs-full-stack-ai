# Full Stack & AI — Faculty Development Programme

Interactive course material for the LBS College of Engineering Faculty Development Programme.

## Course site

The GitHub Pages site is published from [`docs/`](./docs/). Open `docs/index.html` locally or enable GitHub Pages from the `main` branch, `/docs` directory after pushing.

## Repository layout

- [`markdown/`](./markdown/) — detailed, presentation-ready source material and speaker notes.
- [`docs/`](./docs/) — standalone interactive HTML chapters and the GitHub Pages site.
- [`docs/demo/`](./docs/demo/) — a minimal local Ollama prototype: one Python server and one browser UI.

## Run the local open-model prototype

Install and run Ollama with a locally pulled model, then:

```bash
ollama pull <MODEL_TAG>
cd docs/demo
OLLAMA_MODEL=<MODEL_TAG> python3 policy_assistant_demo.py
```

Open `http://127.0.0.1:8080`.

The prototype is intentionally framework-free. It demonstrates authorised context assembly, low-variation decoding, JSON validation, citation checking, and safe failure. GitHub Pages hosts the static course material; it cannot run the local Python/Ollama API.
