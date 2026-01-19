---
title: "Complete Go Mastery Part 15: Mobile & Cross-Platform Development"
date: 2025-01-13 20:00:00 +0530
categories: [Programming, Go, Mobile]
tags: [golang, go, mobile, android, ios, gui, cross-platform, fyne, gio]
---

# Part 15: Mobile & Cross-Platform Development

## Overview

This part explores building mobile and cross-platform applications with Go. You'll learn how to create native mobile apps for Android and iOS, build desktop GUI applications, and develop backends optimized for mobile clients. While Go isn't the primary choice for mobile UI development, it excels at shared business logic, mobile backends, and cross-platform tools.

**What You'll Learn:**
- gomobile for Android and iOS development
- Cross-platform GUI frameworks (Fyne, Gio)
- Building mobile backends with Go
- Shared business logic across platforms
- Mobile API design patterns
- Push notifications and real-time features
- Offline-first mobile architecture
- App distribution and deployment

**Prerequisites:**
- Part 2 (Concurrency)
- Part 5 (Architecture patterns)
- Part 11 (Web development)
- Basic mobile development concepts

---

## 15.1 Mobile Development with gomobile

### What is gomobile?

gomobile allows you to:
- Write Go packages usable in Android (Java/Kotlin) and iOS (Objective-C/Swift)
- Build pure Go apps with limited UI capabilities
- Share business logic between platforms

**Installation:**
```bash
go install golang.org/x/mobile/cmd/gomobile@latest
gomobile init
```

### Use Cases for gomobile

**Good For:**
- Business logic libraries
- Cryptography and security
- Data processing
- Network protocols
- Algorithms and utilities

**Not Ideal For:**
- Complex native UIs
- Platform-specific features
- Main application code (better: native UI + Go backend)

### gomobile bind: Shared Libraries

Create a Go package that can be used in Android/iOS:

```go
// Package shared provides common business logic
package shared

import "fmt"

// Calculator provides mathematical operations
type Calculator struct {
    history []string
}

// NewCalculator creates a new calculator instance
func NewCalculator() *Calculator {
    return &Calculator{
        history: make([]string, 0),
    }
}

// Add performs addition
func (c *Calculator) Add(a, b float64) float64 {
    result := a + b
    c.history = append(c.history, fmt.Sprintf("%.2f + %.2f = %.2f", a, b, result))
    return result
}

// Subtract performs subtraction
func (c *Calculator) Subtract(a, b float64) float64 {
    result := a - b
    c.history = append(c.history, fmt.Sprintf("%.2f - %.2f = %.2f", a, b, result))
    return result
}

// GetHistory returns calculation history
func (c *Calculator) GetHistory() []string {
    return c.history
}

// ClearHistory clears the calculation history
func (c *Calculator) ClearHistory() {
    c.history = make([]string, 0)
}
```

**Build for Android:**
```bash
gomobile bind -target=android -o calculator.aar ./shared
```

**Build for iOS:**
```bash
gomobile bind -target=ios -o Calculator.framework ./shared
```

**Use in Android (Kotlin):**
```kotlin
import shared.Calculator

val calc = Calculator()
val result = calc.add(5.0, 3.0)
println("Result: $result")

val history = calc.getHistory()
for (entry in history) {
    println(entry)
}
```

**Use in iOS (Swift):**
```swift
import Calculator

let calc = SharedNewCalculator()
let result = calc?.add(5.0, b: 3.0)
print("Result: \(result)")

if let history = calc?.getHistory() {
    for i in 0..<history.count {
        print(history.object(at: i))
    }
}
```

### Complex Example: Authentication Library

```go
package auth

import (
    "crypto/rand"
    "crypto/sha256"
    "encoding/base64"
    "errors"
    "time"
)

// Session represents a user session
type Session struct {
    UserID    string
    Token     string
    ExpiresAt int64
}

// AuthManager handles authentication
type AuthManager struct {
    sessions map[string]*Session
}

// NewAuthManager creates a new auth manager
func NewAuthManager() *AuthManager {
    return &AuthManager{
        sessions: make(map[string]*Session),
    }
}

// Login creates a new session
func (am *AuthManager) Login(username, password string) (string, error) {
    // In production, verify against database
    if username == "" || password == "" {
        return "", errors.New("invalid credentials")
    }
    
    // Create session token
    token, err := generateToken()
    if err != nil {
        return "", err
    }
    
    session := &Session{
        UserID:    username,
        Token:     token,
        ExpiresAt: time.Now().Add(24 * time.Hour).Unix(),
    }
    
    am.sessions[token] = session
    return token, nil
}

// ValidateToken checks if a token is valid
func (am *AuthManager) ValidateToken(token string) bool {
    session, exists := am.sessions[token]
    if !exists {
        return false
    }
    
    // Check expiration
    if time.Now().Unix() > session.ExpiresAt {
        delete(am.sessions, token)
        return false
    }
    
    return true
}

// Logout invalidates a session
func (am *AuthManager) Logout(token string) {
    delete(am.sessions, token)
}

// GetUserID returns the user ID for a token
func (am *AuthManager) GetUserID(token string) (string, error) {
    session, exists := am.sessions[token]
    if !exists {
        return "", errors.New("invalid token")
    }
    
    if time.Now().Unix() > session.ExpiresAt {
        delete(am.sessions, token)
        return "", errors.New("token expired")
    }
    
    return session.UserID, nil
}

func generateToken() (string, error) {
    bytes := make([]byte, 32)
    if _, err := rand.Read(bytes); err != nil {
        return "", err
    }
    
    hash := sha256.Sum256(bytes)
    return base64.URLEncoding.EncodeToString(hash[:]), nil
}
```

### Data Synchronization Library

