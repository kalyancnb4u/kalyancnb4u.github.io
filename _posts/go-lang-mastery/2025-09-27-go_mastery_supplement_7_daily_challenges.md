---
title: "Complete Go Mastery - 30-Day Coding Challenge"
date: 2025-01-11 09:00:00 +0530
categories: [Programming, Go, Practice]
tags: [golang, go, challenges, daily-practice, coding]
---

# Complete Go Mastery - 30-Day Coding Challenge

## Overview

Complete one challenge per day for 30 days. Each challenge builds on previous knowledge and takes 30-60 minutes.

**Rules**:
1. Write tests first (TDD)
2. Achieve 80%+ test coverage
3. Push to GitHub daily
4. Review solution next day

---

## Week 1: Fundamentals

### Day 1: FizzBuzz
**Difficulty**: 🟢 Easy

Write a program that prints numbers 1-100. For multiples of 3 print "Fizz", for 5 print "Buzz", for both print "FizzBuzz".

**Extension**: Make it configurable (any two numbers, any words).

```go
func FizzBuzz(n int) []string
func FizzBuzzCustom(n int, div1, div2 int, word1, word2 string) []string
```

---

### Day 2: Palindrome Checker
**Difficulty**: 🟢 Easy

Determine if a string is a palindrome (reads same forwards and backwards).

**Extension**: Ignore case, spaces, and punctuation.

```go
func IsPalindrome(s string) bool
func IsPalindromeAdvanced(s string) bool  // ignore non-alphanumeric
```

---

### Day 3: Anagram Detector
**Difficulty**: 🟢 Easy

Check if two strings are anagrams and group anagrams from a list.

```go
func IsAnagram(s1, s2 string) bool
func GroupAnagrams(words []string) [][]string
```

---

### Day 4: Roman Numerals
**Difficulty**: 🟡 Medium

Convert between integers and Roman numerals.

```go
func IntToRoman(num int) string  // 1-3999
func RomanToInt(s string) int
```

---

### Day 5: Word Frequency Counter
**Difficulty**: 🟢 Easy

Count word frequencies in text, return top N words.

```go
func WordFrequency(text string) map[string]int
func TopNWords(text string, n int) []struct{ Word string; Count int }
```

---

### Day 6: Unique Characters
**Difficulty**: 🟢 Easy

Determine if string has all unique characters without using additional data structures.

```go
func HasUniqueChars(s string) bool
func HasUniqueCharsNoDS(s string) bool  // O(1) space
```

---

### Day 7: String Compression
**Difficulty**: 🟡 Medium

Compress string using character counts. Return original if compressed isn't smaller.

**Example**: "aabcccccaaa" -> "a2b1c5a3"

```go
func Compress(s string) string
```

---

## Week 2: Data Structures

### Day 8: Stack Implementation
**Difficulty**: 🟡 Medium

Implement stack with Push, Pop, Peek, IsEmpty. Use generic types.

```go
type Stack[T any] struct { }
func (s *Stack[T]) Push(item T)
func (s *Stack[T]) Pop() (T, bool)
func (s *Stack[T]) Peek() (T, bool)
func (s *Stack[T]) IsEmpty() bool
```

---

### Day 9: Queue Implementation
**Difficulty**: 🟡 Medium

Implement queue with Enqueue, Dequeue, Front, IsEmpty.

```go
type Queue[T any] struct { }
func (q *Queue[T]) Enqueue(item T)
func (q *Queue[T]) Dequeue() (T, bool)
```

---

### Day 10: Linked List
**Difficulty**: 🟡 Medium

Implement singly linked list with Insert, Delete, Find, Reverse.

```go
type Node struct {
    Value int
    Next  *Node
}
type LinkedList struct { head *Node }
func (ll *LinkedList) Insert(value int)
func (ll *LinkedList) Delete(value int) bool
func (ll *LinkedList) Reverse()
```

---

### Day 11: Binary Search Tree
**Difficulty**: 🟡 Medium

Implement BST with Insert, Search, Delete, InOrder traversal.

```go
type TreeNode struct {
    Value int
    Left  *TreeNode
    Right *TreeNode
}
type BST struct { root *TreeNode }
func (bst *BST) Insert(value int)
func (bst *BST) Search(value int) bool
func (bst *BST) InOrder() []int
```

---

### Day 12: LRU Cache
**Difficulty**: 🔴 Hard

Implement Least Recently Used cache with O(1) get and put.

```go
type LRUCache struct { }
func NewLRUCache(capacity int) *LRUCache
func (lru *LRUCache) Get(key int) (int, bool)
func (lru *LRUCache) Put(key, value int)
```

---

### Day 13: Min Heap
**Difficulty**: 🟡 Medium

Implement min heap with Push, Pop, Peek.

```go
type MinHeap struct { }
func (h *MinHeap) Push(value int)
func (h *MinHeap) Pop() (int, bool)
func (h *MinHeap) Peek() (int, bool)
```

---

### Day 14: Trie (Prefix Tree)
**Difficulty**: 🔴 Hard

Implement trie for autocomplete functionality.

```go
type Trie struct { }
func (t *Trie) Insert(word string)
func (t *Trie) Search(word string) bool
func (t *Trie) StartsWith(prefix string) bool
func (t *Trie) AutoComplete(prefix string) []string
```

---

## Week 3: Algorithms

### Day 15: Sorting Algorithms
**Difficulty**: 🟡 Medium

Implement QuickSort, MergeSort, HeapSort.

```go
func QuickSort(arr []int) []int
func MergeSort(arr []int) []int
func HeapSort(arr []int) []int
```

---

### Day 16: Binary Search Variants
**Difficulty**: 🟡 Medium

