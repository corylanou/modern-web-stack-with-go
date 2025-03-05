# Implementation Details

## Database Schema

We'll create a simple schema for our tasks:

```sql
-- db/schema.sql
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT,
    completed BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

## SQLC Queries

```sql
-- db/queries.sql
-- name: GetTask :one
SELECT * FROM tasks
WHERE id = ? LIMIT 1;

-- name: ListTasks :many
SELECT * FROM tasks
ORDER BY created_at DESC;

-- name: ListTasksByStatus :many
SELECT * FROM tasks
WHERE completed = ?
ORDER BY created_at DESC;

-- name: CreateTask :one
INSERT INTO tasks (
  title, description
) VALUES (
  ?, ?
)
RETURNING *;

-- name: UpdateTask :one
UPDATE tasks
SET title = ?,
    description = ?,
    completed = ?,
    updated_at = CURRENT_TIMESTAMP
WHERE id = ?
RETURNING *;

-- name: DeleteTask :exec
DELETE FROM tasks
WHERE id = ?;
```

## Main Application

```go
// main.go
package main

import (
 "database/sql"
 "log"
 "net/http"
 "os"

 "github.com/yourusername/modern-go-web/handlers"
 _ "github.com/mattn/go-sqlite3"
)

func main() {
 // Create an in-memory SQLite database
 db, err := sql.Open("sqlite3", ":memory:")
 if err != nil {
  log.Fatal("Failed to connect to database:", err)
 }
 defer db.Close()

 // Initialize schema
 initSchema(db)

 // Set up routes
 mux := http.NewServeMux()
 handlers.RegisterRoutes(mux, db)

 // Serve static files
 mux.Handle("/static/", http.StripPrefix("/static/", http.FileServer(http.Dir("static"))))

 // Start server
 port := os.Getenv("PORT")
 if port == "" {
  port = "8080"
 }
 log.Printf("Server starting on http://localhost:%s", port)
 if err := http.ListenAndServe(":"+port, mux); err != nil {
  log.Fatal("Server failed to start:", err)
 }
}

func initSchema(db *sql.DB) {
 // Read schema SQL
 schema := `
 CREATE TABLE tasks (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  description TEXT,
  completed BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
 );
 
 -- Insert some sample data
 INSERT INTO tasks (title, description) VALUES 
  ('Learn Go', 'Complete the Go tour and build a simple project'),
  ('Learn Templ', 'Create components with Templ'),
  ('Build with HTMX', 'Use HTMX for dynamic interactions');
 `
 
 if _, err := db.Exec(schema); err != nil {
  log.Fatal("Failed to initialize schema:", err)
 }
 log.Println("Database schema initialized with sample data")
}
```

## Routes and Handlers

```go
// handlers/routes.go
package handlers

import (
 "database/sql"
 "net/http"

 "github.com/yourusername/modern-go-web/db/sqlc"
)

func RegisterRoutes(mux *http.ServeMux, db *sql.DB) {
 // Create a queries object
 queries := sqlc.New(db)
 
 // Task handlers
 taskHandler := NewTaskHandler(queries)
 
 // Routes
 mux.HandleFunc("GET /", taskHandler.ListTasks)
 mux.HandleFunc("GET /tasks", taskHandler.ListTasks)
 mux.HandleFunc("POST /tasks", taskHandler.CreateTask)
 mux.HandleFunc("GET /tasks/{id}", taskHandler.GetTask)
 mux.HandleFunc("PUT /tasks/{id}", taskHandler.UpdateTask)
 mux.HandleFunc("DELETE /tasks/{id}", taskHandler.DeleteTask)
 mux.HandleFunc("POST /tasks/{id}/toggle", taskHandler.ToggleTaskStatus)
}
```

## Basic Templ Component

```go
// templates/components/task_item.templ
package components

import "github.com/yourusername/modern-go-web/db/sqlc"

templ TaskItem(task sqlc.Task) {
 <div class="task-item border rounded p-4 mb-2" id={ "task-" + fmt.Sprint(task.ID) }>
  <div class="flex items-center justify-between">
   <div class="flex items-center">
    <input 
     type="checkbox" 
     checked?={ task.Completed }
     class="mr-2 h-5 w-5"
     hx-post={ fmt.Sprintf("/tasks/%d/toggle", task.ID) }
     hx-swap="outerHTML"
     hx-target={ "#task-" + fmt.Sprint(task.ID) }
    />
    <h3 class={ "text-lg", templ.KV("line-through", task.Completed) }>{ task.Title }</h3>
   </div>
   <div class="flex">
    <button 
     class="text-blue-500 mr-2"
     hx-get={ fmt.Sprintf("/tasks/%d", task.ID) }
     hx-target="#edit-form-container"
    >
     Edit
    </button>
    <button 
     class="text-red-500"
     hx-delete={ fmt.Sprintf("/tasks/%d", task.ID) }
     hx-swap="outerHTML"
     hx-confirm="Are you sure you want to delete this task?"
    >
     Delete
    </button>
   </div>
  </div>
  if task.Description.Valid {
   <p class="mt-2 text-gray-600">{ task.Description.String }</p>
  }
 </div>
}
```

## HTMX Interactions

```go
// handlers/tasks.go (excerpt)
func (h *TaskHandler) ToggleTaskStatus(w http.ResponseWriter, r *http.Request) {
 idStr := r.PathValue("id")
 id, err := strconv.ParseInt(idStr, 10, 64)
 if err != nil {
  http.Error(w, "Invalid task ID", http.StatusBadRequest)
  return
 }

 // Get current task
 task, err := h.queries.GetTask(r.Context(), id)
 if err != nil {
  http.Error(w, "Task not found", http.StatusNotFound)
  return
 }

 // Toggle completed status
 updatedTask, err := h.queries.UpdateTask(r.Context(), sqlc.UpdateTaskParams{
  ID:          id,
  Title:       task.Title,
  Description: task.Description,
  Completed:   !task.Completed,
 })
 if err != nil {
  http.Error(w, "Failed to update task", http.StatusInternalServerError)
  return
 }

 // Render the updated task item
 components.TaskItem(updatedTask).Render(r.Context(), w)
}
```