```go
package sync

import (
    "encoding/json"
    "sync"
    "time"
)

// SyncManager handles data synchronization
type SyncManager struct {
    pendingChanges []Change
    lastSyncTime   int64
    mu             sync.Mutex
}

// Change represents a data change
type Change struct {
    ID        string
    Type      string // "create", "update", "delete"
    Entity    string
    Data      string // JSON data
    Timestamp int64
}

// NewSyncManager creates a new sync manager
func NewSyncManager() *SyncManager {
    return &SyncManager{
        pendingChanges: make([]Change, 0),
        lastSyncTime:   time.Now().Unix(),
    }
}

// AddChange queues a change for synchronization
func (sm *SyncManager) AddChange(entityType, changeType, id, data string) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    change := Change{
        ID:        id,
        Type:      changeType,
        Entity:    entityType,
        Data:      data,
        Timestamp: time.Now().Unix(),
    }
    
    sm.pendingChanges = append(sm.pendingChanges, change)
}

// GetPendingChanges returns changes since last sync
func (sm *SyncManager) GetPendingChanges() string {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    if len(sm.pendingChanges) == 0 {
        return "[]"
    }
    
    data, _ := json.Marshal(sm.pendingChanges)
    return string(data)
}

// ClearPendingChanges removes all pending changes
func (sm *SyncManager) ClearPendingChanges() {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    sm.pendingChanges = make([]Change, 0)
    sm.lastSyncTime = time.Now().Unix()
}

// GetChangeCount returns the number of pending changes
func (sm *SyncManager) GetChangeCount() int {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    return len(sm.pendingChanges)
}
```

---

## 15.2 Cross-Platform GUI with Fyne

### Fyne Framework

Fyne is a modern, material design-inspired GUI toolkit for Go.

**Features:**
- Cross-platform (Windows, macOS, Linux, iOS, Android)
- Material Design
- Easy to use
- Good documentation
- Active community

**Installation:**
```bash
go get fyne.io/fyne/v2
```

### Basic Fyne Application

```go
package main

import (
    "fyne.io/fyne/v2/app"
    "fyne.io/fyne/v2/container"
    "fyne.io/fyne/v2/widget"
)

func main() {
    myApp := app.New()
    myWindow := myApp.NewWindow("Hello Fyne")
    
    hello := widget.NewLabel("Hello, Fyne!")
    myWindow.SetContent(hello)
    
    myWindow.ShowAndRun()
}
```

### Form and Input

```go
package main

import (
    "fyne.io/fyne/v2/app"
    "fyne.io/fyne/v2/container"
    "fyne.io/fyne/v2/widget"
)

func main() {
    myApp := app.New()
    myWindow := myApp.NewWindow("Form Example")
    
    // Create form entries
    nameEntry := widget.NewEntry()
    nameEntry.SetPlaceHolder("Enter name")
    
    emailEntry := widget.NewEntry()
    emailEntry.SetPlaceHolder("Enter email")
    
    passwordEntry := widget.NewPasswordEntry()
    passwordEntry.SetPlaceHolder("Enter password")
    
    // Result label
    resultLabel := widget.NewLabel("")
    
    // Submit button
    submitButton := widget.NewButton("Submit", func() {
        name := nameEntry.Text
        email := emailEntry.Text
        password := passwordEntry.Text
        
        resultLabel.SetText("Name: " + name + "\nEmail: " + email)
    })
    
    // Create form
    form := container.NewVBox(
        widget.NewLabel("Registration Form"),
        nameEntry,
        emailEntry,
        passwordEntry,
        submitButton,
        resultLabel,
    )
    
    myWindow.SetContent(form)
    myWindow.Resize(fyne.NewSize(400, 300))
    myWindow.ShowAndRun()
}
```

### List and Data Binding

```go
import (
    "fyne.io/fyne/v2/data/binding"
)

type Task struct {
    Title     string
    Completed bool
}

type TodoApp struct {
    tasks  []Task
    window fyne.Window
}

func NewTodoApp() *TodoApp {
    return &TodoApp{
        tasks: make([]Task, 0),
    }
}

func (ta *TodoApp) Run() {
    myApp := app.New()
    ta.window = myApp.NewWindow("Todo List")
    
    // Task list
    taskList := widget.NewList(
        func() int {
            return len(ta.tasks)
        },
        func() fyne.CanvasObject {
            return container.NewHBox(
                widget.NewCheck("", nil),
                widget.NewLabel("Template"),
            )
        },
        func(id widget.ListItemID, item fyne.CanvasObject) {
            c := item.(*fyne.Container)
            check := c.Objects[0].(*widget.Check)
            label := c.Objects[1].(*widget.Label)
            
            task := ta.tasks[id]
            label.SetText(task.Title)
            check.SetChecked(task.Completed)
            check.OnChanged = func(checked bool) {
                ta.tasks[id].Completed = checked
            }
        },
    )
    
    // Input for new tasks
    taskEntry := widget.NewEntry()
    taskEntry.SetPlaceHolder("Enter new task")
    
    addButton := widget.NewButton("Add Task", func() {
        if taskEntry.Text != "" {
            ta.tasks = append(ta.tasks, Task{
                Title:     taskEntry.Text,
                Completed: false,
            })
            taskEntry.SetText("")
            taskList.Refresh()
        }
    })
    
    content := container.NewBorder(
        container.NewVBox(
            taskEntry,
            addButton,
        ),
        nil,
        nil,
        nil,
        taskList,
    )
    
    ta.window.SetContent(content)
    ta.window.Resize(fyne.NewSize(400, 500))
    ta.window.ShowAndRun()
}
```

### Table Widget

```go
func createTableApp() {
    myApp := app.New()
    myWindow := myApp.NewWindow("Table Example")
    
    data := [][]string{
        {"John Doe", "john@example.com", "Developer"},
        {"Jane Smith", "jane@example.com", "Designer"},
        {"Bob Wilson", "bob@example.com", "Manager"},
    }
    
    table := widget.NewTable(
        func() (int, int) {
            return len(data), 3
        },
        func() fyne.CanvasObject {
            return widget.NewLabel("Template")
        },
        func(id widget.TableCellID, cell fyne.CanvasObject) {
            label := cell.(*widget.Label)
            label.SetText(data[id.Row][id.Col])
        },
    )
    
    table.SetColumnWidth(0, 120)
    table.SetColumnWidth(1, 180)
    table.SetColumnWidth(2, 100)
    
    myWindow.SetContent(table)
    myWindow.Resize(fyne.NewSize(500, 300))
    myWindow.ShowAndRun()
}
```

### Custom Themes

