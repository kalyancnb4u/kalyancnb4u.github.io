---
title: "Go Mastery - Part 13: Advanced Networking"
date: 2025-09-15 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, Programming, Networking, TCP, UDP, Protocols, DNS, Load-balancer, Proxy]
---

# Part 13: Advanced Networking

## Overview

This part dives deep into network programming with Go, covering low-level TCP/UDP protocols, custom protocol design, building load balancers and proxies, DNS programming, and service discovery. You'll learn how to build network infrastructure and understand how systems like nginx, HAProxy, and DNS servers work internally.

**What You'll Learn:**
- Network programming fundamentals (OSI model, TCP/IP stack)
- TCP server and client programming
- UDP programming and reliability patterns
- Designing custom binary and text protocols
- Building production-grade load balancers
- Implementing reverse and forward proxies
- DNS server implementation
- Service discovery with Consul and etcd
- Network utilities and tools

**Prerequisites:**
- Part 2 (Concurrency patterns)
- Part 5 (Architecture patterns)
- Basic networking knowledge (IP, ports, sockets)

---

## 13.1 Network Programming Fundamentals

### OSI and TCP/IP Models

**OSI Model (7 Layers):**
1. **Physical**: Hardware, cables, signals
2. **Data Link**: MAC addresses, Ethernet
3. **Network**: IP addressing, routing
4. **Transport**: TCP, UDP (ports, reliability)
5. **Session**: Connection management
6. **Presentation**: Data encoding, encryption
7. **Application**: HTTP, FTP, DNS

**TCP/IP Model (4 Layers):**
1. **Link**: Network interface (Ethernet, WiFi)
2. **Internet**: IP (routing, addressing)
3. **Transport**: TCP, UDP
4. **Application**: HTTP, DNS, SSH

**Go operates primarily at:**
- Transport layer (TCP, UDP)
- Application layer (HTTP, custom protocols)

### Socket Programming Basics

A socket is an endpoint for network communication.

**Socket Types:**
- **Stream Sockets (SOCK_STREAM)**: TCP - reliable, ordered, connection-oriented
- **Datagram Sockets (SOCK_DGRAM)**: UDP - unreliable, unordered, connectionless
- **Raw Sockets (SOCK_RAW)**: Direct IP access (requires privileges)

### The net Package

Go's `net` package provides networking primitives:

```go
import "net"

// Key types:
// - net.Conn: Network connection interface
// - net.Listener: Accepts incoming connections
// - net.Addr: Network address interface
// - net.Dialer: Connection dialer with options
```

### Address Resolution

```go
package main

import (
    "fmt"
    "net"
)

func main() {
    // Resolve hostname to IP
    ips, err := net.LookupIP("google.com")
    if err != nil {
        panic(err)
    }
    
    for _, ip := range ips {
        fmt.Println(ip)
    }
    
    // Resolve hostname to addresses
    addrs, err := net.LookupHost("google.com")
    if err != nil {
        panic(err)
    }
    
    for _, addr := range addrs {
        fmt.Println(addr)
    }
    
    // Reverse lookup (IP to hostname)
    names, err := net.LookupAddr("8.8.8.8")
    if err != nil {
        panic(err)
    }
    
    for _, name := range names {
        fmt.Println(name)
    }
}
```

---

## 13.2 TCP Programming

### TCP Fundamentals

TCP (Transmission Control Protocol) provides:
- **Reliable delivery**: Retransmits lost packets
- **Ordered delivery**: Packets arrive in order
- **Connection-oriented**: Establish before sending
- **Flow control**: Prevents overwhelming receiver
- **Congestion control**: Adapts to network conditions

**TCP Three-Way Handshake:**
1. Client → Server: SYN
2. Server → Client: SYN-ACK
3. Client → Server: ACK

### Basic TCP Server

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
)

func main() {
    // Listen on TCP port 8080
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatal(err)
    }
    defer listener.Close()
    
    log.Println("Server listening on :8080")
    
    for {
        // Accept incoming connection
        conn, err := listener.Accept()
        if err != nil {
            log.Println("Accept error:", err)
            continue
        }
        
        // Handle connection in goroutine
        go handleConnection(conn)
    }
}

func handleConnection(conn net.Conn) {
    defer conn.Close()
    
    log.Printf("New connection from %s", conn.RemoteAddr())
    
    // Read from connection
    scanner := bufio.NewScanner(conn)
    for scanner.Scan() {
        message := scanner.Text()
        log.Printf("Received: %s", message)
        
        // Echo back
        fmt.Fprintf(conn, "Echo: %s\n", message)
    }
    
    if err := scanner.Err(); err != nil {
        log.Printf("Connection error: %v", err)
    }
    
    log.Printf("Connection closed: %s", conn.RemoteAddr())
}
```

### TCP Client

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "os"
)

func main() {
    // Connect to server
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()
    
    log.Println("Connected to server")
    
    // Read from stdin, write to connection
    scanner := bufio.NewScanner(os.Stdin)
    for scanner.Scan() {
        message := scanner.Text()
        
        // Send to server
        fmt.Fprintf(conn, "%s\n", message)
        
        // Read response
        response, err := bufio.NewReader(conn).ReadString('\n')
        if err != nil {
            log.Fatal(err)
        }
        
        fmt.Print("Server: " + response)
    }
}
```

### Connection with Timeout

```go
func connectWithTimeout(address string, timeout time.Duration) (net.Conn, error) {
    dialer := net.Dialer{
        Timeout: timeout,
    }
    
    return dialer.Dial("tcp", address)
}

// Usage
conn, err := connectWithTimeout("localhost:8080", 5*time.Second)
```

### Setting Deadlines

```go
func handleConnectionWithDeadline(conn net.Conn) {
    defer conn.Close()
    
    // Set read deadline (30 seconds)
    conn.SetReadDeadline(time.Now().Add(30 * time.Second))
    
    // Set write deadline
    conn.SetWriteDeadline(time.Now().Add(30 * time.Second))
    
    // Set deadline for both read and write
    conn.SetDeadline(time.Now().Add(30 * time.Second))
    
    // Read with deadline
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil {
        if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
            log.Println("Read timeout")
        } else {
            log.Println("Read error:", err)
        }
        return
    }
    
    log.Printf("Read %d bytes: %s", n, buffer[:n])
}
```

### TCP Keep-Alive

```go
func enableKeepAlive(conn net.Conn) error {
    tcpConn, ok := conn.(*net.TCPConn)
    if !ok {
        return fmt.Errorf("not a TCP connection")
    }
    
    // Enable keep-alive
    if err := tcpConn.SetKeepAlive(true); err != nil {
        return err
    }
    
    // Set keep-alive period
    if err := tcpConn.SetKeepAlivePeriod(30 * time.Second); err != nil {
        return err
    }
    
    return nil
}
```

### TCP Server with Graceful Shutdown

