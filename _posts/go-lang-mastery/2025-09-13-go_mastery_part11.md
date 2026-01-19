---
title: "Complete Go Mastery Part 11: Web & Frontend Integration"
date: 2025-01-13 12:00:00 +0530
categories: [Programming, Go, Web-Development]
tags: [golang, go, web, frontend, htmx, webassembly, graphql, ssr, websockets]
---

# Part 11: Web & Frontend Integration

## Overview

This part explores modern web development with Go, covering server-side rendering, HTMX for reactive UIs, WebAssembly for running Go in the browser, GraphQL APIs, and advanced WebSocket patterns. You'll learn how to build full-stack applications using Go on both the backend and frontend.

**What You'll Learn:**
- Advanced server-side rendering with html/template
- Building reactive UIs with HTMX (no JavaScript required)
- WebAssembly: Running Go code in the browser
- GraphQL APIs with gqlgen
- Advanced WebSocket patterns for real-time applications
- Server-Sent Events for live updates
- Progressive Web Apps with Go
- API versioning strategies

**Prerequisites:**
- Part 5 (API Design basics)
- Part 6 (Logging and monitoring)
- Understanding of HTTP and web fundamentals

---

## 11.1 Server-Side Rendering (SSR)

### What is Server-Side Rendering?

Server-Side Rendering means generating HTML on the server and sending complete pages to the client, as opposed to Single Page Applications (SPAs) where JavaScript builds the UI client-side.

**Benefits of SSR:**
- Better SEO (search engines see full content)
- Faster initial page load
- Works without JavaScript
- Lower client-side complexity
- Progressive enhancement

**Go's Strengths for SSR:**
- Fast template rendering
- Strong standard library (html/template)
- Simple deployment (single binary)
- Excellent performance

### html/template Deep Dive

The `html/template` package provides automatic HTML escaping and powerful template features.

#### Template Syntax Basics

```go
package main

import (
    "html/template"
    "os"
)

func main() {
    // Basic template
    tmpl := template.Must(template.New("hello").Parse(`
        <h1>Hello, {{.Name}}!</h1>
        <p>You are {{.Age}} years old.</p>
    `))
    
    data := struct {
        Name string
        Age  int
    }{"Alice", 30}
    
    tmpl.Execute(os.Stdout, data)
}
```

#### Template Actions

```go
// If/Else
{{if .IsLoggedIn}}
    <p>Welcome back, {{.Username}}!</p>
{{else}}
    <p>Please log in.</p>
{{end}}

// Range (loops)
{{range .Items}}
    <li>{{.Name}}: ${{.Price}}</li>
{{end}}

// With (change context)
{{with .User}}
    <p>User: {{.Name}}</p>
    <p>Email: {{.Email}}</p>
{{end}}

// Variables
{{$total := 0}}
{{range .Items}}
    {{$total = add $total .Price}}
{{end}}
<p>Total: ${{$total}}</p>
```

#### Template Functions

**Built-in Functions:**
```go
// Comparison
{{if eq .Status "active"}}Active{{end}}
{{if ne .Count 0}}Has items{{end}}
{{if lt .Age 18}}Minor{{end}}
{{if gt .Score 90}}Excellent{{end}}

// Logic
{{if and .IsLoggedIn .HasPermission}}Show admin{{end}}
{{if or .IsAdmin .IsModerator}}Can moderate{{end}}
{{if not .IsDisabled}}Enabled{{end}}

// String functions
{{printf "Hello %s" .Name}}
{{print .Value}}
{{println .Message}}

// Indexing
{{index .Slice 0}}
{{index .Map "key"}}

// Length
{{len .Items}}
```

**Custom Functions:**
```go
package main

import (
    "html/template"
    "strings"
    "time"
)

// Custom function map
var funcMap = template.FuncMap{
    "upper": strings.ToUpper,
    "lower": strings.ToLower,
    "formatDate": func(t time.Time) string {
        return t.Format("2006-01-02")
    },
    "add": func(a, b int) int {
        return a + b
    },
    "multiply": func(a, b int) int {
        return a * b
    },
    "truncate": func(s string, length int) string {
        if len(s) > length {
            return s[:length] + "..."
        }
        return s
    },
}

func main() {
    tmpl := template.Must(template.New("custom").Funcs(funcMap).Parse(`
        <h1>{{upper .Title}}</h1>
        <p>{{truncate .Description 100}}</p>
        <time>{{formatDate .CreatedAt}}</time>
        <p>Total: {{add .Price .Tax}}</p>
    `))
    
    data := struct {
        Title       string
        Description string
        CreatedAt   time.Time
        Price       int
        Tax         int
    }{
        Title:       "Product Name",
        Description: "This is a very long description that should be truncated...",
        CreatedAt:   time.Now(),
        Price:       100,
        Tax:         10,
    }
    
    tmpl.Execute(os.Stdout, data)
}
```

#### Template Inheritance and Composition

**Base Layout:**
```go
// templates/base.html
{{define "base"}}
<!DOCTYPE html>
<html>
<head>
    <title>{{block "title" .}}Default Title{{end}}</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    {{block "head" .}}{{end}}
</head>
<body>
    <header>
        {{template "header" .}}
    </header>
    
    <main>
        {{block "content" .}}Default Content{{end}}
    </main>
    
    <footer>
        {{template "footer" .}}
    </footer>
    
    {{block "scripts" .}}{{end}}
</body>
</html>
{{end}}

{{define "header"}}
<nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
</nav>
{{end}}

{{define "footer"}}
<p>&copy; 2025 My Website</p>
{{end}}
```

**Page Template:**
```go
// templates/home.html
{{template "base" .}}

{{define "title"}}Home - My Website{{end}}

{{define "content"}}
<h1>Welcome to My Website</h1>
<p>This is the home page.</p>

{{range .Posts}}
    <article>
        <h2>{{.Title}}</h2>
        <p>{{.Summary}}</p>
        <a href="/post/{{.ID}}">Read more</a>
    </article>
{{end}}
{{end}}

{{define "scripts"}}
<script src="/static/home.js"></script>
{{end}}
```

**Using Templates:**
```go
package main

import (
    "html/template"
    "net/http"
)

var templates *template.Template

func init() {
    // Parse all templates
    templates = template.Must(template.ParseGlob("templates/*.html"))
}

type Post struct {
    ID      int
    Title   string
    Summary string
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    data := struct {
        Posts []Post
    }{
        Posts: []Post{
            {1, "First Post", "Summary of first post"},
            {2, "Second Post", "Summary of second post"},
        },
    }
    
    // Execute the home template
    if err := templates.ExecuteTemplate(w, "home.html", data); err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
}

func main() {
    http.HandleFunc("/", homeHandler)
    http.ListenAndServe(":8080", nil)
}
```

#### Partial Templates (Components)

```go
// templates/components/card.html
{{define "card"}}
<div class="card">
    <h3>{{.Title}}</h3>
    <p>{{.Content}}</p>
    {{if .Link}}
        <a href="{{.Link}}">Learn more</a>
    {{end}}
</div>
{{end}}

// Usage in another template
{{range .Cards}}
    {{template "card" .}}
{{end}}
```

#### Template Pipelines

Pipelines allow chaining operations:

```go
// Chain functions
{{.Name | upper | truncate 20}}

// Use with if
{{if .Title | len | gt 0}}
    <h1>{{.Title}}</h1>
{{end}}

// Multiple arguments
{{.Price | multiply 1.1 | printf "%.2f"}}
```

### Template Security

**Automatic HTML Escaping:**
```go
data := struct {
    UserInput string
}{"<script>alert('XSS')</script>"}

tmpl := template.Must(template.New("safe").Parse(`
    <p>{{.UserInput}}</p>
`))

// Output: <p>&lt;script&gt;alert(&#39;XSS&#39;)&lt;/script&gt;</p>
// The script is escaped and won't execute
```

**When You Need Unescaped HTML:**
```go
import "html/template"

data := struct {
    SafeHTML template.HTML
}{
    SafeHTML: template.HTML("<strong>This is safe HTML</strong>"),
}

tmpl := template.Must(template.New("html").Parse(`
    <div>{{.SafeHTML}}</div>
`))

// Output: <div><strong>This is safe HTML</strong></div>
// Only use template.HTML for content you control!
```

**Other Safe Types:**
```go
template.HTML      // Safe HTML
template.HTMLAttr  // Safe HTML attribute
template.CSS       // Safe CSS
template.JS        // Safe JavaScript
template.JSStr     // Safe JavaScript string
template.URL       // Safe URL
```

### Template Performance

**Caching Parsed Templates:**
```go
type TemplateCache struct {
    templates map[string]*template.Template
    mu        sync.RWMutex
}

func NewTemplateCache() *TemplateCache {
    return &TemplateCache{
        templates: make(map[string]*template.Template),
    }
}

func (tc *TemplateCache) Get(name string) (*template.Template, error) {
    tc.mu.RLock()
    tmpl, exists := tc.templates[name]
    tc.mu.RUnlock()
    
    if exists {
        return tmpl, nil
    }
    
    tc.mu.Lock()
    defer tc.mu.Unlock()
    
    // Double-check after acquiring write lock
    if tmpl, exists := tc.templates[name]; exists {
        return tmpl, nil
    }
    
    // Parse template
    tmpl, err := template.ParseFiles("templates/" + name)
    if err != nil {
        return nil, err
    }
    
    tc.templates[name] = tmpl
    return tmpl, nil
}
```

**Pre-rendering Content:**
```go
// For static content that doesn't change often
var renderedHomePage string
var homePageOnce sync.Once

func getHomePage() string {
    homePageOnce.Do(func() {
        var buf bytes.Buffer
        tmpl.Execute(&buf, homeData)
        renderedHomePage = buf.String()
    })
    return renderedHomePage
}
```

### Modern SSR Frameworks

#### Templ - Type-Safe HTML Templates

Templ generates Go code from template files, providing type safety and better IDE support.

**Installation:**
```bash
go install github.com/a-h/templ/cmd/templ@latest
```

**Template File (page.templ):**
```templ
package templates

templ Page(title string, content string) {
    <!DOCTYPE html>
    <html>
        <head>
            <title>{ title }</title>
        </head>
        <body>
            <h1>{ title }</h1>
            <div>{ content }</div>
        </body>
    </html>
}

templ Card(title string, description string) {
    <div class="card">
        <h3>{ title }</h3>
        <p>{ description }</p>
    </div>
}
```

**Generate Go code:**
```bash
templ generate
```

**Using in Go:**
```go
package main

import (
    "net/http"
    "myapp/templates"
)

func handler(w http.ResponseWriter, r *http.Request) {
    component := templates.Page("My Page", "Welcome to my page!")
    component.Render(r.Context(), w)
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

**Benefits:**
- Type safety (compile-time errors)
- IDE autocomplete
- Better refactoring support
- No runtime template parsing
- Faster rendering

#### Gomponents - HTML in Go

Build HTML using Go functions:

```go
package main

