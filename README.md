# Campus Blog — Student Project

Welcome to **Campus Blog**, a simple student-run blogging platform intended for learning, collaboration, and practicing web development.
This repository contains a minimal PHP-based blog used by the student team to post announcements, share tutorials, and coordinate campus events.

---

## Table of Contents

* [Project Overview](#project-overview)
* [Features](#features)
* [Tech Stack](#tech-stack)
* [Getting Started (Developer Setup)](#getting-started-developer-setup)

  * [Prerequisites](#prerequisites)
  * [Install & Run Locally](#install--run-locally)
* [Project Structure](#project-structure)
* [How the Blog Works (High Level)](#how-the-blog-works-high-level)
* [API / Pages](#api--pages)
* [Database Schema](#database-schema)
* [Contributing](#contributing)
* [Development Notes & TODOs](#development-notes--todos)
* [Security Notes & Known Issues](#security-notes--known-issues)
* [License](#license)
* [Flag (for QA / graders)](#flag-for-qa--graders)

---

## Project Overview

Campus Blog is meant to be a compact codebase for teaching fundamentals: authentication, sessions/cookies, templating, storing posts and comments, and interacting with a relational database. It is intentionally minimal so students can easily inspect, extend, or deliberately break it for hands-on exercises and CTF training.

This repository was created to illustrate:

* a small LAMP-style app (PHP + MySQL)
* basic user registration and login
* a public blog with posts and comments
* how subtle mistakes (like mishandled tokens or exposed files) can lead to sensitive data exposure — useful for security learning.

---

## Features

* Register / Login / Logout flows (simple form-based authentication)
* Post listing, view single post, leave comments
* Comments persisted in the database
* Admin-only page (intended to be hidden or restricted)
* Lightweight, intentionally imperfect security model for training purposes

---

## Tech Stack

* PHP 7.x / 8.x (vanilla)
* MySQL / MariaDB
* Apache (or PHP built-in server for development)
* HTML + CSS (simple styling)
* Optional: Docker for single-image deployments in training/CTF environments

---

## Getting Started (Developer Setup)

### Prerequisites

* PHP (7.4 or 8.x)
* MySQL or MariaDB
* Composer (optional, not required for this project)
* Git

If you prefer, you can use the included Dockerfile to run the entire app (including database) inside a single container for testing / CTF usage.

### Install & Run Locally

1. Clone the repository:

   ```bash
   git clone https://github.com/uvctf-ctf/student-blog.git
   cd student-blog
   ```

2. Create the MySQL database and user, or use a local Docker container. Example MySQL commands:

   ```sql
   CREATE DATABASE ctfdb;
   CREATE USER 'ctfuser'@'localhost' IDENTIFIED BY 'ctfpass';
   GRANT ALL PRIVILEGES ON ctfdb.* TO 'ctfuser'@'localhost';
   FLUSH PRIVILEGES;
   ```

3. Import the initial schema if provided (e.g., `init.sql`):

   ```bash
   mysql -u ctfuser -p ctfdb < init.sql
   ```

4. Configure DB connection in `config.php` or via environment variables.

5. Serve with PHP built-in for quick development:

   ```bash
   php -S 0.0.0.0:8000 -t web
   ```

   Then open `http://localhost:8000` in your browser.

6. (Optional) Use the provided Docker setup to build the single-image container (if available in repo):

   ```bash
   docker build -t campus-blog .
   docker run -d -p 8000:80 campus-blog
   ```

---

## Project Structure

```
/web
  ├─ index.php        # landing page
  ├─ login.php        # login form / auth
  ├─ register.php     # user registration
  ├─ logout.php       # logout page
  ├─ blog.php         # list posts
  ├─ post.php         # view post & comment
  ├─ admin.php        # admin-only page (hidden)
  ├─ config.php       # DB config + helpers
  ├─ jwt_utils.php    # (optional) JWT helpers for token demo
  └─ assets/
       ├─ styles.css
       └─ app.js
/init.sql             # DB initialization SQL
/Dockerfile           # Optional single-image Dockerfile (for CTF usage)
README.md
```

---

## How the Blog Works (High Level)

1. **Registration** — Users create a username and password (hashed with `password_hash()`).
2. **Login** — On successful login, the server issues a cookie (for example `auth`) which contains the session/token (depending on the build). For CTF training, this may be a JWT or a simple session id.
3. **Viewing posts** — Public post listing is available to logged-in users (some variants may restrict it).
4. **Commenting** — Authenticated users can post comments on posts. Comments are written to the `comments` table and displayed on the post page.
5. **Admin page** — A page accessible only to admin accounts; in training mode it contains the flag. The path may be intentionally not linked to simulate a hidden resource.

---

## API / Pages

* `index.php` — Home / status
* `register.php` — Create a new user
* `login.php` — Authenticate and set cookie
* `logout.php` — Clear cookie
* `blog.php` — List all posts
* `post.php?id=<id>` — View a post and leave a comment
* `admin.php` — Admin view (flag) — not linked from navigation

---

## Database Schema

**users**

* `id` INT PRIMARY KEY AUTO_INCREMENT
* `username` VARCHAR(100) UNIQUE
* `password` VARCHAR(255) — hashed
* `role` VARCHAR(30) DEFAULT 'user'
* `created_at` TIMESTAMP

**posts**

* `id` INT PRIMARY KEY AUTO_INCREMENT
* `title` VARCHAR(255)
* `body` TEXT
* `created_at` TIMESTAMP

**comments**

* `id` INT PRIMARY KEY AUTO_INCREMENT
* `post_id` INT — FK to posts.id
* `username` VARCHAR(100)
* `comment` TEXT
* `created_at` TIMESTAMP

---

## Contributing

We welcome contributions from students and instructors. Suggested ways to contribute:

* Add unit tests or integration tests
* Improve UI / accessibility
* Harden authentication and session handling
* Add examples showing secure patterns (SOPs)
* Add additional sample posts for demonstration

To contribute:

1. Fork the repo.
2. Create a branch: `feature/your-feature`
3. Open a Pull Request with a clear description and testing instructions.

---

## Development Notes & TODOs

* [ ] Add CSRF tokens to forms (important for real production apps)
* [ ] Use prepared statements everywhere (currently used — audit to confirm)
* [ ] Sanitize and validate all user inputs server-side and client-side
* [ ] Implement HTTPS/TLS in deployment (for production)
* [ ] Audit JWT usage and signature validation (for training, some builds intentionally accept `alg: none` to teach the vulnerability)
* [ ] Add tests for authentication, DB schema migrations, and comment posting

---

## Security Notes & Known Issues

This project is intentionally imperfect for training purposes. Known weaknesses that are educational by design:

* **JWT `alg: none` demo:** Some training builds accept `alg: none` to teach token forgery. This is *insecure* for real systems.
* **Plaintext credentials in transit:** If you deploy without TLS, HTTP traffic (including credentials) will be sent in cleartext.
* **Comments reflection:** The app may echo comment HTML back to the page for learning XSS vectors — exercise caution.
* **Admin page exposure:** The admin page may be intentionally reachable via token tampering or through obscure URLs for CTF tasks.

---

## License

This project is provided for educational purposes. Use and modify freely for teaching, CTFs, and training. No warranty is provided. If you use it publicly, please credit the original source.

---

## Flag (for QA / graders)

The secret flag is included below:

```
UVCTF{github_detective_7c2b}
```

---

If you want, I can also:

* produce a ready-to-upload zip with `leaked/secrets.txt` containing the flag (so you can push it into the fork), or
* generate a short instructor PDF with hints and the grader acceptance regex, or
* create a commit history example to hide the flag in an earlier revision instead of the main branch.
