# DePaul Slot Booking

A lightweight booking UI and admin panel for scheduling student appointments. The front-end is static (HTML/CSS/JS) and the project uses GitHub repository APIs / Actions to persist bookings to `data/bookings.json` and to send confirmation emails.

## Code structure

- `index.html` — Student-facing booking UI. Renders a calendar, lets students pick a date and time slot, and triggers GitHub Actions to save bookings and send confirmation emails.
- `admin.html` — Admin portal to view, add, edit, delete bookings. It reads/writes `data/bookings.json` via the GitHub API.
- `save_booking.py` — Python helper used by CI/workflows to persist a booking into `data/bookings.json`. Expects environment variables: `BOOKING_DATA` (JSON string), `GH_TOKEN` (GitHub token), and `REPO` (owner/repo).
- `send_email.py` — Sends a confirmation email using Gmail SMTP. Expects `GMAIL_EMAIL`, `GMAIL_APP_PASSWORD`, `ADMIN_EMAIL` and booking fields passed as environment variables.
- `data/bookings.json` — The canonical storage for bookings (JSON array).
- `requirements.txt` — optional dependency list; the core scripts use Python standard library modules.

## Workflow (how it works)

1. A student fills the form in `index.html`, selects date and time, and confirms a booking.
2. The front-end verifies availability by fetching the raw `data/bookings.json` from the repo.
3. On confirmation the front-end triggers a GitHub Actions workflow (example name: `handle_booking.yml`) via the repository dispatch / workflow dispatch endpoint with inputs for `save_booking` and `send_email`.
4. The workflow runs `save_booking.py` (commits the new booking to `data/bookings.json`) and `send_email.py` (sends an email to the student and CCs admin) using secrets stored in the workflow (recommended).
5. The admin UI (`admin.html`) can also modify `data/bookings.json` directly and commit changes back to the repo.

Note: In this repository the token placeholders in the front-end files are for convenience/testing only. Do NOT embed long-lived tokens in client-side code for production — use server-side workflows or GitHub Actions secrets instead.

## Project setup

Prerequisites:

- Python 3.8+ installed
- A GitHub repository (the code expects `data/bookings.json` to exist in the repo)

Quick start (local testing):

1. Clone the repo and enter its directory.

2. (Optional) Create a virtual environment and install dependencies:

```bash
python3 -m venv venv
source venv/bin/activate
# optional: pip install -r requirements.txt
```

3. Serve the static UI locally and open the student page:

```bash
# from the project root
python3 -m http.server 8000
# then open http://localhost:8000/index.html
```

Testing `save_booking.py` locally (example):

```bash
export BOOKING_DATA='{"id":"1","name":"Test Student","email":"test@depaul.edu","studentId":"1234567","course":"CSC 555","subject":"Project Review","reason":"Testing","day":"Monday","dayDate":"2026-04-20","dayDisplay":"Apr 20, 2026","slotId":"1000","timeRange":"10:00 - 10:30"}'
export GH_TOKEN='ghp_xxx'   # GitHub token with repo permissions
export REPO='youruser/your-repo'
python3 save_booking.py
```

Testing `send_email.py` locally (example):

```bash
export GMAIL_EMAIL='you@example.com'
export GMAIL_APP_PASSWORD='your_app_password'
export ADMIN_EMAIL='admin@depaul.edu'
export NAME='Test Student'
export EMAIL='test@depaul.edu'
export STUDENT_ID='1234567'
export COURSE='CSC 555'
export SUBJECT='Project Review'
export REASON='Testing'
export DAY='Monday'
export DAY_DISPLAY='Apr 20, 2026'
export TIME_RANGE='10:00 - 10:30'
python3 send_email.py
```

GitHub Actions integration (recommended for production):

- Create a workflow (e.g. `.github/workflows/handle_booking.yml`) that accepts `workflow_dispatch` or repository dispatch inputs and runs `save_booking.py` and/or `send_email.py` with inputs mapped to environment variables.
- Store secrets (GitHub token, Gmail app password, admin email) in the repository or organization secrets and access them from the workflow. Never commit tokens to the repo.

## Security notes

- Do NOT hard-code tokens or passwords in `index.html` or `admin.html`. Those files are client-side and public if pushed to a public repo.
- The `admin.html` demo uses a simple password check and storing a token in the page for convenience only — replace with secure auth in production.
- Use GitHub Actions secrets and server-side code to keep credentials private.

## License (MIT)

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

----

If you'd like, I can also scaffold a recommended `handle_booking.yml` workflow that runs `save_booking.py` and `send_email.py` using repository secrets. Want me to add that?