import (
    "net/http"
    
    g "github.com/maragudk/gomponents"
    . "github.com/maragudk/gomponents/html"
)

func Page(title string, content g.Node) g.Node {
    return HTML(
        Lang("en"),
        Head(
            TitleEl(g.Text(title)),
            Meta(Charset("UTF-8")),
        ),
        Body(
            H1(g.Text(title)),
            Div(content),
        ),
    )
}

func Card(title, desc string) g.Node {
    return Div(
        Class("card"),
        H3(g.Text(title)),
        P(g.Text(desc)),
    )
}

func handler(w http.ResponseWriter, r *http.Request) {
    page := Page("My Page", Div(
        Card("Card 1", "Description 1"),
        Card("Card 2", "Description 2"),
    ))
    
    page.Render(w)
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

**Benefits:**
- Pure Go (no separate template language)
- Full IDE support
- Easy to test
- Component composition
- No escaping issues (handled automatically)

### HTML Streaming

Stream HTML to the client for faster perceived load times:

```go
func streamHandler(w http.ResponseWriter, r *http.Request) {
    // Set headers for streaming
    w.Header().Set("Content-Type", "text/html; charset=utf-8")
    w.Header().Set("Transfer-Encoding", "chunked")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    // Send header immediately
    fmt.Fprint(w, `<!DOCTYPE html>
    <html>
    <head><title>Streaming Demo</title></head>
    <body>
    <h1>Loading content...</h1>
    `)
    flusher.Flush()
    
    // Simulate loading data
    time.Sleep(500 * time.Millisecond)
    
    // Send first chunk
    fmt.Fprint(w, `<div id="content1">
        <p>First chunk loaded</p>
    </div>
    `)
    flusher.Flush()
    
    // Load more data
    time.Sleep(500 * time.Millisecond)
    
    // Send second chunk
    fmt.Fprint(w, `<div id="content2">
        <p>Second chunk loaded</p>
    </div>
    `)
    flusher.Flush()
    
    // Close HTML
    fmt.Fprint(w, `</body></html>`)
}
```

### Progressive Enhancement

Start with HTML that works without JavaScript, then enhance with JavaScript:

```go
func searchHandler(w http.ResponseWriter, r *http.Request) {
    query := r.URL.Query().Get("q")
    
    // Works without JavaScript (full page reload)
    results := performSearch(query)
    
    // Check if request is from HTMX/AJAX
    if r.Header.Get("HX-Request") == "true" {
        // Return only the results HTML
        renderPartial(w, "results", results)
        return
    }
    
    // Return full page for regular requests
    renderPage(w, "search", map[string]interface{}{
        "Query":   query,
        "Results": results,
    })
}
```

---

## 11.2 HTMX Integration

### What is HTMX?

HTMX allows you to access modern browser features (AJAX, CSS Transitions, WebSockets, Server Sent Events) directly in HTML, without writing JavaScript.

**Core Idea:** Instead of building a JSON API and handling everything client-side with JavaScript, you build HTML-over-the-wire endpoints and use HTMX attributes to make them dynamic.

**Benefits:**
- Reduce JavaScript code (often to zero)
- Simpler architecture
- Better for SEO
- Progressive enhancement
- Faster development
- Smaller bundle size

### HTMX Fundamentals

**Basic Setup:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>HTMX Demo</title>
    <!-- Include HTMX -->
    <script src="https://unpkg.com/htmx.org@1.9.10"></script>
</head>
<body>
    <!-- Your content -->
</body>
</html>
```

**Core Attributes:**
- `hx-get`: Issue GET request
- `hx-post`: Issue POST request
- `hx-put`: Issue PUT request
- `hx-delete`: Issue DELETE request
- `hx-trigger`: When to send request
- `hx-target`: Where to put response
- `hx-swap`: How to swap content

### HTMX with Go Examples

#### Simple Click-to-Load

**HTML:**
```html
<button hx-get="/load-more" 
        hx-target="#content" 
        hx-swap="beforeend">
    Load More
</button>

<div id="content">
    <!-- Initial content -->
</div>
```

**Go Handler:**
```go
func loadMoreHandler(w http.ResponseWriter, r *http.Request) {
    // Return HTML fragment
    fmt.Fprint(w, `
        <div class="item">
            <h3>New Item</h3>
            <p>This was loaded via HTMX!</p>
        </div>
    `)
}

func main() {
    http.HandleFunc("/load-more", loadMoreHandler)
    http.ListenAndServe(":8080", nil)
}
```

#### Live Search

**HTML:**
```html
<input type="search" 
       name="q" 
       hx-get="/search" 
       hx-trigger="keyup changed delay:300ms"
       hx-target="#results">

<div id="results">
    <!-- Search results appear here -->
</div>
```

**Go Handler:**
```go
func searchHandler(w http.ResponseWriter, r *http.Request) {
    query := r.URL.Query().Get("q")
    
    if query == "" {
        w.Write([]byte("<p>Start typing to search...</p>"))
        return
    }
    
    results := performSearch(query)
    
    var html strings.Builder
    for _, result := range results {
        html.WriteString(fmt.Sprintf(`
            <div class="result">
                <h4>%s</h4>
                <p>%s</p>
            </div>
        `, result.Title, result.Description))
    }
    
    w.Write([]byte(html.String()))
}
```

#### Form Submission

**HTML:**
```html
<form hx-post="/contact" 
      hx-target="#message"
      hx-swap="innerHTML">
    <input type="text" name="name" placeholder="Name" required>
    <input type="email" name="email" placeholder="Email" required>
    <textarea name="message" placeholder="Message" required></textarea>
    <button type="submit">Send</button>
</form>

<div id="message"></div>
```

**Go Handler:**
```go
func contactHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    // Parse form
    if err := r.ParseForm(); err != nil {
        fmt.Fprint(w, `<p class="error">Invalid form data</p>`)
        return
    }
    
    name := r.FormValue("name")
    email := r.FormValue("email")
    message := r.FormValue("message")
    
    // Validate
    if name == "" || email == "" || message == "" {
        fmt.Fprint(w, `<p class="error">All fields are required</p>`)
        return
    }
    
    // Process (send email, save to DB, etc.)
    err := sendContactEmail(name, email, message)
    if err != nil {
        fmt.Fprint(w, `<p class="error">Failed to send message. Please try again.</p>`)
        return
    }
    
    // Success
    fmt.Fprint(w, `<p class="success">Thank you! Your message has been sent.</p>`)
}
```

#### Infinite Scroll

**HTML:**
```html
<div id="items">
    {{range .Items}}
        <div class="item">{{.Title}}</div>
    {{end}}
</div>

<div hx-get="/items?page=2" 
     hx-trigger="revealed"
     hx-swap="afterend">
    <p>Loading more...</p>
</div>
```

**Go Handler:**
```go
func itemsHandler(w http.ResponseWriter, r *http.Request) {
    page := r.URL.Query().Get("page")
    pageNum, _ := strconv.Atoi(page)
    
    items := getItems(pageNum, 20) // Get 20 items for this page
    
    var html strings.Builder
    for _, item := range items {
        html.WriteString(fmt.Sprintf(`<div class="item">%s</div>`, item.Title))
    }
    
    // Add next page loader if there are more items
    if hasMoreItems(pageNum) {
        html.WriteString(fmt.Sprintf(`
            <div hx-get="/items?page=%d" 
                 hx-trigger="revealed"
                 hx-swap="afterend">
                <p>Loading more...</p>
            </div>
        `, pageNum+1))
    }
    
    w.Write([]byte(html.String()))
}
```

### Advanced HTMX Patterns

#### Polling for Updates

```html
<div hx-get="/notifications" 
     hx-trigger="every 5s"
     hx-swap="innerHTML">
    <p>Checking for notifications...</p>
</div>
```

```go
func notificationsHandler(w http.ResponseWriter, r *http.Request) {
    notifications := getUnreadNotifications()
    
    if len(notifications) == 0 {
        fmt.Fprint(w, `<p>No new notifications</p>`)
        return
    }
    
    var html strings.Builder
    for _, notif := range notifications {
        html.WriteString(fmt.Sprintf(`
            <div class="notification">%s</div>
        `, notif.Message))
    }
    
    w.Write([]byte(html.String()))
}
```

#### Optimistic UI Updates

```html
<button hx-post="/like" 
        hx-swap="outerHTML"
        hx-disabled-elt="this">
    ❤️ Like
</button>
```

```go
func likeHandler(w http.ResponseWriter, r *http.Request) {
    // Perform like action
    err := addLike(getCurrentUser(r))
    
    if err != nil {
        // Return original button on error
        fmt.Fprint(w, `
            <button hx-post="/like" hx-swap="outerHTML">
                ❤️ Like
            </button>
        `)
        return
    }
    
    // Return updated state
    fmt.Fprint(w, `
        <button hx-delete="/unlike" hx-swap="outerHTML" class="liked">
            💙 Liked
        </button>
    `)
}
```

#### Out-of-Band Swaps (OOB)

Update multiple parts of the page from one request:

```html
<div id="main-content">
    <button hx-get="/update-multiple" hx-target="#main-content">
        Update Multiple
    </button>
</div>

<div id="sidebar">
    Current count: <span id="count">0</span>
</div>
```

```go
func updateMultipleHandler(w http.ResponseWriter, r *http.Request) {
    // Main content update
    mainContent := `<div id="main-content">
        <p>Main content updated!</p>
        <button hx-get="/update-multiple" hx-target="#main-content">
            Update Multiple
        </button>
    </div>`
    
    // Out-of-band update for sidebar
    oobContent := `
        <div id="count" hx-swap-oob="true">42</div>
    `
    
    fmt.Fprint(w, mainContent + oobContent)
}
```

#### Modal Dialogs

```html
<button hx-get="/modal/edit-profile" 
        hx-target="body" 
        hx-swap="beforeend">
    Edit Profile
