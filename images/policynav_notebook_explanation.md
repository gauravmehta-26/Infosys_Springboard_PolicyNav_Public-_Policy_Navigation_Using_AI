# 📘 Detailed Explanation: PolicyNav Google Colab Notebook

---

## 🧭 Overview

This notebook builds and runs **PolicyNav** — a full-stack, AI-powered web application that helps citizens understand public policy documents. Users can upload PDF/HTML/TXT policy files, ask questions about them using AI (RAG), get summaries, analyze readability, and visualize entity relationships as knowledge graphs. The app is served using **Streamlit** and made publicly accessible over the internet using **ngrok** (a tunneling tool).

The notebook is divided into **12 code cells**, each writing a separate Python file or running a specific setup step.

---

## 📦 Summary of All Cells

| Cell | Purpose | File Written |
|------|---------|-------------|
| 1 | Install all Python libraries | — |
| 2 | Mount Google Drive, create folders | — |
| 3 | Write database layer | `db.py` |
| 4 | Write configuration constants | `config.py` |
| 5 | Write authentication utilities | `auth.py` |
| 6 | Write readability analyzer | `readability.py` |
| 7 | Write FAISS vector search engine | `vector_store.py` |
| 8 | Write knowledge graph builder | `knowledge_graph.py` |
| 9 | Write AI/NLP engine | `nlp_engine.py` |
| 10 | Write the main Streamlit web app | `app.py` |
| 11 | Seed sample feedback into the database | — |
| 12 | Launch the app publicly via ngrok | — |

---

---

# 🔷 Cell 1 — Install All Required Libraries

**Cell Type:** Code

## Summary
This cell installs every Python library the project needs using `pip`, Python's package manager. The `!` at the start of each line tells Colab to run a shell command (not Python).

---

## Line-by-Line Explanation

```python
!pip install streamlit pyjwt bcrypt python-dotenv pyngrok nltk streamlit-option-menu plotly textstat PyPDF2 beautifulsoup4 streamlit-autorefresh wordcloud matplotlib
```

**What it does:** Installs the first group of libraries. The `!` prefix runs this as a terminal command inside Colab.

**Libraries explained:**

- **`streamlit`** — Turns Python scripts into interactive web applications. This is the main framework for the entire app's UI.
- **`pyjwt`** — Handles JSON Web Tokens (JWT). Used to securely keep users "logged in" without storing passwords in the session.
- **`bcrypt`** — A password hashing library. Instead of storing plain-text passwords (unsafe), it stores a scrambled version that can't be reversed.
- **`python-dotenv`** — Loads environment variables from a `.env` file. Used to keep secrets like API keys out of the code.
- **`pyngrok`** — A Python wrapper around ngrok, a tool that creates a public URL for a server running on your local/Colab machine.
- **`nltk`** — Natural Language Toolkit. A library for working with human language data (tokenizing sentences, etc.).
- **`streamlit-option-menu`** — An extra Streamlit component that creates stylish navigation menus in the sidebar.
- **`plotly`** — Creates interactive, animated charts and graphs for the web.
- **`textstat`** — Calculates text readability scores (e.g., how hard a document is to read).
- **`PyPDF2`** — Reads and extracts text from PDF files.
- **`beautifulsoup4`** — Parses HTML files and extracts their text content.
- **`streamlit-autorefresh`** — Automatically refreshes the Streamlit page at set intervals (used for live dashboards).
- **`wordcloud`** — Generates word cloud images from text.
- **`matplotlib`** — A foundational Python library for creating static charts and plots.

---

```python
!pip install langdetect pandas transformers torch sentence-transformers faiss-cpu accelerate spacy networkx pyvis bitsandbytes -q
```

**What it does:** Installs the second group of libraries. The `-q` flag means "quiet" — it suppresses most output to keep things tidy.

**Libraries explained:**

- **`langdetect`** — Detects what language a piece of text is written in (e.g., English, Hindi).
- **`pandas`** — The most popular Python library for working with tabular data (like Excel spreadsheets, but in code).
- **`transformers`** — A Hugging Face library that provides thousands of pre-trained AI models for NLP (text generation, translation, summarization, etc.).
- **`torch`** — PyTorch, the deep learning framework used to run all the AI models.
- **`sentence-transformers`** — Converts sentences into numerical vectors (embeddings) that capture their meaning. Used for finding similar documents.
- **`faiss-cpu`** — Facebook AI Similarity Search. A library for very fast similarity search over large sets of vector embeddings. Used to find relevant policy document chunks.
- **`accelerate`** — A Hugging Face library that makes running large AI models on GPU/CPU easier.
- **`spacy`** — An industrial-strength NLP library. Used here for Named Entity Recognition (identifying names of people, organizations, laws in text).
- **`networkx`** — A library for creating and analyzing graphs (networks of nodes and edges). Used to build the knowledge graph structure.
- **`pyvis`** — Renders interactive, visual network graphs in the browser. Built on top of networkx.
- **`bitsandbytes`** — Allows AI models to run in lower precision (less memory), which is important on GPUs.

---

```python
!python -m spacy download en_core_web_sm -q
```

**What it does:** Downloads spaCy's small English language model (`en_core_web_sm`). This model is what spaCy uses to recognize named entities (people, organizations, laws, places) in text.

- **`-m spacy`** — Runs spaCy as a module.
- **`download en_core_web_sm`** — Downloads the pre-trained small English model.
- **`-q`** — Quiet mode.

---

```python
!pip install anthropic -q
```

