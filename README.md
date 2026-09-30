# FitBuddy

FitBuddy is a local-first fitness planning web app. It collects a profile, generates a seven-day movement plan and a short nutrition or recovery tip, lets the user revise the plan with feedback, and stores both original and current versions in SQLite. FastAPI serves a responsive Jinja2 interface and a small JSON API.

When `GOOGLE_API_KEY` is configured, workout generation and feedback revisions use Gemini 2.5 Pro and tips use Gemini 2.5 Flash. Without a key, the app runs in local demo mode with built-in content so the complete flow can be tried without external services. The AI output is general wellness guidance, not medical advice.

## Requirements

- Python 3.10 or newer
- VS Code with the Python extension
- A Google AI Studio API key only if you want live Gemini generation

## Open in VS Code and install

1. Open the `fitbuddy` project folder in VS Code (`File > Open Folder`).
2. Open **Terminal > New Terminal** in the project folder.
3. Create and activate a virtual environment.

   **Windows PowerShell**

   ```powershell
   py -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

   If PowerShell blocks activation, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` in that terminal, then activate again.

   **macOS/Linux**

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

4. Install dependencies:

   ```bash
   python -m pip install --upgrade pip
   pip install -r requirements.txt
   ```

5. In VS Code, choose the interpreter from `.venv` (`Ctrl+Shift+P` / `Cmd+Shift+P` → **Python: Select Interpreter**).

## Configure Gemini (optional)

Copy `.env.example` to `.env` and add your Google AI Studio key:

```dotenv
GOOGLE_API_KEY=your_api_key_here
GEMINI_WORKOUT_MODEL=gemini-2.5-pro
GEMINI_TIP_MODEL=gemini-2.5-flash
```

Keep `.env` private; it is excluded from Git. If the key is absent, no Gemini request is made and the app uses the local demo generator. Model access depends on the Google account and API availability; model names can be changed in `.env`.

## Run

From the project root with the virtual environment active:

```bash
uvicorn app.main:app --reload
```

Open [http://127.0.0.1:8000](http://127.0.0.1:8000). The SQLite database `fitbuddy.db` is created automatically in the project root on first launch. Stop the server with `Ctrl+C`.

## Try the application

1. Create a profile using a unique user ID (letters, numbers, `_` and `-`), age, weight, goal, and preferred effort.
2. Review the seven-day plan and nutrition/recovery tip.
3. Submit feedback such as “add an extra recovery day” and review the revised plan.
4. Open **Coach dashboard** to compare the original and current plans.

The dashboard is intentionally a local development view and has no authentication. Do not expose this development server or dashboard to the public internet with real user information.

## API and interactive docs

FastAPI's interactive API explorer is at [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs).

- `GET /health` — application and AI mode status.
- `POST /api/plans` — create a plan. JSON fields: `username`, `user_code`, `age`, `weight_kg`, `goal`, `intensity`.
- `GET /api/plans/{plan_id}` — fetch a plan, profile, tip, and latest feedback.
- `POST /api/plans/{plan_id}/feedback` — JSON fields `plan_id` and `feedback` to revise a plan.
- `POST /generate-workout` and `POST /submit-feedback` — HTML form flows used by the website.
- `GET /view-all-users` — local coach dashboard.

Valid goals are `weight_loss`, `muscle_gain`, `general_wellness`, and `flexibility`; valid intensity values are `low`, `medium`, and `high`.

## Manual checks

With the server running, check `GET /health` in a browser or in the docs page. Create a profile from the homepage, revise the plan with feedback, and confirm both versions appear on the coach dashboard. For the JSON flow, use `/docs` to call `POST /api/plans`, copy its `plan_id`, then call the feedback and read endpoints. In demo mode, the health response reports `local fallback`; after configuring a valid key and restarting, it reports `Gemini`.

## Project layout

```text
app/
  ai.py           Gemini prompts and local fallback
  config.py       Environment configuration
  database.py     SQLAlchemy engine and session
  main.py         FastAPI routes and app startup
  models.py       SQLite user and plan models
  schemas.py      Request validation
templates/        Jinja2 pages
static/css/       Responsive app styling
requirements.txt  Python dependencies
.env.example      Optional configuration template
```