Binary search, first occurrence, last occurrence, closest element.

```go
func BinarySearch(arr []int, target int) int
func FirstOccurrence(arr []int, target int) int
func LastOccurrence(arr []int, target int) int
```

---

### Day 17: Two Pointers
**Difficulty**: 🟡 Medium

Container with most water, 3Sum, Remove duplicates from sorted array.

```go
func MaxArea(height []int) int
func ThreeSum(nums []int) [][]int
func RemoveDuplicates(nums []int) int
```

---

### Day 18: Sliding Window
**Difficulty**: 🟡 Medium

Longest substring without repeating, min window substring.

```go
func LengthOfLongestSubstring(s string) int
func MinWindow(s, t string) string
```

---

### Day 19: Dynamic Programming (Easy)
**Difficulty**: 🟡 Medium

Fibonacci, climbing stairs, house robber.

```go
func Fibonacci(n int) int
func ClimbStairs(n int) int
func Rob(nums []int) int
```

---

### Day 20: Dynamic Programming (Medium)
**Difficulty**: 🔴 Hard

Coin change, longest increasing subsequence, edit distance.

```go
func CoinChange(coins []int, amount int) int
func LengthOfLIS(nums []int) int
func MinDistance(word1, word2 string) int
```

---

### Day 21: Graph Algorithms
**Difficulty**: 🔴 Hard

DFS, BFS, number of islands, course schedule.

```go
func NumIslands(grid [][]byte) int
func CanFinish(numCourses int, prerequisites [][]int) bool
```

---

## Week 4: Concurrency & Real-World

### Day 22: Concurrent Counter
**Difficulty**: 🟡 Medium

Thread-safe counter with increment, decrement, get.

```go
type Counter struct { }
func (c *Counter) Increment()
func (c *Counter) Decrement()
func (c *Counter) Get() int
```

---

### Day 23: Worker Pool
**Difficulty**: 🟡 Medium

Process jobs with fixed number of workers.

```go
type WorkerPool struct { }
func NewWorkerPool(workers int) *WorkerPool
func (wp *WorkerPool) Submit(job func())
func (wp *WorkerPool) Close()
```

---

### Day 24: Rate Limiter
**Difficulty**: 🔴 Hard

Token bucket rate limiter.

```go
type RateLimiter struct { }
func NewRateLimiter(rate int, burst int) *RateLimiter
func (rl *RateLimiter) Allow() bool
```

---

### Day 25: Pub/Sub System
**Difficulty**: 🔴 Hard

In-memory publish-subscribe system.

```go
type PubSub struct { }
func (ps *PubSub) Subscribe(topic string) <-chan interface{}
func (ps *PubSub) Publish(topic string, msg interface{})
```

---

### Day 26: URL Shortener
**Difficulty**: 🟡 Medium

Generate short URLs, redirect, track clicks.

```go
type URLShortener struct { }
func (us *URLShortener) Shorten(longURL string) string
func (us *URLShortener) Expand(shortURL string) (string, bool)
func (us *URLShortener) Clicks(shortURL string) int
```

---

### Day 27: Simple HTTP Server
**Difficulty**: 🟡 Medium

REST API for TODO items with CRUD operations.

```go
// Endpoints: GET /todos, POST /todos, PUT /todos/:id, DELETE /todos/:id
```

---

### Day 28: File Watcher
**Difficulty**: 🟡 Medium

Watch directory for file changes, notify subscribers.

```go
type FileWatcher struct { }
func (fw *FileWatcher) Watch(dir string) error
func (fw *FileWatcher) Subscribe() <-chan FileEvent
```

---

### Day 29: Command-Line Tool
**Difficulty**: 🟡 Medium

CLI tool with subcommands (like git). Use cobra or urfave/cli.

```bash
$ mytool command [flags]
$ mytool config set key value
$ mytool status
```

---

### Day 30: Capstone Challenge
**Difficulty**: 🔴 Hard

**Build a Chat Server**:
- WebSocket connections
- Multiple rooms
- User authentication
- Message history
- Concurrent clients (100+)

```go
type ChatServer struct { }
func (cs *ChatServer) HandleConnection(conn *websocket.Conn)
func (cs *ChatServer) CreateRoom(name string) error
func (cs *ChatServer) BroadcastToRoom(room, message string)
```

---

## Bonus Challenges (Days 31-40)

### Day 31: In-Memory Database
Simple key-value store with transactions.

### Day 32: Load Balancer
Round-robin load balancer for HTTP requests.

### Day 33: Circuit Breaker
Fault-tolerance pattern implementation.

### Day 34: Event Bus
Event-driven communication system.

### Day 35: Template Engine
Simple text template rendering.

### Day 36: JSON Parser
Parse JSON without using encoding/json.

### Day 37: HTTP Middleware Chain
Composable middleware system.

### Day 38: Connection Pool
Generic connection pool implementation.

### Day 39: Metrics Collector
Prometheus-style metrics collection.

### Day 40: Distributed Lock
Redis-based distributed locking.

---

## Daily Workflow

**Morning (15 min)**:
1. Read challenge requirements
2. Plan approach
3. Write pseudocode

**Coding (30-45 min)**:
1. Write tests first
2. Implement solution
3. Refactor

**Evening (15 min)**:
1. Review code
2. Run tests
3. Push to GitHub
4. Update tracker

---

## Success Metrics

- [ ] All 30 challenges completed
- [ ] 80%+ test coverage on each
- [ ] All code on GitHub
- [ ] Code reviewed by peer (optional)
- [ ] Blog post about favorite challenge (optional)

---

**Start tomorrow! One challenge a day keeps unemployment away! 🚀**
