# Snippetbox

Snippetbox lets people paste and share snippets of text — a bit like Pastebin or GitHub’s Gists. This project was developed while following along with Alex Edwards' book [Learn to Build Professional Web Applications with Go](https://lets-go.alexedwards.net). Topics covered included project structuring, query routing, database management, form processing, and secure display of dynamic data.

*P.S Each commit in the project, except for the last one, is made at the end of a chapter in the book. A commit message is referred to as a book chapter after which it was created*

## How to Use

To get started with the Snippetbox:

1. **Clone the Repository**

```bash
git clone https://github.com/kirinyoku/snippetbox.git
```
2. **Set up MySQL database**

You’ll need to install MySQL on your computer. After installing the database, connect to it. Copy and paste the following commands into the mysql prompt to create a new snippetbox database using UTF8 encoding.

```sql
-- Create a new UTF-8 `snippetbox` database.
CREATE DATABASE snippetbox CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Switch to using the `snippetbox` database.
USE snippetbox;
```
Then copy and paste the following SQL statement to create a new snippets table to hold the text snippets for our application:

```sql
    -- Create a `snippets` table.
CREATE TABLE snippets (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,
    created DATETIME NOT NULL,
    expires DATETIME NOT NULL
);

-- Add an index on the created column.
CREATE INDEX idx_snippets_created ON snippets(created);
```
Create a session table in the database to store session data for our users.

```sql
USE snippetbox;

CREATE TABLE sessions (
    token CHAR(43) PRIMARY KEY,
    data BLOB NOT NULL,
    expiry TIMESTAMP(6) NOT NULL
);

CREATE INDEX sessions_expiry_idx ON sessions (expiry);
```
Create a users table.

```sql
USE snippetbox;

CREATE TABLE users (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    hashed_password CHAR(60) NOT NULL,
    created DATETIME NOT NULL
);

ALTER TABLE users ADD CONSTRAINT users_uc_email UNIQUE (email);
```
From a security point of view it’s not a good idea to connect to MySQL as the root user from a web application. Instead it’s better to create a database user with restricted permissions on the database.

```sql
CREATE USER 'web'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON snippetbox.* TO 'web'@'localhost';
-- Important: Make sure to swap 'pass' with a password of your own choosing.
ALTER USER 'web'@'localhost' IDENTIFIED BY 'pass';
```
When launching the app, use the **dsn** flasg to specify your database credentials.

3. **Generate a self-signed TLS certificate**

**crypto/tls** in the Go standard library contains the **generate_cert.go** tool, which can be used to easily create your own self-signed certificate. To run the **generate_cert.go** tool, you’ll need to know the place on your computer where the source code for the Go standard library is installed. If you’re using Linux, macOS or FreeBSD and followed the [official install instructions](https://go.dev/doc/install#install), then the **generate_cert.go** file should be located under /usr/local/go/src/crypto/tls.

```bash
cd snippetbox
mkdir tls
cd tls
go run /usr/local/go/src/crypto/tls/generate_cert.go --rsa-bits=2048 --host=localhost
```
4. **Install Dependencies, Build, and Run**

Make sure you're in the project directory and run the following commands.

```bash
go mod tidy
go build -o ./bin/ ./cmd/web/
./bin/web
```
After the server is launched, the application will be available at https://localhost:4000