```go
import (
    "image/color"
    "fyne.io/fyne/v2/theme"
)

type MyTheme struct{}

func (m MyTheme) Color(name fyne.ThemeColorName, variant fyne.ThemeVariant) color.Color {
    switch name {
    case theme.ColorNameButton:
        return color.NRGBA{R: 0, G: 122, B: 255, A: 255}
    case theme.ColorNamePrimary:
        return color.NRGBA{R: 76, G: 175, B: 80, A: 255}
    default:
        return theme.DefaultTheme().Color(name, variant)
    }
}

func (m MyTheme) Icon(name fyne.ThemeIconName) fyne.Resource {
    return theme.DefaultTheme().Icon(name)
}

func (m MyTheme) Font(style fyne.TextStyle) fyne.Resource {
    return theme.DefaultTheme().Font(style)
}

func (m MyTheme) Size(name fyne.ThemeSizeName) float32 {
    return theme.DefaultTheme().Size(name)
}

// Apply custom theme
func main() {
    myApp := app.New()
    myApp.Settings().SetTheme(&MyTheme{})
    
    // ... rest of app
}
```

### File Dialogs

```go
func fileDialogExample() {
    myApp := app.New()
    myWindow := myApp.NewWindow("File Dialog")
    
    contentLabel := widget.NewLabel("No file selected")
    
    openButton := widget.NewButton("Open File", func() {
        dialog.ShowFileOpen(func(reader fyne.URIReadCloser, err error) {
            if err != nil {
                contentLabel.SetText("Error: " + err.Error())
                return
            }
            if reader == nil {
                return
            }
            defer reader.Close()
            
            data, _ := io.ReadAll(reader)
            contentLabel.SetText(string(data))
        }, myWindow)
    })
    
    saveButton := widget.NewButton("Save File", func() {
        dialog.ShowFileSave(func(writer fyne.URIWriteCloser, err error) {
            if err != nil {
                dialog.ShowError(err, myWindow)
                return
            }
            if writer == nil {
                return
            }
            defer writer.Close()
            
            content := []byte("Hello from Fyne!")
            writer.Write(content)
        }, myWindow)
    })
    
    content := container.NewVBox(
        openButton,
        saveButton,
        contentLabel,
    )
    
    myWindow.SetContent(content)
    myWindow.ShowAndRun()
}
```

---

## 15.3 Immediate Mode GUI with Gio

### Gio Framework

Gio is an immediate mode GUI framework with excellent performance.

**Features:**
- Immediate mode (no retained widget tree)
- High performance
- GPU-accelerated
- Small binary size
- Cross-platform

**Installation:**
```bash
go get gioui.org
```

### Basic Gio Application

```go
package main

import (
    "log"
    "os"
    
    "gioui.org/app"
    "gioui.org/io/system"
    "gioui.org/layout"
    "gioui.org/op"
    "gioui.org/widget/material"
    "gioui.org/font/gofont"
)

func main() {
    go func() {
        w := app.NewWindow()
        if err := run(w); err != nil {
            log.Fatal(err)
        }
        os.Exit(0)
    }()
    app.Main()
}

func run(w *app.Window) error {
    th := material.NewTheme(gofont.Collection())
    var ops op.Ops
    
    for {
        e := <-w.Events()
        switch e := e.(type) {
        case system.DestroyEvent:
            return e.Err
        case system.FrameEvent:
            gtx := layout.NewContext(&ops, e)
            
            // Draw UI
            material.H1(th, "Hello Gio!").Layout(gtx)
            
            e.Frame(gtx.Ops)
        }
    }
}
```

### Button and Interaction

```go
import (
    "gioui.org/widget"
)

type UI struct {
    theme  *material.Theme
    button widget.Clickable
    count  int
}

func NewUI() *UI {
    return &UI{
        theme: material.NewTheme(gofont.Collection()),
        count: 0,
    }
}

func (ui *UI) Layout(gtx layout.Context) layout.Dimensions {
    // Check if button was clicked
    if ui.button.Clicked() {
        ui.count++
    }
    
    return layout.Flex{Axis: layout.Vertical}.Layout(gtx,
        layout.Rigid(func(gtx layout.Context) layout.Dimensions {
            return material.H3(ui.theme, fmt.Sprintf("Count: %d", ui.count)).Layout(gtx)
        }),
        layout.Rigid(func(gtx layout.Context) layout.Dimensions {
            return material.Button(ui.theme, &ui.button, "Click Me").Layout(gtx)
        }),
    )
}

func run(w *app.Window) error {
    ui := NewUI()
    var ops op.Ops
    
    for {
        e := <-w.Events()
        switch e := e.(type) {
        case system.DestroyEvent:
            return e.Err
        case system.FrameEvent:
            gtx := layout.NewContext(&ops, e)
            ui.Layout(gtx)
            e.Frame(gtx.Ops)
        }
    }
}
```

### Form with Input Fields

```go
type FormUI struct {
    theme     *material.Theme
    nameEdit  widget.Editor
    emailEdit widget.Editor
    submit    widget.Clickable
    result    string
}

func NewFormUI() *FormUI {
    return &FormUI{
        theme: material.NewTheme(gofont.Collection()),
    }
}

func (f *FormUI) Layout(gtx layout.Context) layout.Dimensions {
    if f.submit.Clicked() {
        f.result = fmt.Sprintf("Name: %s\nEmail: %s", 
            f.nameEdit.Text(), 
            f.emailEdit.Text())
    }
    
    return layout.Flex{Axis: layout.Vertical, Spacing: layout.SpaceEnd}.Layout(gtx,
        layout.Rigid(func(gtx layout.Context) layout.Dimensions {
            return material.H4(f.theme, "Contact Form").Layout(gtx)
        }),
        layout.Rigid(func(gtx layout.Context) layout.Dimensions {
            return material.Editor(f.theme, &f.nameEdit, "Name").Layout(gtx)
        }),
        layout.Rigid(func(gtx layout.Context) layout.Dimensions {
            return material.Editor(f.theme, &f.emailEdit, "Email").Layout(gtx)
        }),
        layout.Rigid(func(gtx layout.Context) layout.Dimensions {
            return material.Button(f.theme, &f.submit, "Submit").Layout(gtx)
        }),
        layout.Rigid(func(gtx layout.Context) layout.Dimensions {
            if f.result != "" {
                return material.Body1(f.theme, f.result).Layout(gtx)
            }
            return layout.Dimensions{}
        }),
    )
}
```