</button>
```

```go
func modalHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, `
        <div id="modal" class="modal-overlay">
            <div class="modal-content">
                <h2>Edit Profile</h2>
                <form hx-post="/profile/update" 
                      hx-target="#modal" 
                      hx-swap="outerHTML">
                    <input type="text" name="name" placeholder="Name">
                    <input type="email" name="email" placeholder="Email">
                    <button type="submit">Save</button>
                    <button type="button" 
                            onclick="document.getElementById('modal').remove()">
                        Cancel
                    </button>
                </form>
            </div>
        </div>
    `)
}
```

### HTMX Response Headers

Go can send special HTMX response headers:

```go
func handler(w http.ResponseWriter, r *http.Request) {
    // Trigger client-side event
    w.Header().Set("HX-Trigger", "itemUpdated")
    
    // Redirect (HTMX will follow)
    w.Header().Set("HX-Redirect", "/dashboard")
    
    // Refresh the page
    w.Header().Set("HX-Refresh", "true")
    
    // Push URL to browser history
    w.Header().Set("HX-Push-Url", "/new-url")
    
    // Replace URL in browser history
    w.Header().Set("HX-Replace-Url", "/replace-url")
    
    // Retarget the response
    w.Header().Set("HX-Retarget", "#different-element")
    
    // Change swap method
    w.Header().Set("HX-Reswap", "beforeend")
    
    fmt.Fprint(w, "<p>Updated content</p>")
}
```

### HTMX Request Headers

Check if request is from HTMX:

```go
func handler(w http.ResponseWriter, r *http.Request) {
    // Check if HTMX request
    isHTMX := r.Header.Get("HX-Request") == "true"
    
    // Get current URL
    currentURL := r.Header.Get("HX-Current-URL")
    
    // Get target element
    target := r.Header.Get("HX-Target")
    
    // Get trigger element
    trigger := r.Header.Get("HX-Trigger")
    
    if isHTMX {
        // Return HTML fragment
        fmt.Fprint(w, "<div>Fragment for HTMX</div>")
    } else {
        // Return full page
        renderFullPage(w, data)
    }
}
```

### Form Validation with HTMX

```html
<form hx-post="/validate-email" 
      hx-target="#email-error" 
      hx-trigger="change from:#email">
    <input type="email" 
           id="email" 
           name="email" 
           placeholder="Email">
    <span id="email-error"></span>
    
    <button type="submit">Submit</button>
</form>
```

```go
func validateEmailHandler(w http.ResponseWriter, r *http.Request) {
    email := r.FormValue("email")
    
    if email == "" {
        w.Write([]byte(""))
        return
    }
    
    if !isValidEmail(email) {
        fmt.Fprint(w, `<span class="error">Invalid email address</span>`)
        return
    }
    
    // Check if email already exists
    if emailExists(email) {
        fmt.Fprint(w, `<span class="error">Email already registered</span>`)
        return
    }
    
    fmt.Fprint(w, `<span class="success">✓ Email available</span>`)
}
```

### HTMX Best Practices

**1. Progressive Enhancement:**
```go
func handler(w http.ResponseWriter, r *http.Request) {
    data := getData()
    
    if r.Header.Get("HX-Request") == "true" {
        // HTMX request: return fragment
        renderFragment(w, data)
    } else {
        // Regular request: return full page
        renderFullPage(w, data)
    }
}
```

**2. Use Appropriate HTTP Methods:**
```go
// GET for fetching data (idempotent)
http.HandleFunc("/items", getItemsHandler)

// POST for creating
http.HandleFunc("/items", createItemHandler)

// PUT for updating
http.HandleFunc("/items/{id}", updateItemHandler)

// DELETE for deleting
http.HandleFunc("/items/{id}", deleteItemHandler)
```

**3. Return Appropriate Status Codes:**
```go
func createHandler(w http.ResponseWriter, r *http.Request) {
    item, err := createItem(r)
    if err != nil {
        w.WriteHeader(http.StatusBadRequest)
        fmt.Fprint(w, `<p class="error">Failed to create item</p>`)
        return
    }
    
    w.WriteHeader(http.StatusCreated)
    renderItem(w, item)
}
```

**4. Handle Errors Gracefully:**
```go
func handler(w http.ResponseWriter, r *http.Request) {
    data, err := fetchData()
    if err != nil {
        // Return user-friendly error HTML
        fmt.Fprint(w, `
            <div class="error-message">
                <p>Sorry, something went wrong.</p>
                <button hx-get="/retry" hx-swap="outerHTML">
                    Try Again
                </button>
            </div>
        `)
        return
    }
    
    renderData(w, data)
}
```

**5. Security with HTMX:**
```go
// CSRF protection
func handler(w http.ResponseWriter, r *http.Request) {
    // Verify CSRF token
    token := r.Header.Get("X-CSRF-Token")
    if !validCSRFToken(token) {
        w.WriteHeader(http.StatusForbidden)
        fmt.Fprint(w, "<p>Invalid CSRF token</p>")
        return
    }
    
    // Process request...
}

// Include CSRF token in responses
fmt.Fprintf(w, `
    <form hx-post="/submit" hx-headers='{"X-CSRF-Token": "%s"}'>
        <!-- form fields -->
    </form>
`, csrfToken)
```

### Complete HTMX Example: Todo Application

```go
package main

import (
    "fmt"
    "html/template"
    "net/http"
    "strconv"
    "sync"
)

type Todo struct {
    ID        int
    Title     string
    Completed bool
}

var (
    todos   = make(map[int]*Todo)
    nextID  = 1
    todosMu sync.RWMutex
)

func main() {
    http.HandleFunc("/", homeHandler)
    http.HandleFunc("/todos", todosHandler)
    http.HandleFunc("/todos/add", addTodoHandler)
    http.HandleFunc("/todos/toggle/", toggleTodoHandler)
    http.HandleFunc("/todos/delete/", deleteTodoHandler)
    
    fmt.Println("Server starting on :8080")
    http.ListenAndServe(":8080", nil)
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    tmpl := `
    <!DOCTYPE html>
    <html>
    <head>
        <title>HTMX Todo App</title>
        <script src="https://unpkg.com/htmx.org@1.9.10"></script>
        <style>
            body { font-family: Arial, sans-serif; max-width: 600px; margin: 50px auto; }
            .todo { padding: 10px; border-bottom: 1px solid #ddd; }
            .completed { text-decoration: line-through; color: #888; }
            button { margin-left: 10px; }
        </style>
    </head>
    <body>
        <h1>Todo List</h1>
        
        <form hx-post="/todos/add" hx-target="#todos" hx-swap="afterbegin">
            <input type="text" name="title" placeholder="New todo" required>
            <button type="submit">Add</button>
        </form>
        
        <div id="todos" hx-get="/todos" hx-trigger="load"></div>
    </body>
    </html>
    `
    
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, tmpl)
}

func todosHandler(w http.ResponseWriter, r *http.Request) {
    todosMu.RLock()
    defer todosMu.RUnlock()
    
    for _, todo := range todos {
        renderTodo(w, todo)
    }
}

func addTodoHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    title := r.FormValue("title")
    if title == "" {
        http.Error(w, "Title required", http.StatusBadRequest)
        return
    }
    
    todosMu.Lock()
    todo := &Todo{
        ID:        nextID,
        Title:     title,
        Completed: false,
    }
    todos[nextID] = todo
    nextID++
    todosMu.Unlock()
    
    renderTodo(w, todo)
}

func toggleTodoHandler(w http.ResponseWriter, r *http.Request) {
    idStr := r.URL.Path[len("/todos/toggle/"):]
    id, _ := strconv.Atoi(idStr)
    
    todosMu.Lock()
    defer todosMu.Unlock()
    
    todo, exists := todos[id]
    if !exists {
        http.Error(w, "Todo not found", http.StatusNotFound)
        return
    }
    
    todo.Completed = !todo.Completed
    renderTodo(w, todo)
}

func deleteTodoHandler(w http.ResponseWriter, r *http.Request) {
    idStr := r.URL.Path[len("/todos/delete/"):]
    id, _ := strconv.Atoi(idStr)
    
    todosMu.Lock()
    defer todosMu.Unlock()
    
    delete(todos, id)
    // Return empty response (todo will be removed from DOM)
}

func renderTodo(w http.ResponseWriter, todo *Todo) {
    class := ""
    if todo.Completed {
        class = "completed"
    }
    
    fmt.Fprintf(w, `
        <div id="todo-%d" class="todo %s">
            <input type="checkbox" 
                   hx-post="/todos/toggle/%d" 
                   hx-target="#todo-%d" 
                   hx-swap="outerHTML"
                   %s>
            <span>%s</span>
            <button hx-delete="/todos/delete/%d" 
                    hx-target="#todo-%d" 
                    hx-swap="outerHTML swap:1s">
                Delete
            </button>
        </div>
    `, todo.ID, class, todo.ID, todo.ID, 
       map[bool]string{true: "checked", false: ""}[todo.Completed],
       template.HTMLEscapeString(todo.Title), 
       todo.ID, todo.ID)
}
```

This example demonstrates:
- Form submission without page reload
- Dynamic list updates
- Toggle state
- Delete with smooth removal
- All without writing custom JavaScript!

---

*Due to length constraints, I'll continue with the remaining sections in the next response. Shall I continue with WebSockets Advanced, WebAssembly, GraphQL, and the remaining sections?*

## 11.3 WebSockets Advanced

### WebSocket Fundamentals

WebSockets provide full-duplex communication channels over a single TCP connection, enabling real-time bidirectional data flow between client and server.

**When to Use WebSockets:**
- Real-time chat applications
- Live notifications
- Collaborative editing
- Gaming
- Live feeds (stock prices, sports scores)
- IoT device communication

**When NOT to Use WebSockets:**
- Simple request-response patterns (use HTTP)
- Unidirectional server → client updates (use SSE)
- Rare updates (use polling)

### WebSocket Libraries

**1. gorilla/websocket** - Most popular, production-ready:
```bash
go get github.com/gorilla/websocket
```

**2. nhooyr.io/websocket** - Modern, minimal API:
```bash
go get nhooyr.io/websocket
```

**3. gobwas/ws** - Low-level, high-performance:
```bash
go get github.com/gobwas/ws
```

### Basic WebSocket Server (gorilla/websocket)

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    
    "github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
    ReadBufferSize:  1024,
    WriteBufferSize: 1024,
    // Allow all origins (configure properly in production)
    CheckOrigin: func(r *http.Request) bool {
        return true
    },
}

func wsHandler(w http.ResponseWriter, r *http.Request) {
    // Upgrade HTTP connection to WebSocket
    conn, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Println("Upgrade error:", err)
        return
    }
    defer conn.Close()
    
    log.Println("Client connected")
    
    // Read/write loop
    for {
        messageType, message, err := conn.ReadMessage()
        if err != nil {
            log.Println("Read error:", err)
            break
        }
        
        log.Printf("Received: %s", message)
        
        // Echo message back
        err = conn.WriteMessage(messageType, message)
        if err != nil {
            log.Println("Write error:", err)
            break
        }
    }
    
    log.Println("Client disconnected")
}

func main() {
    http.HandleFunc("/ws", wsHandler)
    
    log.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**HTML Client:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket Demo</title>
</head>
<body>
    <h1>WebSocket Demo</h1>
    <div id="messages"></div>
    <input type="text" id="input" placeholder="Type a message">
    <button onclick="send()">Send</button>
    
    <script>
        const ws = new WebSocket('ws://localhost:8080/ws');
        const messages = document.getElementById('messages');
        const input = document.getElementById('input');
        
        ws.onopen = () => {
            console.log('Connected');
            messages.innerHTML += '<p>Connected to server</p>';
        };
        
        ws.onmessage = (event) => {
            console.log('Message:', event.data);
            messages.innerHTML += `<p>Server: ${event.data}</p>`;
        };
        
        ws.onerror = (error) => {
            console.error('WebSocket error:', error);
        };
        
        ws.onclose = () => {
            console.log('Disconnected');
            messages.innerHTML += '<p>Disconnected from server</p>';
        };
        
        function send() {
            if (input.value) {
                ws.send(input.value);
                messages.innerHTML += `<p>You: ${input.value}</p>`;
                input.value = '';
            }
        }
        
        input.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') send();
        });
    </script>
</body>
</html>
```

