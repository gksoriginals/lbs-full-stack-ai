# Project setup

## Take the project home

1. Clone the repository.

   ```bash
   git clone https://github.com/gksoriginals/lbs-full-stack-ai.git
   cd lbs-full-stack-ai
   ```

2. Create a Groq API key and select a model.

   - Open [Groq Console — API Keys](https://console.groq.com/keys) and create an API key.
   - Choose an available chat model from the [Groq model list](https://console.groq.com/docs/models).

3. Create a project-root `.env` file.

   ```env
   GROQ_API_KEY=gsk_your_key_here
   GROQ_MODEL=openai/gpt-oss-20b
   ```

4. Serve the course material.

   ```bash
   python3 -m http.server 8765 --directory docs
   ```

   Open `http://127.0.0.1:8765`.

5. Run the LLM integration demo.

   ```bash
   python3 -m pip install openai python-dotenv
   cd docs/demo
   python3 policy_assistant_demo.py
   ```

   Open `http://127.0.0.1:8080`.

Keep `.env` private. It is ignored by Git and must never be committed.