```go
type TCPServer struct {
    listener net.Listener
    conns    map[net.Conn]struct{}
    mu       sync.Mutex
    wg       sync.WaitGroup
    shutdown chan struct{}
}

func NewTCPServer(addr string) (*TCPServer, error) {
    listener, err := net.Listen("tcp", addr)
    if err != nil {
        return nil, err
    }
    
    return &TCPServer{
        listener: listener,
        conns:    make(map[net.Conn]struct{}),
        shutdown: make(chan struct{}),
    }, nil
}

func (s *TCPServer) Start() error {
    defer s.wg.Wait()
    
    for {
        conn, err := s.listener.Accept()
        if err != nil {
            select {
            case <-s.shutdown:
                return nil
            default:
                log.Println("Accept error:", err)
                continue
            }
        }
        
        s.mu.Lock()
        s.conns[conn] = struct{}{}
        s.mu.Unlock()
        
        s.wg.Add(1)
        go s.handleConnection(conn)
    }
}

func (s *TCPServer) handleConnection(conn net.Conn) {
    defer s.wg.Done()
    defer func() {
        s.mu.Lock()
        delete(s.conns, conn)
        s.mu.Unlock()
        conn.Close()
    }()
    
    // Handle connection...
    scanner := bufio.NewScanner(conn)
    for scanner.Scan() {
        select {
        case <-s.shutdown:
            return
        default:
            // Process message
            message := scanner.Text()
            fmt.Fprintf(conn, "Echo: %s\n", message)
        }
    }
}

func (s *TCPServer) Shutdown() error {
    close(s.shutdown)
    
    // Stop accepting new connections
    s.listener.Close()
    
    // Close existing connections
    s.mu.Lock()
    for conn := range s.conns {
        conn.Close()
    }
    s.mu.Unlock()
    
    // Wait for all handlers to finish
    s.wg.Wait()
    
    return nil
}
```

**Usage:**
```go
func main() {
    server, err := NewTCPServer(":8080")
    if err != nil {
        log.Fatal(err)
    }
    
    // Handle shutdown signal
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)
    
    go func() {
        <-sigChan
        log.Println("Shutting down server...")
        server.Shutdown()
    }()
    
    log.Println("Server starting on :8080")
    if err := server.Start(); err != nil {
        log.Fatal(err)
    }
}
```

### Connection Pool

```go
type ConnectionPool struct {
    address  string
    maxConns int
    conns    chan net.Conn
    mu       sync.Mutex
    closed   bool
}

func NewConnectionPool(address string, maxConns int) *ConnectionPool {
    return &ConnectionPool{
        address:  address,
        maxConns: maxConns,
        conns:    make(chan net.Conn, maxConns),
    }
}

func (p *ConnectionPool) Get() (net.Conn, error) {
    p.mu.Lock()
    if p.closed {
        p.mu.Unlock()
        return nil, fmt.Errorf("pool is closed")
    }
    p.mu.Unlock()
    
    select {
    case conn := <-p.conns:
        // Reuse existing connection
        return conn, nil
    default:
        // Create new connection
        return net.Dial("tcp", p.address)
    }
}

func (p *ConnectionPool) Put(conn net.Conn) {
    p.mu.Lock()
    if p.closed {
        p.mu.Unlock()
        conn.Close()
        return
    }
    p.mu.Unlock()
    
    select {
    case p.conns <- conn:
        // Connection returned to pool
    default:
        // Pool is full, close connection
        conn.Close()
    }
}

func (p *ConnectionPool) Close() {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    if p.closed {
        return
    }
    
    p.closed = true
    close(p.conns)
    
    for conn := range p.conns {
        conn.Close()
    }
}
```

### TCP Optimization

**Nagle's Algorithm:**
```go
// Disable Nagle's algorithm for low-latency applications
tcpConn.SetNoDelay(true) // Disable buffering, send immediately
tcpConn.SetNoDelay(false) // Enable Nagle (default)
```

**Buffer Sizes:**
```go
// Set read buffer size
tcpConn.SetReadBuffer(1024 * 1024) // 1 MB

// Set write buffer size
tcpConn.SetWriteBuffer(1024 * 1024) // 1 MB
```

**Linger:**
```go
// Set linger timeout (time to wait for unsent data on close)
tcpConn.SetLinger(10) // Wait 10 seconds
tcpConn.SetLinger(0)  // Close immediately, discard unsent data
tcpConn.SetLinger(-1) // Use system default
```

---

## 13.3 UDP Programming

### UDP Fundamentals

UDP (User Datagram Protocol) provides:
- **Connectionless**: No handshake
- **Unreliable**: No guaranteed delivery
- **Unordered**: Packets may arrive out of order
- **Faster**: Lower overhead than TCP
- **Message-oriented**: Preserves message boundaries

**Use Cases:**
- DNS queries
- Video/audio streaming
- Online gaming
- IoT sensor data
- Metrics collection
- Broadcasting/multicasting

### UDP Server

```go
package main

import (
    "fmt"
    "log"
    "net"
)

func main() {
    // Listen on UDP port 8080
    addr, err := net.ResolveUDPAddr("udp", ":8080")
    if err != nil {
        log.Fatal(err)
    }
    
    conn, err := net.ListenUDP("udp", addr)
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()
    
    log.Println("UDP server listening on :8080")
    
    buffer := make([]byte, 1024)
    
    for {
        // Read datagram
        n, clientAddr, err := conn.ReadFromUDP(buffer)
        if err != nil {
            log.Println("Read error:", err)
            continue
        }
        
        message := string(buffer[:n])
        log.Printf("Received from %s: %s", clientAddr, message)
        
        // Send response
        response := fmt.Sprintf("Echo: %s", message)
        _, err = conn.WriteToUDP([]byte(response), clientAddr)
        if err != nil {
            log.Println("Write error:", err)
        }
    }
}
```

### UDP Client

```go
package main

import (
    "fmt"
    "log"
    "net"
)

func main() {
    // Resolve server address
    serverAddr, err := net.ResolveUDPAddr("udp", "localhost:8080")
    if err != nil {
        log.Fatal(err)
    }
    
    // Create UDP connection
    conn, err := net.DialUDP("udp", nil, serverAddr)
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()
    
    // Send message
    message := []byte("Hello, UDP!")
    _, err = conn.Write(message)
    if err != nil {
        log.Fatal(err)
    }
    
    // Receive response
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Server response: %s\n", buffer[:n])
}
```

### UDP Broadcast

```go
func broadcastUDP(message string, port int) error {
    addr, err := net.ResolveUDPAddr("udp", fmt.Sprintf("255.255.255.255:%d", port))
    if err != nil {
        return err
    }
    
    conn, err := net.DialUDP("udp", nil, addr)
    if err != nil {
        return err
    }
    defer conn.Close()
    
    _, err = conn.Write([]byte(message))
    return err
}

func receiveBroadcast(port int) error {
    addr, err := net.ResolveUDPAddr("udp", fmt.Sprintf(":%d", port))
    if err != nil {
        return err
    }
    
    conn, err := net.ListenUDP("udp", addr)
    if err != nil {
        return err
    }
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    for {
        n, remoteAddr, err := conn.ReadFromUDP(buffer)
        if err != nil {
            return err
        }
        
        log.Printf("Broadcast from %s: %s", remoteAddr, buffer[:n])
    }
}
```