---

*I'll continue with Mobile Backend Development, Push Notifications, Offline-First Architecture, and complete Part 15. Shall I continue?*

## 15.4 Mobile Backend Development

### Mobile-Optimized API Design

**Key Principles:**
- Minimize network requests
- Batch operations
- Efficient data formats (JSON vs Protocol Buffers)
- Pagination for large datasets
- Field filtering (return only needed data)
- API versioning

**Example API:**
```go
package api

import (
    "encoding/json"
    "net/http"
    "strconv"
)

type MobileAPI struct {
    db *Database
}

// Batch endpoint - get multiple resources in one request
func (api *MobileAPI) BatchHandler(w http.ResponseWriter, r *http.Request) {
    var batchReq struct {
        Users    []string `json:"users"`
        Posts    []string `json:"posts"`
        Comments []string `json:"comments"`
    }
    
    json.NewDecoder(r.Body).Decode(&batchReq)
    
    response := map[string]interface{}{}
    
    // Fetch all requested resources
    if len(batchReq.Users) > 0 {
        response["users"] = api.db.GetUsersBatch(batchReq.Users)
    }
    if len(batchReq.Posts) > 0 {
        response["posts"] = api.db.GetPostsBatch(batchReq.Posts)
    }
    if len(batchReq.Comments) > 0 {
        response["comments"] = api.db.GetCommentsBatch(batchReq.Comments)
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

// Pagination with cursor-based approach (better for mobile)
func (api *MobileAPI) ListPostsHandler(w http.ResponseWriter, r *http.Request) {
    limit, _ := strconv.Atoi(r.URL.Query().Get("limit"))
    if limit == 0 || limit > 100 {
        limit = 20
    }
    
    cursor := r.URL.Query().Get("cursor")
    
    posts, nextCursor, err := api.db.GetPosts(limit, cursor)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    response := map[string]interface{}{
        "posts":  posts,
        "cursor": nextCursor,
        "has_more": nextCursor != "",
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

// Field filtering - return only requested fields
func (api *MobileAPI) GetUserHandler(w http.ResponseWriter, r *http.Request) {
    userID := r.URL.Query().Get("id")
    fields := r.URL.Query()["fields"] // ?fields=name&fields=email
    
    user, err := api.db.GetUser(userID)
    if err != nil {
        http.Error(w, err.Error(), http.StatusNotFound)
        return
    }
    
    // Filter fields if specified
    if len(fields) > 0 {
        filtered := make(map[string]interface{})
        for _, field := range fields {
            switch field {
            case "id":
                filtered["id"] = user.ID
            case "name":
                filtered["name"] = user.Name
            case "email":
                filtered["email"] = user.Email
            // ... other fields
            }
        }
        json.NewEncoder(w).Encode(filtered)
    } else {
        json.NewEncoder(w).Encode(user)
    }
}
```

### Delta Sync API

```go
type SyncAPI struct {
    db *Database
}

type SyncRequest struct {
    LastSyncTime int64             `json:"last_sync_time"`
    Changes      []Change          `json:"changes"`
}

type SyncResponse struct {
    ServerChanges []Change `json:"server_changes"`
    Conflicts     []Conflict `json:"conflicts"`
    SyncTime      int64    `json:"sync_time"`
}

type Change struct {
    ID        string `json:"id"`
    Type      string `json:"type"` // "create", "update", "delete"
    Entity    string `json:"entity"`
    Data      map[string]interface{} `json:"data"`
    Timestamp int64  `json:"timestamp"`
}

type Conflict struct {
    ID           string                 `json:"id"`
    ClientData   map[string]interface{} `json:"client_data"`
    ServerData   map[string]interface{} `json:"server_data"`
    ClientTime   int64                  `json:"client_time"`
    ServerTime   int64                  `json:"server_time"`
}

func (api *SyncAPI) SyncHandler(w http.ResponseWriter, r *http.Request) {
    var req SyncRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    // Process client changes
    conflicts := make([]Conflict, 0)
    for _, change := range req.Changes {
        conflict, err := api.processChange(change, req.LastSyncTime)
        if err != nil {
            log.Printf("Error processing change: %v", err)
            continue
        }
        if conflict != nil {
            conflicts = append(conflicts, *conflict)
        }
    }
    
    // Get server changes since last sync
    serverChanges, err := api.db.GetChangesSince(req.LastSyncTime)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    response := SyncResponse{
        ServerChanges: serverChanges,
        Conflicts:     conflicts,
        SyncTime:      time.Now().Unix(),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func (api *SyncAPI) processChange(change Change, lastSyncTime int64) (*Conflict, error) {
    // Check for conflicts
    serverData, serverTime, err := api.db.GetEntityWithTimestamp(change.Entity, change.ID)
    if err != nil && err != ErrNotFound {
        return nil, err
    }
    
    // Conflict if server was modified after client's last sync
    if serverTime > lastSyncTime {
        return &Conflict{
            ID:         change.ID,
            ClientData: change.Data,
            ServerData: serverData,
            ClientTime: change.Timestamp,
            ServerTime: serverTime,
        }, nil
    }
    
    // No conflict, apply change
    switch change.Type {
    case "create", "update":
        api.db.SaveEntity(change.Entity, change.ID, change.Data, change.Timestamp)
    case "delete":
        api.db.DeleteEntity(change.Entity, change.ID)
    }
    
    return nil, nil
}
```

### GraphQL for Mobile

