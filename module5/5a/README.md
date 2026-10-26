# Module 5a: Using .env

1. [Preliminaries](#1-preliminaries)
2. [What is an .env file?](#2-what-is-an-env-file)
3. [Instructions to setup .env](#3-instructions-to-setup-env)

## 1. Preliminaries

Welcome to Module 5 where we go into a critical step of app development, deploying your application for demonstration.
Application deployment is a major undertaking that requires additional setup and maintenance you may not be familiar with, but it is one of the biggest indicators of your proficiency to develop applications that provide value to potential clients.

The contents of this module are for the purposes of demonstrating coursework, this is not meant to actually setup your app in a production environment.
However doing this will allow anyone with an internet connection to access your web application (provided they know the URL).

The first step described in this submodule can be done at any time during development (the earlier the better!).
This utilizes a common practice in collaborative development where credentials such as your database username or password is stored on a separate file locally in your computer.

## 2. What is an .env file?

An `.env` file is a plain text file that stores settings for your application as `NAME=value` pairs, one per line:

```text
DB_HOST=localhost
DB_PASSWORD=admin
```

These pairs are called **environment variables**. When your app starts, a library called `python-dotenv` reads the file and makes each pair available to your Python code through `os.environ`:

```python
os.environ['DB_PASSWORD']   # gives you 'admin'
```

This fixes the repeated changing of credentials everytime you pull your application for the remote repository only to find someone changed the API script (your dbconnect.py).
More importantly, this is a far more secure and flexible way to manage credentials in the context of handling your information locally and online.

Just as a gentle reminder, GitHub repositories can be made public, so storing passwords and sensitive information is a very bad idea and lacks the privacy that other cloud storage providers offer.

## 3. Instructions to setup .env

### 3.1 Step 1: Install python-dotenv

In your terminal, with your virtual environment active, run the following:

```bash
pip install python dot-env
```

### 3.2 Step 2: Protect .env with .gitignore

Before creating your `.env` file, update your .gitignore to exclude the file:

```text
/venv
<your other ignored files>

.env
```

### Step 3: Create the .env file

In the root folder, create a new file with the name ".env"
Similar to .gitignore, this is a text file where you can add credentials specific to you and your local environment.

Add the following in your newly created file. Change the credentials according to your own use case.

```text
DB_HOST=localhost
DB_NAME=ie172sampledb
DB_USER=postgres
DB_PORT=5432
DB_PASSWORD=admin
```

- No spaces around the `=` sign.
- No quotes needed around the values.
- No comments on the same line as a value.

### Step 4: Update dbconnect.py

Replace the hardcoded values with the lookup from the environment.

```python
import os    # Additional import

import pandas as pd
import psycopg2
from dotenv import load_dotenv     # Additional import

load_dotenv()  # reads the .env file in your project folder


def getdblocation():
    db = psycopg2.connect(
        host=os.environ['DB_HOST'],
        database=os.environ['DB_NAME'],
        user=os.environ['DB_USER'],
        port=os.environ['DB_PORT'],
        password=os.environ['DB_PASSWORD'],
    )

    return db


def modifyDB(sql, values):
    db = getdblocation()
...
```

### Step 5: Test your app

Don't forget to save your changes first then run the app normally. If an error occurs please double-check your setup.

### Step 6: Create the .env template

Since `.env` is not shared, others need to know which variables to set. Create a second file called `.env.example` with the same names but with placeholder values:

```text
DB_HOST=your-database-host
DB_NAME=your-database-name
DB_USER=your-database-user
DB_PORT=5432
DB_PASSWORD=your-database-password
```

Unlike `.env`, this file **is** committed to Git, because it contains no real credentials. Anyone who clones your repo copies it to `.env` and fills in their own values.

### Step 7: Check that Git is ignoring the file

```bash
git status
```

`.env` must **not** appear in the list of files to be committed. `.env.example` should.

### Next step

When you deploy in the next module, you will not upload your `.env` file. Instead, you will enter these same variable names (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PORT`, `DB_PASSWORD`) in the hosting platform's dashboard, with values for your online database. Feel free to use this setup to streamline your development process!