### UDP Multicast

```go
func sendMulticast(group string, port int, message string) error {
    addr, err := net.ResolveUDPAddr("udp", fmt.Sprintf("%s:%d", group, port))
    if err != nil {
        return err
    }
    
    conn, err := net.DialUDP("udp", nil, addr)
    if err != nil {
        return err
    }
    defer conn.Close()
    
    _, err = conn.Write([]byte(message))
    return err
}

func receiveMulticast(group string, port int) error {
    addr, err := net.ResolveUDPAddr("udp", fmt.Sprintf("%s:%d", group, port))
    if err != nil {
        return err
    }
    
    conn, err := net.ListenMulticastUDP("udp", nil, addr)
    if err != nil {
        return err
    }
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    for {
        n, remoteAddr, err := conn.ReadFromUDP(buffer)
        if err != nil {
            return err
        }
        
        log.Printf("Multicast from %s: %s", remoteAddr, buffer[:n])
    }
}

// Usage
go receiveMulticast("224.0.0.1", 9999)
time.Sleep(time.Second)
sendMulticast("224.0.0.1", 9999, "Hello, multicast!")
```

### Reliable UDP Pattern

Implementing reliability on top of UDP:

```go
type ReliableUDP struct {
    conn         *net.UDPConn
    pendingAcks  map[uint32]*Packet
    mu           sync.Mutex
    nextSeq      uint32
    retryTimeout time.Duration
    maxRetries   int
}

type Packet struct {
    Seq     uint32
    Ack     uint32
    Data    []byte
    SentAt  time.Time
    Retries int
}

func (r *ReliableUDP) Send(data []byte, addr *net.UDPAddr) error {
    r.mu.Lock()
    seq := r.nextSeq
    r.nextSeq++
    r.mu.Unlock()
    
    packet := &Packet{
        Seq:    seq,
        Data:   data,
        SentAt: time.Now(),
    }
    
    // Serialize packet
    packetBytes := serializePacket(packet)
    
    // Send packet
    _, err := r.conn.WriteToUDP(packetBytes, addr)
    if err != nil {
        return err
    }
    
    // Store for potential retransmission
    r.mu.Lock()
    r.pendingAcks[seq] = packet
    r.mu.Unlock()
    
    // Start retry timer
    go r.retryIfNeeded(seq, addr)
    
    return nil
}

func (r *ReliableUDP) retryIfNeeded(seq uint32, addr *net.UDPAddr) {
    ticker := time.NewTicker(r.retryTimeout)
    defer ticker.Stop()
    
    for range ticker.C {
        r.mu.Lock()
        packet, exists := r.pendingAcks[seq]
        if !exists {
            r.mu.Unlock()
            return // ACK received
        }
        
        packet.Retries++
        if packet.Retries > r.maxRetries {
            delete(r.pendingAcks, seq)
            r.mu.Unlock()
            log.Printf("Packet %d dropped after %d retries", seq, r.maxRetries)
            return
        }
        r.mu.Unlock()
        
        // Retransmit
        packetBytes := serializePacket(packet)
        r.conn.WriteToUDP(packetBytes, addr)
        log.Printf("Retransmitting packet %d (attempt %d)", seq, packet.Retries)
    }
}

func (r *ReliableUDP) handleAck(seq uint32) {
    r.mu.Lock()
    delete(r.pendingAcks, seq)
    r.mu.Unlock()
}

func serializePacket(p *Packet) []byte {
    buf := new(bytes.Buffer)
    binary.Write(buf, binary.BigEndian, p.Seq)
    binary.Write(buf, binary.BigEndian, p.Ack)
    binary.Write(buf, binary.BigEndian, uint32(len(p.Data)))
    buf.Write(p.Data)
    return buf.Bytes()
}
```

---

*I'll continue with Custom Protocol Design, Load Balancer Implementation, and complete Part 13. Shall I continue?*

## 13.4 Custom Protocol Design

### Protocol Design Principles

**Good protocol design:**
- **Versioning**: Support protocol evolution
- **Extensibility**: Allow adding fields without breaking
- **Error handling**: Clear error codes and messages
- **Efficiency**: Minimize overhead
- **Security**: Authentication, encryption support

### Binary Protocol Design

**Message Format:**
```
+--------+--------+--------+--------+
| Magic  |Version | Type   | Length |  (Header: 4 bytes)
+--------+--------+--------+--------+
|                                   |
|           Payload                 |  (Variable length)
|                                   |
+-----------------------------------+
|            Checksum               |  (Footer: 4 bytes)
+-----------------------------------+
```

**Implementation:**
```go
package protocol

import (
    "encoding/binary"
    "hash/crc32"
    "io"
)

const (
    MagicNumber  = 0xCAFE
    ProtocolVersion = 1
    
    // Message types
    TypePing     = 0x01
    TypePong     = 0x02
    TypeRequest  = 0x10
    TypeResponse = 0x11
    TypeError    = 0xFF
)

type Message struct {
    Type    uint8
    Payload []byte
}

// Encode message to binary
func (m *Message) Encode() ([]byte, error) {
    payloadLen := len(m.Payload)
    totalLen := 4 + 4 + payloadLen + 4 // header + payload + checksum
    
    buf := make([]byte, totalLen)
    
    // Write header
    binary.BigEndian.PutUint16(buf[0:2], MagicNumber)
    buf[2] = ProtocolVersion
    buf[3] = m.Type
    binary.BigEndian.PutUint32(buf[4:8], uint32(payloadLen))
    
    // Write payload
    copy(buf[8:], m.Payload)
    
    // Calculate and write checksum
    checksum := crc32.ChecksumIEEE(buf[:8+payloadLen])
    binary.BigEndian.PutUint32(buf[8+payloadLen:], checksum)
    
    return buf, nil
}

// Decode message from binary
func DecodeMessage(r io.Reader) (*Message, error) {
    // Read header
    header := make([]byte, 8)
    if _, err := io.ReadFull(r, header); err != nil {
        return nil, err
    }
    
    // Verify magic number
    magic := binary.BigEndian.Uint16(header[0:2])
    if magic != MagicNumber {
        return nil, fmt.Errorf("invalid magic number: %x", magic)
    }
    
    // Verify version
    version := header[2]
    if version != ProtocolVersion {
        return nil, fmt.Errorf("unsupported version: %d", version)
    }
    
    msgType := header[3]
    payloadLen := binary.BigEndian.Uint32(header[4:8])
    
    // Read payload
    payload := make([]byte, payloadLen)
    if _, err := io.ReadFull(r, payload); err != nil {
        return nil, err
    }
    
    // Read and verify checksum
    checksumBytes := make([]byte, 4)
    if _, err := io.ReadFull(r, checksumBytes); err != nil {
        return nil, err
    }
    
    expectedChecksum := binary.BigEndian.Uint32(checksumBytes)
    actualChecksum := crc32.ChecksumIEEE(append(header, payload...))
    
    if expectedChecksum != actualChecksum {
        return nil, fmt.Errorf("checksum mismatch")
    }
    
    return &Message{
        Type:    msgType,
        Payload: payload,
    }, nil
}
```

