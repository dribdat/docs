# Dribdat CLI Documentation

---

## Overview

This Python script provides a command-line interface (CLI) for managing the Dribdat application, built on Flask and Click. It enables administrators to perform bulk operations on events, users, projects, and challenges.

> **Entry Point**: Run with `python cli.py`

The CLI automatically selects the configuration based on the `DRIBDAT_ENV` environment variable:

| Environment Variable | Configuration Used |
|-----------------------|--------------------|
| `DRIBDAT_ENV=prod`    | `ProdConfig`       |
| Otherwise             | `DevConfig`       |

For more details see [Configuration docs](https://dribdat.cc/deploy.html#from-source).

---

## Commands Reference

---

### `ls`
List all visible events.

**Usage**
```
python cli.py ls
```

**Output**
- Total count of non-hidden events
- Tab-separated list: `EVENT_ID    EVENT_NAME    PROJECT_COUNT`

---

### `socialize`
Reset or refresh user profile data.

**Usage**
```
python cli.py socialize [kind...]
```

**Arguments**
| Argument | Required | Description |
|----------|----------|-------------|
| `kind`   | No       | (Future) Specify which models to refresh (e.g., `users`). Currently refreshes all users. |

**Behavior**
- Calls `User.socialize()` on all users in the database.
- Output: Number of updated users.

---

### `numerise`
Assign numeric identifiers to projects or challenges for a given event.

**Usage**
```
python cli.py numerise EVENT_ID [--clear] [--primes] [--challenges]
```

**Arguments**
| Argument    | Required | Type      | Default | Description |
|-------------|----------|-----------|---------|-------------|
| `event`     | Yes      | `int`     | -       | ID of the event to process. |
| `--clear`   | No       | `bool`    | `False` | If set, clears identifiers instead of setting them. |
| `--primes`  | No       | `bool`    | `False` | Use prime numbers (2, 3, 5, 7, ...) for identifiers. |
| `--challenges` | No    | `bool`    | `False` | Apply numbering to challenges only (default: projects only). |

**Behavior**
- Only processes non-hidden projects with `progress >= 0`.
- Projects are ordered by `id` (ascending).
- Identifiers are padded with zeros if needed (e.g., `001`, `012`).

**Example Output**
```
Applying numbers to event: My Hackathon
Enumerated 15 projects.
```

---

### `event_start`
Create a new event.

**Usage**
```
python cli.py event_start NAME [START] [FINISH]
```

**Arguments**
| Argument | Required | Type       | Default | Description |
|----------|----------|------------|---------|-------------|
| `name`   | Yes      | `str`      | -       | Name of the event. |
| `start`  | No       | `datetime` | Tomorrow | Start time (ISO format, e.g., `2026-06-01T10:00:00`). |
| `finish` | No       | `datetime` | 2 days after `start` | End time (ISO format). |

**Behavior**
- Creates an `Event` with the given name, start, and end times.
- Output: ID of the created event.

---

### `imports`
Import events, projects, and users from a datapackage (URI or file).

**Usage**
```
python cli.py imports URI_OR_PATH [LEVEL]
```

**Arguments**
| Argument | Required | Type   | Default | Description |
|----------|----------|--------|---------|-------------|
| `url`    | Yes      | `str`  | -       | URI (HTTP/HTTPS) or local file path to the datapackage. |
| `level`  | No       | `str`  | `full`  | Import mode: `dry run`, `basic`, or `full`. |

**Import Modes**
| Level    | Dry Run | All Data | Description |
|----------|---------|----------|-------------|
| `dry run`| Yes     | No       | Preview changes without saving. |
| `basic`  | No      | No       | Import with minimal data. |
| `full`   | No      | Yes      | Import all available data. |

**Output**
- List of created event names.
- Errors (if any).

---
---
### `exports`
Export system data (users or events) to CSV (stdout).

**Usage**
```
python cli.py exports kind...
```

**Arguments**
| Argument | Required | Type   | Description |
|----------|----------|--------|-------------|
| `kind`   | Yes      | `str`  | One or more of: `people`, `events`. |

**Exports**

**`people` (Users)**
Columns: `id, username, email, updated_at, fullname, my_skills, my_wishes, roles, teams, project_ids`

**`events`**
Columns: `name, starts_at, ends_at`

**Example**
```bash
python cli.py exports people > users.csv
python cli.py exports events > events.csv
```

---
---
### `register`
Import users from a CSV file.

**Usage**
```
python cli.py register FILENAME [TESTONLY]
```

**Arguments**
| Argument  | Required | Type      | Default | Description |
|-----------|----------|-----------|---------|-------------|
| `filename`| Yes      | `str`     | -       | Path to the CSV file. |
| `testonly`| No       | `bool`    | `False` | If `True`, does not save changes (dry run). |

**Output**
- Number of users imported and updated.

---
---
### `kick`
Clean up inactive or low-scoring user accounts.

**Usage**
```
python cli.py kick [OPTIONS]
```

**Options**
| Option       | Type      | Default | Description |
|--------------|-----------|---------|-------------|
| `--lowscore` | `bool`    | `False` | Target users with no content (low score). |
| `--inactive` | `bool`    | `False` | Target inactive users. |
| `--withsso`  | `bool`    | `False` | Include users with active SSO. |
| `--delete`   | `bool`    | `False` | Delete users (default: deactivate only). |
| `--score`    | `int`     | `0`     | Minimum score threshold for `--lowscore`. |

**Behavior**
- Targets non-admin users only.
- If `--inactive` and `--lowscore` are both set, only inactive users with no content are targeted.
- If `--delete` is set, users are permanently deleted; otherwise, they are deactivated (`active=False`).
- Safety: Prints a 5-second countdown (Ctrl+C to abort) before execution.
- Output: CSV list of affected users (`username,fullname,email,webpage_url`).

**Example**
```bash
# Deactivate inactive users
python cli.py kick --inactive

# Delete users with no content and score < 5
python cli.py kick --lowscore --delete --score 5

# Deactivate inactive users, including those with SSO
python cli.py kick --inactive --withsso
```

---
---
---
## Helper Functions

| Function | Description |
|----------|-------------|
| `create_app()` | Initializes the Flask app with the correct config (`DevConfig` or `ProdConfig`). |
| `fetch_datapackage()` | Fetches and processes a datapackage from a URI. |
| `load_file_datapackage()` | Loads and processes a datapackage from a local file. |
| `import_users_csv()` | Imports users from a CSV file. |

---
---
## Notes

1. All commands requiring database access use `create_app().app_context()`.
2. Uses `UTC` (from `dribdat.futures`) for datetime operations.
3. Commands like `imports` and `register` support dry-run modes for safety.
4. The `kick` command with `--delete` is irreversible.
5. Future Work:
   - `socialize`: The `kind` parameter is not yet implemented (currently refreshes all users).
   - `numerise`: Sort order (alphabetic, ID-based, etc.) is hardcoded to `Project.id`.