### Connection Management

```go
type Client struct {
    conn     *websocket.Conn
    send     chan []byte
    hub      *Hub
    userID   string
}

func (c *Client) readPump() {
    defer func() {
        c.hub.unregister <- c
        c.conn.Close()
    }()
    
    // Configure read deadline
    c.conn.SetReadDeadline(time.Now().Add(60 * time.Second))
    c.conn.SetPongHandler(func(string) error {
        c.conn.SetReadDeadline(time.Now().Add(60 * time.Second))
        return nil
    })
    
    for {
        _, message, err := c.conn.ReadMessage()
        if err != nil {
            if websocket.IsUnexpectedCloseError(err, websocket.CloseGoingAway, websocket.CloseAbnormalClosure) {
                log.Printf("error: %v", err)
            }
            break
        }
        
        // Handle message
        c.hub.broadcast <- message
    }
}

func (c *Client) writePump() {
    ticker := time.NewTicker(54 * time.Second)
    defer func() {
        ticker.Stop()
        c.conn.Close()
    }()
    
    for {
        select {
        case message, ok := <-c.send:
            c.conn.SetWriteDeadline(time.Now().Add(10 * time.Second))
            if !ok {
                c.conn.WriteMessage(websocket.CloseMessage, []byte{})
                return
            }
            
            w, err := c.conn.NextWriter(websocket.TextMessage)
            if err != nil {
                return
            }
            w.Write(message)
            
            if err := w.Close(); err != nil {
                return
            }
            
        case <-ticker.C:
            c.conn.SetWriteDeadline(time.Now().Add(10 * time.Second))
            if err := c.conn.WriteMessage(websocket.PingMessage, nil); err != nil {
                return
            }
        }
    }
}
```

### Hub Pattern for Broadcasting

```go
type Hub struct {
    clients    map[*Client]bool
    broadcast  chan []byte
    register   chan *Client
    unregister chan *Client
    mu         sync.RWMutex
}

func NewHub() *Hub {
    return &Hub{
        broadcast:  make(chan []byte),
        register:   make(chan *Client),
        unregister: make(chan *Client),
        clients:    make(map[*Client]bool),
    }
}

func (h *Hub) Run() {
    for {
        select {
        case client := <-h.register:
            h.mu.Lock()
            h.clients[client] = true
            h.mu.Unlock()
            log.Printf("Client registered. Total clients: %d", len(h.clients))
            
        case client := <-h.unregister:
            h.mu.Lock()
            if _, ok := h.clients[client]; ok {
                delete(h.clients, client)
                close(client.send)
            }
            h.mu.Unlock()
            log.Printf("Client unregistered. Total clients: %d", len(h.clients))
            
        case message := <-h.broadcast:
            h.mu.RLock()
            for client := range h.clients {
                select {
                case client.send <- message:
                default:
                    close(client.send)
                    delete(h.clients, client)
                }
            }
            h.mu.RUnlock()
        }
    }
}
```

### Room-Based Communication

```go
type Room struct {
    ID      string
    clients map[*Client]bool
    mu      sync.RWMutex
}

type Hub struct {
    rooms      map[string]*Room
    roomsMu    sync.RWMutex
    register   chan *Client
    unregister chan *Client
}

func (h *Hub) JoinRoom(client *Client, roomID string) {
    h.roomsMu.Lock()
    room, exists := h.rooms[roomID]
    if !exists {
        room = &Room{
            ID:      roomID,
            clients: make(map[*Client]bool),
        }
        h.rooms[roomID] = room
    }
    h.roomsMu.Unlock()
    
    room.mu.Lock()
    room.clients[client] = true
    room.mu.Unlock()
    
    log.Printf("Client joined room %s. Room size: %d", roomID, len(room.clients))
}

func (h *Hub) LeaveRoom(client *Client, roomID string) {
    h.roomsMu.RLock()
    room, exists := h.rooms[roomID]
    h.roomsMu.RUnlock()
    
    if !exists {
        return
    }
    
    room.mu.Lock()
    delete(room.clients, client)
    roomSize := len(room.clients)
    room.mu.Unlock()
    
    // Delete room if empty
    if roomSize == 0 {
        h.roomsMu.Lock()
        delete(h.rooms, roomID)
        h.roomsMu.Unlock()
    }
}

func (h *Hub) BroadcastToRoom(roomID string, message []byte) {
    h.roomsMu.RLock()
    room, exists := h.rooms[roomID]
    h.roomsMu.RUnlock()
    
    if !exists {
        return
    }
    
    room.mu.RLock()
    defer room.mu.RUnlock()
    
    for client := range room.clients {
        select {
        case client.send <- message:
        default:
            // Client send buffer is full
            log.Printf("Failed to send to client in room %s", roomID)
        }
    }
}
```

### Message Protocol

Define a structured message format:

```go
type Message struct {
    Type      string          `json:"type"`
    Room      string          `json:"room,omitempty"`
    From      string          `json:"from"`
    To        string          `json:"to,omitempty"`
    Content   string          `json:"content"`
    Timestamp time.Time       `json:"timestamp"`
    Metadata  json.RawMessage `json:"metadata,omitempty"`
}

func (c *Client) handleMessage(data []byte) {
    var msg Message
    if err := json.Unmarshal(data, &msg); err != nil {
        log.Printf("Invalid message format: %v", err)
        return
    }
    
    msg.From = c.userID
    msg.Timestamp = time.Now()
    
    switch msg.Type {
    case "chat":
        c.handleChatMessage(&msg)
    case "join":
        c.handleJoinRoom(&msg)
    case "leave":
        c.handleLeaveRoom(&msg)
    case "typing":
        c.handleTypingIndicator(&msg)
    default:
        log.Printf("Unknown message type: %s", msg.Type)
    }
}

func (c *Client) handleChatMessage(msg *Message) {
    // Store message in database
    if err := saveMessage(msg); err != nil {
        log.Printf("Failed to save message: %v", err)
        return
    }
    
    // Broadcast to room
    messageJSON, _ := json.Marshal(msg)
    c.hub.BroadcastToRoom(msg.Room, messageJSON)
}
```

### Scaling WebSockets with Redis

For horizontal scaling, use Redis pub/sub to synchronize messages across servers:

```go
package main

import (
    "context"
    "encoding/json"
    
    "github.com/go-redis/redis/v8"
)

type RedisHub struct {
    localHub    *Hub
    redisClient *redis.Client
    ctx         context.Context
}

func NewRedisHub(localHub *Hub, redisAddr string) *RedisHub {
    return &RedisHub{
        localHub: localHub,
        redisClient: redis.NewClient(&redis.Options{
            Addr: redisAddr,
        }),
        ctx: context.Background(),
    }
}

func (rh *RedisHub) Subscribe(channel string) {
    pubsub := rh.redisClient.Subscribe(rh.ctx, channel)
    defer pubsub.Close()
    
    ch := pubsub.Channel()
    for msg := range ch {
        // Broadcast to local clients
        rh.localHub.broadcast <- []byte(msg.Payload)
    }
}

func (rh *RedisHub) Publish(channel string, message []byte) error {
    return rh.redisClient.Publish(rh.ctx, channel, message).Err()
}

func (rh *RedisHub) BroadcastToRoom(roomID string, message []byte) {
    // Publish to Redis (all servers will receive)
    if err := rh.Publish("room:"+roomID, message); err != nil {
        log.Printf("Failed to publish to Redis: %v", err)
    }
}

func (rh *RedisHub) Run() {
    // Subscribe to Redis channels
    go rh.Subscribe("global")
    
    // Run local hub
    go rh.localHub.Run()
    
    // Forward local broadcasts to Redis
    for message := range rh.localHub.broadcast {
        rh.Publish("global", message)
    }
}
```

### Complete Chat Application

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "time"
    
    "github.com/gorilla/websocket"
)

type ChatServer struct {
    hub      *Hub
    upgrader websocket.Upgrader
}

func NewChatServer() *ChatServer {
    return &ChatServer{
        hub: NewHub(),
        upgrader: websocket.Upgrader{
            ReadBufferSize:  1024,
            WriteBufferSize: 1024,
            CheckOrigin: func(r *http.Request) bool {
                return true // Configure properly in production
            },
        },
    }
}

func (cs *ChatServer) ServeWS(w http.ResponseWriter, r *http.Request) {
    conn, err := cs.upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Println(err)
        return
    }
    
    // Get user ID from query params or auth token
    userID := r.URL.Query().Get("user")
    if userID == "" {
        userID = "anonymous"
    }
    
    client := &Client{
        conn:   conn,
        send:   make(chan []byte, 256),
        hub:    cs.hub,
        userID: userID,
    }
    
    cs.hub.register <- client
    
    // Start goroutines
    go client.writePump()
    go client.readPump()
}