**Usage:**
```go
func main() {
    // Create message
    msg := &Message{
        Type:    TypeRequest,
        Payload: []byte("Hello, Protocol!"),
    }
    
    // Encode
    encoded, _ := msg.Encode()
    
    // Decode
    reader := bytes.NewReader(encoded)
    decoded, _ := DecodeMessage(reader)
    
    fmt.Printf("Decoded: Type=%d, Payload=%s\n", decoded.Type, decoded.Payload)
}
```

### Text-Based Protocol (Line-Based)

**Example: Simple Command Protocol**
```
COMMAND arg1 arg2 arg3\r\n
```

**Implementation:**
```go
type CommandProtocol struct {
    conn net.Conn
}

func (p *CommandProtocol) SendCommand(cmd string, args ...string) error {
    message := cmd
    if len(args) > 0 {
        message += " " + strings.Join(args, " ")
    }
    message += "\r\n"
    
    _, err := p.conn.Write([]byte(message))
    return err
}

func (p *CommandProtocol) ReadCommand() (string, []string, error) {
    reader := bufio.NewReader(p.conn)
    line, err := reader.ReadString('\n')
    if err != nil {
        return "", nil, err
    }
    
    line = strings.TrimSpace(line)
    parts := strings.Fields(line)
    
    if len(parts) == 0 {
        return "", nil, fmt.Errorf("empty command")
    }
    
    cmd := parts[0]
    args := parts[1:]
    
    return cmd, args, nil
}

// Example commands:
// SET key value
// GET key
// DELETE key
// PING
```

### Request-Response Pattern

```go
type RequestResponse struct {
    conn        net.Conn
    nextID      uint32
    pending     map[uint32]chan *Message
    mu          sync.Mutex
}

func NewRequestResponse(conn net.Conn) *RequestResponse {
    rr := &RequestResponse{
        conn:    conn,
        pending: make(map[uint32]chan *Message),
    }
    
    go rr.readLoop()
    
    return rr
}

func (rr *RequestResponse) Request(payload []byte) (*Message, error) {
    rr.mu.Lock()
    id := rr.nextID
    rr.nextID++
    responseChan := make(chan *Message, 1)
    rr.pending[id] = responseChan
    rr.mu.Unlock()
    
    // Send request
    req := &Message{
        Type:    TypeRequest,
        Payload: append([]byte{byte(id >> 24), byte(id >> 16), byte(id >> 8), byte(id)}, payload...),
    }
    
    encoded, _ := req.Encode()
    if _, err := rr.conn.Write(encoded); err != nil {
        rr.mu.Lock()
        delete(rr.pending, id)
        rr.mu.Unlock()
        return nil, err
    }
    
    // Wait for response
    select {
    case response := <-responseChan:
        return response, nil
    case <-time.After(5 * time.Second):
        rr.mu.Lock()
        delete(rr.pending, id)
        rr.mu.Unlock()
        return nil, fmt.Errorf("request timeout")
    }
}

func (rr *RequestResponse) readLoop() {
    for {
        msg, err := DecodeMessage(rr.conn)
        if err != nil {
            log.Printf("Read error: %v", err)
            return
        }
        
        if msg.Type == TypeResponse {
            // Extract request ID
            if len(msg.Payload) < 4 {
                continue
            }
            
            id := binary.BigEndian.Uint32(msg.Payload[:4])
            msg.Payload = msg.Payload[4:]
            
            rr.mu.Lock()
            if ch, exists := rr.pending[id]; exists {
                ch <- msg
                delete(rr.pending, id)
            }
            rr.mu.Unlock()
        }
    }
}
```

---

## 13.5 Load Balancer Implementation

### Load Balancing Algorithms

**1. Round Robin:**
```go
type RoundRobin struct {
    backends []*Backend
    current  uint32
}

func (rr *RoundRobin) Next() *Backend {
    n := atomic.AddUint32(&rr.current, 1)
    return rr.backends[(int(n)-1)%len(rr.backends)]
}
```

**2. Least Connections:**
```go
type LeastConnections struct {
    backends []*Backend
    mu       sync.RWMutex
}

func (lc *LeastConnections) Next() *Backend {
    lc.mu.RLock()
    defer lc.mu.RUnlock()
    
    var selected *Backend
    minConns := int(^uint(0) >> 1) // Max int
    
    for _, backend := range lc.backends {
        if backend.ActiveConnections() < minConns {
            minConns = backend.ActiveConnections()
            selected = backend
        }
    }
    
    return selected
}
```

**3. IP Hash (Sticky Sessions):**
```go
type IPHash struct {
    backends []*Backend
}

func (ih *IPHash) Next(clientIP string) *Backend {
    hash := fnv.New32a()
    hash.Write([]byte(clientIP))
    index := int(hash.Sum32()) % len(ih.backends)
    return ih.backends[index]
}
```

**4. Weighted Round Robin:**
```go
type WeightedRoundRobin struct {
    backends []*Backend
    weights  []int
    current  int
    gcd      int
    maxWeight int
    mu       sync.Mutex
}

func (wrr *WeightedRoundRobin) Next() *Backend {
    wrr.mu.Lock()
    defer wrr.mu.Unlock()
    
    for {
        wrr.current = (wrr.current + 1) % len(wrr.backends)
        
        if wrr.current == 0 {
            wrr.maxWeight -= wrr.gcd
            if wrr.maxWeight <= 0 {
                wrr.maxWeight = wrr.getMaxWeight()
            }
        }
        
        if wrr.weights[wrr.current] >= wrr.maxWeight {
            return wrr.backends[wrr.current]
        }
    }
}
```

### Backend Health Checking

```go
type Backend struct {
    URL              *url.URL
    Alive            bool
    mu               sync.RWMutex
    activeConns      int32
    ReverseProxy     *httputil.ReverseProxy
}

func (b *Backend) SetAlive(alive bool) {
    b.mu.Lock()
    b.Alive = alive
    b.mu.Unlock()
}

func (b *Backend) IsAlive() bool {
    b.mu.RLock()
    alive := b.Alive
    b.mu.RUnlock()
    return alive
}

func (b *Backend) ActiveConnections() int {
    return int(atomic.LoadInt32(&b.activeConns))
}

func (b *Backend) IncrementConnections() {
    atomic.AddInt32(&b.activeConns, 1)
}

func (b *Backend) DecrementConnections() {
    atomic.AddInt32(&b.activeConns, -1)
}

// Health check
func (b *Backend) HealthCheck() {
    timeout := 2 * time.Second
    client := &http.Client{Timeout: timeout}
    
    resp, err := client.Get(b.URL.String() + "/health")
    if err != nil || resp.StatusCode != http.StatusOK {
        b.SetAlive(false)
        return
    }
    
    b.SetAlive(true)
}
```

