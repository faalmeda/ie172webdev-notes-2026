# Module 5b: App Deployment Part 1

1. [Preliminaries](#1-preliminaries)
2. [What is Supabase?](#2-what-is-supabase)
3. [Create your Supabase project](#3-create-your-supabase-project)
4. [Get your connection details](#4-get-your-connection-details)
5. [Copy your local database to Supabase](#5-copy-your-local-database-to-supabase)
6. [Point your app to Supabase]
7. [Things to know about the free plan]
8. [Troubleshooting]

## 1. Preliminaries

So far your app has been talking to a PostgreSQL database running on your own computer (`localhost`). The database is invisible and not connected to the internet, so when your app is deployed online, it will not be able to reach it.

In this module you will create a free PostgreSQL database in the cloud using **Supabase**, copy your local database into it, and test your app against it from your own computer. Deploying the app itself comes in the next module.

  - Complete **Module 5a** first. 
  - Your database should be at least close to its final version.

You will need:

- A Supabase account (sign up at <https://supabase.com>; signing in with GitHub is the quickest)
- Your local database running via pgAdmin, with the tables and data you want to deploy

## 2. What is Supabase?

Supabase is a Backend-as-a-Service platform built around PostgreSQL. It also offers authentication, file storage, and an automatic API, but we only need one thing from it: a hosted PostgreSQL database that your app can connect to over the internet.

Because it is regular PostgreSQL, your SQL, your tables, and your Python code all work unchanged. Only the database credentials change, which is exactly why we moved them into a `.env` file.

## 3. Create your Supabase project

1. Log in to Supabase and click **New project**.

<add-screenshot>

2. Fill in the form:

| Field | What to enter |
| ------- | --------------- |
| Name | Anything you like, e.g. `ie172-demo` |
| Database password | A long, strong password. **Use only letters and digits**; special characters can break connection strings. |
| Region | The option closest to you (for the Philippines, choose Southeast Asia / Singapore if it is offered) |
| Plan | Free |

1. **Save your database password now**, in a password manager or a secure note. It is shown only once; if you lose it you can only reset it, not view it.
2. If the form shows security options:
   - **Enable Data API:** turn this **off**. Your app connects directly to the database and does not use Supabase's REST API.
   - **Automatic RLS:** you can leave this on.
3. Leave everything else alone. You do not need Authentication, Storage, Realtime, or Edge Functions.
4. Click **Create new project** and wait a couple of minutes while it is set up.

## 4. Get your connection details

In your project, click the **Connect** button at the top of the page.

<add-screenshot>

1. Click 'Direct' tab among 'Framework, Server, Direct, ORM, MCP'.
2. Choose the **Session pooler** option.
3. Copy the connection details. They look like this:

```
postgresql://postgres.[PROJECT-REF]:[YOUR-PASSWORD]@aws-[N]-[REGION].pooler.supabase.com:5432/postgres
```

Now match the pieces of that string to your five `.env` variables:

| `.env` variable | Value from the connection string |
| ----------------- | ---------------------------------- |
| `DB_HOST` | `aws-[N]-[REGION].pooler.supabase.com` |
| `DB_NAME` | `postgres` |
| `DB_USER` | `postgres.[PROJECT-REF]` (the username includes your project reference, **not** just `postgres`) |
| `DB_PORT` | `5432` |
| `DB_PASSWORD` | the password you saved in Step 3 |

Keep these handy for Sections 5 and 6.

## 5. Copy your local database to Supabase

Your Supabase database starts empty. You need to copy your tables and data into it. Pick **one** of the two methods below.

### Method A: Export a SQL file from pgAdmin (recommended for small databases)

**Part 1: Export from your local database**

1. In pgAdmin, right-click your local database (`ie172sampledb`) and choose **Backup...**
2. Set the options:
   - **Filename:** choose a location and name, e.g. `ie172sampledb.sql`
   - **Format:** `Plain`
   - On the **Data Options** tab (or **Options**, depending on your version), turn on **Owner: No / Do not save owner** and **Privileges: No / Do not save privileges** if you see them.
3. Click **Backup** and wait for the success message.

**Part 2: Import into Supabase**

1. In Supabase, open **SQL Editor** from the left sidebar and start a new query.
2. Open your `.sql` file in a text editor, copy its contents, paste them into the SQL Editor, and click **Run**.

If the file is very large and the editor struggles, use Method B instead.

### Method B: Restore with psql (works for any size)

1. Export your local database to a plain SQL file (you can use the pgAdmin steps above, or this command):

```bash
pg_dump -h localhost -U postgres -d ie172sampledb --no-owner --no-privileges -f ie172sampledb.sql
```

1. Load it into Supabase using your session pooler connection string from Section 4:

```bash
psql "postgresql://postgres.[PROJECT-REF]:[YOUR-PASSWORD]@aws-[N]-[REGION].pooler.supabase.com:5432/postgres" -f ie172sampledb.sql
```

### Check that it worked

1. In Supabase, open **Table Editor** from the left sidebar.
2. Your tables should be listed, with your data in them. Click a table to check that the rows are there.

You can also run a quick query in the **SQL Editor**:

```sql
select count(*) from your_table_name;
```

## 6. Point your app to Supabase

Open your `.env` file (from Module 5a) and replace the local values with the Supabase ones from Section 4:

```
DB_HOST=aws-[N]-[REGION].pooler.supabase.com
DB_NAME=postgres
DB_USER=postgres.[PROJECT-REF]
DB_PORT=5432
DB_PASSWORD=your-supabase-password
```

**Tip:** before you overwrite them, save your old local values in a comment block or a second file (for example `.env.local`), so you can switch back to your local database later. Remember that `.env.local` must also be listed in `.gitignore`.

Now run your app as usual:

```bash
python index.py
```

If your pages load the same data as before, your app is now reading from the cloud. Your code did not change. Only `.env` did.

## 7. Things to know about the free plan

- **Inactivity pause:** free projects are paused after about a week without activity. Your data is not lost, but the app will not connect until you open the Supabase dashboard and restore the project. **Open the dashboard the day before any demo.**
- **Storage limit:** the free database is limited to about 500 MB. This is plenty for coursework.
- **Project limit:** the free plan allows only a small number of active projects (two at the time of writing), so delete old test projects you no longer need.
- **Limits change.** Check Supabase's pricing page if something here looks out of date.

## 8. Troubleshooting

| Symptom | Likely cause and fix |
| --------- | ---------------------- |
| `could not translate host name` or `network is unreachable` | You used the direct connection host (`db.[...].supabase.co`). Use the **Session pooler** host instead. |
| `password authentication failed` | Wrong password, or you used `postgres` as the user. The user must be `postgres.[PROJECT-REF]`. If you forgot the password, reset it under **Project Settings → Database**. |
| `Tenant or user not found` | The username is missing the `.[PROJECT-REF]` part. |
| `KeyError: 'DB_HOST'` | The variable is missing or misspelled in `.env`, or you ran the app from a different folder. |
| Connection works but tables are missing | The import in Section 5 did not finish. Check the SQL Editor output for errors and re-run it. |
| Errors like `unrecognized configuration parameter` when importing | Your local PostgreSQL is a newer version than the dump expects Supabase to support. Delete the offending `SET ...;` lines from the `.sql` file and run it again. |
| `role "..." does not exist` or permission errors on import | The dump contains owner or privilege lines. Re-export with owner and privileges turned off (or use `--no-owner --no-privileges`). |
| `relation already exists` | You ran the import twice. Delete the tables in **Table Editor** (or drop them in the SQL Editor) and run it once more. |
| App worked yesterday, fails today | The project may have been paused. Open the Supabase dashboard and restore it. |

## Next step

In the next module you will deploy your app online. You will not upload your `.env` file. Instead, you will enter the same five variables (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PORT`, `DB_PASSWORD`) in the hosting platform's dashboard, using the Supabase values from this module.
