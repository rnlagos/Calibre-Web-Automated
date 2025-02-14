# Calibre-Web-Automated
## Migrate Users from CalibreServe (users.sqlite) to Calibre-Web-Automated (app.db)

I have had problems migrating from CW to CWA, although I have put the path for :config (/srv/calibre) the users have not been added so I have made a small script in Python. 

This script helps migrate user accounts from CalibreServe (users.sqlite) to Calibre-Web-Automated (app.db). It ensures that plain-text passwords are securely hashed using PBKDF2-SHA256, matching the password format required by Calibre-Web-Automated.

**Features**

- Extracts users from users.sqlite.
- Hashes passwords using PBKDF2-SHA256.
- Inserts users into app.db, setting email to NULL and role to 1 (regular user).
- Skips users that already exist in app.db to avoid duplicates.

**Requirements**

Ensure you have Python installed along with the required dependencies:

`pip install werkzeug`

**Usage**

Run the script in the same directory where users.sqlite and app.db are located.

```
import sqlite3
from werkzeug.security import generate_password_hash

# Connect to the databases
source_db = sqlite3.connect("users.sqlite")
dest_db = sqlite3.connect("app.db")

source_cursor = source_db.cursor()
dest_cursor = dest_db.cursor()

# Fetch users from users.sqlite
source_cursor.execute("SELECT id, name, pw FROM users")
users = source_cursor.fetchall()

# Insert users into app.db with hashed passwords
for user in users:
    user_id, name, plain_password = user
    hashed_password = generate_password_hash(plain_password, method="pbkdf2:sha256", salt_length=16)

    try:
        dest_cursor.execute(
            """
            INSERT INTO user (name, email, role, password, kindle_mail, locale, sidebar_view, default_language, denied_tags, allowed_tags, denied_column_value, allowed_column_value, view_settings, kobo_only_shelves_sync)
            VALUES (?, NULL, 1, ?, NULL, 'en', 1, 'all', NULL, NULL, NULL, NULL, '{}', 0)
            """,
            (name, hashed_password)
        )
    except sqlite3.IntegrityError:
        print(f"⚠️  User '{name}' already exists in app.db, skipping...")

# Commit changes and close connections
dest_db.commit()
source_db.close()
dest_db.close()

print("✅ Migration completed successfully.")
```

**Notes**

- The script assigns role = 1 (regular user) by default.
- email is set to NULL since users.sqlite does not store emails.
- If an existing user is found in app.db, it will be skipped to prevent conflicts.

Feel free to use, modify, and share this script. Contributions are welcome!