### Complete Load Balancer

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "net/http/httputil"
    "net/url"
    "sync/atomic"
    "time"
)

type LoadBalancer struct {
    backends []*Backend
    current  uint32
}

func NewLoadBalancer(backendURLs []string) *LoadBalancer {
    backends := make([]*Backend, len(backendURLs))
    
    for i, urlStr := range backendURLs {
        url, _ := url.Parse(urlStr)
        backends[i] = &Backend{
            URL:          url,
            Alive:        true,
            ReverseProxy: httputil.NewSingleHostReverseProxy(url),
        }
    }
    
    lb := &LoadBalancer{
        backends: backends,
    }
    
    // Start health checks
    go lb.healthCheckLoop()
    
    return lb
}

func (lb *LoadBalancer) getNextBackend() *Backend {
    // Round-robin selection
    for i := 0; i < len(lb.backends); i++ {
        idx := atomic.AddUint32(&lb.current, 1) % uint32(len(lb.backends))
        backend := lb.backends[idx]
        
        if backend.IsAlive() {
            return backend
        }
    }
    
    return nil
}

func (lb *LoadBalancer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    backend := lb.getNextBackend()
    if backend == nil {
        http.Error(w, "No backends available", http.StatusServiceUnavailable)
        return
    }
    
    backend.IncrementConnections()
    defer backend.DecrementConnections()
    
    // Add load balancer headers
    r.Header.Set("X-Forwarded-For", r.RemoteAddr)
    r.Header.Set("X-Forwarded-Host", r.Host)
    
    backend.ReverseProxy.ServeHTTP(w, r)
}

func (lb *LoadBalancer) healthCheckLoop() {
    ticker := time.NewTicker(10 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        for _, backend := range lb.backends {
            go backend.HealthCheck()
        }
    }
}

func main() {
    backends := []string{
        "http://localhost:8081",
        "http://localhost:8082",
        "http://localhost:8083",
    }
    
    lb := NewLoadBalancer(backends)
    
    server := &http.Server{
        Addr:         ":8080",
        Handler:      lb,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
    }
    
    log.Println("Load balancer starting on :8080")
    log.Fatal(server.ListenAndServe())
}
```

### Circuit Breaker Pattern

```go
type CircuitBreaker struct {
    maxFailures  int
    timeout      time.Duration
    failures     int
    lastAttempt  time.Time
    state        string // "closed", "open", "half-open"
    mu           sync.Mutex
}

func NewCircuitBreaker(maxFailures int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        maxFailures: maxFailures,
        timeout:     timeout,
        state:       "closed",
    }
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    if cb.state == "open" {
        if time.Since(cb.lastAttempt) > cb.timeout {
            cb.state = "half-open"
        } else {
            return fmt.Errorf("circuit breaker is open")
        }
    }
    
    err := fn()
    
    if err != nil {
        cb.failures++
        cb.lastAttempt = time.Now()
        
        if cb.failures >= cb.maxFailures {
            cb.state = "open"
        }
        
        return err
    }
    
    // Success
    cb.failures = 0
    cb.state = "closed"
    return nil
}
```

---

## 13.6 DNS Programming

### DNS Basics

DNS (Domain Name System) translates domain names to IP addresses.

**DNS Record Types:**
- **A**: IPv4 address
- **AAAA**: IPv6 address
- **CNAME**: Canonical name (alias)
- **MX**: Mail exchanger
- **NS**: Name server
- **TXT**: Text records
- **SOA**: Start of authority

### DNS Queries with net Package

```go
func dnsLookups() {
    // Lookup A records
    ips, _ := net.LookupIP("google.com")
    for _, ip := range ips {
        fmt.Println("IP:", ip)
    }
    
    // Lookup CNAME
    cname, _ := net.LookupCNAME("www.google.com")
    fmt.Println("CNAME:", cname)
    
    // Lookup MX records
    mxs, _ := net.LookupMX("gmail.com")
    for _, mx := range mxs {
        fmt.Printf("MX: %s (priority %d)\n", mx.Host, mx.Pref)
    }
    
    // Lookup NS records
    nss, _ := net.LookupNS("google.com")
    for _, ns := range nss {
        fmt.Println("NS:", ns.Host)
    }
    
    // Lookup TXT records
    txts, _ := net.LookupTXT("google.com")
    for _, txt := range txts {
        fmt.Println("TXT:", txt)
    }
}
```

### Custom DNS Resolver

```go
func customResolver() {
    resolver := &net.Resolver{
        PreferGo: true,
        Dial: func(ctx context.Context, network, address string) (net.Conn, error) {
            d := net.Dialer{
                Timeout: 2 * time.Second,
            }
            // Use Google's DNS
            return d.DialContext(ctx, network, "8.8.8.8:53")
        },
    }
    
    ips, err := resolver.LookupIP(context.Background(), "ip4", "example.com")
    if err != nil {
        log.Fatal(err)
    }
    
    for _, ip := range ips {
        fmt.Println(ip)
    }
}
```

### Building a DNS Server (with miekg/dns)

```bash
go get github.com/miekg/dns
```

**Simple DNS Server:**
```go
package main

import (
    "log"
    "net"
    
    "github.com/miekg/dns"
)

type DNSHandler struct {
    records map[string]string
}

func (h *DNSHandler) ServeDNS(w dns.ResponseWriter, r *dns.Msg) {
    msg := new(dns.Msg)
    msg.SetReply(r)
    msg.Authoritative = true
    
    for _, question := range r.Question {
        log.Printf("Query: %s", question.Name)
        
        switch question.Qtype {
        case dns.TypeA:
            if ip, exists := h.records[question.Name]; exists {
                rr := &dns.A{
                    Hdr: dns.RR_Header{
                        Name:   question.Name,
                        Rrtype: dns.TypeA,
                        Class:  dns.ClassINET,
                        Ttl:    300,
                    },
                    A: net.ParseIP(ip),
                }
                msg.Answer = append(msg.Answer, rr)
            }
        }
    }
    
    w.WriteMsg(msg)
}

func main() {
    handler := &DNSHandler{
        records: map[string]string{
            "example.local.": "192.168.1.10",
            "test.local.":    "192.168.1.20",
        },
    }
    
    server := &dns.Server{
        Addr:    ":53",
        Net:     "udp",
        Handler: handler,
    }
    
    log.Println("DNS server starting on :53")
    log.Fatal(server.ListenAndServe())
}
```

**DNS Forwarding Server:**
```go
type ForwardingDNSHandler struct {
    upstream string
}

func (h *ForwardingDNSHandler) ServeDNS(w dns.ResponseWriter, r *dns.Msg) {
    // Forward to upstream DNS
    client := new(dns.Client)
    response, _, err := client.Exchange(r, h.upstream)
    if err != nil {
        log.Printf("Forward error: %v", err)
        return
    }
    
    w.WriteMsg(response)
}