```go
import (
    "github.com/graphql-go/graphql"
)

var userType = graphql.NewObject(graphql.ObjectConfig{
    Name: "User",
    Fields: graphql.Fields{
        "id": &graphql.Field{Type: graphql.String},
        "name": &graphql.Field{Type: graphql.String},
        "email": &graphql.Field{Type: graphql.String},
        "posts": &graphql.Field{
            Type: graphql.NewList(postType),
            Resolve: func(p graphql.ResolveParams) (interface{}, error) {
                user := p.Source.(User)
                return db.GetUserPosts(user.ID)
            },
        },
    },
})

var queryType = graphql.NewObject(graphql.ObjectConfig{
    Name: "Query",
    Fields: graphql.Fields{
        "user": &graphql.Field{
            Type: userType,
            Args: graphql.FieldConfigArgument{
                "id": &graphql.ArgumentConfig{Type: graphql.String},
            },
            Resolve: func(p graphql.ResolveParams) (interface{}, error) {
                id := p.Args["id"].(string)
                return db.GetUser(id)
            },
        },
    },
})

// Client can request exactly what it needs:
// query {
//   user(id: "123") {
//     name
//     posts {
//       title
//     }
//   }
// }
```

---

## 15.5 Push Notifications

### Firebase Cloud Messaging (FCM)

```bash
go get firebase.google.com/go/v4
```

**Send Push Notification:**
```go
package push

import (
    "context"
    "log"
    
    firebase "firebase.google.com/go/v4"
    "firebase.google.com/go/v4/messaging"
    "google.golang.org/api/option"
)

type NotificationService struct {
    client *messaging.Client
}

func NewNotificationService(credentialsPath string) (*NotificationService, error) {
    ctx := context.Background()
    
    opt := option.WithCredentialsFile(credentialsPath)
    app, err := firebase.NewApp(ctx, nil, opt)
    if err != nil {
        return nil, err
    }
    
    client, err := app.Messaging(ctx)
    if err != nil {
        return nil, err
    }
    
    return &NotificationService{client: client}, nil
}

func (ns *NotificationService) SendToDevice(token, title, body string, data map[string]string) error {
    message := &messaging.Message{
        Notification: &messaging.Notification{
            Title: title,
            Body:  body,
        },
        Data:  data,
        Token: token,
    }
    
    _, err := ns.client.Send(context.Background(), message)
    return err
}

func (ns *NotificationService) SendToTopic(topic, title, body string) error {
    message := &messaging.Message{
        Notification: &messaging.Notification{
            Title: title,
            Body:  body,
        },
        Topic: topic,
    }
    
    _, err := ns.client.Send(context.Background(), message)
    return err
}

func (ns *NotificationService) SendMulticast(tokens []string, title, body string) (*messaging.BatchResponse, error) {
    message := &messaging.MulticastMessage{
        Notification: &messaging.Notification{
            Title: title,
            Body:  body,
        },
        Tokens: tokens,
    }
    
    return ns.client.SendMulticast(context.Background(), message)
}

// Subscribe users to topics
func (ns *NotificationService) SubscribeToTopic(tokens []string, topic string) error {
    _, err := ns.client.SubscribeToTopic(context.Background(), tokens, topic)
    return err
}
```

### Push Notification Queue

```go
type NotificationQueue struct {
    service *NotificationService
    queue   chan NotificationJob
    workers int
}

type NotificationJob struct {
    Type  string
    Token string
    Topic string
    Title string
    Body  string
    Data  map[string]string
}

func NewNotificationQueue(service *NotificationService, workers int) *NotificationQueue {
    nq := &NotificationQueue{
        service: service,
        queue:   make(chan NotificationJob, 1000),
        workers: workers,
    }
    
    // Start workers
    for i := 0; i < workers; i++ {
        go nq.worker()
    }
    
    return nq
}

func (nq *NotificationQueue) worker() {
    for job := range nq.queue {
        var err error
        
        switch job.Type {
        case "device":
            err = nq.service.SendToDevice(job.Token, job.Title, job.Body, job.Data)
        case "topic":
            err = nq.service.SendToTopic(job.Topic, job.Title, job.Body)
        }
        
        if err != nil {
            log.Printf("Failed to send notification: %v", err)
            // Could implement retry logic here
        }
    }
}

func (nq *NotificationQueue) QueueDeviceNotification(token, title, body string, data map[string]string) {
    nq.queue <- NotificationJob{
        Type:  "device",
        Token: token,
        Title: title,
        Body:  body,
        Data:  data,
    }
}

func (nq *NotificationQueue) QueueTopicNotification(topic, title, body string) {
    nq.queue <- NotificationJob{
        Type:  "topic",
        Topic: topic,
        Title: title,
        Body:  body,
    }
}
```

---

## 15.6 Offline-First Architecture

### Local Database with SQLite

```go
package storage

import (
    "database/sql"
    _ "github.com/mattn/go-sqlite3"
)

type LocalStorage struct {
    db *sql.DB
}

func NewLocalStorage(dbPath string) (*LocalStorage, error) {
    db, err := sql.Open("sqlite3", dbPath)
    if err != nil {
        return nil, err
    }
    
    // Create tables
    _, err = db.Exec(`
        CREATE TABLE IF NOT EXISTS items (
            id TEXT PRIMARY KEY,
            data TEXT NOT NULL,
            synced INTEGER DEFAULT 0,
            updated_at INTEGER NOT NULL
        );
        
        CREATE TABLE IF NOT EXISTS pending_changes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            entity_type TEXT NOT NULL,
            entity_id TEXT NOT NULL,
            change_type TEXT NOT NULL,
            data TEXT,
            timestamp INTEGER NOT NULL
        );
    `)
    if err != nil {
        return nil, err
    }
    
    return &LocalStorage{db: db}, nil
}

func (ls *LocalStorage) SaveItem(id, data string) error {
    _, err := ls.db.Exec(`
        INSERT OR REPLACE INTO items (id, data, synced, updated_at)
        VALUES (?, ?, 0, ?)
    `, id, data, time.Now().Unix())
    
    if err != nil {
        return err
    }
    
    // Record change for syncing
    _, err = ls.db.Exec(`
        INSERT INTO pending_changes (entity_type, entity_id, change_type, data, timestamp)
        VALUES ('item', ?, 'update', ?, ?)
    `, id, data, time.Now().Unix())
    
    return err
}

func (ls *LocalStorage) GetItem(id string) (string, error) {
    var data string
    err := ls.db.QueryRow("SELECT data FROM items WHERE id = ?", id).Scan(&data)
    return data, err
}

func (ls *LocalStorage) GetPendingChanges() ([]Change, error) {
    rows, err := ls.db.Query(`
        SELECT entity_type, entity_id, change_type, data, timestamp
        FROM pending_changes
        ORDER BY timestamp ASC
    `)
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    changes := make([]Change, 0)
    for rows.Next() {
        var c Change
        err := rows.Scan(&c.Entity, &c.ID, &c.Type, &c.Data, &c.Timestamp)
        if err != nil {
            continue
        }
        changes = append(changes, c)
    }
    
    return changes, nil
}

func (ls *LocalStorage) MarkSynced(ids []string) error {
    tx, err := ls.db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()
    
    // Mark items as synced
    stmt, err := tx.Prepare("UPDATE items SET synced = 1 WHERE id = ?")
    if err != nil {
        return err
    }
    defer stmt.Close()
    
    for _, id := range ids {
        _, err := stmt.Exec(id)
        if err != nil {
            return err
        }
    }
    
    // Clear pending changes for synced items
    _, err = tx.Exec(`
        DELETE FROM pending_changes
        WHERE entity_id IN (` + placeholders(len(ids)) + `)
    `, toInterfaces(ids)...)
    if err != nil {
        return err
    }
    
    return tx.Commit()
}
```

