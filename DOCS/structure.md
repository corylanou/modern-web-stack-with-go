# Project Structure

Our task manager application will have the following structure:

```
modern-go-web/
├── .air.toml                 # Air configuration for hot reloading
├── db/
│   ├── schema.sql            # Database schema definition
│   ├── queries.sql           # SQL queries for SQLC
│   └── sqlc/                 # Generated Go code from SQLC
├── handlers/
│   ├── tasks.go              # HTTP handlers for tasks
│   └── routes.go             # Route definitions
├── main.go                   # Application entry point
├── go.mod                    # Go module file
├── go.sum                    # Go module checksum
├── package.json              # NPM package configuration
├── sqlc.yaml                 # SQLC configuration
├── static/
│   ├── css/
│   │   ├── input.css         # Tailwind CSS input
│   │   └── output.css        # Generated Tailwind CSS
│   └── js/
│       └── htmx.min.js       # HTMX library
└── templates/
    ├── base.templ            # Base layout template
    ├── components/
    │   ├── task_form.templ   # Task creation/edit form
    │   └── task_item.templ   # Individual task component
    └── pages/
        ├── home.templ        # Home page with task list
        └── error.templ       # Error page
```

This structure follows best practices for organizing a modern Go web application:

1. **Separation of concerns**: Database, handlers, templates, and static files are kept separate
2. **Feature-based organization**: Components and templates are organized by feature
3. **Simple to navigate**: Clear folder naming and hierarchy
4. **Scalable**: Easy to add new features or expand existing ones