// Usage
handler := &ForwardingDNSHandler{
    upstream: "8.8.8.8:53",
}
```

---

## 13.7 Service Discovery

### Consul Integration

```bash
go get github.com/hashicorp/consul/api
```

**Service Registration:**
```go
package main

import (
    "fmt"
    "log"
    
    "github.com/hashicorp/consul/api"
)

func registerService() {
    config := api.DefaultConfig()
    client, err := api.NewClient(config)
    if err != nil {
        log.Fatal(err)
    }
    
    registration := &api.AgentServiceRegistration{
        ID:      "my-service-1",
        Name:    "my-service",
        Port:    8080,
        Address: "192.168.1.10",
        Tags:    []string{"api", "v1"},
        Check: &api.AgentServiceCheck{
            HTTP:     "http://192.168.1.10:8080/health",
            Interval: "10s",
            Timeout:  "2s",
        },
    }
    
    err = client.Agent().ServiceRegister(registration)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Println("Service registered")
}

func deregisterService() {
    config := api.DefaultConfig()
    client, _ := api.NewClient(config)
    
    client.Agent().ServiceDeregister("my-service-1")
}
```

**Service Discovery:**
```go
func discoverServices(serviceName string) ([]*api.ServiceEntry, error) {
    config := api.DefaultConfig()
    client, err := api.NewClient(config)
    if err != nil {
        return nil, err
    }
    
    services, _, err := client.Health().Service(serviceName, "", true, nil)
    if err != nil {
        return nil, err
    }
    
    return services, nil
}

// Usage
services, _ := discoverServices("my-service")
for _, service := range services {
    fmt.Printf("Service: %s at %s:%d\n", 
        service.Service.ID,
        service.Service.Address,
        service.Service.Port)
}
```

### etcd Integration

```bash
go get go.etcd.io/etcd/client/v3
```

**Service Registration with etcd:**
```go
package main

import (
    "context"
    "fmt"
    "time"
    
    clientv3 "go.etcd.io/etcd/client/v3"
)

func registerWithEtcd() {
    cli, err := clientv3.New(clientv3.Config{
        Endpoints:   []string{"localhost:2379"},
        DialTimeout: 5 * time.Second,
    })
    if err != nil {
        log.Fatal(err)
    }
    defer cli.Close()
    
    // Create a lease (TTL)
    lease, err := cli.Grant(context.Background(), 10)
    if err != nil {
        log.Fatal(err)
    }
    
    // Register service with lease
    key := "/services/my-service/instance-1"
    value := "192.168.1.10:8080"
    
    _, err = cli.Put(context.Background(), key, value, clientv3.WithLease(lease.ID))
    if err != nil {
        log.Fatal(err)
    }
    
    // Keep lease alive
    ch, err := cli.KeepAlive(context.Background(), lease.ID)
    if err != nil {
        log.Fatal(err)
    }
    
    for ka := range ch {
        fmt.Printf("Lease kept alive: %d\n", ka.ID)
    }
}

func discoverFromEtcd(servicePrefix string) ([]string, error) {
    cli, _ := clientv3.New(clientv3.Config{
        Endpoints:   []string{"localhost:2379"},
        DialTimeout: 5 * time.Second,
    })
    defer cli.Close()
    
    resp, err := cli.Get(context.Background(), servicePrefix, clientv3.WithPrefix())
    if err != nil {
        return nil, err
    }
    
    instances := make([]string, 0, len(resp.Kvs))
    for _, kv := range resp.Kvs {
        instances = append(instances, string(kv.Value))
    }
    
    return instances, nil
}
```

**Watch for Service Changes:**
```go
func watchServices(servicePrefix string) {
    cli, _ := clientv3.New(clientv3.Config{
        Endpoints: []string{"localhost:2379"},
    })
    defer cli.Close()
    
    watchChan := cli.Watch(context.Background(), servicePrefix, clientv3.WithPrefix())
    
    for watchResp := range watchChan {
        for _, event := range watchResp.Events {
            switch event.Type {
            case clientv3.EventTypePut:
                fmt.Printf("Service added: %s = %s\n", event.Kv.Key, event.Kv.Value)
            case clientv3.EventTypeDelete:
                fmt.Printf("Service removed: %s\n", event.Kv.Key)
            }
        }
    }
}
```

---

*I'll complete Part 13 with Network Utilities, FAQs, Interview Questions, and exercises. Shall I continue?*

## 13.8 Network Utilities and Tools

### Port Scanner

```go
package main

import (
    "fmt"
    "net"
    "sync"
    "time"
)

type PortScanner struct {
    host    string
    timeout time.Duration
}

func NewPortScanner(host string, timeout time.Duration) *PortScanner {
    return &PortScanner{
        host:    host,
        timeout: timeout,
    }
}

func (ps *PortScanner) ScanPort(port int) bool {
    address := fmt.Sprintf("%s:%d", ps.host, port)
    conn, err := net.DialTimeout("tcp", address, ps.timeout)
    if err != nil {
        return false
    }
    conn.Close()
    return true
}

func (ps *PortScanner) ScanRange(startPort, endPort int) []int {
    openPorts := make([]int, 0)
    var mu sync.Mutex
    var wg sync.WaitGroup
    
    // Limit concurrent scans
    semaphore := make(chan struct{}, 100)
    
    for port := startPort; port <= endPort; port++ {
        wg.Add(1)
        go func(p int) {
            defer wg.Done()
            
            semaphore <- struct{}{}
            defer func() { <-semaphore }()
            
            if ps.ScanPort(p) {
                mu.Lock()
                openPorts = append(openPorts, p)
                fmt.Printf("Port %d is open\n", p)
                mu.Unlock()
            }
        }(port)
    }
    
    wg.Wait()
    return openPorts
}

func main() {
    scanner := NewPortScanner("localhost", 500*time.Millisecond)
    openPorts := scanner.ScanRange(1, 1024)
    
    fmt.Printf("\nOpen ports: %v\n", openPorts)
}
```

### Ping Implementation (ICMP)

```go
package main

import (
    "fmt"
    "net"
    "os"
    "time"
    
    "golang.org/x/net/icmp"
    "golang.org/x/net/ipv4"
)