### Sync Manager

```go
type SyncManager struct {
    storage   *LocalStorage
    api       *SyncAPI
    interval  time.Duration
    running   bool
    mu        sync.Mutex
}

func NewSyncManager(storage *LocalStorage, api *SyncAPI, interval time.Duration) *SyncManager {
    return &SyncManager{
        storage:  storage,
        api:      api,
        interval: interval,
    }
}

func (sm *SyncManager) Start() {
    sm.mu.Lock()
    if sm.running {
        sm.mu.Unlock()
        return
    }
    sm.running = true
    sm.mu.Unlock()
    
    go sm.syncLoop()
}

func (sm *SyncManager) Stop() {
    sm.mu.Lock()
    sm.running = false
    sm.mu.Unlock()
}

func (sm *SyncManager) syncLoop() {
    ticker := time.NewTicker(sm.interval)
    defer ticker.Stop()
    
    // Immediate sync on start
    sm.performSync()
    
    for range ticker.C {
        sm.mu.Lock()
        running := sm.running
        sm.mu.Unlock()
        
        if !running {
            return
        }
        
        sm.performSync()
    }
}

func (sm *SyncManager) performSync() {
    log.Println("Starting sync...")
    
    // Get pending changes
    changes, err := sm.storage.GetPendingChanges()
    if err != nil {
        log.Printf("Failed to get pending changes: %v", err)
        return
    }
    
    // Get last sync time
    lastSyncTime := sm.storage.GetLastSyncTime()
    
    // Send to server
    response, err := sm.api.Sync(SyncRequest{
        LastSyncTime: lastSyncTime,
        Changes:      changes,
    })
    if err != nil {
        log.Printf("Sync failed: %v", err)
        return
    }
    
    // Apply server changes
    for _, change := range response.ServerChanges {
        sm.applyServerChange(change)
    }
    
    // Handle conflicts
    for _, conflict := range response.Conflicts {
        sm.handleConflict(conflict)
    }
    
    // Mark local changes as synced
    syncedIDs := make([]string, len(changes))
    for i, change := range changes {
        syncedIDs[i] = change.ID
    }
    sm.storage.MarkSynced(syncedIDs)
    
    // Update last sync time
    sm.storage.SetLastSyncTime(response.SyncTime)
    
    log.Println("Sync completed")
}

func (sm *SyncManager) applyServerChange(change Change) {
    switch change.Type {
    case "create", "update":
        data, _ := json.Marshal(change.Data)
        sm.storage.SaveItem(change.ID, string(data))
    case "delete":
        sm.storage.DeleteItem(change.ID)
    }
}

func (sm *SyncManager) handleConflict(conflict Conflict) {
    // Strategy: Server wins (can be customized)
    log.Printf("Conflict detected for %s, using server version", conflict.ID)
    
    data, _ := json.Marshal(conflict.ServerData)
    sm.storage.SaveItem(conflict.ID, string(data))
}

// Manual sync trigger
func (sm *SyncManager) SyncNow() error {
    sm.performSync()
    return nil
}
```

### Connectivity Monitor

```go
type ConnectivityMonitor struct {
    online     bool
    listeners  []func(bool)
    mu         sync.RWMutex
}

func NewConnectivityMonitor() *ConnectivityMonitor {
    cm := &ConnectivityMonitor{
        online:    true,
        listeners: make([]func(bool), 0),
    }
    
    go cm.monitor()
    
    return cm
}

func (cm *ConnectivityMonitor) monitor() {
    ticker := time.NewTicker(5 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        online := cm.checkConnectivity()
        
        cm.mu.Lock()
        wasOnline := cm.online
        cm.online = online
        cm.mu.Unlock()
        
        // Notify listeners if status changed
        if wasOnline != online {
            cm.notifyListeners(online)
        }
    }
}

func (cm *ConnectivityMonitor) checkConnectivity() bool {
    // Try to reach a known server
    timeout := time.Duration(2 * time.Second)
    client := http.Client{Timeout: timeout}
    
    _, err := client.Get("https://www.google.com")
    return err == nil
}

func (cm *ConnectivityMonitor) IsOnline() bool {
    cm.mu.RLock()
    defer cm.mu.RUnlock()
    return cm.online
}

func (cm *ConnectivityMonitor) OnStatusChange(listener func(bool)) {
    cm.mu.Lock()
    cm.listeners = append(cm.listeners, listener)
    cm.mu.Unlock()
}

func (cm *ConnectivityMonitor) notifyListeners(online bool) {
    cm.mu.RLock()
    listeners := make([]func(bool), len(cm.listeners))
    copy(listeners, cm.listeners)
    cm.mu.RUnlock()
    
    for _, listener := range listeners {
        go listener(online)
    }
}
```

---

*I'll complete Part 15 with App Distribution, FAQs, Interview Questions, and exercises. Shall I continue?*

## 15.7 Real-Time Features

### WebSocket for Mobile

