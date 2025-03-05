# Installation Guide

## Prerequisites

- A code editor (VS Code recommended)
- Git
- Terminal/Command Prompt

## Go Installation

### Windows

1. Download the Windows installer from [golang.org/dl](https://golang.org/dl/)
2. Run the installer and follow the prompts
3. Verify installation by opening Command Prompt and typing:

```sh
go version
```

### macOS

1. Using Homebrew (recommended):

```sh
brew install go
```

2. Or download the macOS package from [golang.org/dl](https://golang.org/dl/)

3. Verify installation:

```sh
go version
```

## MySQL

### Windows

1. For in-memory development purposes, we'll use SQLite, which doesn't require installation
2. For a full MySQL installation (optional):
   - Download the MySQL Installer from [mysql.com/downloads](https://dev.mysql.com/downloads/installer/)
   - Run the installer and select "Developer Default" configuration

### macOS

1. For in-memory development purposes, we'll use SQLite, which doesn't require installation
2. For a full MySQL installation (optional):

   ```
   brew install mysql
   brew services start mysql
   ```

## Tailwind CSS Setup

### Both Windows and macOS

We'll set up Tailwind CSS as part of our project using npm:

1. Ensure you have Node.js and npm installed:
   - Windows: Download from [nodejs.org](https://nodejs.org/)
   - macOS: `brew install node`
2. Verification:

   ```
   node --version
   npm --version
   ```

## Create Project Directory

### Both Windows and macOS

1. Create a new directory for your project:

   ```
   mkdir modern-go-web
   cd modern-go-web
   ```

2. Initialize Go module:

   ```
   go mod init github.com/yourusername/modern-go-web
   ```

## Install Go Dependencies

### Both Windows and macOS

Run the following commands:

```bash
# Initialize the Go project
go mod init github.com/yourusername/modern-go-web

# Install templ
go get github.com/a-h/templ/cmd/templ

# Install air for hot-reloading
go install github.com/cosmtrek/air@latest

# Install SQLC
go install github.com/kyleconroy/sqlc/cmd/sqlc@latest

# Create main.go file
touch main.go  # On Windows: type nul > main.go
```

## Set Up HTMX and Tailwind

### Both Windows and macOS

1. Create a static directory:

   ```
   mkdir -p static/css static/js
   ```

2. Download HTMX:

   ```
   # Windows (PowerShell)
   Invoke-WebRequest -Uri https://unpkg.com/htmx.org@1.9.10/dist/htmx.min.js -OutFile static/js/htmx.min.js

   # macOS
   curl -L https://unpkg.com/htmx.org@1.9.10/dist/htmx.min.js -o static/js/htmx.min.js
   ```

3. Initialize npm for Tailwind:

   ```
   npm init -y
   npm install tailwindcss
   npx tailwindcss init
   ```

4. Create `tailwind.config.js`:

   ```javascript
   /** @type {import('tailwindcss').Config} */
   module.exports = {
     content: ["./templates/**/*.templ", "./static/**/*.{html,js}"],
     theme: {
       extend: {},
     },
     plugins: [],
   }
   ```

5. Create `static/css/input.css`:

   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```

6. Add build script to `package.json`:

   ```json
   "scripts": {
     "dev": "tailwindcss -i ./static/css/input.css -o ./static/css/output.css --watch"
   }
   ```

## Setup Air for Hot Reloading

Create a `.air.toml` file in your project root:

```toml
root = "."
tmp_dir = "tmp"

[build]
  cmd = "templ generate && go build -o ./tmp/main ."
  bin = "./tmp/main"
  delay = 1000
  exclude_dir = ["assets", "tmp", "vendor", "node_modules"]
  exclude_file = []
  exclude_regex = ["_test.go", "_templ.go"]
  exclude_unchanged = false
  follow_symlink = false
  full_bin = ""
  include_dir = []
  include_ext = ["go", "templ", "sql"]
  kill_delay = "0s"
  log = "build-errors.log"
  send_interrupt = false
  stop_on_error = true

[color]
  app = ""
  build = "yellow"
  main = "magenta"
  runner = "green"
  watcher = "cyan"

[log]
  time = false

[misc]
  clean_on_exit = false

[screen]
  clear_on_rebuild = false
```

## Create SQLC Config

Create a `sqlc.yaml` file:

```yaml
version: "2"
sql:
  - engine: "sqlite"
    queries: "db/queries.sql"
    schema: "db/schema.sql"
    gen:
      go:
        package: "db"
        out: "db/sqlc"
        emit_json_tags: true
        emit_prepared_queries: false
        emit_interface: true
        emit_exact_table_names: false
```