func ping(host string) error {
    // Resolve host
    addr, err := net.ResolveIPAddr("ip4", host)
    if err != nil {
        return err
    }
    
    // Create ICMP connection
    conn, err := icmp.ListenPacket("ip4:icmp", "0.0.0.0")
    if err != nil {
        return err
    }
    defer conn.Close()
    
    // Create ICMP echo request
    msg := icmp.Message{
        Type: ipv4.ICMPTypeEcho,
        Code: 0,
        Body: &icmp.Echo{
            ID:   os.Getpid() & 0xffff,
            Seq:  1,
            Data: []byte("PING"),
        },
    }
    
    msgBytes, err := msg.Marshal(nil)
    if err != nil {
        return err
    }
    
    // Send echo request
    start := time.Now()
    if _, err := conn.WriteTo(msgBytes, addr); err != nil {
        return err
    }
    
    // Receive echo reply
    reply := make([]byte, 1500)
    conn.SetReadDeadline(time.Now().Add(3 * time.Second))
    
    n, peer, err := conn.ReadFrom(reply)
    if err != nil {
        return err
    }
    
    duration := time.Since(start)
    
    // Parse reply
    rm, err := icmp.ParseMessage(ipv4.ICMPTypeEchoReply.Protocol(), reply[:n])
    if err != nil {
        return err
    }
    
    switch rm.Type {
    case ipv4.ICMPTypeEchoReply:
        fmt.Printf("Reply from %s: bytes=%d time=%v\n", peer, n, duration)
    default:
        fmt.Printf("Unexpected ICMP message: %+v\n", rm)
    }
    
    return nil
}

func main() {
    if err := ping("8.8.8.8"); err != nil {
        fmt.Fprintf(os.Stderr, "Error: %v\n", err)
        os.Exit(1)
    }
}
```

### Traceroute

```go
func traceroute(host string, maxHops int) error {
    addr, err := net.ResolveIPAddr("ip4", host)
    if err != nil {
        return err
    }
    
    conn, err := icmp.ListenPacket("ip4:icmp", "0.0.0.0")
    if err != nil {
        return err
    }
    defer conn.Close()
    
    fmt.Printf("Traceroute to %s (%s), %d hops max\n", host, addr, maxHops)
    
    for ttl := 1; ttl <= maxHops; ttl++ {
        // Set TTL
        if err := conn.IPv4PacketConn().SetTTL(ttl); err != nil {
            return err
        }
        
        msg := icmp.Message{
            Type: ipv4.ICMPTypeEcho,
            Code: 0,
            Body: &icmp.Echo{
                ID:   os.Getpid() & 0xffff,
                Seq:  ttl,
                Data: []byte("TRACE"),
            },
        }
        
        msgBytes, _ := msg.Marshal(nil)
        
        start := time.Now()
        conn.WriteTo(msgBytes, addr)
        
        reply := make([]byte, 1500)
        conn.SetReadDeadline(time.Now().Add(2 * time.Second))
        
        n, peer, err := conn.ReadFrom(reply)
        duration := time.Since(start)
        
        if err != nil {
            fmt.Printf("%2d  *  *  * (timeout)\n", ttl)
            continue
        }
        
        rm, _ := icmp.ParseMessage(ipv4.ICMPTypeEchoReply.Protocol(), reply[:n])
        
        fmt.Printf("%2d  %s  %v\n", ttl, peer, duration)
        
        if rm.Type == ipv4.ICMPTypeEchoReply {
            fmt.Println("Reached destination")
            break
        }
    }
    
    return nil
}
```

### Network Monitor

```go
type NetworkMonitor struct {
    interfaces map[string]*InterfaceStats
    mu         sync.RWMutex
}

type InterfaceStats struct {
    Name         string
    BytesReceived uint64
    BytesSent     uint64
    PacketsReceived uint64
    PacketsSent     uint64
}

func NewNetworkMonitor() *NetworkMonitor {
    return &NetworkMonitor{
        interfaces: make(map[string]*InterfaceStats),
    }
}

func (nm *NetworkMonitor) Update() error {
    interfaces, err := net.Interfaces()
    if err != nil {
        return err
    }
    
    nm.mu.Lock()
    defer nm.mu.Unlock()
    
    for _, iface := range interfaces {
        stats, err := getInterfaceStats(iface.Name)
        if err != nil {
            continue
        }
        
        nm.interfaces[iface.Name] = stats
    }
    
    return nil
}

func (nm *NetworkMonitor) GetStats(ifaceName string) *InterfaceStats {
    nm.mu.RLock()
    defer nm.mu.RUnlock()
    
    return nm.interfaces[ifaceName]
}

func (nm *NetworkMonitor) StartMonitoring(interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()
    
    for range ticker.C {
        nm.Update()
        nm.printStats()
    }
}

func (nm *NetworkMonitor) printStats() {
    nm.mu.RLock()
    defer nm.mu.RUnlock()
    
    for name, stats := range nm.interfaces {
        fmt.Printf("Interface: %s\n", name)
        fmt.Printf("  RX: %d bytes (%d packets)\n", stats.BytesReceived, stats.PacketsReceived)
        fmt.Printf("  TX: %d bytes (%d packets)\n", stats.BytesSent, stats.PacketsSent)
    }
}
```

### Connection Info Tool

```go
func printConnectionInfo(conn net.Conn) {
    fmt.Println("Connection Information:")
    fmt.Printf("  Local Address:  %s\n", conn.LocalAddr())
    fmt.Printf("  Remote Address: %s\n", conn.RemoteAddr())
    
    if tcpConn, ok := conn.(*net.TCPConn); ok {
        fmt.Println("\nTCP Details:")
        
        // Get file descriptor for socket options
        file, _ := tcpConn.File()
        defer file.Close()
        
        fd := int(file.Fd())
        
        // You can use syscalls to get socket options
        fmt.Printf("  File Descriptor: %d\n", fd)
    }
}

func listActiveConnections() error {
    // This would typically use /proc/net/tcp on Linux
    // or netstat/ss commands
    
    // Example: Parse /proc/net/tcp
    data, err := os.ReadFile("/proc/net/tcp")
    if err != nil {
        return err
    }
    
    lines := strings.Split(string(data), "\n")
    for i, line := range lines {
        if i == 0 {
            continue // Skip header
        }
        
        fields := strings.Fields(line)
        if len(fields) < 10 {
            continue
        }
        
        localAddr := parseAddress(fields[1])
        remoteAddr := parseAddress(fields[2])
        state := fields[3]
        
        fmt.Printf("%s -> %s [%s]\n", localAddr, remoteAddr, state)
    }
    
    return nil
}
```

### Bandwidth Tester

```go
type BandwidthTester struct {
    conn net.Conn
}

func (bt *BandwidthTester) TestDownload(duration time.Duration) float64 {
    buffer := make([]byte, 32*1024) // 32KB buffer
    totalBytes := 0
    
    deadline := time.Now().Add(duration)
    bt.conn.SetReadDeadline(deadline)
    
    for time.Now().Before(deadline) {
        n, err := bt.conn.Read(buffer)
        if err != nil {
            break
        }
        totalBytes += n
    }
    
    // Return Mbps
    seconds := duration.Seconds()
    mbps := (float64(totalBytes) * 8) / (seconds * 1000000)
    return mbps
}