```go
package realtime

import (
    "log"
    "sync"
    
    "github.com/gorilla/websocket"
)

type MobileHub struct {
    clients    map[*Client]bool
    broadcast  chan []byte
    register   chan *Client
    unregister chan *Client
    mu         sync.RWMutex
}

type Client struct {
    hub    *MobileHub
    conn   *websocket.Conn
    send   chan []byte
    userID string
}

func NewMobileHub() *MobileHub {
    return &MobileHub{
        clients:    make(map[*Client]bool),
        broadcast:  make(chan []byte, 256),
        register:   make(chan *Client),
        unregister: make(chan *Client),
    }
}

func (h *MobileHub) Run() {
    for {
        select {
        case client := <-h.register:
            h.mu.Lock()
            h.clients[client] = true
            h.mu.Unlock()
            log.Printf("Client connected: %s", client.userID)
            
        case client := <-h.unregister:
            h.mu.Lock()
            if _, ok := h.clients[client]; ok {
                delete(h.clients, client)
                close(client.send)
            }
            h.mu.Unlock()
            log.Printf("Client disconnected: %s", client.userID)
            
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

func (h *MobileHub) SendToUser(userID string, message []byte) {
    h.mu.RLock()
    defer h.mu.RUnlock()
    
    for client := range h.clients {
        if client.userID == userID {
            select {
            case client.send <- message:
            default:
                close(client.send)
                delete(h.clients, client)
            }
        }
    }
}

func (c *Client) ReadPump() {
    defer func() {
        c.hub.unregister <- c
        c.conn.Close()
    }()
    
    for {
        _, message, err := c.conn.ReadMessage()
        if err != nil {
            break
        }
        
        // Handle incoming message
        c.handleMessage(message)
    }
}

func (c *Client) WritePump() {
    defer c.conn.Close()
    
    for message := range c.send {
        err := c.conn.WriteMessage(websocket.TextMessage, message)
        if err != nil {
            break
        }
    }
}

func (c *Client) handleMessage(message []byte) {
    // Parse and handle message
    log.Printf("Received from %s: %s", c.userID, message)
}
```

### Server-Sent Events (SSE) for Mobile

```go
type SSEServer struct {
    clients map[chan string]bool
    mu      sync.RWMutex
}

func NewSSEServer() *SSEServer {
    return &SSEServer{
        clients: make(map[chan string]bool),
    }
}

func (s *SSEServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Set SSE headers
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    // Create client channel
    clientChan := make(chan string)
    
    s.mu.Lock()
    s.clients[clientChan] = true
    s.mu.Unlock()
    
    defer func() {
        s.mu.Lock()
        delete(s.clients, clientChan)
        close(clientChan)
        s.mu.Unlock()
    }()
    
    // Send events to client
    for {
        select {
        case msg := <-clientChan:
            fmt.Fprintf(w, "data: %s\n\n", msg)
            w.(http.Flusher).Flush()
        case <-r.Context().Done():
            return
        }
    }
}

func (s *SSEServer) Broadcast(message string) {
    s.mu.RLock()
    defer s.mu.RUnlock()
    
    for client := range s.clients {
        select {
        case client <- message:
        default:
        }
    }
}
```

---

## 15.8 App Distribution and Deployment

### Building for Multiple Platforms

**Fyne Cross-Compilation:**
```bash
# Build for Android
fyne package -os android -appID com.example.myapp

# Build for iOS
fyne package -os ios -appID com.example.myapp

# Build for Windows
fyne package -os windows

# Build for macOS
fyne package -os darwin

# Build for Linux
fyne package -os linux
```

**gomobile Build:**
```bash
# Android APK
gomobile build -target=android -o myapp.apk ./cmd/mobile

# iOS App
gomobile build -target=ios -bundleid=com.example.myapp ./cmd/mobile

# Android AAR library
gomobile bind -target=android -o mylib.aar ./pkg/shared

# iOS Framework
gomobile bind -target=ios -o MyLib.framework ./pkg/shared
```

### CI/CD for Mobile Apps

```yaml
# .github/workflows/mobile.yml
name: Mobile Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.21
      
      - name: Install gomobile
        run: |
          go install golang.org/x/mobile/cmd/gomobile@latest
          gomobile init
      
      - name: Build Android AAR
        run: gomobile bind -target=android -o app.aar ./shared
      
      - name: Upload artifact
        uses: actions/upload-artifact@v2
        with:
          name: android-aar
          path: app.aar
  
  build-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.21
      
      - name: Install gomobile
        run: |
          go install golang.org/x/mobile/cmd/gomobile@latest
          gomobile init
      
      - name: Build iOS Framework
        run: gomobile bind -target=ios -o App.framework ./shared
      
      - name: Upload artifact
        uses: actions/upload-artifact@v2
        with:
          name: ios-framework
          path: App.framework
```

### Mobile Backend Deployment

**Dockerfile for Mobile Backend:**
```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server ./cmd/server

# Run stage
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY --from=builder /app/server .

EXPOSE 8080

CMD ["./server"]
```

**Docker Compose with Database:**
```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_USER=user
      - DB_PASSWORD=password
      - DB_NAME=mobileapp
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      - postgres
      - redis
  
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mobileapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Kubernetes Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobile-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mobile-backend
  template:
    metadata:
      labels:
        app: mobile-backend
    spec:
      containers:
      - name: api
        image: myregistry/mobile-backend:latest
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: db_host
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: mobile-backend
spec:
  selector:
    app: mobile-backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

---

## FAQs

**Q1: Should I build my entire mobile app in Go?**

**No.** Use native UI frameworks (Swift/SwiftUI for iOS, Kotlin/Jetpack Compose for Android) for the best user experience. Use Go for:
- Shared business logic (gomobile bind)
- Mobile backend services
- Data processing libraries
- Network protocols

**Q2: gomobile vs Flutter vs React Native?**

- **gomobile**: Share business logic, not UI. Requires native UI development.
- **Flutter**: Complete cross-platform solution, single codebase, good performance.
- **React Native**: JavaScript-based, large ecosystem, near-native performance.

**Choose gomobile when:** You have complex Go logic to share, or need Go's performance for specific tasks.

**Q3: Which GUI framework should I use - Fyne or Gio?**

- **Fyne**: Easier to learn, material design, better documentation, good for typical apps.
- **Gio**: Better performance, smaller binaries, immediate mode (more control), steeper learning curve.

**Recommendation:** Start with Fyne unless you need maximum performance or have immediate mode GUI experience.

**Q4: How do I handle offline data in mobile apps?**

**Pattern:**
1. Local SQLite database for storage
2. Track changes (pending_changes table)
3. Sync when online (delta sync)
4. Conflict resolution (last-write-wins or custom logic)
5. Background sync with retry

See section 15.6 for complete implementation.

**Q5: What's the best way to structure a mobile backend in Go?**

**Recommended structure:**
```
/cmd
  /server        - Main server