func (cs *ChatServer) Start() {
    go cs.hub.Run()
    
    http.HandleFunc("/ws", cs.ServeWS)
    http.HandleFunc("/", cs.serveHome)
    
    log.Println("Chat server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}

func (cs *ChatServer) serveHome(w http.ResponseWriter, r *http.Request) {
    html := `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Chat Application</title>
        <style>
            #messages { 
                height: 400px; 
                overflow-y: scroll; 
                border: 1px solid #ccc; 
                padding: 10px;
                margin-bottom: 10px;
            }
            .message { margin: 5px 0; }
            .user { font-weight: bold; }
            .timestamp { color: #888; font-size: 0.8em; }
        </style>
    </head>
    <body>
        <h1>Chat Application</h1>
        <div id="messages"></div>
        <input type="text" id="username" placeholder="Username" value="User">
        <input type="text" id="room" placeholder="Room" value="general">
        <button onclick="joinRoom()">Join Room</button>
        <br><br>
        <input type="text" id="messageInput" placeholder="Type a message" style="width: 300px;">
        <button onclick="sendMessage()">Send</button>
        
        <script>
            let ws;
            let currentRoom = 'general';
            
            function connect() {
                const username = document.getElementById('username').value;
                ws = new WebSocket('ws://localhost:8080/ws?user=' + encodeURIComponent(username));
                
                ws.onopen = () => {
                    console.log('Connected');
                    joinRoom();
                };
                
                ws.onmessage = (event) => {
                    const msg = JSON.parse(event.data);
                    displayMessage(msg);
                };
                
                ws.onerror = (error) => {
                    console.error('WebSocket error:', error);
                };
                
                ws.onclose = () => {
                    console.log('Disconnected');
                    setTimeout(connect, 3000); // Reconnect after 3 seconds
                };
            }
            
            function joinRoom() {
                const room = document.getElementById('room').value;
                currentRoom = room;
                
                ws.send(JSON.stringify({
                    type: 'join',
                    room: room
                }));
                
                document.getElementById('messages').innerHTML = '<p>Joined room: ' + room + '</p>';
            }
            
            function sendMessage() {
                const input = document.getElementById('messageInput');
                if (!input.value) return;
                
                const message = {
                    type: 'chat',
                    room: currentRoom,
                    content: input.value
                };
                
                ws.send(JSON.stringify(message));
                input.value = '';
            }
            
            function displayMessage(msg) {
                const messages = document.getElementById('messages');
                const time = new Date(msg.timestamp).toLocaleTimeString();
                
                messages.innerHTML += \`
                    <div class="message">
                        <span class="user">\${msg.from}:</span>
                        <span class="content">\${msg.content}</span>
                        <span class="timestamp">(\${time})</span>
                    </div>
                \`;
                
                messages.scrollTop = messages.scrollHeight;
            }
            
            document.getElementById('messageInput').addEventListener('keypress', (e) => {
                if (e.key === 'Enter') sendMessage();
            });
            
            // Connect on page load
            connect();
        </script>
    </body>
    </html>
    `
    
    w.Header().Set("Content-Type", "text/html")
    w.Write([]byte(html))
}

func main() {
    server := NewChatServer()
    server.Start()
}
```

### WebSocket Best Practices

**1. Heartbeat/Ping-Pong:**
```go
// Client side keeps connection alive
ticker := time.NewTicker(54 * time.Second)
defer ticker.Stop()

for {
    select {
    case <-ticker.C:
        if err := conn.WriteMessage(websocket.PingMessage, nil); err != nil {
            return
        }
    }
}

// Server side responds to pings
conn.SetPongHandler(func(string) error {
    conn.SetReadDeadline(time.Now().Add(60 * time.Second))
    return nil
})
```

**2. Graceful Shutdown:**
```go
func (c *Client) Close() {
    // Send close message
    c.conn.WriteMessage(
        websocket.CloseMessage,
        websocket.FormatCloseMessage(websocket.CloseNormalClosure, ""),
    )
    
    // Wait for close confirmation or timeout
    select {
    case <-time.After(time.Second):
    }
    
    c.conn.Close()
}
```

**3. Message Size Limits:**
```go
const maxMessageSize = 512 * 1024 // 512 KB

conn.SetReadLimit(maxMessageSize)
```

**4. Rate Limiting:**
```go
type RateLimiter struct {
    limiter *rate.Limiter
}

func (c *Client) readPump() {
    limiter := rate.NewLimiter(10, 20) // 10 msg/sec, burst of 20
    
    for {
        if !limiter.Allow() {
            // Too many messages, slow down
            time.Sleep(100 * time.Millisecond)
            continue
        }
        
        _, message, err := c.conn.ReadMessage()
        // Process message...
    }
}
```

**5. Authentication:**
```go
func (cs *ChatServer) ServeWS(w http.ResponseWriter, r *http.Request) {
    // Verify JWT token
    token := r.Header.Get("Authorization")
    userID, err := verifyJWT(token)
    if err != nil {
        http.Error(w, "Unauthorized", http.StatusUnauthorized)
        return
    }
    
    conn, err := cs.upgrader.Upgrade(w, r, nil)
    if err != nil {
        return
    }
    
    client := &Client{
        conn:   conn,
        send:   make(chan []byte, 256),
        hub:    cs.hub,
        userID: userID,
    }
    
    // Continue with client setup...
}
```

---

## 11.4 WebAssembly (WASM) with Go

### What is WebAssembly?

WebAssembly is a binary instruction format that runs in web browsers at near-native speed. Go can compile to WebAssembly, allowing you to run Go code in the browser.

**Use Cases:**
- CPU-intensive computations in the browser
- Image/video processing
- Data compression
- Cryptography
- Game logic
- Data visualization

**Limitations:**
- Large binary size (~2-10 MB even for simple programs)
- Slower startup time compared to JavaScript
- Limited DOM access (requires syscall/js)
- No direct file system access
- Garbage collection overhead

### Compiling Go to WebAssembly

**Simple Go Program:**
```go
// main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello from WebAssembly!")
}
```

**Compile to WASM:**
```bash
GOOS=js GOARCH=wasm go build -o main.wasm main.go
```

**Copy wasm_exec.js (required for loading WASM):**
```bash
cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" .
```

**HTML File:**
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Go WASM Demo</title>
</head>
<body>
    <script src="wasm_exec.js"></script>
    <script>
        const go = new Go();
        WebAssembly.instantiateStreaming(
            fetch("main.wasm"),
            go.importObject
        ).then((result) => {
            go.run(result.instance);
        });
    </script>
</body>
</html>
```

**Serve with HTTP server:**
```bash
# Simple Python server
python3 -m http.server 8080

# Or Go server
go run server.go
```

```go
// server.go
package main

import (
    "log"
    "net/http"
)

func main() {
    http.Handle("/", http.FileServer(http.Dir(".")))
    log.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### JavaScript Interop with syscall/js

**Calling JavaScript from Go:**
```go
package main

import (
    "syscall/js"
)

func main() {
    // Get global objects
    document := js.Global().Get("document")
    window := js.Global().Get("window")
    console := js.Global().Get("console")
    
    // Call JavaScript functions
    console.Call("log", "Hello from Go!")
    
    // Alert
    window.Call("alert", "This is from Go!")
    
    // DOM manipulation
    body := document.Get("body")
    h1 := document.Call("createElement", "h1")
    h1.Set("textContent", "Hello from Go WebAssembly!")
    body.Call("appendChild", h1)
    
    // Prevent Go program from exiting
    select {}
}
```

**Exposing Go Functions to JavaScript:**
```go
package main

import (
    "syscall/js"
)

func add(this js.Value, args []js.Value) interface{} {
    if len(args) != 2 {
        return "Error: need exactly 2 arguments"
    }
    
    a := args[0].Int()
    b := args[1].Int()
    return a + b
}

func greet(this js.Value, args []js.Value) interface{} {
    if len(args) == 0 {
        return "Hello, World!"
    }
    
    name := args[0].String()
    return "Hello, " + name + "!"
}

func main() {
    // Register Go functions as global JavaScript functions
    js.Global().Set("goAdd", js.FuncOf(add))
    js.Global().Set("goGreet", js.FuncOf(greet))
    
    println("Go WebAssembly initialized!")
    
    select {} // Keep the program running
}
```

**Using from JavaScript:**
```html
<script>
    const go = new Go();
    WebAssembly.instantiateStreaming(fetch("main.wasm"), go.importObject)
        .then((result) => {
            go.run(result.instance);
            
            // Now we can call Go functions
            console.log(goAdd(5, 3));        // 8
            console.log(goGreet("Alice"));   // Hello, Alice!
        });
</script>
```

### DOM Manipulation

```go
package main

import (
    "strconv"
    "syscall/js"
)

func setupCounter() {
    count := 0
    
    document := js.Global().Get("document")
    
    // Create elements
    container := document.Call("createElement", "div")
    display := document.Call("createElement", "h1")
    display.Set("id", "counter")
    display.Set("textContent", strconv.Itoa(count))
    
    incrementBtn := document.Call("createElement", "button")
    incrementBtn.Set("textContent", "Increment")
    
    decrementBtn := document.Call("createElement", "button")
    decrementBtn.Set("textContent", "Decrement")
    
    // Add event listeners
    incrementBtn.Call("addEventListener", "click", js.FuncOf(func(this js.Value, args []js.Value) interface{} {
        count++
        display.Set("textContent", strconv.Itoa(count))
        return nil
    }))
    
    decrementBtn.Call("addEventListener", "click", js.FuncOf(func(this js.Value, args []js.Value) interface{} {
        count--
        display.Set("textContent", strconv.Itoa(count))
        return nil
    }))
    
    // Append to DOM
    container.Call("appendChild", display)
    container.Call("appendChild", incrementBtn)
    container.Call("appendChild", decrementBtn)
    document.Get("body").Call("appendChild", container)
}

func main() {
    setupCounter()
    select {}
}
```

### Promises and Async Operations

```go
package main

import (
    "syscall/js"
    "time"
)

func fetchData() js.Func {
    return js.FuncOf(func(this js.Value, args []js.Value) interface{} {
        // Create a Promise
        handler := js.FuncOf(func(this js.Value, args []js.Value) interface{} {
            resolve := args[0]
            reject := args[1]
            
            go func() {
                // Simulate async operation
                time.Sleep(2 * time.Second)
                
                // Resolve promise
                resolve.Invoke("Data loaded successfully!")
            }()
            
            return nil
        })
        
        promiseConstructor := js.Global().Get("Promise")
        return promiseConstructor.New(handler)
    })
}

func main() {
    js.Global().Set("fetchData", fetchData())
    
    select {}
}
```

**Using from JavaScript:**
```javascript
fetchData().then(result => {
    console.log(result); // "Data loaded successfully!" after 2 seconds
});
```

### Canvas Graphics

```go
package main

import (
    "math"
    "syscall/js"
    "time"
)

func drawAnimation() {
    document := js.Global().Get("document")
    canvas := document.Call("createElement", "canvas")
    canvas.Set("width", 400)
    canvas.Set("height", 400)
    document.Get("body").Call("appendChild", canvas)
    
    ctx := canvas.Call("getContext", "2d")
    
    angle := 0.0
    
    var renderFrame js.Func
    renderFrame = js.FuncOf(func(this js.Value, args []js.Value) interface{} {
        // Clear canvas
        ctx.Call("clearRect", 0, 0, 400, 400)
        
        // Draw circle
        ctx.Call("beginPath")
        x := 200 + math.Cos(angle)*100
        y := 200 + math.Sin(angle)*100
        ctx.Call("arc", x, y, 20, 0, 2*math.Pi)
        ctx.Set("fillStyle", "blue")
        ctx.Call("fill")
        
        angle += 0.05
        
        // Request next frame
        js.Global().Call("requestAnimationFrame", renderFrame)
        return nil
    })
    
    // Start animation
    js.Global().Call("requestAnimationFrame", renderFrame)
}

func main() {
    drawAnimation()
    select {}
}
```

### Real-World Example: Image Processing

```go
package main

import (
    "image"
    "image/color"
    "syscall/js"
)

func processImage(this js.Value, args []js.Value) interface{} {
    if len(args) == 0 {
        return "Error: no image data provided"
    }
    
    imageData := args[0]
    width := imageData.Get("width").Int()
    height := imageData.Get("height").Int()
    data := imageData.Get("data")
    
    // Convert to grayscale
    for i := 0; i < width*height*4; i += 4 {
        r := data.Index(i).Int()
        g := data.Index(i + 1).Int()
        b := data.Index(i + 2).Int()
        
        // Calculate grayscale value
        gray := int(0.299*float64(r) + 0.587*float64(g) + 0.114*float64(b))
        
        data.SetIndex(i, gray)
        data.SetIndex(i+1, gray)
        data.SetIndex(i+2, gray)
        // Alpha channel (i+3) remains unchanged
    }
    
    return imageData
}

func main() {
    js.Global().Set("processImage", js.FuncOf(processImage))
    println("Image processor ready!")
    select {}
}
```

**HTML + JavaScript:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Image Processing with Go WASM</title>
</head>
<body>
    <h1>Image Processing</h1>
    <input type="file" id="fileInput" accept="image/*">
    <br><br>
    <canvas id="canvas"></canvas>
    
    <script src="wasm_exec.js"></script>
    <script>
        const go = new Go();
        WebAssembly.instantiateStreaming(fetch("main.wasm"), go.importObject)
            .then((result) => {
                go.run(result.instance);
                
                document.getElementById('fileInput').addEventListener('change', (e) => {
                    const file = e.target.files[0];
                    const reader = new FileReader();
                    
                    reader.onload = (event) => {
                        const img = new Image();
                        img.onload = () => {
                            const canvas = document.getElementById('canvas');
                            canvas.width = img.width;
                            canvas.height = img.height;
                            
                            const ctx = canvas.getContext('2d');
                            ctx.drawImage(img, 0, 0);
                            
                            const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                            
                            // Process image with Go function
                            const processed = processImage(imageData);
                            
                            ctx.putImageData(processed, 0, 0);
                        };
                        img.src = event.target.result;
                    };
                    
                    reader.readAsDataURL(file);
                });
            });
    </script>
</body>
</html>
```

### TinyGo for Smaller Binaries

TinyGo is a Go compiler for microcontrollers and WebAssembly that produces much smaller binaries.

**Install TinyGo:**
```bash
# macOS
brew tap tinygo-org/tools
brew install tinygo

# Linux - see https://tinygo.org/getting-started/install/
```

**Compile with TinyGo:**
```bash
tinygo build -o main.wasm -target wasm ./main.go
```

**Size comparison:**
- Regular Go: 2-10 MB
- TinyGo: 10-100 KB

**Limitations:**
- Not all standard library packages supported
- Some reflection features limited
- Different garbage collector

---

*I'll continue with GraphQL, SSE, PWA, and complete the part. Shall I continue?*

## 11.5 GraphQL with gqlgen

### What is GraphQL?

GraphQL is a query language for APIs that lets clients request exactly the data they need. Unlike REST where you have multiple endpoints, GraphQL typically has a single endpoint.

**GraphQL vs REST:**
- **REST**: Multiple endpoints, fixed responses, over/under-fetching
- **GraphQL**: Single endpoint, flexible queries, request exactly what you need

### gqlgen - GraphQL Code Generator

gqlgen is a Go library that generates a type-safe GraphQL server from your schema.

**Installation:**
```bash
go get github.com/99designs/gqlgen
```

### Basic GraphQL Server

**1. Initialize gqlgen project:**
```bash
mkdir graphql-server && cd graphql-server
go mod init graphql-server
go run github.com/99designs/gqlgen init
```

This creates:
- `graph/schema.graphqls` - Your GraphQL schema
- `graph/model/` - Generated models
- `graph/` - Generated resolver stubs
- `server.go` - HTTP server

**2. Define Schema (graph/schema.graphqls):**
```graphql
type Query {
  users: [User!]!
  user(id: ID!): User
  posts: [Post!]!
  post(id: ID!): Post
}

type Mutation {
  createUser(input: NewUser!): User!
  createPost(input: NewPost!): Post!
  deletePost(id: ID!): Boolean!
}

type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: Time!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  published: Boolean!
  createdAt: Time!
}

input NewUser {
  name: String!
  email: String!
}

input NewPost {
  title: String!
  content: String!
  authorID: ID!
}

scalar Time
```

**3. Generate Code:**
```bash
go run github.com/99designs/gqlgen generate
```

**4. Implement Resolvers (graph/schema.resolvers.go):**
```go
package graph

import (
    "context"
    "fmt"
    "strconv"
    "time"
    
    "graphql-server/graph/model"
)

// In-memory storage (use database in production)
var (
    users = make(map[string]*model.User)
    posts = make(map[string]*model.Post)
    nextUserID = 1
    nextPostID = 1
)

func (r *queryResolver) Users(ctx context.Context) ([]*model.User, error) {
    result := make([]*model.User, 0, len(users))
    for _, user := range users {
        result = append(result, user)
    }
    return result, nil
}

func (r *queryResolver) User(ctx context.Context, id string) (*model.User, error) {
    user, exists := users[id]
    if !exists {
        return nil, fmt.Errorf("user not found")
    }
    return user, nil
}

func (r *queryResolver) Posts(ctx context.Context) ([]*model.Post, error) {
    result := make([]*model.Post, 0, len(posts))
    for _, post := range posts {
        result = append(result, post)
    }
    return result, nil
}

func (r *queryResolver) Post(ctx context.Context, id string) (*model.Post, error) {
    post, exists := posts[id]
    if !exists {
        return nil, fmt.Errorf("post not found")
    }
    return post, nil
}

func (r *mutationResolver) CreateUser(ctx context.Context, input model.NewUser) (*model.User, error) {
    id := strconv.Itoa(nextUserID)
    nextUserID++
    
    user := &model.User{
        ID:        id,
        Name:      input.Name,
        Email:     input.Email,
        CreatedAt: time.Now(),
    }
    
    users[id] = user
    return user, nil
}

func (r *mutationResolver) CreatePost(ctx context.Context, input model.NewPost) (*model.Post, error) {
    // Verify author exists
    _, exists := users[input.AuthorID]
    if !exists {
        return nil, fmt.Errorf("author not found")
    }
    
    id := strconv.Itoa(nextPostID)
    nextPostID++
    
    post := &model.Post{
        ID:        id,
        Title:     input.Title,
        Content:   input.Content,
        Published: false,
        CreatedAt: time.Now(),
    }
    
    posts[id] = post
    return post, nil
}

func (r *mutationResolver) DeletePost(ctx context.Context, id string) (bool, error) {
    _, exists := posts[id]
    if !exists {
        return false, fmt.Errorf("post not found")
    }
    
    delete(posts, id)
    return true, nil
}

// User resolver for nested fields
func (r *userResolver) Posts(ctx context.Context, obj *model.User) ([]*model.Post, error) {
    result := make([]*model.Post, 0)
    for _, post := range posts {
        // Simplified: would check post.AuthorID == obj.ID
        result = append(result, post)
    }
    return result, nil
}

// Post resolver for nested fields
func (r *postResolver) Author(ctx context.Context, obj *model.Post) (*model.User, error) {
    // Simplified: would look up by obj.AuthorID
    for _, user := range users {
        return user, nil
    }
    return nil, fmt.Errorf("author not found")
}
```

**5. Run Server (server.go):**
```go
package main

import (
    "log"
    "net/http"
    "os"
    
    "graphql-server/graph"
    
    "github.com/99designs/gqlgen/graphql/handler"
    "github.com/99designs/gqlgen/graphql/playground"
)

const defaultPort = "8080"

func main() {
    port := os.Getenv("PORT")
    if port == "" {
        port = defaultPort
    }
    
    srv := handler.NewDefaultServer(graph.NewExecutableSchema(graph.Config{
        Resolvers: &graph.Resolver{},
    }))
    
    http.Handle("/", playground.Handler("GraphQL playground", "/query"))
    http.Handle("/query", srv)
    
    log.Printf("connect to http://localhost:%s/ for GraphQL playground", port)
    log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

**6. Test Queries:**

Open `http://localhost:8080` in browser (GraphQL Playground)

```graphql
# Create a user
mutation {
  createUser(input: {
    name: "Alice"
    email: "alice@example.com"
  }) {
    id
    name
    email
  }
}

# Query users
query {
  users {
    id
    name
    email
    createdAt
  }
}

# Query with nested fields
query {
  users {
    id
    name
    posts {
      id
      title
    }
  }
}

# Query with variables
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    posts {
      title
      content
    }
  }
}
```

### DataLoader Pattern (N+1 Problem)

The N+1 problem occurs when loading a list of items and their related data requires N+1 database queries.

**Example N+1 Problem:**
```go
// BAD: Makes N+1 queries
func (r *userResolver) Posts(ctx context.Context, obj *model.User) ([]*model.Post, error) {
    // This gets called for EACH user in a list query
    return db.GetPostsByUserID(obj.ID) // Query for each user!
}
```

**Solution: DataLoader**

```bash
go get github.com/graph-gophers/dataloader/v7
```

```go
package dataloader

import (
    "context"
    "fmt"
    
    "github.com/graph-gophers/dataloader/v7"
    "graphql-server/graph/model"
)

type ctxKey string

const (
    loadersKey = ctxKey("dataloaders")
)

type Loaders struct {
    PostsByUserID *dataloader.Loader[string, []*model.Post]
}

func NewLoaders() *Loaders {
    return &Loaders{
        PostsByUserID: dataloader.NewBatchedLoader(
            batchGetPostsByUserIDs,
            dataloader.WithWait[string, []*model.Post](10*time.Millisecond),
        ),
    }
}

func batchGetPostsByUserIDs(ctx context.Context, userIDs []string) []*dataloader.Result[[]*model.Post] {
    // Fetch all posts for all user IDs in a SINGLE query
    postsByUser := make(map[string][]*model.Post)
    
    // Example: SELECT * FROM posts WHERE user_id IN (...)
    allPosts := db.GetPostsByUserIDs(userIDs)
    
    for _, post := range allPosts {
        postsByUser[post.AuthorID] = append(postsByUser[post.AuthorID], post)
    }
    
    // Return results in same order as input
    results := make([]*dataloader.Result[[]*model.Post], len(userIDs))
    for i, userID := range userIDs {
        posts := postsByUser[userID]
        if posts == nil {
            posts = []*model.Post{}
        }
        results[i] = &dataloader.Result[[]*model.Post]{Data: posts}
    }
    
    return results
}

// Middleware to attach loaders to context
func Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        ctx := context.WithValue(r.Context(), loadersKey, NewLoaders())
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

func For(ctx context.Context) *Loaders {
    return ctx.Value(loadersKey).(*Loaders)
}
```

**Use DataLoader in Resolver:**
```go
func (r *userResolver) Posts(ctx context.Context, obj *model.User) ([]*model.Post, error) {
    // Use DataLoader - batches multiple calls into single query
    return dataloader.For(ctx).PostsByUserID.Load(ctx, obj.ID)()
}
```

### Pagination

**Cursor-Based Pagination (Relay Style):**

```graphql
type Query {
  posts(first: Int, after: String): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PostEdge {
  cursor: String!
  node: Post!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

```go
func (r *queryResolver) Posts(ctx context.Context, first *int, after *string) (*model.PostConnection, error) {
    limit := 10
    if first != nil && *first > 0 {
        limit = *first
    }
    
    var startID int
    if after != nil {
        startID, _ = strconv.Atoi(*after)
    }
    
    posts := db.GetPosts(startID, limit+1) // Get one extra to check hasNextPage
    
    hasNextPage := len(posts) > limit
    if hasNextPage {
        posts = posts[:limit]
    }
    
    edges := make([]*model.PostEdge, len(posts))
    for i, post := range posts {
        edges[i] = &model.PostEdge{
            Cursor: post.ID,
            Node:   post,
        }
    }
    
    pageInfo := &model.PageInfo{
        HasNextPage:     hasNextPage,
        HasPreviousPage: after != nil,
    }
    
    if len(edges) > 0 {
        pageInfo.StartCursor = &edges[0].Cursor
        pageInfo.EndCursor = &edges[len(edges)-1].Cursor
    }
    
    return &model.PostConnection{
        Edges:      edges,
        PageInfo:   pageInfo,
        TotalCount: db.GetPostCount(),
    }, nil
}
```

### Authentication & Authorization

```go
// Middleware for authentication
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        
        userID, err := verifyToken(token)
        if err != nil {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        
        ctx := context.WithValue(r.Context(), "userID", userID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// Use in resolver
func (r *mutationResolver) CreatePost(ctx context.Context, input model.NewPost) (*model.Post, error) {
    userID, ok := ctx.Value("userID").(string)
    if !ok {
        return nil, fmt.Errorf("unauthorized")
    }
    
    // Create post...
}
```

**Directive-Based Authorization:**

```graphql
directive @auth on FIELD_DEFINITION

type Mutation {
  createPost(input: NewPost!): Post! @auth
  deletePost(id: ID!): Boolean! @auth
}
```

```go
// Implement directive
func (r *mutationResolver) CreatePost(ctx context.Context, input model.NewPost) (*model.Post, error) {
    // Directive middleware checks auth before calling this
    // ...
}
```

### Subscriptions (Real-Time Updates)

```graphql
type Subscription {
  postAdded: Post!
  postUpdated(id: ID!): Post!
}
```

```go
package graph

import (
    "context"
    "time"
)

// Channel for broadcasting post additions
var postAddedChan = make(chan *model.Post, 100)

func (r *mutationResolver) CreatePost(ctx context.Context, input model.NewPost) (*model.Post, error) {
    post := createPost(input)
    
    // Broadcast to subscribers
    select {
    case postAddedChan <- post:
    default:
    }
    
    return post, nil
}

func (r *subscriptionResolver) PostAdded(ctx context.Context) (<-chan *model.Post, error) {
    posts := make(chan *model.Post, 1)
    
    go func() {
        for {
            select {
            case post := <-postAddedChan:
                posts <- post
            case <-ctx.Done():
                close(posts)
                return
            }
        }
    }()
    
    return posts, nil
}
```

---

## 11.6 Server-Sent Events (SSE)

### What are Server-Sent Events?

SSE allows servers to push updates to clients over HTTP. Unlike WebSockets, SSE is:
- Unidirectional (server → client only)
- Simpler to implement
- Automatic reconnection
- Works over HTTP (no special protocol)

**Use Cases:**
- Live notifications
- Real-time dashboards
- Live feeds
- Progress updates
- Server-to-client only communication

### Basic SSE Server

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "time"
)

func sseHandler(w http.ResponseWriter, r *http.Request) {
    // Set headers for SSE
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    w.Header().Set("Access-Control-Allow-Origin", "*") // CORS
    
    // Get flusher for streaming
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming unsupported", http.StatusInternalServerError)
        return
    }
    
    // Create a channel for client disconnect
    notify := r.Context().Done()
    
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-notify:
            log.Println("Client disconnected")
            return
            
        case t := <-ticker.C:
            // Send event
            fmt.Fprintf(w, "data: Server time: %s\n\n", t.Format(time.RFC3339))
            flusher.Flush()
        }
    }
}

func main() {
    http.HandleFunc("/events", sseHandler)
    
    log.Println("SSE server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**HTML Client:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>SSE Demo</title>
</head>
<body>
    <h1>Server-Sent Events</h1>
    <div id="events"></div>
    
    <script>
        const eventSource = new EventSource('/events');
        const eventsDiv = document.getElementById('events');
        
        eventSource.onmessage = (event) => {
            const p = document.createElement('p');
            p.textContent = event.data;
            eventsDiv.appendChild(p);
        };
        
        eventSource.onerror = (error) => {
            console.error('EventSource error:', error);
            // EventSource automatically reconnects
        };
        
        // Close connection
        // eventSource.close();
    </script>
</body>
</html>
```

### Named Events

```go
func sseHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    flusher, _ := w.(http.Flusher)
    
    // Send named events
    fmt.Fprintf(w, "event: user_connected\n")
    fmt.Fprintf(w, "data: {\"user\": \"Alice\"}\n\n")
    flusher.Flush()
    
    time.Sleep(2 * time.Second)
    
    fmt.Fprintf(w, "event: message\n")
    fmt.Fprintf(w, "data: {\"text\": \"Hello!\"}\n\n")
    flusher.Flush()
    
    time.Sleep(2 * time.Second)
    
    fmt.Fprintf(w, "event: user_disconnected\n")
    fmt.Fprintf(w, "data: {\"user\": \"Alice\"}\n\n")
    flusher.Flush()
}
```

**Client with Event Listeners:**
```javascript
const eventSource = new EventSource('/events');

eventSource.addEventListener('user_connected', (event) => {
    const data = JSON.parse(event.data);
    console.log('User connected:', data.user);
});

eventSource.addEventListener('message', (event) => {
    const data = JSON.parse(event.data);
    console.log('Message:', data.text);
});

eventSource.addEventListener('user_disconnected', (event) => {
    const data = JSON.parse(event.data);
    console.log('User disconnected:', data.user);
});
```

### SSE with Event ID and Reconnection

```go
func sseHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    flusher, _ := w.(http.Flusher)
    
    // Get last event ID from client (for reconnection)
    lastEventID := r.Header.Get("Last-Event-ID")
    startID := 0
    if lastEventID != "" {
        startID, _ = strconv.Atoi(lastEventID)
    }
    
    eventID := startID
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-r.Context().Done():
            return
            
        case <-ticker.C:
            eventID++
            
            // Send event with ID
            fmt.Fprintf(w, "id: %d\n", eventID)
            fmt.Fprintf(w, "data: Event %d\n\n", eventID)
            flusher.Flush()
        }
    }
}
```

### Notification System with SSE

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "sync"
)

type Notification struct {
    Type    string `json:"type"`
    Message string `json:"message"`
}

type NotificationHub struct {
    clients map[chan Notification]bool
    mu      sync.RWMutex
}

func NewNotificationHub() *NotificationHub {
    return &NotificationHub{
        clients: make(map[chan Notification]bool),
    }
}

func (h *NotificationHub) Subscribe() chan Notification {
    ch := make(chan Notification, 10)
    h.mu.Lock()
    h.clients[ch] = true
    h.mu.Unlock()
    return ch
}

func (h *NotificationHub) Unsubscribe(ch chan Notification) {
    h.mu.Lock()
    delete(h.clients, ch)
    close(ch)
    h.mu.Unlock()
}

func (h *NotificationHub) Broadcast(notification Notification) {
    h.mu.RLock()
    defer h.mu.RUnlock()
    
    for ch := range h.clients {
        select {
        case ch <- notification:
        default:
            // Channel full, skip
        }
    }
}

func (h *NotificationHub) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming unsupported", http.StatusInternalServerError)
        return
    }
    
    // Subscribe to notifications
    ch := h.Subscribe()
    defer h.Unsubscribe(ch)
    
    for {
        select {
        case <-r.Context().Done():
            return
            
        case notification := <-ch:
            data, _ := json.Marshal(notification)
            fmt.Fprintf(w, "data: %s\n\n", data)
            flusher.Flush()
        }
    }
}

