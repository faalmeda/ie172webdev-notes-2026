# Module 5d: Deploying your app on Render

1. [Preliminaries](#1-preliminaries)
2. [What is Render?](#2-what-is-render)
3. [Prepare your project](#3-prepare-your-project)
4. [Push your project to GitHub](#4-push-your-project-to-github)
5. [Create a Render account](#5-create-a-render-account)
6. [Create the web service](#6-create-the-web-service)
7. [Add your environment variables](#7-add-your-environment-variables)
8. [Deploy and test](#8-deploy-and-test)
9. [Updating your app](#9-updating-your-app)
10. [Before your demo](#10-before-your-demo)
11. [Troubleshooting](#11-troubleshooting)
12. [When you are done](#12-when-you-are-done)

## 1. Preliminaries

In this module you will put your Dash app online using **Render**, so anyone with the link can open it, even when your computer is off. Your database lives in Supabase (Module 5b), and the app on Render connects to it.

```text
Viewer's browser --> Render (your Dash app) --> Supabase (your PostgreSQL database)
```

Before you start, make sure that:

- **Module 5a is done:** your database code reads credentials from environment variables (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PORT`, `DB_PASSWORD`), and `.env` is listed in `.gitignore`.
- **Module 5b is done:** your tables and data are in Supabase, and you have your **session pooler** connection details.
- **Your app works locally against Supabase:** with your `.env` pointing at Supabase, `python index.py` loads your pages and data. If it does not work locally, it will not work on Render, so fix it here first.
- You have a **GitHub** account and your project is (or will be) in a GitHub repository.

## 2. What is Render?

Render is a hosting platform. You connect it to your GitHub repository, tell it how to install and start your app, and it runs the app on its servers and gives you a public `https://` link.

It has a free plan that is enough for a course demo. The trade-offs:

- **The app goes to sleep when nobody has used it for about 15 minutes.** The next visitor waits roughly 30-60 seconds while it wakes up.
- **The free server has limited memory (512 MB).** Heavy libraries can use a lot of it.
- **Free plan details change.** Check Render's pricing page if something looks different.

## 3. Prepare your project

### 3.1 Check your project structure

Your project should look like this, with `index.py` as the file that loads everything:

```
my-dash-app/
├── app.py              creates the Dash app and the Flask server
├── index.py            layout, callbacks, imports
├── (other modules)     e.g. database script, pages, layouts
├── requirements.txt
├── .gitignore
├── .env                (local only, never committed)
├── .env.example
└── README.md
```

Render starts your app through a production server called **gunicorn**. For that to work, `app.py` must expose the Flask server:

```python
# app.py
from dash import Dash

app = Dash(__name__)
server = app.server   # gunicorn needs this
```

And `index.py` must import it:

```python
# index.py
from app import app, server   # importing server here is what makes it available
```

The `if __name__ == "__main__": app.run(...)` block at the bottom of `index.py` is only for running on your own computer. Gunicorn ignores it, so you can leave it as is.

### 3.2 Create `requirements.txt`

Render installs your packages from `requirements.txt`. Generate it from your working environment:

```bash
pip freeze > requirements.txt
```

Then open the file and make these changes:

1. **Replace `psycopg2` with `psycopg2-binary`.** The plain `psycopg2` package must be compiled from source, which fails on Render's build servers. Use the same version number, for example `psycopg2-binary==2.9.11`.
2. **Add `gunicorn`** on its own line. This is the server Render uses to run your app.
3. **Make sure `python-dotenv` is in the list.** Your code imports it, so the app will crash on Render without it.
4. **Optional:** remove packages you only use for testing (for example `pytest`, `pluggy`, `iniconfig`) to speed up the build.

Delete any line that looks Windows-only (for example `pywin32`), because the build will fail on Render's Linux servers. `colorama` is fine to leave in.

<details>
<summary>Example of a finished requirements.txt (yours may differ)</summary>

```
babel==2.18.0
blinker==1.9.0
certifi==2026.2.25
charset-normalizer==3.4.6
click==8.3.1
colorama==0.4.6
dash==4.0.0
dash-bootstrap-components==2.0.4
dash_mantine_components==2.7.0
DateTime==6.0
docxcompose==2.1.0
docxtpl==0.20.2
Flask==3.1.3
gunicorn
idna==3.11
importlib_metadata==8.7.1
itsdangerous==2.2.0
Jinja2==3.1.6
lxml==6.1.0
MarkupSafe==3.0.3
narwhals==2.18.0
nest-asyncio==1.6.0
numpy==2.4.3
packaging==26.0
pandas==3.0.1
patsy==1.0.2
plotly==6.6.0
plotly-express==0.4.1
psycopg2-binary==2.9.11
Pygments==2.20.0
python-dateutil==2.9.0.post0
python-docx==1.2.0
python-dotenv==1.2.2
pytz==2026.1.post1
requests==2.32.5
retrying==1.4.2
scipy==1.17.1
setuptools==82.0.1
six==1.17.0
statsmodels==0.14.6
typing_extensions==4.15.0
tzdata==2025.3
urllib3==2.6.3
Werkzeug==3.1.6
zipp==3.23.0
zope.interface==8.3
```

</details>

### 3.3 Note your Python version

Run this and write down the result (for example `3.12.8`). You will need it in Section 7:

```bash
python --version
```

### 3.4 Check `.gitignore` and `.env.example`

Your `.gitignore` must contain at least:

```
.env
__pycache__/
venv/
.venv/
```

Your `.env.example` (committed to Git) should list the variable names with fake values:

```
DB_HOST=your-database-host
DB_NAME=your-database-name
DB_USER=your-database-user
DB_PORT=5432
DB_PASSWORD=your-database-password
```

## 4. Push your project to GitHub

If your project is not on GitHub yet:

1. On GitHub, click **New repository**, give it a name, and create it (you can keep it private; Render can still access it once you authorize it).
2. In your project folder, run:

```bash
git init
git add .
git commit -m "Prepare app for deployment"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

If your project is already on GitHub, just commit and push your new `requirements.txt`:

```bash
git add .
git commit -m "Prepare app for deployment"
git push
```

**Before you move on, check your repository on GitHub in the browser:**

- `.env` must **not** be there. `.env.example` should be.
- `requirements.txt`, `app.py`, and `index.py` should be at the top level of the repository (not inside a subfolder).

If `.env` was pushed, change your Supabase database password immediately (see Module 5a, "If you already committed the password").

## 5. Create a Render account

1. Go to <https://render.com> and click **Get Started** or **Sign Up**.
2. Choose **Sign up with GitHub**. This is the quickest option because it links your repositories to Render at the same time.
3. Authorize Render. If GitHub asks which repositories to share, you can choose only your project's repository.

You may be asked to verify your email address.

## 6. Create the web service

1. In the Render dashboard, click **New +**, then **Web Service**.
2. Choose to build from a **Git repository** and select your repository from the list. If it is missing, click the option to configure GitHub access and grant Render access to the repository.
3. Fill in the settings:

| Setting | Value |
| --------- | ------- |
| Name | Anything, e.g. `ie172-dash-demo`. This becomes part of your link. |
| Language | `Python 3` |
| Branch | `main` |
| Region | The one closest to your Supabase project, if offered |
| Root Directory | Leave empty (unless your app is inside a subfolder) |
| Build Command | `pip install -r requirements.txt` |
| Start Command | `gunicorn index:server --workers 1 --timeout 120` |
| Instance Type | **Free** |

**About the start command:**

- `index:server` means "in `index.py`, use the variable called `server`". It must be `index`, not `app`. Gunicorn has to load `index.py` so that your layout and callbacks are registered.
- `--workers 1` keeps memory use low enough for the free server.
- `--timeout 120` gives the app time to respond on its first, slow request.

Do **not** click **Create Web Service** yet. Add your environment variables first (Section 7).

## 7. Add your environment variables

On the same page, scroll to **Environment Variables** (it may be under **Advanced**) and click **Add Environment Variable** for each row.

Use the same values that are in your working local `.env` file, the **Supabase session pooler** values from Module 5b:

| Key | Value |
| ----- | ------- |
| `DB_HOST` | `aws-[N]-[REGION].pooler.supabase.com` |
| `DB_NAME` | `postgres` |
| `DB_USER` | `postgres.[PROJECT-REF]` |
| `DB_PORT` | `5432` |
| `DB_PASSWORD` | your Supabase database password |
| `PYTHON_VERSION` | the **full** version from Section 3.3, e.g. `3.12.8` |

Rules to follow:

- Type the variable names **exactly** as they appear in your code, including capital letters.
- Do not put quotes around the values.
- `PYTHON_VERSION` needs all three numbers (`3.12.8`). Something like `3.12` will not work.
- Make sure `DB_HOST` is the **pooler** host, not `db.[...].supabase.co`. Render cannot connect to the direct host.

When everything looks right, click **Create Web Service**.

## 8. Deploy and test

### 8.1 Watch the build

Render opens a page with a live **Logs** view. You will see it installing packages and then starting gunicorn. The first build can take several minutes, especially with heavy packages like `pandas`, `scipy`, and `statsmodels`.

The deploy is finished when the status at the top changes to **Live**, and the logs show gunicorn starting (a line similar to `Listening at: http://0.0.0.0:10000`).

If the build fails, read the end of the log. The message is usually clear (for example, a missing package). See the troubleshooting table in Section 11. A failed build does not break anything, and you can fix your code and push again.

### 8.2 Open your app

Your link is shown near the top of the service page, in the form:

```
https://<your-service-name>.onrender.com
```

Click it. The first load may take up to a minute. Check that your pages open and that the data from Supabase appears.

### 8.3 Test from another device

Open the link on your phone using mobile data. This confirms the app works for people outside your computer and network.

## 9. Updating your app

Render redeploys automatically every time you push to the `main` branch:

```bash
git add .
git commit -m "Describe your change"
git push
```

Watch the **Events** or **Logs** tab to see the new deploy run. If a deploy fails, Render keeps the previous working version running until you push a fix.

If you only changed an environment variable, Render will redeploy for you after you save it.

## 10. Before your demo

Both free services go idle, and you do not want to discover that in front of the class:

- **Open your Render link 1-2 minutes before presenting** so the app is awake. The first visit after a period of inactivity is slow.
- **Open your Supabase dashboard the day before.** Free Supabase projects are paused after about a week of inactivity. If yours was paused, restore it from the dashboard.
- **Do a full run-through** from your phone or another computer, including any page that writes to the database.

## 11. Troubleshooting

Always start by opening the **Logs** tab on your Render service and reading the last lines.

| Symptom | Likely cause and fix |
| --------- | ---------------------- |
| `pg_config executable not found` or a psycopg2 build error | `requirements.txt` still has `psycopg2`. Change it to `psycopg2-binary` and push. |
| `ModuleNotFoundError: No module named '...'` | That package is missing from `requirements.txt`. Add it and push. |
| `Failed to find attribute 'server' in 'index'` | `index.py` does not import `server`. Add `from app import app, server`. |
| `ImportError: cannot import name 'app' from partially initialized module` | Circular import. `app.py` must not import `index.py` or anything that imports it. |
| Page loads but is blank, or callbacks do nothing | The start command points at the wrong module. It must be `gunicorn index:server`, not `app:server`. |
| `KeyError: 'DB_HOST'` (or another name) | The variable is missing or misspelled under Environment Variables. Fix it and save. |
| `network is unreachable` or a connection timeout | `DB_HOST` is the direct Supabase host. Use the **session pooler** host. |
| `password authentication failed` or `Tenant or user not found` | `DB_USER` must be `postgres.[PROJECT-REF]`, and `DB_PASSWORD` must be your Supabase database password. |
| Build fails with a Python version error | `PYTHON_VERSION` is missing, or is not a full version like `3.12.8`. |
| Log shows the worker was killed or ran out of memory | The free server has 512 MB. Keep `--workers 1`, remove unused packages from `requirements.txt`, and avoid loading large datasets at startup. |
| The site takes about a minute to load | The free service was asleep. Wait, then refresh. |
| App worked yesterday, errors today | The Supabase project may have been paused. Open the Supabase dashboard and restore it. |
| Something is clearly stale after a change | On the service page, use **Manual Deploy**, then **Clear build cache & deploy**. |

## 12. When you are done

After your course demo or grading period, you can stop the service so it does not stay online:

1. Open your service in Render, go to **Settings**, and scroll to the bottom.
2. Choose **Suspend Web Service** (you can resume it later) or **Delete Web Service** (permanent).

If you no longer need the database, you can also delete or pause the Supabase project from its dashboard.

## Deployment checklist

- [ ] App works locally against Supabase (`python index.py`)
- [ ] `requirements.txt` has `psycopg2-binary`, `gunicorn`, and `python-dotenv`
- [ ] `.env` is not on GitHub; `.env.example` is
- [ ] Render start command is `gunicorn index:server --workers 1 --timeout 120`
- [ ] All six environment variables are set (five `DB_*` plus `PYTHON_VERSION`)
- [ ] Link opens on a phone using mobile data
- [ ] Render link opened shortly before the demo; Supabase dashboard checked the day before