**What it does:** Installs the **Anthropic** Python library, which allows the app to connect to Claude AI (Anthropic's AI assistant). This adds an extra Claude-powered AI chat option on top of the local model.

---

## Output
The output shows installation progress bars and "Successfully installed" messages for each new package downloaded.

---

---

# 🔷 Cell 2 — Mount Google Drive & Set Up Folders

**Cell Type:** Code

## Summary
This cell connects Google Colab to the user's Google Drive so that all files (the database, uploaded documents, AI indexes) are stored **persistently**. Without this, everything would be deleted each time Colab restarts.

---

## Line-by-Line Explanation

```python
import os
```
- **`os`** — A built-in Python module that lets you interact with the operating system (create folders, read environment variables, build file paths, etc.).

---

```python
from google.colab import drive
```
- **`google.colab`** — A special package only available in Google Colab environments.
- **`drive`** — The submodule for mounting Google Drive into the Colab file system.

---

```python
drive.mount('/content/drive', force_remount=True)
```
- **`drive.mount()`** — Connects your Google Drive to Colab's file system.
- **`'/content/drive'`** — The path where Google Drive will appear in Colab. Think of it like plugging in a USB drive at this location.
- **`force_remount=True`** — If Drive is already mounted, unmounts and remounts it fresh. This prevents stale connections.

---

```python
APP_DIR = "/content/drive/MyDrive/PolicyNav"
```
- Creates a Python string variable `APP_DIR` that points to the main app folder inside Google Drive.
- All app files (database, uploaded documents, AI graphs) will be stored here so they survive Colab restarts.

---

```python
os.makedirs(APP_DIR, exist_ok=True)
```
- **`os.makedirs()`** — Creates a directory (folder) and all parent directories if they don't exist.
- **`exist_ok=True`** — If the folder already exists, don't throw an error — just continue.

---

```python
os.makedirs(os.path.join(APP_DIR, 'documents'), exist_ok=True)
```
- **`os.path.join()`** — Safely combines path parts into one path string (handles `/` separators automatically).
- Creates the `documents/` subfolder inside `APP_DIR`. This is where uploaded policy PDF/HTML/TXT files are stored.

---

```python
os.makedirs(os.path.join(APP_DIR, 'graphs'), exist_ok=True)
```
- Creates the `graphs/` subfolder where generated HTML knowledge graph files are saved.

---

```python
os.makedirs(os.path.join(APP_DIR, 'avatars'), exist_ok=True)
```
- Creates the `avatars/` subfolder where user profile pictures are stored.

---

```python
os.environ['APP_DIR'] = APP_DIR
```
- **`os.environ`** — A dictionary of environment variables (system-wide key-value settings).
- This saves `APP_DIR` as an environment variable so other Python scripts (like `db.py`, `app.py`) can read it with `os.environ.get('APP_DIR')`. This avoids hard-coding the path in every file.

---

```python
print(f"✅ Persistent App Directory set to: {APP_DIR}")
```
- Prints a success message. The `f"..."` is an f-string — it inserts the variable value into the string.

## Output
```
Mounted at /content/drive
✅ Persistent App Directory set to: /content/drive/MyDrive/PolicyNav
```

---

---

# 🔷 Cell 3 — Write `db.py` (Database Layer)

**Cell Type:** Code

## Summary
This cell writes the entire **database management file** (`db.py`) to disk using the special `%%writefile` magic command. This file handles all interactions with a **SQLite database** — creating tables, inserting records, updating data, and querying it.

The `%%writefile db.py` at the top means: "Everything below this line gets written into a file called `db.py`."

---

## Imports Inside `db.py`

```python
import os
import sqlite3
import json
```
- **`os`** — Used to build the database file path.
- **`sqlite3`** — Python's built-in module for working with SQLite, a lightweight file-based database. No separate server needed — the database is just one `.db` file.
- **`json`** — Used to serialize (convert to string) and deserialize Python lists/dicts when storing them in the database (SQLite doesn't natively store lists).

---

## Global Configuration

```python
DB_NAME = os.path.join(os.environ.get('APP_DIR', '.'), "policynav_users.db")
```
- Reads `APP_DIR` from environment variables (set in Cell 2). If not set, defaults to `'.'` (current directory).
- The database file will be named `policynav_users.db` inside `APP_DIR`.

---

## Function: `get_connection()`

```python
def get_connection():
    return sqlite3.connect(DB_NAME, check_same_thread=False)
```
- **Purpose:** Creates and returns a connection to the SQLite database.
- **`sqlite3.connect(DB_NAME)`** — Opens a connection to the database file. If it doesn't exist yet, it creates it.
- **`check_same_thread=False`** — By default, SQLite connections can only be used from the thread that created them. This disables that restriction, which is needed for Streamlit (which uses multiple threads).
- **Returns:** A `Connection` object used to run SQL queries.

---

## Function: `init_db()`

**Purpose:** Creates all the database tables when the app starts for the first time (if they don't already exist).

### Tables Created:

**1. `users` table**
```python
cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE,
    email TEXT UNIQUE,
    password_hash TEXT,
    security_question TEXT,
    security_answer_hash TEXT,
    failed_attempts INTEGER DEFAULT 0,
    lock_until TEXT,
    password_history TEXT,
    avatar_path TEXT,
    is_admin INTEGER DEFAULT 0,
    is_online INTEGER DEFAULT 0,
    last_seen TEXT
)
""")
```
- Stores all registered users.
- `PRIMARY KEY AUTOINCREMENT` — Each row gets a unique ID automatically.
- `TEXT UNIQUE` — No two rows can have the same value in that column.
- `DEFAULT 0` — If not specified, the value defaults to 0.
- `failed_attempts` — Counts wrong password attempts (for account lockout).
- `lock_until` — Timestamp when the account gets unlocked.
- `password_history` — JSON list of old password hashes (prevents reusing recent passwords).
- `is_admin` — 1 if the user is an admin, 0 if regular user.

**2. `deleted_accounts` table** — Records emails of deleted accounts so they can't re-register without admin approval.

**3. `pending_registrations` table** — Holds sign-up requests from previously deleted account emails, awaiting admin review.

**4. `activity_log` table** — Logs every action a user takes in the app (which section they used, what they asked, what the AI responded).

**5. `feedback` table** — Stores user ratings and comments for each app section.

**6. `deadlines` table** — Stores policy deadlines that the AI extracts from documents for a given user.

**7. `user_points` table** — Tracks gamification data: total points, login streak, badges earned.

**8. `points_log` table** — Logs every event that awarded points.

### Schema Migration:
```python
cursor.execute("PRAGMA table_info(users)")
cols = [col[1] for col in cursor.fetchall()]
for col, defval in [("avatar_path","TEXT"), ("is_admin","INTEGER DEFAULT 0"), ...]:
    if col not in cols:
        cursor.execute(f"ALTER TABLE users ADD COLUMN {col} {defval}")
```
- **`PRAGMA table_info()`** — SQLite command that returns info about each column in a table.
- This loop checks if certain columns exist. If they don't (because the database was created with an older version of the code), it adds them using `ALTER TABLE`.
- This is called a **database migration** — keeping an existing database up to date with code changes without destroying data.

---

## User CRUD Functions

### `create_user(username, email, password_hash, question, answer_hash)`
- **Purpose:** Inserts a new user into the `users` table.
- `history = json.dumps([password_hash])` — Stores the initial password hash as a JSON list (for history tracking).
- Uses a `try/except/finally` block: if insertion fails (e.g., duplicate email), returns `False`. `finally` always closes the connection.
- **Returns:** `(True, "User created")` on success or `(False, "User exists")` on failure.

### `get_user_by_email(email)`
- **Purpose:** Fetches all data for a single user by their email.
- `cursor.fetchone()` — Returns just the first matching row (or `None` if not found).
- **Returns:** A tuple of all column values for the user.

### `update_login_attempts(email, attempts, lock_until=None)`
- **Purpose:** Updates how many failed login attempts a user has made, and optionally sets a lockout time.

### `update_password(email, new_hash)` / `update_password_history(email, history_json)`
- **Purpose:** Updates the stored password hash and the JSON list of recent password hashes.

### `update_avatar(email, avatar_path)` / `set_user_online(email, is_online)` / `promote_user_to_admin(email)` / `remove_admin_status(email)`
- **Purpose:** Simple database update functions for changing specific user fields.

### `delete_user(email)`
- Records the email in `deleted_accounts` (so the history isn't lost), then removes the user from the `users` table.

### `is_deleted_account(email)`
- **Returns:** `True` if the email is in the `deleted_accounts` table.

---

## Registration Functions

### `add_pending_registration(...)` / `get_pending_registrations()` / `approve_pending_registration(email)`
- Manage a queue of sign-up requests from previously-deleted-account emails that need admin approval before the account is created.

---

## Admin & Activity Functions

### `get_all_users()` / `submit_feedback(...)` / `log_activity(...)` / `get_user_activity(email)` / `get_all_feedback()` / `get_all_activity_logs()`
- Standard read/write functions for admin panels and user activity history.

### `update_email(old_email, new_email)`
- Updates email in `users`, `activity_log`, and `feedback` tables atomically (all changes happen together, or none do — using `rollback()` if something fails).

---

## Deadline Functions

### `save_deadline(...)` / `get_user_deadlines(email)` / `toggle_deadline_done(deadline_id)` / `delete_deadline(deadline_id)`
- CRUD (Create, Read, Update, Delete) operations for storing AI-detected policy deadlines.
- `toggle_deadline_done` uses a SQL `CASE WHEN` to flip a boolean: if done=1, set to 0; else set to 1.

---

## Gamification (Points & Badges)

### `POINTS_MAP` dictionary
```python
POINTS_MAP = {
    "upload_doc":      10,
    "rag_question":    5,
    "summarize":       8,
    "readability":     4,
    "feedback":        3,
    "login_streak_7":  20,
    "login_streak_30": 50,
}
```
- Maps user actions to the number of points they earn. For example, uploading a document gives 10 points.

### `BADGES` list
- A list of dictionaries, each defining a badge:
  - `id` — Unique identifier string.
  - `name` — Display name.
  - `icon` — Emoji icon.
  - `desc` — Description of how to earn it.
  - `condition` — A tuple like `("points", 500)` meaning "needs 500 points".

### `_ensure_points_row(cursor, email)`
- A helper function that inserts a zero-points row for the user if one doesn't exist yet (`INSERT OR IGNORE`).
- The `_` prefix is a Python convention meaning "internal/private helper".

### `award_points(email, action)`
- Looks up how many points the `action` is worth from `POINTS_MAP`.
- Updates the user's total in `user_points` and logs the event in `points_log`.
- Calls `_check_and_award_badges(email)` afterward.

### `update_login_streak(email)`
- Gets today's date and the last login date from the database.
- If the user logged in yesterday: `new_streak = streak + 1` (streak continues).
- If not: `new_streak = 1` (streak resets).
- Awards bonus points at 7-day and 30-day streak milestones.

### `_check_and_award_badges(email)`
- Checks each badge's condition against the user's current state.
- Uses `action_counts` (a dict of how many times each action was done) for action-based badges.
- If new badges were earned, writes the updated list to the database as JSON.

### `get_user_points_data(email)` / `get_leaderboard(limit=20)` / `get_points_history(email, limit=20)`
- Read functions for displaying points info, the top-20 leaderboard, and a user's points history.

## Output
```
Writing db.py
```

---

---

# 🔷 Cell 4 — Write `config.py` (Configuration)

**Cell Type:** Code

## Summary
Writes a simple configuration file that holds all the app's global settings, reading secret values from environment variables (so they're never hard-coded in the source code).

---

## Line-by-Line Explanation

```python
import os
```
- Imports `os` to access environment variables.

---

```python
JWT_SECRET = os.getenv("JWT_SECRET_KEY")
```
- **`os.getenv()`** — Reads an environment variable. If it doesn't exist, returns `None`.
- **`JWT_SECRET`** — The secret key used to sign and verify JWT tokens. If this is stolen, someone could forge login tokens.

---

```python
JWT_ALGORITHM = "HS256"
```
- The hashing algorithm used for JWT tokens. "HS256" = HMAC with SHA-256.

---

```python
TOKEN_EXPIRY_MINUTES = 60
```
- JWT tokens expire after 60 minutes. After that, the user is automatically logged out.

---

```python
MAX_LOGIN_ATTEMPTS = 3
LOCK_TIME_MINUTES = 5
PASSWORD_HISTORY_COUNT = 3
```
- **`MAX_LOGIN_ATTEMPTS`** — After 3 wrong passwords, the account is locked.
- **`LOCK_TIME_MINUTES`** — The account stays locked for 5 minutes.
- **`PASSWORD_HISTORY_COUNT`** — The app remembers the last 3 passwords and won't let users reuse them.

---

```python
EMAIL_ID = os.getenv("EMAIL_ID")
EMAIL_APP_PASSWORD = os.getenv("EMAIL_APP_PASSWORD")
```
- The Gmail address and app password used by the system to send OTP (One-Time Password) emails.

---

```python
ADMIN_EMAIL = os.getenv("ADMIN_EMAIL_ID")
ADMIN_PASSWORD = os.getenv("ADMIN_PASSWORD")
```
- Credentials for the built-in admin account. Loaded from Colab Secrets.

## Output
```
Writing config.py
```

---

---

# 🔷 Cell 5 — Write `auth.py` (Authentication Utilities)

**Cell Type:** Code

## Summary
This file contains all the helper functions for security: hashing passwords, validating emails, generating and verifying JWT tokens, sending OTP emails, and creating admin notification emails.

---

## Imports

```python
import re
import bcrypt
import jwt
import random
import smtplib
from datetime import datetime, timedelta
from email.mime.text import MIMEText
from config import *
```

- **`re`** — Regular expressions (patterns for matching/validating text, like email formats).
- **`bcrypt`** — Password hashing.
- **`jwt`** — JSON Web Token encoding/decoding.
- **`random`** — For generating random OTP numbers.
- **`smtplib`** — Python's built-in library for sending emails via SMTP.
- **`datetime, timedelta`** — Date/time types. `timedelta` represents a duration (e.g., 60 minutes).
- **`MIMEText`** — Creates email message objects with text/HTML content.
- **`from config import *`** — Imports all variables from `config.py` (JWT_SECRET, EMAIL_ID, etc.).

---

## Function: `hash_text(text)`

```python
def hash_text(text: str) -> str:
    return bcrypt.hashpw(text.encode(), bcrypt.gensalt()).decode()
```

- **Purpose:** Converts a plain-text password (or security answer) into a secure hash.
- `text.encode()` — Converts the string to bytes (bcrypt works with bytes).
- `bcrypt.gensalt()` — Generates a random "salt" (extra random data added before hashing to make each hash unique even for the same password).
- `bcrypt.hashpw()` — Applies bcrypt hashing.
- `.decode()` — Converts the resulting bytes back to a string for storage.
- **Example:** `hash_text("MyPass123")` → `"$2b$12$..."` (a long scrambled string)

---

## Function: `verify_text(text, hashed)`

```python
def verify_text(text: str, hashed: str) -> bool:
    return bcrypt.checkpw(text.encode(), hashed.encode())
```

- **Purpose:** Checks if a plain-text password matches its stored hash.
- **Returns:** `True` if they match, `False` otherwise.
- bcrypt handles salts automatically — you don't need to extract or store the salt separately.

---

## Function: `validate_email(email)`

```python
def validate_email(email: str) -> bool:
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None
```

- **Purpose:** Checks if an email address is formatted correctly.
- `r'^[a-zA-Z0-9._%+-]+@...'` — A regex pattern. Let's break it down:
  - `^` — Start of the string.
  - `[a-zA-Z0-9._%+-]+` — One or more letters, numbers, dots, underscores, etc. (the username part).
  - `@` — Literal @ symbol.
  - `[a-zA-Z0-9.-]+` — Domain name.
  - `\.` — Literal dot.
  - `[a-zA-Z]{2,}$` — Top-level domain (like "com", "org") — at least 2 letters, at the end.
- **Returns:** `True` if valid, `False` if not.

---

## Function: `validate_password(password)`

```python
def validate_password(password: str):
    if len(password) < 8:
        return False, "Password must be at least 8 characters"
    if not re.search(r"[A-Z]", password):
        return False, "Password must contain at least 1 uppercase letter"
    if not re.search(r"[a-z]", password):
        return False, "Password must contain at least 1 lowercase letter"
    if not re.search(r"\d", password):
        return False, "Password must contain at least 1 number"
    return True, "Valid password"
```

- **Purpose:** Validates that a password meets complexity requirements.
- `re.search(r"[A-Z]", password)` — Searches for at least one uppercase letter anywhere in the string.
- **Returns:** A tuple `(bool, message)` — e.g., `(False, "Too short")` or `(True, "Valid password")`.

---

## Function: `generate_token(email)`

```python
def generate_token(email: str):
    payload = {
        "email": email,
        "exp": datetime.utcnow() + timedelta(minutes=TOKEN_EXPIRY_MINUTES)
    }
    return jwt.encode(payload, JWT_SECRET, algorithm=JWT_ALGORITHM)
```

- **Purpose:** Creates a JWT token after a user logs in.
- `payload` — A dictionary of data to embed in the token.
  - `"email"` — Identifies which user the token belongs to.
  - `"exp"` — Expiry timestamp (current time + 60 minutes).
- `jwt.encode()` — Signs the payload with the secret key and returns a token string.
- **The token is like a tamper-proof ID card** — anyone can read it, but only the server can verify it hasn't been faked.

---

## Function: `verify_token(token)`

```python
def verify_token(token: str):
    try:
        return jwt.decode(token, JWT_SECRET, algorithms=[JWT_ALGORITHM])
    except:
        return None
```

- **Purpose:** Decodes and validates a JWT token.
- `jwt.decode()` — Verifies the signature and checks expiry. Returns the payload dict if valid.
- If the token is expired, tampered with, or invalid, an exception is raised — caught by `except`, which returns `None`.

---

## Function: `generate_otp()`

```python
def generate_otp():
    return str(random.randint(100000, 999999))
```

- **Purpose:** Generates a random 6-digit OTP code.
- `random.randint(100000, 999999)` — Random integer between 100000 and 999999 (always 6 digits).

---

## Function: `send_html_email(receiver, subject, html_content)`

```python
def send_html_email(receiver, subject, html_content):
    msg = MIMEText(html_content, "html")
    msg["Subject"] = subject
    msg["From"] = EMAIL_ID
    msg["To"] = receiver
    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
        server.login(EMAIL_ID, EMAIL_APP_PASSWORD)
        server.send_message(msg)
```

- **Purpose:** Sends an HTML-formatted email.
- `MIMEText(html_content, "html")` — Creates an email message with HTML content (so it can have styling and colors).
- `smtplib.SMTP_SSL("smtp.gmail.com", 465)` — Opens an encrypted (SSL) connection to Gmail's email server on port 465.
- `server.login()` — Authenticates with Gmail using the app's email and password.
- `server.send_message(msg)` — Sends the email.

---

## Function: `send_otp_email(receiver, otp)`
- **Purpose:** Creates a branded HTML email template with the OTP code and sends it using `send_html_email`.
- The email has a dark-themed HTML layout with the OTP displayed prominently in green.

---

## Function: `send_admin_action_email(receiver, action_text)`
- **Purpose:** Sends a notification email to a user when an admin takes action on their account (e.g., deletion, approval).

## Output
```
Writing auth.py
```

---

---

# 🔷 Cell 6 — Write `readability.py`

**Cell Type:** Code

## Summary
Writes a class that analyzes how readable a piece of text is, using multiple standard readability formulas.

---

## Imports

```python
import textstat
```
- **`textstat`** — A library that computes many readability scores. It counts syllables, words, sentences, and applies formulas.

---

## Class: `ReadabilityAnalyzer`

```python
class ReadabilityAnalyzer:
    def __init__(self, text):
        self.text = text
```

- **`class`** — Defines a blueprint for creating objects.
- **`__init__`** — The constructor method, called when you create a new object. It receives the text to analyze.
- **`self.text = text`** — Saves the text as an instance variable so other methods can use it.
- **Example usage:** `analyzer = ReadabilityAnalyzer("This is my policy document text...")`

---

## Method: `get_all_metrics()`

**Purpose:** Calculates and returns all readability scores for the stored text.

```python
"Flesch Reading Ease": textstat.flesch_reading_ease(self.text),
```
- **Flesch Reading Ease** — Score from 0–100. Higher = easier to read. 70+ is easy (plain English), below 30 is very hard (legal/technical documents). Formula uses sentence length and syllable count.

```python
"Flesch-Kincaid Grade": textstat.flesch_kincaid_grade(self.text),
```
- **Flesch-Kincaid Grade** — U.S. school grade level needed to understand the text. Grade 8 = 8th grader can read it.

```python
"SMOG Index": textstat.smog_index(self.text),
```
- **SMOG Index** (Simple Measure of Gobbledygook) — Another grade level formula, especially good for health documents.

```python
"Gunning Fog": textstat.gunning_fog(self.text),
```
- **Gunning Fog Index** — Grade level based on complex word percentage.

```python
"Coleman-Liau": textstat.coleman_liau_index(self.text)
```
- **Coleman-Liau Index** — Grade level based on characters per word and sentences per 100 words.

```python
metrics["Overall Grade Level"] = (
    metrics["Flesch-Kincaid Grade"]
    + metrics["SMOG Index"]
    + metrics["Gunning Fog"]
    + metrics["Coleman-Liau"]
) / 4
```
- Calculates the **average** of the four grade-level scores for an overall estimate.

```python
metrics["Total Sentences"] = textstat.sentence_count(self.text)
metrics["Total Words"] = textstat.lexicon_count(self.text)
metrics["Total Syllables"] = textstat.syllable_count(self.text)
metrics["Complex Words"] = textstat.difficult_words(self.text)
metrics["Total Characters"] = len(self.text)
```
- Basic text statistics.
- **`textstat.difficult_words()`** — Counts words not found in a list of common/easy words.
- **`len(self.text)`** — Total character count.
- **Returns:** A dictionary of all metrics.

## Output
```
Writing readability.py
```

---

---

# 🔷 Cell 7 — Write `vector_store.py` (AI Document Search)

**Cell Type:** Code

## Summary
This is the heart of the RAG (Retrieval-Augmented Generation) system. It handles converting documents to embeddings, storing them in a FAISS index, and finding relevant chunks when a user asks a question.

**Concept explanation:** Think of this like a very smart library system. Every book (document chunk) is described by a unique numerical fingerprint (embedding). When you ask a question, your question also gets a fingerprint, and the system finds the books whose fingerprints are most similar.

---

## Imports

```python
import os, glob, pickle, faiss, PyPDF2
from bs4 import BeautifulSoup
from sentence_transformers import SentenceTransformer
```

- **`glob`** — Finds all files matching a pattern in a directory (like `*.pdf`).
- **`pickle`** — Serializes Python objects to a file and reads them back. Used to save/load the document metadata.
- **`faiss`** — Facebook AI Similarity Search — the fast vector search engine.
- **`PyPDF2`** — Reads PDF files.
- **`BeautifulSoup`** — Parses HTML files to extract plain text.
- **`SentenceTransformer`** — The embedding model that converts text to vectors.

---

## Global Variables

```python
APP_DIR = os.environ.get('APP_DIR', '.')
INDEX_PATH = os.path.join(APP_DIR, "faiss_index.bin")
META_PATH = os.path.join(APP_DIR, "faiss_meta.pkl")
_embedder = None
```

- **`INDEX_PATH`** — Path to save/load the FAISS vector index (the binary file of embeddings).
- **`META_PATH`** — Path to save/load the metadata (which chunk came from which file).
- **`_embedder = None`** — The embedding model starts as `None` and is loaded lazily (only when first needed).

---

## Function: `get_embedder()`

```python
def get_embedder():
    global _embedder
    if _embedder is None:
        _embedder = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')
    return _embedder
```

- **Purpose:** Returns the embedding model, loading it only once (lazy initialization).
- **`global _embedder`** — Declares that we're modifying the module-level variable, not creating a local one.
- **`'paraphrase-multilingual-MiniLM-L12-v2'`** — A multilingual sentence embedding model. It converts sentences into 384-dimensional vectors. "Multilingual" means it understands many languages.
- The model is cached in `_embedder` after first load to avoid reloading it on every function call.

---

## Function: `extract_text_from_pdf(pdf_path)`

```python
def extract_text_from_pdf(pdf_path):
    text = ""
    try:
        for page in PyPDF2.PdfReader(pdf_path).pages:
            text += (page.extract_text() or "") + "\n"
    except:
        pass
    return text
```

- **Purpose:** Reads all text from a PDF file.
- `PyPDF2.PdfReader(pdf_path)` — Opens the PDF.
- `.pages` — Returns a list of page objects.
- `page.extract_text()` — Extracts the text from one page. Returns `None` if the page has no extractable text (e.g., it's a scanned image).
- `or ""` — Uses empty string if `extract_text()` returns `None`.
- `try/except/pass` — If any error occurs (corrupted PDF), the function silently returns whatever text was collected.

---

## Function: `ingest_documents(docs_dir)`

**Purpose:** Reads all documents in a folder, converts them to chunks, embeds them, and adds them to the FAISS index. Only processes **new** files (files not already in the index).

```python
existing_metadata = []
if os.path.exists(META_PATH):
    try:
        with open(META_PATH, 'rb') as f:
            existing_metadata = pickle.load(f)
    except:
        pass
existing_filenames = set([d['filename'] for d in existing_metadata])
```
- Loads already-processed document metadata. Creates a set of filenames already in the index so they won't be reprocessed.

```python
files = glob.glob(os.path.join(docs_dir, "*"))
```
- Gets a list of all files in the documents directory.

```python
for filepath in files:
    filename = os.path.basename(filepath)
    if filename in existing_filenames:
        continue
```
- **`os.path.basename()`** — Extracts just the filename from the full path (e.g., `"policy.pdf"` from `"/content/.../policy.pdf"`).
- `continue` — Skips to the next file (this one is already indexed).

```python
    if filepath.lower().endswith(".pdf"):
        text = extract_text_from_pdf(filepath)
    elif filepath.lower().endswith((".htm", ".html")):
        with open(filepath, ...) as f:
            text = BeautifulSoup(f.read(), 'html.parser').get_text(separator=' ')
    elif filepath.lower().endswith(".txt"):
        with open(filepath, ...) as f:
            text = f.read()
```
- Chooses the right parsing method based on file extension.
- `BeautifulSoup(...).get_text(separator=' ')` — Strips all HTML tags and returns just the readable text.

```python
    for i in range(0, len(text), 1500):
        chunk = text[i:i+1500]
        if len(chunk) > 50:
            new_chunks.append(chunk)
            new_metadata.append({"filename": filename, "content": chunk})
```
- **Chunking:** Splits the full document text into 1500-character chunks. This is because AI models and FAISS work better with smaller pieces.
- `range(0, len(text), 1500)` — Generates indices 0, 1500, 3000, ... up to the end of the text.
- Skips chunks shorter than 50 characters (likely empty/garbage).
- Each chunk is stored alongside its source filename in `new_metadata`.

```python
embedder = get_embedder()
embeddings = embedder.encode(new_chunks, convert_to_numpy=True)
```
- **`embedder.encode()`** — Converts all new text chunks into numerical vectors. `convert_to_numpy=True` returns a NumPy array (needed for FAISS).

```python
if os.path.exists(INDEX_PATH):
    index = faiss.read_index(INDEX_PATH)
else:
    index = faiss.IndexFlatL2(embeddings.shape[1])
```
- Loads the existing index (if it exists) or creates a new one.
- **`faiss.IndexFlatL2`** — An exact L2 (Euclidean distance) search index. `embeddings.shape[1]` is the dimension of the embedding vectors (384).

```python
index.add(embeddings)
faiss.write_index(index, INDEX_PATH)
```
- Adds the new vectors to the index and saves it to disk.

---

## Function: `search_documents(query, top_k=5)`

**Purpose:** Finds the most relevant document chunks for a given question.

- `embedder.encode([query], convert_to_numpy=True)` — Converts the question into a vector.
- `index.search(..., top_k)` — Finds the `top_k` (5) closest vectors in the index. Returns `distances` (how similar) and `indices` (which chunks matched).
- The function limits results to at most 2 chunks per file (`file_count[fname] <= 2`) to ensure diversity across documents.
- **Returns:** A list of matching metadata dicts, each with `filename` and `content`.

---

## Function: `get_all_documents()`
- Loads and returns all metadata from the pickle file — used for building the knowledge graph.

## Output
```
Writing vector_store.py
```

---

---

# 🔷 Cell 8 — Write `knowledge_graph.py`

**Cell Type:** Code

## Summary
Writes the knowledge graph builder. It reads document chunks, extracts named entities (people, organizations, laws, places), and creates an interactive visual graph showing how they're connected.

---

## Imports

```python
import spacy, networkx as nx, os
from pyvis.network import Network
```

- **`spacy`** — NLP library used for Named Entity Recognition.
- **`networkx as nx`** — The `nx` alias is convention. Used to build the graph structure (nodes and edges).
- **`Network`** — From pyvis; renders the networkx graph as an interactive HTML file.

---

```python
try:
    nlp = spacy.load("en_core_web_sm")
except:
    nlp = None
```
- Tries to load the spaCy English model. If it fails (not installed), sets `nlp = None` so the rest of the code can check for it.

---

## Function: `build_graph_from_documents(docs)`

**Purpose:** Builds a networkx graph from document chunks.

```python
G = nx.Graph()
```
- Creates an empty undirected graph. Think of it as a blank whiteboard.

```python
max_docs = min(len(docs), 50)
```
- Processes at most 50 document chunks to avoid making the graph too cluttered.

```python
doc = nlp(text)
entities = [
    (ent.text.strip(), ent.label_) for ent in doc.ents
    if ent.label_ in ['ORG', 'GPE', 'LAW', 'PERSON'] and len(ent.text.strip()) > 2
]
```
- `nlp(text)` — Runs spaCy's NLP pipeline on the text.
- `doc.ents` — All named entities found in the text.
- **Filters:** Only keeps entities of type:
  - `ORG` — Organizations (e.g., "Ministry of Finance").
  - `GPE` — Geopolitical entities (e.g., "India").
  - `LAW` — Laws and regulations (e.g., "Right to Education Act").
  - `PERSON` — People's names.
- Discards entities shorter than 2 characters (noise).

```python
source_node = f"Doc: {docs[i]['filename']}"
G.add_node(source_node, label=source_node, title=..., color="#38bdf8", size=18)
```
- Adds the document file itself as a blue node in the graph.

```python
for ent, label in entities:
    if label == "ORG":   color = "#22c55e"
    elif label == "PERSON": color = "#f472b6"
    ...
    G.add_node(ent, label=ent, title=..., color=color, size=20)
    G.add_edge(source_node, ent)
```
- Adds each entity as a colored node (green=org, pink=person, amber=law, yellow=place).
- Draws an edge (line) from the document node to each entity it contains.

```python
for a in range(min(len(entity_names), 5)):
    for b in range(a + 1, min(len(entity_names), 5)):
        G.add_edge(entity_names[a], entity_names[b])
```
- Connects co-occurring entities to each other (entities appearing in the same document chunk are likely related).
- Only connects the first 5 entities per chunk to avoid clutter.

---

## Function: `generate_interactive_graph(docs)`

**Purpose:** Converts the networkx graph into an interactive, zoomable HTML visualization using pyvis.

```python
net = Network(height="650px", width="100%", bgcolor="#0b0f19", font_color="white", notebook=False)
```
- Creates a pyvis Network object with a dark background theme.

```python
net.barnes_hut(gravity=-8000, central_gravity=0.3, spring_length=200)
```
- **Barnes-Hut** is a physics simulation algorithm that determines how nodes repel/attract each other so the graph looks neat.
- `gravity=-8000` — Strong repulsion between nodes (they spread out).
- `spring_length=200` — Edges try to be 200 pixels long.

```python
net.from_nx(G)
```
- Converts the networkx graph into a pyvis graph.

```python
net.set_options("var options = {...}")
```
- Sets visual options like font size, edge color, hover behavior, zoom, drag, and physics settings (as a JSON string).

```python
output_path = os.path.join(APP_DIR, "graphs", "policy_kg.html")
net.save_graph(output_path)
return output_path
```
- Saves the interactive graph as an HTML file. This file is later displayed in the Streamlit app using an iframe.

## Output
```
Writing knowledge_graph.py
```

---

---

# 🔷 Cell 9 — Write `nlp_engine.py` (AI & Translation Engine)

**Cell Type:** Code

## Summary
This file manages the two AI models at the core of PolicyNav: a language model (Qwen2.5) for answering questions and summarizing text, and a translation model (NLLB-200) for supporting 7 Indian languages.

---

## Imports

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, AutoModelForSeq2SeqLM
import vector_store
```

- **`torch`** — PyTorch, the deep learning framework.
- **`AutoModelForCausalLM`** — Loads auto-regressive (text generation) models. Used for Qwen2.5.
- **`AutoTokenizer`** — Loads the tokenizer for a model (converts text to token IDs).
- **`AutoModelForSeq2SeqLM`** — Loads sequence-to-sequence models. Used for NLLB translation.
- **`vector_store`** — The document search module from Cell 7.

---

## Model Configuration

```python
MODEL_ID = "Qwen/Qwen2.5-1.5B-Instruct"
TRANSLATOR_ID = "facebook/nllb-200-distilled-600M"
```

- **`Qwen/Qwen2.5-1.5B-Instruct`** — Qwen 2.5 is a small but capable 1.5 billion parameter instruction-tuned language model by Alibaba. It answers questions in a chat format.
- **`facebook/nllb-200-distilled-600M`** — Facebook's No Language Left Behind model, a 600M parameter multilingual translation model that supports 200 languages (distilled = compressed for speed).

```python
model = None
tokenizer = None
translator_model = None
translator_tokenizer = None
```
- All models start as `None` and are loaded lazily (only on first use). This way, the login page appears instantly without waiting for model loading.

---

## Function: `init_model()`

**Purpose:** Loads both AI models into memory (GPU/CPU).

```python
tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)
model = AutoModelForCausalLM.from_pretrained(
    MODEL_ID,
    device_map="auto",
    torch_dtype=torch.float16
)
```
- **`from_pretrained(MODEL_ID)`** — Downloads and loads the model from Hugging Face Hub.
- **`device_map="auto"`** — Automatically places the model on GPU (if available) or CPU.
- **`torch_dtype=torch.float16`** — Loads the model in 16-bit floating point (half precision). Uses half the GPU memory of normal 32-bit, with minimal accuracy loss.

---

## `LANG_CODES` dictionary

```python
LANG_CODES = {
    "English": "eng_Latn",
    "Hindi": "hin_Deva",
    "Tamil": "tam_Taml",
    ...
}
```
- Maps human-readable language names to NLLB's internal language codes.
- Format: `language_script` (e.g., `hin_Deva` = Hindi in Devanagari script).

---

## Function: `translate_fast(text, source_lang, target_lang)`

**Purpose:** Translates text from one language to another.

```python
if source_lang == target_lang:
    return text
```
- If no translation needed, returns text immediately.

```python
translator_tokenizer.src_lang = src_code
inputs = translator_tokenizer(text, return_tensors="pt", max_length=512, truncation=True).to(translator_model.device)
```
- Sets the source language.
- Tokenizes the input text into PyTorch tensors (`"pt"`), limits to 512 tokens.
- `.to(translator_model.device)` — Moves the tensors to the same device as the model (GPU/CPU).

```python
tgt_token_id = translator_tokenizer.convert_tokens_to_ids(tgt_code)
outputs = translator_model.generate(**inputs, forced_bos_token_id=tgt_token_id, max_length=512, num_beams=2)
```
- **`forced_bos_token_id`** — Forces the model to start generating in the target language.
- **`num_beams=2`** — Beam search with 2 beams (explores 2 possible translation paths, picks the better one).

```python
return translator_tokenizer.decode(outputs[0], skip_special_tokens=True)
```
- Converts token IDs back to a readable string.

---

## Function: `generate_english_response(prompt_text)`

**Purpose:** Sends a prompt to the Qwen model and gets a response.

```python
messages = [
    {"role": "system", "content": "You are the Public Policy Compass AI..."},
    {"role": "user", "content": prompt_text}
]
text = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
```
- Formats the messages as a chat (system + user roles) using the model's expected template.

```python
with torch.no_grad():
    outputs = model.generate(
        inputs.input_ids,
        max_new_tokens=200,
        temperature=0.2,
        do_sample=True,
        pad_token_id=tokenizer.eos_token_id
    )
```
- **`torch.no_grad()`** — Disables gradient calculation (not needed during inference, saves memory).
- **`max_new_tokens=200`** — Generate up to 200 new tokens (words/sub-words).
- **`temperature=0.2`** — Low temperature = more deterministic/factual responses (less creative).
- **`do_sample=True`** — Uses sampling (not greedy decoding).

```python
return tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
```
- Decodes only the **newly generated** tokens (slices off the prompt part).

---

## Function: `answer_policy_question(native_question, target_lang, simplify)`

**Purpose:** Full RAG pipeline — takes a question in any supported language and returns an answer.

1. Translates the question to English.
2. Searches relevant document chunks using `vector_store.search_documents()`.
3. Builds a prompt with context + question.
4. If `simplify=True`, adds "Simplify the language for a middle-school student."
5. Generates an English answer.
6. Translates the answer back to the user's language.
- **Returns:** `(final_answer, source_docs)`.

---

## Function: `summarize_document(text, target_lang)`

**Purpose:** Summarizes a document into 3 bullet points in the user's language.

```python
eng_sum = generate_english_response(f"Summarize this policy into 3 highly concise bullet points:\n\n{text[:3000]}")
return translate_fast(eng_sum, "English", target_lang)
```
- Takes the first 3000 characters of the document (to fit in the model's context window).
- Generates a 3-bullet summary in English, then translates it.

## Output
```
Writing nlp_engine.py
```

---

---

# 🔷 Cell 10 — Write `app.py` (Main Streamlit Application)

**Cell Type:** Code

## Summary
This is the largest cell — it writes the complete Streamlit web application. It contains all the UI pages, routing logic, user authentication flows, and integration with all the other modules. Think of this as the "main control room" of PolicyNav.

---

## Imports (app.py)

```python
import io, matplotlib.pyplot as plt
from wordcloud import WordCloud
import plotly.graph_objects as go
import os, time, uuid, PyPDF2, smtplib, secrets, hmac, hashlib, struct
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from readability import ReadabilityAnalyzer
import streamlit as st
import json
from datetime import datetime, timedelta
from db import *
from auth import *
from config import *
import base64
import vector_store, nlp_engine, knowledge_graph
import pandas as pd
import plotly.express as px
from streamlit_option_menu import option_menu
import streamlit.components.v1 as components
from streamlit_autorefresh import st_autorefresh
import re
```

Key highlights:
- **`io`** — In-memory file-like objects (for passing images without saving to disk).
- **`uuid`** — Generates unique IDs (for avatar filenames).
- **`secrets`, `hmac`, `hashlib`, `struct`** — Used to implement a secure HOTP/TOTP-style OTP generator.
- **`base64`** — Encodes binary files (like images) as text strings for embedding in HTML.
- **`from db import *`, `from auth import *`, `from config import *`** — Imports all functions from the custom modules.
- **`plotly.express as px`** — High-level Plotly interface for common chart types.
- **`option_menu`** — Sidebar navigation menu component.
- **`components`** — Allows embedding raw HTML/JavaScript into Streamlit (used to show the knowledge graph iframe).

---

## Page Configuration

```python
st.set_page_config(
    page_title="PolicyNav",
    page_icon="⚡",
    layout="wide",
    initial_sidebar_state="expanded"
)
```
- Sets browser tab title, favicon, wide layout (uses full screen width), and opens the sidebar by default.

---

## `apply_theme()` Function

**Purpose:** Injects a comprehensive block of custom CSS into the Streamlit app to create PolicyNav's dark, modern design system.

Key CSS elements:
- **Root background** — `#0a0e1a` (very dark navy blue).
- **Fonts** — Google Fonts: `Poppins` for headings, `Inter` for body text.
- **Inputs** — Dark styled text fields with purple focus glow (`#6366f1` = indigo).
- **Buttons** — Purple gradient with hover lift effect.
- **Sidebar** — Even darker than main content.
- **Tabs** — Dark pill-style with active tab in purple.
- **Password strength** — `pw-weak` (red), `pw-medium` (yellow), `pw-strong` (green).
- **`pn-card`** — A reusable dark card container with subtle border.
- **History table** — Custom table styling for activity logs.
- **Chat messages** — Dark rounded chat bubbles.

```python
apply_theme()
```
- Calls the function immediately so styles are applied when the app starts.

---

## OTP System (Secure HOTP-Style)

```python
OTP_EXPIRY = 600  # 10 minutes
```

### `generate_otp_code()`
```python
def generate_otp_code():
    secret = secrets.token_bytes(20)
    msg    = struct.pack(">Q", int(time.time()))
    h      = hmac.new(secret, msg, hashlib.sha1).digest()
    offset = h[19] & 0xf
    code   = ((h[offset]&0x7f)<<24|(h[offset+1]&0xff)<<16|(h[offset+2]&0xff)<<8|(h[offset+3]&0xff))
    return f"{code%1000000:06d}"
```
- Implements a **HOTP (HMAC-based One-Time Password)** algorithm:
  - Generates a random 20-byte secret.
  - Packs the current Unix timestamp as 8 bytes (big-endian).
  - Creates an HMAC-SHA1 hash.
  - Extracts a dynamic offset from the last nibble of the hash.
  - Reads 4 bytes at the offset and extracts 6 digits.
- The result is a 6-digit OTP (zero-padded if needed with `f"{...:06d}"`).
- This is much more secure than a simple `random.randint()`.

### `send_otp_styled(to_email)`
- **Purpose:** Sends a branded dark-themed HTML email containing the OTP.
- Uses `smtplib.SMTP("smtp.gmail.com", 587)` with `starttls()` (TLS encryption on port 587).
- Stores the OTP, the recipient email, and the send time in `st.session_state` for later verification.
- **Returns:** `(True, "Sent")` on success or `(False, error_message)` on failure.

### `otp_is_valid(entered, for_email)`
- Checks three things: (1) OTP is not expired, (2) the email matches, (3) the code is correct.
- **Returns:** `(True, "Valid")` or `(False, error_message)`.

### `clear_otp()`
- Removes OTP data from `st.session_state` after successful verification.

---

## Password Strength

```python
def check_password_strength(pw):
    if re.search(r"\s", pw):  return "Weak", ["No spaces allowed"]
    ok = (re.search(r"[A-Za-z]", pw) and re.search(r"\d", pw))
    if len(pw) >= 8 and ok:   return "Strong", []
    if len(pw) >= 6 and ok:   return "Medium", ["Add 2+ chars for Strong"]
    return "Weak", ["Too short (aim for 8+)"]
```
- Returns a strength label ("Weak"/"Medium"/"Strong") and a list of feedback messages.

```python
def pw_strength_html(pw):
    s, f = check_password_strength(pw)
    cls  = {"Weak":"pw-weak","Medium":"pw-medium","Strong":"pw-strong"}[s]
    icon = {"Weak":"✗","Medium":"◑","Strong":"✓"}[s]
    hint = f" — {', '.join(f)}" if f else ""
    return f"<span class='{cls}'>{icon} {s}{hint}</span>"
```
- Returns an HTML string with a colored strength indicator using the CSS classes defined in `apply_theme()`.

---

## UI Helper Functions

### `auth_header(title, subtitle)`
- Renders a centered header with the PolicyNav logo (⚡), title, subtitle, and page name.

### `section_header(icon, title, subtitle)`
- Renders a content-area header with an icon, large title, and optional subtitle.

### `card_start()` / `card_end()`
- Wraps content in a dark card `<div>` (adds `<div class="pn-card">` and `</div>`).

### `divider_text(text)`
- Renders a horizontal line with centered text (used as a section divider like `── or ──`).

### `kpi_card(col, icon, label, value, accent)`
- Renders a styled KPI (Key Performance Indicator) metric card inside a Streamlit column with an accent color border.

### `create_gauge(value, title, min_val, max_val, color)`
- Creates a Plotly gauge chart (like a speedometer) for displaying readability scores visually.

---

## Database & Model Initialization

```python
init_db()
```
- Calls the `init_db()` function from `db.py` to create all tables when the app first starts.

```python
@st.cache_resource
def load_and_cache_models():
    nlp_engine.init_model()
    vector_store.get_embedder()
    return True
```
- **`@st.cache_resource`** — A Streamlit decorator that caches the result of this function. The function only runs **once** per server session, and the result (loaded models) is shared across all user sessions.
- Models are loaded lazily (this function is called only when the user first uses a feature that needs them).

---

## Session State Initialization

```python
for k, v in [
    ("token", None), ("page", "Login"), ("reset_stage", 0),
    ("rag_chat", []), ("summary", ""), ("doc_text", ""), ...
]:
    if k not in st.session_state:
        st.session_state[k] = v
```
- **`st.session_state`** — Streamlit's server-side storage for each user's data. It persists between page reruns within the same session.
- This loop sets default values for all session keys only if they haven't been set yet (avoids resetting on every page reload).
- **Key state variables:**
  - `"token"` — The user's JWT login token.
  - `"page"` — Current page name (used for routing).
  - `"rag_chat"` — Chat history list for the RAG Q&A feature.
  - `"otp_code"`, `"otp_email"`, `"otp_time"` — OTP verification data.
  - `"signup_data"` — Temporarily holds registration form data during OTP verification.

---

## `go_to(page)` Function

```python
def go_to(page):
    st.session_state.page = page
    st.rerun()
```
- **Purpose:** Client-side navigation. Changes the current page and forces Streamlit to rerun the script (which re-renders the appropriate page).

---

## Avatar Helper Functions

### `get_avatar_path(user)`
- Looks at column index 9 of the user tuple (the `avatar_path` column) and checks if the file exists.

### `save_avatar(uploaded_file, email)`
- Validates the file extension (only PNG, JPG, JPEG).
- Creates a safe filename by replacing `@` with `_at_` and `.` with `_` in the email, then appending a random UUID hex string.
- Saves the file to the `avatars/` directory.

---

## `render_feedback_ui(section_name, generated_text, unique_key)` Function

**Purpose:** Renders a collapsible feedback widget at the bottom of each AI feature section.

- Verifies the user is logged in.
- Shows a 1–5 star rating (radio buttons) and an optional text comment.
- On submit, calls `submit_feedback()` to save to DB and `award_points()` to give +3 points.
- `unique_key` — Each Streamlit widget must have a unique key; this parameter ensures the feedback widgets on different pages don't conflict.

---

## `render_sidebar()` Function

**Purpose:** Renders the left sidebar with the app logo, user info, navigation menu, and logout button.

- If logged in: shows username, avatar, online status dot, points, and streak.
- Uses `option_menu()` from `streamlit-option-menu` for the stylish navigation menu.
- Menu items: Dashboard, Upload & Search, Summarization, Readability, Knowledge Graph, My Activity, Feedback, Leaderboard.
- Admin users also see an "Admin" menu item.
- Logout button: calls `set_user_online(email, False)`, clears the token, and returns to the Login page.

---

## Page Functions

### `login_page()`
- Two-column layout (left=branding, right=form).
- Handles admin login (direct, no OTP), regular login (with OTP), account lockout logic.
- On successful password verification, sends OTP via email.

### `otp_page()`
- Verifies the 6-digit code entered by the user.
- Handles both login OTP flow and forgot-password OTP flow.
- On success: generates JWT token, updates login streak, navigates to Dashboard.

### `signup_page()`
- Two-stage: (1) Fill the registration form and validate, (2) Verify OTP.
- Handles the case where the email was previously deleted — sends request to admin instead.
- Password strength indicator shown in real time.
- On successful OTP: calls `create_user()` and logs in the user.

### `forgot_password_page()` / `set_new_password_page()`
- Multi-step password reset flow:
  1. Enter email.
  2. Verify OTP sent to email.
  3. Set a new password.
  4. Checks that new password isn't in recent password history.

---

## Main App Pages (after login)

### Dashboard Page
- Shows a welcome message, KPI cards (total users, documents, activity logs, feedback entries).
- Displays a word cloud of frequently asked questions.
- Shows the user's gamification profile: points, streak, badges, points history chart.
- Admin dashboard shows: user management table, pending registrations, activity logs, feedback chart.

### Upload & RAG Search Page
- File uploader for PDF/HTML/TXT.
- On upload: calls `vector_store.ingest_documents()` to index new files.
- Chat-style Q&A interface: user types a question, AI finds relevant chunks and answers.
- Language selector (English, Hindi, Tamil, etc.).
- "Simplify language" toggle.
- Deadline Tracker: AI extracts policy deadlines from uploaded documents.

### Summarization Page
- User selects an already-uploaded file.
- Calls `nlp_engine.summarize_document()` for a 3-bullet summary.
- Shows a word cloud of the document text.
- Allows downloading the summary as a text file.

### Readability Page
- User pastes text or uploads a file.
- Calls `ReadabilityAnalyzer.get_all_metrics()`.
- Displays gauge charts for Flesch Reading Ease and Overall Grade Level.
- Shows all metric values in a bar chart.
- Provides an interpretation of the scores.

### Knowledge Graph Page
- Calls `knowledge_graph.generate_interactive_graph()`.
- Embeds the resulting HTML file inside Streamlit using `components.html()`.
- The graph is interactive: users can zoom, drag, and hover over nodes.

### My Activity Page
- Shows a table of the user's past AI queries and responses.
- Hover over AI outputs to see the full response (via CSS hover cards).

### Feedback Page
- Star rating form for each app section.
- Admin can see all feedback in a chart broken down by section and rating.

### Leaderboard Page
- Shows the top 20 users ranked by points.
- Uses medal emojis (🥇🥈🥉) for top 3.

## Output
```
Writing app.py
```

---

---

# 🔷 Cell 11 — Seed Sample Feedback Data

**Cell Type:** Code

## Summary
This maintenance cell wipes existing feedback from the database and inserts a fresh set of 13 sample feedback records. It's used to populate the admin's feedback dashboard with realistic-looking demo data.

---

## Line-by-Line Explanation

```python
import sqlite3
import os
```
- Direct imports (not using `db.py`) to connect to the database manually.

---

```python
APP_DIR = os.environ.get('APP_DIR', '/content/drive/MyDrive/PolicyNav')
DB_NAME = os.path.join(APP_DIR, "policynav_users.db")
```
- Reads the app directory from the environment (set in Cell 2).
- Constructs the database file path.

---

```python
conn = sqlite3.connect(DB_NAME)
cursor = conn.cursor()
```
- Opens a direct connection to the database.
- `cursor` — An object used to execute SQL commands. Think of it as a "pointer" that can run queries.

---

```python
cursor.execute("DELETE FROM feedback")
```
- **`DELETE FROM feedback`** — Removes ALL rows from the feedback table (a complete wipe).

---

```python
cursor.execute("DELETE FROM sqlite_sequence WHERE name='feedback'")
```
- **`sqlite_sequence`** — An internal SQLite table that tracks auto-increment counters.
- This resets the auto-increment counter so the next inserted feedback starts with `id=1` (not some large number).

---

```python
sample_feedbacks = [
    ("alice@example.com", "RAG Search", 5, "The search is incredibly fast..."),
    ("bob@example.com", "Summarization", 4, "Great summary, but..."),
    ...
]
```
- A list of 13 tuples. Each tuple has: `(user_email, section, rating, comments)`.
- These represent fake reviews from different users across all major app sections (RAG Search, Summarization, Knowledge Graph, Readability).

---

```python
cursor.executemany("""
    INSERT INTO feedback (user_email, section, rating, comments)
    VALUES (?, ?, ?, ?)
""", sample_feedbacks)
```
- **`cursor.executemany()`** — Runs the same SQL INSERT statement multiple times, once for each tuple in `sample_feedbacks`.
- **`?` placeholders** — SQLite parameter substitution. The values from each tuple are safely inserted in place of the `?`s (prevents SQL injection).

---

```python
conn.commit()
conn.close()
```
- **`conn.commit()`** — Saves all the changes to the database file permanently.
- **`conn.close()`** — Closes the database connection to free resources.

---

```python
print(f"✅ Successfully wiped old data and inserted {len(sample_feedbacks)} sample feedback records!")
```
- Prints a success message with the count of inserted records.

## Output
```
✅ Successfully wiped old data and inserted 13 sample feedback records!
```

---

---

# 🔷 Cell 12 — Launch the App via Ngrok

**Cell Type:** Code

## Summary
This is the **final launch cell**. It starts the Streamlit web server and creates a public internet URL using ngrok so anyone can access the app from their browser.

---

## Line-by-Line Explanation

```python
import os, subprocess, time
from google.colab import userdata
from pyngrok import ngrok
```

- **`subprocess`** — Python's module for running external programs. Used to start the `streamlit` command.
- **`userdata`** — Colab's module for securely reading "Secrets" stored in Colab's secret manager.
- **`ngrok`** — The tunneling library that creates public URLs.

---

```python
os.environ['APP_DIR'] = APP_DIR
```
- Re-sets `APP_DIR` as an environment variable in the current process. The `subprocess` that runs Streamlit will inherit this.

---

```python
ngrok_token = userdata.get('NGROK_AUTHTOKEN')
```
- **`userdata.get('NGROK_AUTHTOKEN')`** — Reads the ngrok authentication token from Colab Secrets (the padlock icon in the left sidebar of Colab). This token is required by ngrok to create public tunnels.

---

```python
if ngrok_token:
    ngrok.set_auth_token(ngrok_token)
    ngrok.kill()
```
- Only proceeds if the token was found.
- **`ngrok.set_auth_token()`** — Authenticates with ngrok's service.
- **`ngrok.kill()`** — Kills any existing ngrok processes (to avoid conflicts from previous runs).

---

```python
process = subprocess.Popen(['streamlit', 'run', 'app.py'], env=os.environ.copy())
```
- **`subprocess.Popen()`** — Starts a new process in the background (doesn't wait for it to finish).
- **`['streamlit', 'run', 'app.py']`** — The command: runs `streamlit run app.py`.
- **`env=os.environ.copy()`** — Passes the current environment variables (including `APP_DIR`) to the Streamlit process.

---

```python
time.sleep(6)
```
- **`time.sleep(6)`** — Pauses for 6 seconds to give Streamlit time to start up before creating the tunnel.

---

```python
print(f"\n🔗 Streamlit URL: {ngrok.connect(8501).public_url}\n")
```
- **`ngrok.connect(8501)`** — Creates a public tunnel to port 8501 (where Streamlit runs by default).
- **`.public_url`** — The public HTTPS URL (e.g., `https://unlaving-chin-huggingly.ngrok-free.dev`).
- Anyone with this URL can access the PolicyNav app.

---

```python
print("🛑 To stop Streamlit, stop this cell.")
try:
    input("Press ENTER to stop...\n")
except:
    pass
finally:
    process.terminate()
    ngrok.kill()
    print("Stopped.")
```
- `input()` — Keeps the cell running (waiting for user input). The app stays live as long as this cell is running.
- **`try/except/finally`** — `finally` always runs, even if interrupted:
  - `process.terminate()` — Stops the Streamlit server.
  - `ngrok.kill()` — Closes the ngrok tunnel.

---

```python
else:
    print("❌ Ngrok Token missing from Colab secrets.")
```
- If no ngrok token was found, prints an error message instead of trying to launch.

## Output
```
🔗 Streamlit URL: https://unlaving-chin-huggingly.ngrok-free.dev

🛑 To stop Streamlit, stop this cell.
```

---

---

# 🏗️ Overall Architecture Summary

```
policynav_project.ipynb
│
├── Cell 1  → Install libraries
├── Cell 2  → Mount Google Drive, create folders
│
├── Cell 3  → db.py         ← SQLite database layer (users, logs, points, badges)
├── Cell 4  → config.py     ← App secrets and settings
├── Cell 5  → auth.py       ← Hashing, JWT, email OTP
├── Cell 6  → readability.py ← Text readability scoring
├── Cell 7  → vector_store.py ← FAISS-based RAG document search
├── Cell 8  → knowledge_graph.py ← spaCy NER + networkx + pyvis graph
├── Cell 9  → nlp_engine.py  ← Qwen2.5 (Q&A) + NLLB-200 (translation)
├── Cell 10 → app.py         ← Main Streamlit UI (all pages and logic)
│
├── Cell 11 → Seed sample feedback data
└── Cell 12 → Launch Streamlit + ngrok public URL
```

**Data flow for a user question:**
1. User uploads PDF → `vector_store.ingest_documents()` chunks + embeds it into FAISS.
2. User types a question → `nlp_engine.answer_policy_question()` is called.
3. Question is translated to English (if needed) → `translate_fast()`.
4. Relevant chunks fetched from FAISS → `vector_store.search_documents()`.
5. Chunks + question sent to Qwen2.5 → `generate_english_response()`.
6. Answer translated back to user's language → `translate_fast()`.
7. Answer shown in the chat UI, logged to DB, points awarded.