func main() {
    hub := NewNotificationHub()
    
    // SSE endpoint
    http.Handle("/events", hub)
    
    // Endpoint to trigger notifications
    http.HandleFunc("/notify", func(w http.ResponseWriter, r *http.Request) {
        notification := Notification{
            Type:    r.URL.Query().Get("type"),
            Message: r.URL.Query().Get("message"),
        }
        
        hub.Broadcast(notification)
        w.Write([]byte("Notification sent"))
    })
    
    log.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## FAQs

**Q1: When should I use HTMX vs a JavaScript framework like React?**

Use HTMX when:
- You want minimal JavaScript
- Server-side rendering is preferred
- Simpler architecture is important
- SEO is critical
- Team is backend-focused

Use React/Vue/Svelte when:
- Complex client-side state
- Offline functionality needed
- Rich interactions required
- Client-side routing essential

**Q2: Is WebAssembly ready for production?**

Yes, but consider:
- Large binary sizes (use TinyGo to reduce)
- Slower startup than JavaScript
- Good for CPU-intensive tasks
- Not ideal for DOM-heavy applications
- Limited browser API access

**Q3: GraphQL vs REST - which should I choose?**

Use GraphQL when:
- Clients need flexible data fetching
- Multiple client types (web, mobile)
- Avoiding over/under-fetching is important
- Real-time updates needed (subscriptions)

Use REST when:
- Simple CRUD operations
- Caching is important
- Team familiarity with REST
- Simpler is better

**Q4: How do I handle HTMX and direct browser visits?**

Check the HX-Request header:
```go
if r.Header.Get("HX-Request") == "true" {
    // Return HTML fragment
} else {
    // Return full page
}
```

**Q5: Can I mix HTMX with WebSockets?**

Yes! HTMX supports WebSocket extensions:
```html
<div hx-ws="connect:/ws">
    <form hx-ws="send">
        <input name="message">
    </form>
</div>
```

---

## Interview Questions

**Q1: Explain the difference between Server-Side Rendering and Client-Side Rendering.**

**A**: SSR generates HTML on the server and sends complete pages. CSR sends a minimal HTML shell and builds the UI with JavaScript. SSR benefits: better SEO, faster initial load, works without JS. CSR benefits: richer interactions, less server load, better for SPAs.

**Q2: What is the N+1 problem in GraphQL and how do you solve it?**

**A**: The N+1 problem occurs when fetching a list of items and their relations requires N+1 queries (1 for list + N for each item's relations). Solution: Use DataLoader to batch requests into a single query.

**Q3: How do WebSockets differ from HTTP polling?**

**A**: HTTP polling: client repeatedly requests updates, wasteful, higher latency. WebSockets: persistent bidirectional connection, real-time, lower overhead. Use WebSockets for real-time apps, polling for simple periodic updates.

**Q4: What are the security considerations with HTMX?**

**A**: Same as any web app: CSRF protection (verify tokens), XSS prevention (escape output), authentication/authorization, rate limiting, input validation. HTMX doesn't add new vulnerabilities but follow standard security practices.

**Q5: How do you optimize WebAssembly binary size?**

**A**: Use TinyGo instead of standard Go (10-100KB vs 2-10MB), strip debug info, use compression (gzip/brotli), code splitting, lazy loading modules, minimize dependencies.

---

## Key Takeaways

1. **Server-Side Rendering** with Go provides excellent performance and SEO
2. **HTMX** enables reactive UIs without writing JavaScript
3. **WebSockets** enable real-time bidirectional communication
4. **WebAssembly** runs Go code in the browser for CPU-intensive tasks
5. **GraphQL** provides flexible, efficient data fetching
6. **SSE** is perfect for server-to-client real-time updates
7. Choose the right tool for your use case - don't over-engineer

---

## Practice Exercises

### Exercise 1: HTMX Todo App
Build a complete todo application using:
- HTMX for all interactions
- No custom JavaScript
- Server-side validation
- Optimistic UI updates

### Exercise 2: Real-Time Chat
Create a chat application with:
- WebSocket connections
- Multiple rooms
- User presence
- Message history
- Redis for scaling

### Exercise 3: GraphQL Blog API
Build a blog API with:
- Posts and comments
- Pagination
- DataLoader for N+1 prevention
- Authentication
- Subscriptions for real-time updates

### Exercise 4: WebAssembly Image Editor
Create a browser-based image editor:
- Image filters (grayscale, blur, etc.)
- Canvas manipulation
- File upload/download
- Compare Go WASM vs JavaScript performance

### Exercise 5: Live Dashboard
Build a real-time dashboard using:
- SSE for server updates
- HTMX for user interactions
- Polling for metrics
- Chart visualization

---

## Additional Resources

**HTMX:**
- https://htmx.org
- https://htmx.org/examples/
- Book: "Hypermedia Systems"

**WebAssembly:**
- https://github.com/golang/go/wiki/WebAssembly
- https://tinygo.org
- https://webassembly.org

**GraphQL:**
- https://gqlgen.com
- https://graphql.org
- https://www.apollographql.com

**WebSockets:**
- https://github.com/gorilla/websocket
- https://websockets.spec.whatwg.org

---

## Next Steps

- **Part 12**: Data Engineering & Processing (Kafka, streams, ETL)
- Practice building full-stack applications with these tools
- Experiment with different approaches for your use cases
- Contribute to open-source projects using these technologies

**Congratulations!** You now have comprehensive knowledge of modern web development with Go! 🎉

## 11.7 API Versioning Strategies

### Why Version APIs?

- Breaking changes need migration path
- Support multiple clients (old apps still work)
- Allow gradual adoption of new features
- Maintain backwards compatibility

### URL Versioning

Most common and explicit approach.

```go
// v1/user.go
package v1

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func GetUser(w http.ResponseWriter, r *http.Request) {
    // v1 implementation
}

// v2/user.go
package v2

type User struct {
    ID        int    `json:"id"`
    FirstName string `json:"first_name"`
    LastName  string `json:"last_name"`
    Email     string `json:"email"`
    Phone     string `json:"phone"`
}

func GetUser(w http.ResponseWriter, r *http.Request) {
    // v2 implementation
}

// main.go
package main

import (
    v1 "myapp/api/v1"
    v2 "myapp/api/v2"
)

func main() {
    // Version in path
    http.HandleFunc("/api/v1/users", v1.GetUser)
    http.HandleFunc("/api/v2/users", v2.GetUser)
    
    http.ListenAndServe(":8080", nil)
}
```

### Header Versioning

Version specified in custom header.

```go
func versionMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        version := r.Header.Get("API-Version")
        
        switch version {
        case "2":
            // Add v2 marker to context
            ctx := context.WithValue(r.Context(), "api_version", 2)
            next.ServeHTTP(w, r.WithContext(ctx))
        default:
            // Default to v1
            ctx := context.WithValue(r.Context(), "api_version", 1)
            next.ServeHTTP(w, r.WithContext(ctx))
        }
    })
}

func getUserHandler(w http.ResponseWriter, r *http.Request) {
    version := r.Context().Value("api_version").(int)
    
    if version == 2 {
        // Return v2 response
        json.NewEncoder(w).Encode(v2.GetUser())
    } else {
        // Return v1 response
        json.NewEncoder(w).Encode(v1.GetUser())
    }
}
```

### Content Negotiation

Version in Accept header.

```go
func contentNegotiationHandler(w http.ResponseWriter, r *http.Request) {
    accept := r.Header.Get("Accept")
    
    switch {
    case strings.Contains(accept, "application/vnd.myapp.v2+json"):
        w.Header().Set("Content-Type", "application/vnd.myapp.v2+json")
        json.NewEncoder(w).Encode(v2.GetUser())
    default:
        w.Header().Set("Content-Type", "application/vnd.myapp.v1+json")
        json.NewEncoder(w).Encode(v1.GetUser())
    }
}
```

### Deprecation Strategy

```go
type DeprecationInfo struct {
    Version     string    `json:"version"`
    DeprecatedAt time.Time `json:"deprecated_at"`
    SunsetAt     time.Time `json:"sunset_at"`
    Message      string    `json:"message"`
}

func deprecationMiddleware(version string, sunset time.Time) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Add deprecation headers
            w.Header().Set("Deprecation", "true")
            w.Header().Set("Sunset", sunset.Format(time.RFC1123))
            w.Header().Set("Link", `</api/v2>; rel="successor-version"`)
            
            next.ServeHTTP(w, r)
        })
    }
}

// Usage
sunsetDate := time.Date(2025, 12, 31, 0, 0, 0, 0, time.UTC)
http.Handle("/api/v1/users", 
    deprecationMiddleware("v1", sunsetDate)(
        http.HandlerFunc(v1.GetUser),
    ),
)
```

---

## 11.8 Progressive Web Apps (PWA)

### What is a PWA?

Progressive Web App features:
- Installable on devices
- Works offline
- Push notifications
- App-like experience
- Served over HTTPS

### Service Worker

Go serves the service worker JavaScript file:

```go
// static/sw.js
const CACHE_NAME = 'my-app-v1';
const urlsToCache = [
    '/',
    '/static/style.css',
    '/static/app.js',
    '/offline.html'
];

// Install event - cache resources
self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then((cache) => cache.addAll(urlsToCache))
    );
});

// Fetch event - serve from cache, fallback to network
self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request)
            .then((response) => {
                if (response) {
                    return response; // Serve from cache
                }
                return fetch(event.request); // Fetch from network
            })
            .catch(() => {
                // Offline fallback
                if (event.request.mode === 'navigate') {
                    return caches.match('/offline.html');
                }
            })
    );
});
```

### Web App Manifest

```go
// Generate manifest.json
func manifestHandler(w http.ResponseWriter, r *http.Request) {
    manifest := map[string]interface{}{
        "name":             "My Go PWA",
        "short_name":       "GoPWA",
        "description":      "A Progressive Web App built with Go",
        "start_url":        "/",
        "display":          "standalone",
        "background_color": "#ffffff",
        "theme_color":      "#2196F3",
        "icons": []map[string]string{
            {
                "src":   "/static/icon-192.png",
                "sizes": "192x192",
                "type":  "image/png",
            },
            {
                "src":   "/static/icon-512.png",
                "sizes": "512x512",
                "type":  "image/png",
            },
        },
    }
    
    w.Header().Set("Content-Type", "application/manifest+json")
    json.NewEncoder(w).Encode(manifest)
}

func main() {
    http.HandleFunc("/manifest.json", manifestHandler)
    http.Handle("/static/", http.StripPrefix("/static/", http.FileServer(http.Dir("static"))))
    
    http.ListenAndServe(":8080", nil)
}
```

### HTML with PWA Features

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Go PWA</title>
    
    <!-- PWA Manifest -->
    <link rel="manifest" href="/manifest.json">
    
    <!-- Theme color -->
    <meta name="theme-color" content="#2196F3">
    
    <!-- Icons -->
    <link rel="icon" type="image/png" sizes="192x192" href="/static/icon-192.png">
    <link rel="apple-touch-icon" href="/static/icon-192.png">
</head>
<body>
    <h1>My Progressive Web App</h1>
    
    <button id="install-button" style="display: none;">Install App</button>
    
    <script>
        // Register service worker
        if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('/static/sw.js')
                .then(reg => console.log('Service Worker registered', reg))
                .catch(err => console.error('Service Worker registration failed', err));
        }
        
        // Install prompt
        let deferredPrompt;
        const installButton = document.getElementById('install-button');
        
        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            installButton.style.display = 'block';
        });
        
        installButton.addEventListener('click', async () => {
            if (deferredPrompt) {
                deferredPrompt.prompt();
                const { outcome } = await deferredPrompt.userChoice;
                console.log('Install outcome:', outcome);
                deferredPrompt = null;
                installButton.style.display = 'none';
            }
        });
    </script>