func (bt *BandwidthTester) TestUpload(duration time.Duration) float64 {
    buffer := make([]byte, 32*1024)
    totalBytes := 0
    
    deadline := time.Now().Add(duration)
    bt.conn.SetWriteDeadline(deadline)
    
    for time.Now().Before(deadline) {
        n, err := bt.conn.Write(buffer)
        if err != nil {
            break
        }
        totalBytes += n
    }
    
    seconds := duration.Seconds()
    mbps := (float64(totalBytes) * 8) / (seconds * 1000000)
    return mbps
}
```

---

## FAQs

**Q1: When should I use TCP vs UDP?**

**TCP:** When reliability is critical (file transfer, HTTP, email, database connections)  
**UDP:** When speed matters more than reliability (video streaming, gaming, DNS, VoIP, IoT sensors)

**Q2: How do I handle connection timeouts properly?**

Set appropriate deadlines:
```go
conn.SetDeadline(time.Now().Add(30 * time.Second))      // Both read/write
conn.SetReadDeadline(time.Now().Add(30 * time.Second))  // Read only
conn.SetWriteDeadline(time.Now().Add(30 * time.Second)) // Write only
```

Check for timeout errors:
```go
if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
    // Handle timeout
}
```

**Q3: What's the difference between Layer 4 and Layer 7 load balancing?**

**Layer 4 (Transport):** Routes based on IP/port, faster, less overhead, no content inspection  
**Layer 7 (Application):** Routes based on HTTP headers/URLs, content-aware, more flexible, higher overhead

**Q4: How do I implement connection pooling?**

Reuse connections to avoid overhead:
```go
pool := make(chan net.Conn, maxConns)

// Get connection
conn := <-pool // or create new if empty

// Use connection
// ...

// Return to pool
pool <- conn
```

See complete example in section 13.2.

**Q5: What's the best way to handle network errors?**

- Check for temporary errors: `err.(net.Error).Temporary()`
- Check for timeout: `err.(net.Error).Timeout()`
- Check for specific errors: `errors.Is(err, io.EOF)`
- Implement retry logic with exponential backoff
- Log errors with context
- Use circuit breakers for failing services

---

## Interview Questions

**Q1: Explain the TCP three-way handshake.**

**A:** TCP establishes connection with 3 steps:
1. **SYN**: Client sends SYN packet to server
2. **SYN-ACK**: Server responds with SYN-ACK
3. **ACK**: Client sends ACK, connection established

This ensures both sides are ready and agree on initial sequence numbers for reliable, ordered delivery.

**Q2: How does a load balancer detect failed backends?**

**A:** Through health checks:
- **Active probing**: Periodically send HTTP/TCP requests to health endpoint
- **Passive monitoring**: Track request success/failure rates
- **Circuit breaker**: Temporarily remove failing backends
- Health checks run at intervals (e.g., every 10 seconds)
- Failed backends marked unavailable until health check passes

**Q3: What is the difference between connection timeout and read/write timeout?**

**A:**
- **Connection timeout**: Max time to establish connection (TCP handshake)
- **Read timeout**: Max time waiting for data after connection established
- **Write timeout**: Max time to send data

Example: `net.DialTimeout` sets connection timeout, `conn.SetReadDeadline` sets read timeout.

**Q4: How would you design a load balancer that handles 1 million requests per second?**

**A:**
- Use Layer 4 load balancing (lower overhead)
- Connection pooling to backends
- Async I/O with goroutines
- Efficient algorithms (round-robin, least connections)
- Health check optimization (batch checks, longer intervals)
- Horizontal scaling (multiple load balancer instances)
- Hardware acceleration (use kernel bypass like DPDK if needed)
- Metrics and monitoring (track latency, throughput, errors)

**Q5: Explain how DNS works.**

**A:**
1. Client queries local DNS resolver
2. Resolver checks cache
3. If not cached, queries root name servers
4. Root servers refer to TLD (Top-Level Domain) servers (.com, .org)
5. TLD servers refer to authoritative name servers
6. Authoritative server returns IP address
7. Resolver caches result and returns to client

DNS uses UDP for queries (fast, small packets) and TCP for zone transfers (reliable, large data).

---

## Key Takeaways

1. **TCP** provides reliable, ordered delivery; **UDP** provides fast, connectionless delivery
2. **Custom protocols** require careful design for versioning, error handling, and efficiency
3. **Load balancers** distribute traffic using algorithms like round-robin, least connections, IP hash
4. **Health checks** ensure load balancers only route to healthy backends
5. **DNS** translates domain names to IP addresses using hierarchical resolution
6. **Service discovery** (Consul, etcd) enables dynamic service registration and lookup
7. **Network programming** in Go is powerful due to goroutines and built-in concurrency
8. **Proper error handling** and timeouts are critical for robust network applications

---

## Practice Exercises

### Exercise 1: Build a Chat Server
- TCP server supporting multiple clients
- Broadcast messages to all connected clients
- Handle client disconnections gracefully
- Implement commands (/nick, /list, /quit)

### Exercise 2: Custom Protocol Implementation
- Design binary protocol with versioning
- Implement request-response pattern
- Add compression and encryption
- Handle message fragmentation

### Exercise 3: HTTP Load Balancer
- Implement round-robin and least connections
- Add health checking
- Implement sticky sessions (IP hash)
- Add metrics (requests/sec, latency)
- Graceful shutdown

### Exercise 4: DNS Server
- Implement authoritative DNS server
- Support A, AAAA, CNAME, TXT records
- Add caching
- Implement forwarding to upstream DNS

### Exercise 5: Service Discovery System
- Register services with health checks
- Query available services
- Watch for service changes
- Implement client-side load balancing

---

## Additional Resources

**Networking:**
- Computer Networks (Tanenbaum) - Book
- TCP/IP Illustrated (Stevens) - Book
- https://beej.us/guide/bgnet/

**Go Networking:**
- https://pkg.go.dev/net
- https://golang.org/doc/articles/wiki/ (includes network examples)
- Network Programming with Go (Jan Newmarch) - Book

**Load Balancing:**
- https://github.com/gorilla/websocket (WebSocket examples)
- https://www.nginx.com/resources/glossary/load-balancing/
- HAProxy documentation

**DNS:**
- https://github.com/miekg/dns
- DNS and BIND (Cricket Liu) - Book
- RFC 1035 (DNS specification)

**Service Discovery:**
- https://www.consul.io/docs
- https://etcd.io/docs
- https://github.com/hashicorp/consul

---

## Summary

Part 13 covered advanced networking with Go:

1. **Network Fundamentals**: OSI/TCP-IP models, sockets, address resolution
2. **TCP Programming**: Servers, clients, connection management, optimization
3. **UDP Programming**: Datagrams, broadcast, multicast, reliability patterns
4. **Custom Protocols**: Binary and text protocols, message framing
5. **Load Balancers**: Algorithms, health checks, circuit breakers
6. **DNS Programming**: Queries, custom resolver, DNS server implementation
7. **Service Discovery**: Consul and etcd integration
8. **Network Utilities**: Port scanner, ping, traceroute, monitoring

You now have the skills to build network infrastructure and understand how systems like nginx, DNS servers, and service meshes work!

---

**Total Word Count**: ~14,000 words  
**Status**: ✅ Complete

**Next**: Part 14 - Machine Learning & AI Integration