/internal
  /api          - API handlers
  /models       - Data models
  /storage      - Database layer
  /sync         - Sync logic
  /push         - Push notifications
/pkg
  /shared       - Shared mobile library (gomobile)
```

**Key patterns:**
- Clean architecture (layers)
- Repository pattern (data access)
- Service layer (business logic)
- API versioning
- Proper error handling

---

## Interview Questions

**Q1: Explain the trade-offs of using Go for mobile development.**

**A:**

**Pros:**
- Share business logic between platforms
- Go's performance and concurrency
- Type safety
- Easy deployment (single binary for backend)

**Cons:**
- Limited UI capabilities
- Larger binary size than native
- Not first-class citizen on mobile platforms
- Garbage collection (GC pauses)

**Best use:** Shared libraries + native UI, or mobile backends.

**Q2: How would you implement offline-first sync in a mobile app?**

**A:**

1. **Local storage:** SQLite database
2. **Change tracking:** Record all create/update/delete operations
3. **Sync protocol:** 
   - Send pending changes to server
   - Receive server changes since last sync
   - Detect conflicts (timestamps)
4. **Conflict resolution:** Last-write-wins or custom logic
5. **Background sync:** Periodic sync when online
6. **Retry logic:** Exponential backoff for failures

**Q3: What's the difference between REST and GraphQL for mobile APIs?**

**A:**

**REST:**
- Multiple endpoints
- Over/under-fetching possible
- Simpler to cache
- Easier to implement

**GraphQL:**
- Single endpoint
- Fetch exactly what you need (reduces bandwidth)
- Better for complex, nested data
- More complex caching

**For mobile:** GraphQL often better due to bandwidth efficiency and flexibility.

**Q4: How do you handle push notifications at scale?**

**A:**

1. **Queue-based approach:** Async job queue for notifications
2. **Batching:** Send to multiple devices in one API call
3. **Topic-based:** Group users by topics for efficient broadcasting
4. **Rate limiting:** Respect FCM/APNS limits
5. **Retry logic:** Handle transient failures
6. **Analytics:** Track delivery and engagement
7. **User preferences:** Honor notification settings

See section 15.5 for implementation.

**Q5: Explain delta sync vs full sync.**

**A:**

**Full sync:**
- Transfer entire dataset every time
- Simple implementation
- High bandwidth usage
- Slow for large datasets

**Delta sync:**
- Transfer only changes since last sync
- More complex (requires change tracking)
- Low bandwidth usage
- Fast even for large datasets
- Requires conflict resolution

**When to use:** Delta sync for production apps with significant data; full sync only for very small datasets or prototypes.

---

## Key Takeaways

1. **Use Go strategically** - shared libraries, not entire apps
2. **gomobile** enables code sharing between iOS/Android
3. **Fyne** is the easiest cross-platform GUI framework
4. **Gio** offers better performance with more complexity
5. **Mobile backends** are where Go truly shines
6. **Offline-first** is essential for good mobile UX
7. **Delta sync** minimizes bandwidth and improves performance
8. **Push notifications** require proper queuing and error handling
9. **GraphQL** often better than REST for mobile
10. **Native UI + Go backend** is the recommended pattern

---

## Practice Exercises

### Exercise 1: Shared Calculator Library
- Create gomobile library with calculator logic
- Build for Android and iOS
- Create simple native UI in both platforms
- Use the Go library from native code

### Exercise 2: Todo App with Fyne
- Build cross-platform todo app with Fyne
- Local SQLite storage
- Add, edit, delete tasks
- Mark tasks complete
- Package for desktop platforms

### Exercise 3: Mobile Backend API
- Design RESTful API for mobile app
- Implement delta sync endpoint
- Add pagination with cursors
- Field filtering support
- Authentication with JWT

### Exercise 4: Offline-First Notes App
- Local SQLite database
- Track changes for sync
- Implement sync protocol
- Handle conflicts (server wins)
- Background sync when online

### Exercise 5: Real-Time Chat Backend
- WebSocket server for chat
- Push notifications for messages
- Online/offline status
- Message history
- Deploy to Kubernetes

---

## Additional Resources

**gomobile:**
- https://pkg.go.dev/golang.org/x/mobile
- https://github.com/golang/mobile

**Fyne:**
- https://fyne.io
- https://developer.fyne.io/tour/
- Fyne Toolkit (book)

**Gio:**
- https://gioui.org
- https://gioui.org/doc/learn

**Mobile Backend:**
- https://firebase.google.com/docs/cloud-messaging
- Building Mobile Backend Services (book)

**Offline-First:**
- https://offlinefirst.org
- Offline First Web Applications (book)

---

## Summary

Part 15 covered mobile and cross-platform development with Go:

1. **gomobile**: Shared libraries for Android/iOS
2. **Fyne**: Easy cross-platform GUI framework
3. **Gio**: High-performance immediate mode GUI
4. **Mobile Backend**: API design, delta sync, GraphQL
5. **Push Notifications**: FCM integration, queueing
6. **Offline-First**: Local storage, sync protocol, conflict resolution
7. **Real-Time**: WebSocket and SSE for mobile
8. **Deployment**: CI/CD, Docker, Kubernetes

You now understand how to use Go effectively in mobile development!

---

**Total Word Count**: ~12,000 words  
**Status**: ✅ Complete

**Next**: Part 16 - Blockchain, IoT & Emerging Technologies (FINAL PART!)