</body>
</html>
```

### Push Notifications (Server Side)

```bash
go get github.com/SherClockHolmes/webpush-go
```

```go
package main

import (
    "encoding/json"
    
    webpush "github.com/SherClockHolmes/webpush-go"
)

type PushSubscription struct {
    Endpoint string `json:"endpoint"`
    Keys     struct {
        P256dh string `json:"p256dh"`
        Auth   string `json:"auth"`
    } `json:"keys"`
}

func sendPushNotification(subscription PushSubscription, message string) error {
    // VAPID keys (generate once, store securely)
    vapidPublicKey := "YOUR_PUBLIC_KEY"
    vapidPrivateKey := "YOUR_PRIVATE_KEY"
    
    payload := map[string]string{
        "title": "New Notification",
        "body":  message,
    }
    
    payloadJSON, _ := json.Marshal(payload)
    
    // Send notification
    resp, err := webpush.SendNotification(payloadJSON, &webpush.Subscription{
        Endpoint: subscription.Endpoint,
        Keys: webpush.Keys{
            P256dh: subscription.Keys.P256dh,
            Auth:   subscription.Keys.Auth,
        },
    }, &webpush.Options{
        Subscriber:      "mailto:admin@example.com",
        VAPIDPublicKey:  vapidPublicKey,
        VAPIDPrivateKey: vapidPrivateKey,
        TTL:             30,
    })
    
    if err != nil {
        return err
    }
    
    defer resp.Body.Close()
    return nil
}

func subscribeHandler(w http.ResponseWriter, r *http.Request) {
    var subscription PushSubscription
    if err := json.NewDecoder(r.Body).Decode(&subscription); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    // Store subscription in database
    saveSubscription(subscription)
    
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(map[string]string{"status": "subscribed"})
}
```

---

## Summary

Part 11 covered modern web and frontend integration with Go:

1. **Server-Side Rendering**: html/template, Templ, Gomponents
2. **HTMX**: Building reactive UIs without JavaScript
3. **WebSockets**: Real-time bidirectional communication
4. **WebAssembly**: Running Go in the browser
5. **GraphQL**: Flexible API queries with gqlgen
6. **Server-Sent Events**: Unidirectional real-time updates
7. **API Versioning**: Managing API evolution
8. **Progressive Web Apps**: Installable, offline-capable apps

You now have the tools to build modern, full-stack web applications entirely in Go!

---

**Total Word Count**: ~15,000 words  
**Status**: ✅ Complete

**Next**: Part 12 - Data Engineering & Processing
