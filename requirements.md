  # System Requirements

  To run **AI DevCompanion** on your local machine, ensure your environment meets the following specifications.

  ## 1. Prerequisites

  ### Software
  *   **Operating System:** Windows 10/11, macOS, or Linux.
  *   **Node.js:** Version 18.0.0 or higher (for the frontend).
  *   **Python:** Version 3.10 or higher (for the backend).
  *   **Git:** Required to clone the repository.

  ### API Keys
  *   **Google Gemini API Key:** You must have a valid API key from [Google AI Studio](https://aistudio.google.com/). It is free to generate.

  ## 2. Python Dependencies (Backend)
  The following Python packages are required (install via `pip install -r requirements.txt`):
  *   `fastapi`: For the web server.
  *   `uvicorn[standard]`: For the ASGI server implementation.
  *   `python-dotenv`: For loading environment variables.
  *   `google-genai`: The official Google Generative AI SDK.
  *   `pydantic`: For data validation.

  ## 3. Node Dependencies (Frontend)
  The following Node packages are required (install via `npm install`):
  *   `react`, `react-dom`: Core UI library (v19+).
  *   `vite`: Build tool and dev server.
  *   `tailwindcss`: CSS framework (v4+).
  *   `framer-motion`: Animation library.
  *   `lucide-react`: Icon set.
  *   `react-markdown`: For rendering the AI output.

  ## 4. Hardware Recommendations
  *   **RAM:** Minimum 4GB (8GB recommended for smooth development).
  *   **Processor:** Dual-core CPU or better.
