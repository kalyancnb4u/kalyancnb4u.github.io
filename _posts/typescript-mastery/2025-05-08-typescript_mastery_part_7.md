---
title: "TypeScript Mastery - Part 7: Real-World Applications & Advanced Scenarios"
date: 2025-05-08 00:00:00 +0530
categories: [TypeScript, TS Mastery]
tags: [TypeScript, Programming, Web Development, Micro-services, Web-sockets, CLI, Monorepo, Real-time, Production, Advanced]
---

# Complete TypeScript Mastery Part 7: Real-World Applications & Advanced Scenarios

## Introduction

Welcome to Part 7, the final part of the Complete TypeScript Mastery series! In Parts 1-6, we covered TypeScript fundamentals, advanced features, design patterns, and framework integrations. Now we'll explore advanced real-world scenarios and build complete production applications.

This part focuses on advanced topics:
- Microservices architecture with TypeScript
- Real-time applications with WebSockets
- CLI tools and command-line applications
- Monorepo management and tooling
- E-commerce application architecture
- Authentication and authorization systems
- File upload and processing
- Payment integration
- Advanced debugging and profiling
- Production deployment strategies

By the end of this part, you'll have the expertise to architect and build enterprise-grade TypeScript applications for any scenario.

---

## 7.1 Microservices Architecture

### Microservices Overview

```typescript
// shared/types.ts - Shared types across services
export interface User {
  id: string
  email: string
  name: string
  createdAt: Date
}

export interface Product {
  id: string
  name: string
  price: number
  stock: number
}

export interface Order {
  id: string
  userId: string
  items: OrderItem[]
  total: number
  status: OrderStatus
  createdAt: Date
}

export interface OrderItem {
  productId: string
  quantity: number
  price: number
}

export enum OrderStatus {
  Pending = 'pending',
  Processing = 'processing',
  Shipped = 'shipped',
  Delivered = 'delivered',
  Cancelled = 'cancelled'
}

// shared/events.ts - Domain events
export interface DomainEvent {
  id: string
  type: string
  timestamp: Date
  payload: any
}

export interface UserCreatedEvent extends DomainEvent {
  type: 'user.created'
  payload: {
    userId: string
    email: string
    name: string
  }
}

export interface OrderPlacedEvent extends DomainEvent {
  type: 'order.placed'
  payload: {
    orderId: string
    userId: string
    items: OrderItem[]
    total: number
  }
}

export interface PaymentProcessedEvent extends DomainEvent {
  type: 'payment.processed'
  payload: {
    orderId: string
    amount: number
    status: 'success' | 'failed'
  }
}
```

### Message Broker Integration

```typescript
// shared/message-broker.ts
import { EventEmitter } from 'events'

export interface MessageBroker {
  publish(topic: string, message: any): Promise<void>
  subscribe(topic: string, handler: (message: any) => Promise<void>): void
  connect(): Promise<void>
  disconnect(): Promise<void>
}

// RabbitMQ implementation
import * as amqp from 'amqplib'

export class RabbitMQBroker implements MessageBroker {
  private connection: amqp.Connection | null = null
  private channel: amqp.Channel | null = null
  private readonly url: string

  constructor(url: string) {
    this.url = url
  }

  async connect(): Promise<void> {
    this.connection = await amqp.connect(this.url)
    this.channel = await this.connection.createChannel()
    console.log('Connected to RabbitMQ')
  }

  async disconnect(): Promise<void> {
    await this.channel?.close()
    await this.connection?.close()
    console.log('Disconnected from RabbitMQ')
  }

  async publish(topic: string, message: any): Promise<void> {
    if (!this.channel) {
      throw new Error('Channel not initialized')
    }

    await this.channel.assertQueue(topic, { durable: true })
    
    const content = Buffer.from(JSON.stringify(message))
    this.channel.sendToQueue(topic, content, { persistent: true })
    
    console.log(`Published message to ${topic}:`, message)
  }

  subscribe(topic: string, handler: (message: any) => Promise<void>): void {
    if (!this.channel) {
      throw new Error('Channel not initialized')
    }

    this.channel.assertQueue(topic, { durable: true })
    
    this.channel.consume(topic, async (msg) => {
      if (msg) {
        try {
          const content = JSON.parse(msg.content.toString())
          await handler(content)
          this.channel!.ack(msg)
        } catch (error) {
          console.error('Error processing message:', error)
          this.channel!.nack(msg, false, false)
        }
      }
    })

    console.log(`Subscribed to ${topic}`)
  }
}

// Redis Pub/Sub implementation
import Redis from 'ioredis'

export class RedisBroker implements MessageBroker {
  private publisher: Redis
  private subscriber: Redis
  private handlers: Map<string, (message: any) => Promise<void>> = new Map()

  constructor(options: Redis.RedisOptions) {
    this.publisher = new Redis(options)
    this.subscriber = new Redis(options)
  }

  async connect(): Promise<void> {
    console.log('Connected to Redis')
  }

  async disconnect(): Promise<void> {
    await this.publisher.quit()
    await this.subscriber.quit()
    console.log('Disconnected from Redis')
  }

  async publish(topic: string, message: any): Promise<void> {
    await this.publisher.publish(topic, JSON.stringify(message))
    console.log(`Published message to ${topic}:`, message)
  }

  subscribe(topic: string, handler: (message: any) => Promise<void>): void {
    this.handlers.set(topic, handler)
    
    this.subscriber.subscribe(topic, (err) => {
      if (err) {
        console.error(`Failed to subscribe to ${topic}:`, err)
      } else {
        console.log(`Subscribed to ${topic}`)
      }
    })

    this.subscriber.on('message', async (channel, message) => {
      const handler = this.handlers.get(channel)
      if (handler) {
        try {
          const parsed = JSON.parse(message)
          await handler(parsed)
        } catch (error) {
          console.error('Error processing message:', error)
        }
      }
    })
  }
}
```

### User Service

```typescript
// services/user-service/src/app.ts
import express, { Application } from 'express'
import { UserController } from './controllers/user.controller'
import { UserService } from './services/user.service'
import { UserRepository } from './repositories/user.repository'
import { MessageBroker } from '@shared/message-broker'

export class UserServiceApp {
  private app: Application
  private messageBroker: MessageBroker

  constructor(messageBroker: MessageBroker) {
    this.app = express()
    this.messageBroker = messageBroker
    this.setupMiddleware()
    this.setupRoutes()
  }

  private setupMiddleware(): void {
    this.app.use(express.json())
  }

  private setupRoutes(): void {
    const repository = new UserRepository()
    const service = new UserService(repository, this.messageBroker)
    const controller = new UserController(service)

    this.app.post('/users', (req, res) => controller.createUser(req, res))
    this.app.get('/users/:id', (req, res) => controller.getUser(req, res))
    this.app.put('/users/:id', (req, res) => controller.updateUser(req, res))
    this.app.delete('/users/:id', (req, res) => controller.deleteUser(req, res))
  }

  async start(port: number): Promise<void> {
    await this.messageBroker.connect()
    
    this.app.listen(port, () => {
      console.log(`User service listening on port ${port}`)
    })
  }
}

// services/user-service/src/services/user.service.ts
import { User, UserCreatedEvent } from '@shared/types'
import { UserRepository } from '../repositories/user.repository'
import { MessageBroker } from '@shared/message-broker'
import { v4 as uuidv4 } from 'uuid'

export class UserService {
  constructor(
    private repository: UserRepository,
    private messageBroker: MessageBroker
  ) {}

  async createUser(data: Omit<User, 'id' | 'createdAt'>): Promise<User> {
    const user: User = {
      id: uuidv4(),
      ...data,
      createdAt: new Date()
    }

    await this.repository.save(user)

    // Publish event
    const event: UserCreatedEvent = {
      id: uuidv4(),
      type: 'user.created',
      timestamp: new Date(),
      payload: {
        userId: user.id,
        email: user.email,
        name: user.name
      }
    }

    await this.messageBroker.publish('user.events', event)

    return user
  }

  async getUser(id: string): Promise<User | null> {
    return this.repository.findById(id)
  }

  async updateUser(id: string, data: Partial<User>): Promise<User> {
    const user = await this.repository.findById(id)
    
    if (!user) {
      throw new Error('User not found')
    }

    const updated = { ...user, ...data }
    await this.repository.save(updated)

    return updated
  }

  async deleteUser(id: string): Promise<void> {
    await this.repository.delete(id)
  }
}
```

### Order Service

```typescript
// services/order-service/src/services/order.service.ts
import { Order, OrderPlacedEvent, OrderStatus } from '@shared/types'
import { OrderRepository } from '../repositories/order.repository'
import { MessageBroker } from '@shared/message-broker'
import { v4 as uuidv4 } from 'uuid'

export class OrderService {
  constructor(
    private repository: OrderRepository,
    private messageBroker: MessageBroker
  ) {
    this.subscribeToEvents()
  }

  private subscribeToEvents(): void {
    // Listen for payment processed events
    this.messageBroker.subscribe('payment.events', async (event) => {
      if (event.type === 'payment.processed') {
        await this.handlePaymentProcessed(event)
      }
    })
  }

  async createOrder(data: Omit<Order, 'id' | 'status' | 'createdAt'>): Promise<Order> {
    const order: Order = {
      id: uuidv4(),
      ...data,
      status: OrderStatus.Pending,
      createdAt: new Date()
    }

    await this.repository.save(order)

    // Publish order placed event
    const event: OrderPlacedEvent = {
      id: uuidv4(),
      type: 'order.placed',
      timestamp: new Date(),
      payload: {
        orderId: order.id,
        userId: order.userId,
        items: order.items,
        total: order.total
      }
    }

    await this.messageBroker.publish('order.events', event)

    return order
  }

  async getOrder(id: string): Promise<Order | null> {
    return this.repository.findById(id)
  }

  async getUserOrders(userId: string): Promise<Order[]> {
    return this.repository.findByUserId(userId)
  }

  async updateOrderStatus(id: string, status: OrderStatus): Promise<Order> {
    const order = await this.repository.findById(id)
    
    if (!order) {
      throw new Error('Order not found')
    }

    order.status = status
    await this.repository.save(order)

    return order
  }

  private async handlePaymentProcessed(event: PaymentProcessedEvent): Promise<void> {
    const { orderId, status } = event.payload

    if (status === 'success') {
      await this.updateOrderStatus(orderId, OrderStatus.Processing)
    } else {
      await this.updateOrderStatus(orderId, OrderStatus.Cancelled)
    }
  }
}
```

### API Gateway

```typescript
// services/api-gateway/src/app.ts
import express, { Application, Request, Response, NextFunction } from 'express'
import { createProxyMiddleware } from 'http-proxy-middleware'
import jwt from 'jsonwebtoken'

interface AuthRequest extends Request {
  user?: {
    id: string
    email: string
  }
}

export class APIGateway {
  private app: Application
  private readonly jwtSecret: string

  constructor(jwtSecret: string) {
    this.app = express()
    this.jwtSecret = jwtSecret
    this.setupMiddleware()
    this.setupRoutes()
  }

  private setupMiddleware(): void {
    this.app.use(express.json())
    this.app.use(this.logRequests)
  }

  private logRequests = (req: Request, res: Response, next: NextFunction): void => {
    console.log(`${req.method} ${req.path}`)
    next()
  }

  private authenticate = (req: AuthRequest, res: Response, next: NextFunction): void => {
    const token = req.headers.authorization?.split(' ')[1]

    if (!token) {
      res.status(401).json({ error: 'No token provided' })
      return
    }

    try {
      const decoded = jwt.verify(token, this.jwtSecret) as {
        id: string
        email: string
      }
      req.user = decoded
      next()
    } catch (error) {
      res.status(401).json({ error: 'Invalid token' })
    }
  }

  private setupRoutes(): void {
    // Public routes - no authentication
    this.app.use(
      '/api/auth',
      createProxyMiddleware({
        target: 'http://localhost:3001',
        pathRewrite: { '^/api/auth': '' },
        changeOrigin: true
      })
    )

    // Protected routes - require authentication
    this.app.use(
      '/api/users',
      this.authenticate,
      createProxyMiddleware({
        target: 'http://localhost:3002',
        pathRewrite: { '^/api/users': '/users' },
        changeOrigin: true
      })
    )

    this.app.use(
      '/api/products',
      createProxyMiddleware({
        target: 'http://localhost:3003',
        pathRewrite: { '^/api/products': '/products' },
        changeOrigin: true
      })
    )

    this.app.use(
      '/api/orders',
      this.authenticate,
      createProxyMiddleware({
        target: 'http://localhost:3004',
        pathRewrite: { '^/api/orders': '/orders' },
        changeOrigin: true
      })
    )

    // Health check
    this.app.get('/health', (req, res) => {
      res.json({ status: 'ok', timestamp: new Date().toISOString() })
    })
  }

  start(port: number): void {
    this.app.listen(port, () => {
      console.log(`API Gateway listening on port ${port}`)
    })
  }
}

// services/api-gateway/src/index.ts
import { APIGateway } from './app'

const gateway = new APIGateway(process.env.JWT_SECRET || 'secret')
gateway.start(3000)
```

### Service Discovery

```typescript
// shared/service-registry.ts
export interface ServiceInfo {
  name: string
  url: string
  health: string
  lastSeen: Date
}

export class ServiceRegistry {
  private services = new Map<string, ServiceInfo>()
  private readonly heartbeatInterval = 30000 // 30 seconds

  register(name: string, url: string): void {
    this.services.set(name, {
      name,
      url,
      health: 'healthy',
      lastSeen: new Date()
    })

    console.log(`Service registered: ${name} at ${url}`)
  }

  unregister(name: string): void {
    this.services.delete(name)
    console.log(`Service unregistered: ${name}`)
  }

  getService(name: string): ServiceInfo | undefined {
    return this.services.get(name)
  }

  getAllServices(): ServiceInfo[] {
    return Array.from(this.services.values())
  }

  updateHeartbeat(name: string): void {
    const service = this.services.get(name)
    if (service) {
      service.lastSeen = new Date()
    }
  }

  startHealthCheck(): void {
    setInterval(() => {
      const now = Date.now()
      
      for (const [name, service] of this.services.entries()) {
        const timeSinceLastSeen = now - service.lastSeen.getTime()
        
        if (timeSinceLastSeen > this.heartbeatInterval * 2) {
          service.health = 'unhealthy'
          console.warn(`Service unhealthy: ${name}`)
        }
      }
    }, this.heartbeatInterval)
  }
}

// Service health endpoint
import express from 'express'

export function setupHealthEndpoint(
  app: express.Application,
  serviceName: string,
  registry: ServiceRegistry
): void {
  app.get('/health', (req, res) => {
    registry.updateHeartbeat(serviceName)
    
    res.json({
      service: serviceName,
      status: 'healthy',
      timestamp: new Date().toISOString()
    })
  })
}
```

---

## 7.2 Real-Time Applications with WebSockets

### WebSocket Server Setup

```typescript
// server/websocket-server.ts
import { WebSocketServer, WebSocket } from 'ws'
import { IncomingMessage } from 'http'
import jwt from 'jsonwebtoken'

interface AuthenticatedWebSocket extends WebSocket {
  userId?: string
  rooms: Set<string>
}

interface WebSocketMessage {
  type: string
  payload: any
}

interface JoinRoomMessage extends WebSocketMessage {
  type: 'join_room'
  payload: {
    roomId: string
  }
}

interface LeaveRoomMessage extends WebSocketMessage {
  type: 'leave_room'
  payload: {
    roomId: string
  }
}

interface SendMessagePayload extends WebSocketMessage {
  type: 'send_message'
  payload: {
    roomId: string
    message: string
  }
}

export class WebSocketManager {
  private wss: WebSocketServer
  private clients = new Map<string, AuthenticatedWebSocket>()
  private rooms = new Map<string, Set<string>>()
  private readonly jwtSecret: string

  constructor(port: number, jwtSecret: string) {
    this.jwtSecret = jwtSecret
    this.wss = new WebSocketServer({ port })
    this.setupConnectionHandler()
  }

  private setupConnectionHandler(): void {
    this.wss.on('connection', (ws: WebSocket, req: IncomingMessage) => {
      const userId = this.authenticateConnection(req)
      
      if (!userId) {
        ws.close(1008, 'Authentication failed')
        return
      }

      const client = ws as AuthenticatedWebSocket
      client.userId = userId
      client.rooms = new Set()

      this.clients.set(userId, client)
      console.log(`Client connected: ${userId}`)

      this.setupMessageHandlers(client)
      this.setupCloseHandler(client)
    })
  }

  private authenticateConnection(req: IncomingMessage): string | null {
    try {
      const url = new URL(req.url || '', `http://${req.headers.host}`)
      const token = url.searchParams.get('token')

      if (!token) {
        return null
      }

      const decoded = jwt.verify(token, this.jwtSecret) as { id: string }
      return decoded.id
    } catch (error) {
      return null
    }
  }

  private setupMessageHandlers(client: AuthenticatedWebSocket): void {
    client.on('message', (data: Buffer) => {
      try {
        const message: WebSocketMessage = JSON.parse(data.toString())
        this.handleMessage(client, message)
      } catch (error) {
        console.error('Error parsing message:', error)
      }
    })
  }

  private handleMessage(client: AuthenticatedWebSocket, message: WebSocketMessage): void {
    switch (message.type) {
      case 'join_room':
        this.handleJoinRoom(client, message as JoinRoomMessage)
        break
      case 'leave_room':
        this.handleLeaveRoom(client, message as LeaveRoomMessage)
        break
      case 'send_message':
        this.handleSendMessage(client, message as SendMessagePayload)
        break
      default:
        console.warn('Unknown message type:', message.type)
    }
  }

  private handleJoinRoom(client: AuthenticatedWebSocket, message: JoinRoomMessage): void {
    const { roomId } = message.payload

    client.rooms.add(roomId)

    if (!this.rooms.has(roomId)) {
      this.rooms.set(roomId, new Set())
    }

    this.rooms.get(roomId)!.add(client.userId!)

    console.log(`User ${client.userId} joined room ${roomId}`)

    // Notify room members
    this.broadcastToRoom(roomId, {
      type: 'user_joined',
      payload: {
        userId: client.userId,
        roomId
      }
    })
  }

  private handleLeaveRoom(client: AuthenticatedWebSocket, message: LeaveRoomMessage): void {
    const { roomId } = message.payload

    client.rooms.delete(roomId)
    this.rooms.get(roomId)?.delete(client.userId!)

    console.log(`User ${client.userId} left room ${roomId}`)

    this.broadcastToRoom(roomId, {
      type: 'user_left',
      payload: {
        userId: client.userId,
        roomId
      }
    })
  }

  private handleSendMessage(client: AuthenticatedWebSocket, message: SendMessagePayload): void {
    const { roomId, message: messageText } = message.payload

    if (!client.rooms.has(roomId)) {
      client.send(JSON.stringify({
        type: 'error',
        payload: { message: 'Not in room' }
      }))
      return
    }

    this.broadcastToRoom(roomId, {
      type: 'message',
      payload: {
        userId: client.userId,
        roomId,
        message: messageText,
        timestamp: new Date().toISOString()
      }
    })
  }

  private broadcastToRoom(roomId: string, message: any): void {
    const userIds = this.rooms.get(roomId)
    
    if (!userIds) return

    const messageStr = JSON.stringify(message)

    userIds.forEach(userId => {
      const client = this.clients.get(userId)
      if (client && client.readyState === WebSocket.OPEN) {
        client.send(messageStr)
      }
    })
  }

  private setupCloseHandler(client: AuthenticatedWebSocket): void {
    client.on('close', () => {
      console.log(`Client disconnected: ${client.userId}`)

      // Remove from all rooms
      client.rooms.forEach(roomId => {
        this.rooms.get(roomId)?.delete(client.userId!)
        
        this.broadcastToRoom(roomId, {
          type: 'user_left',
          payload: {
            userId: client.userId,
            roomId
          }
        })
      })

      this.clients.delete(client.userId!)
    })
  }

  sendToUser(userId: string, message: any): void {
    const client = this.clients.get(userId)
    
    if (client && client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify(message))
    }
  }

  broadcast(message: any): void {
    const messageStr = JSON.stringify(message)
    
    this.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(messageStr)
      }
    })
  }
}

// Usage
const wsManager = new WebSocketManager(8080, process.env.JWT_SECRET!)
```

### Socket.IO Implementation

```typescript
// server/socket-io-server.ts
import { Server, Socket } from 'socket.io'
import { createServer } from 'http'
import jwt from 'jsonwebtoken'

interface AuthenticatedSocket extends Socket {
  userId?: string
}

interface ServerToClientEvents {
  message: (data: MessageData) => void
  user_joined: (data: { userId: string; roomId: string }) => void
  user_left: (data: { userId: string; roomId: string }) => void
  typing: (data: { userId: string; roomId: string; isTyping: boolean }) => void
}

interface ClientToServerEvents {
  join_room: (roomId: string) => void
  leave_room: (roomId: string) => void
  send_message: (data: { roomId: string; message: string }) => void
  typing: (data: { roomId: string; isTyping: boolean }) => void
}

interface MessageData {
  userId: string
  roomId: string
  message: string
  timestamp: string
}

export class SocketIOManager {
  private io: Server<ClientToServerEvents, ServerToClientEvents>
  private readonly jwtSecret: string

  constructor(port: number, jwtSecret: string) {
    this.jwtSecret = jwtSecret
    const httpServer = createServer()
    
    this.io = new Server<ClientToServerEvents, ServerToClientEvents>(httpServer, {
      cors: {
        origin: '*',
        methods: ['GET', 'POST']
      }
    })

    this.setupMiddleware()
    this.setupConnectionHandler()

    httpServer.listen(port, () => {
      console.log(`Socket.IO server listening on port ${port}`)
    })
  }

  private setupMiddleware(): void {
    this.io.use((socket, next) => {
      const token = socket.handshake.auth.token

      if (!token) {
        return next(new Error('Authentication error'))
      }

      try {
        const decoded = jwt.verify(token, this.jwtSecret) as { id: string }
        (socket as AuthenticatedSocket).userId = decoded.id
        next()
      } catch (error) {
        next(new Error('Authentication error'))
      }
    })
  }

  private setupConnectionHandler(): void {
    this.io.on('connection', (socket: Socket) => {
      const authSocket = socket as AuthenticatedSocket
      console.log(`Client connected: ${authSocket.userId}`)

      this.setupEventHandlers(authSocket)

      socket.on('disconnect', () => {
        console.log(`Client disconnected: ${authSocket.userId}`)
      })
    })
  }

  private setupEventHandlers(socket: AuthenticatedSocket): void {
    socket.on('join_room', (roomId: string) => {
      socket.join(roomId)
      console.log(`User ${socket.userId} joined room ${roomId}`)

      socket.to(roomId).emit('user_joined', {
        userId: socket.userId!,
        roomId
      })
    })

    socket.on('leave_room', (roomId: string) => {
      socket.leave(roomId)
      console.log(`User ${socket.userId} left room ${roomId}`)

      socket.to(roomId).emit('user_left', {
        userId: socket.userId!,
        roomId
      })
    })

    socket.on('send_message', (data: { roomId: string; message: string }) => {
      const messageData: MessageData = {
        userId: socket.userId!,
        roomId: data.roomId,
        message: data.message,
        timestamp: new Date().toISOString()
      }

      this.io.to(data.roomId).emit('message', messageData)
    })

    socket.on('typing', (data: { roomId: string; isTyping: boolean }) => {
      socket.to(data.roomId).emit('typing', {
        userId: socket.userId!,
        roomId: data.roomId,
        isTyping: data.isTyping
      })
    })
  }

  sendToUser(userId: string, event: string, data: any): void {
    this.io.sockets.sockets.forEach(socket => {
      if ((socket as AuthenticatedSocket).userId === userId) {
        socket.emit(event, data)
      }
    })
  }

  sendToRoom(roomId: string, event: string, data: any): void {
    this.io.to(roomId).emit(event, data)
  }

  broadcast(event: string, data: any): void {
    this.io.emit(event, data)
  }
}
```

### Real-Time Client

```typescript
// client/websocket-client.ts
export class WebSocketClient {
  private ws: WebSocket | null = null
  private readonly url: string
  private readonly token: string
  private reconnectAttempts = 0
  private readonly maxReconnectAttempts = 5
  private messageHandlers = new Map<string, (data: any) => void>()

  constructor(url: string, token: string) {
    this.url = url
    this.token = token
  }

  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.ws = new WebSocket(`${this.url}?token=${this.token}`)

      this.ws.onopen = () => {
        console.log('WebSocket connected')
        this.reconnectAttempts = 0
        resolve()
      }

      this.ws.onerror = (error) => {
        console.error('WebSocket error:', error)
        reject(error)
      }

      this.ws.onclose = () => {
        console.log('WebSocket disconnected')
        this.attemptReconnect()
      }

      this.ws.onmessage = (event) => {
        try {
          const message = JSON.parse(event.data)
          this.handleMessage(message)
        } catch (error) {
          console.error('Error parsing message:', error)
        }
      }
    })
  }

  private attemptReconnect(): void {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++
      console.log(`Reconnecting... Attempt ${this.reconnectAttempts}`)
      
      setTimeout(() => {
        this.connect().catch(() => {
          console.error('Reconnection failed')
        })
      }, 1000 * this.reconnectAttempts)
    } else {
      console.error('Max reconnection attempts reached')
    }
  }

  private handleMessage(message: { type: string; payload: any }): void {
    const handler = this.messageHandlers.get(message.type)
    if (handler) {
      handler(message.payload)
    }
  }

  on(type: string, handler: (data: any) => void): void {
    this.messageHandlers.set(type, handler)
  }

  off(type: string): void {
    this.messageHandlers.delete(type)
  }

  send(type: string, payload: any): void {
    if (this.ws && this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify({ type, payload }))
    } else {
      console.error('WebSocket not connected')
    }
  }

  joinRoom(roomId: string): void {
    this.send('join_room', { roomId })
  }

  leaveRoom(roomId: string): void {
    this.send('leave_room', { roomId })
  }

  sendMessage(roomId: string, message: string): void {
    this.send('send_message', { roomId, message })
  }

  disconnect(): void {
    if (this.ws) {
      this.ws.close()
      this.ws = null
    }
  }
}

// Usage in React
import { useEffect, useState, useCallback } from 'react'

interface Message {
  userId: string
  message: string
  timestamp: string
}

export function useChatRoom(roomId: string, token: string) {
  const [messages, setMessages] = useState<Message[]>([])
  const [client, setClient] = useState<WebSocketClient | null>(null)
  const [connected, setConnected] = useState(false)

  useEffect(() => {
    const ws = new WebSocketClient('ws://localhost:8080', token)

    ws.connect()
      .then(() => {
        setConnected(true)
        ws.joinRoom(roomId)
      })
      .catch((error) => {
        console.error('Connection failed:', error)
      })

    ws.on('message', (data: Message) => {
      setMessages(prev => [...prev, data])
    })

    ws.on('user_joined', (data) => {
      console.log('User joined:', data)
    })

    ws.on('user_left', (data) => {
      console.log('User left:', data)
    })

    setClient(ws)

    return () => {
      ws.leaveRoom(roomId)
      ws.disconnect()
    }
  }, [roomId, token])

  const sendMessage = useCallback((message: string) => {
    if (client) {
      client.sendMessage(roomId, message)
    }
  }, [client, roomId])

  return {
    messages,
    sendMessage,
    connected
  }
}
```

---

## 7.3 CLI Tools with TypeScript

### CLI Framework Setup

```bash
npm install commander inquirer chalk ora
npm install -D @types/inquirer
```

### Basic CLI Structure

```typescript
// src/cli.ts
import { Command } from 'commander'
import chalk from 'chalk'
import { createProject } from './commands/create'
import { buildProject } from './commands/build'
import { deployProject } from './commands/deploy'

const program = new Command()

program
  .name('my-cli')
  .description('A powerful CLI tool built with TypeScript')
  .version('1.0.0')

program
  .command('create <project-name>')
  .description('Create a new project')
  .option('-t, --template <template>', 'Project template', 'basic')
  .option('-d, --directory <directory>', 'Target directory', '.')
  .action(async (projectName: string, options) => {
    try {
      await createProject(projectName, options)
      console.log(chalk.green('✓ Project created successfully!'))
    } catch (error) {
      console.error(chalk.red('✗ Error creating project:'), error)
      process.exit(1)
    }
  })

program
  .command('build')
  .description('Build the project')
  .option('-w, --watch', 'Watch mode')
  .option('-p, --production', 'Production build')
  .action(async (options) => {
    try {
      await buildProject(options)
      console.log(chalk.green('✓ Build completed!'))
    } catch (error) {
      console.error(chalk.red('✗ Build failed:'), error)
      process.exit(1)
    }
  })

program
  .command('deploy')
  .description('Deploy the project')
  .option('-e, --environment <env>', 'Deployment environment', 'staging')
  .action(async (options) => {
    try {
      await deployProject(options)
      console.log(chalk.green('✓ Deployment successful!'))
    } catch (error) {
      console.error(chalk.red('✗ Deployment failed:'), error)
      process.exit(1)
    }
  })

program.parse()
```

### Interactive Prompts

```typescript
// src/commands/create.ts
import inquirer from 'inquirer'
import chalk from 'chalk'
import ora from 'ora'
import * as fs from 'fs/promises'
import * as path from 'path'

interface ProjectConfig {
  name: string
  template: string
  features: string[]
  packageManager: 'npm' | 'yarn' | 'pnpm'
  typescript: boolean
  git: boolean
}

export async function createProject(
  projectName: string,
  options: { template?: string; directory?: string }
): Promise<void> {
  console.log(chalk.blue('\n🚀 Creating a new project...\n'))

  // Interactive prompts
  const answers = await inquirer.prompt<{
    template: string
    features: string[]
    packageManager: string
    typescript: boolean
    git: boolean
  }>([
    {
      type: 'list',
      name: 'template',
      message: 'Select a project template:',
      choices: [
        { name: 'Basic', value: 'basic' },
        { name: 'React', value: 'react' },
        { name: 'Node.js API', value: 'nodejs' },
        { name: 'Full-stack', value: 'fullstack' }
      ],
      default: options.template || 'basic'
    },
    {
      type: 'checkbox',
      name: 'features',
      message: 'Select features:',
      choices: [
        { name: 'ESLint', value: 'eslint', checked: true },
        { name: 'Prettier', value: 'prettier', checked: true },
        { name: 'Testing (Jest)', value: 'jest' },
        { name: 'Docker', value: 'docker' },
        { name: 'CI/CD', value: 'cicd' }
      ]
    },
    {
      type: 'list',
      name: 'packageManager',
      message: 'Select package manager:',
      choices: ['npm', 'yarn', 'pnpm'],
      default: 'npm'
    },
    {
      type: 'confirm',
      name: 'typescript',
      message: 'Use TypeScript?',
      default: true
    },
    {
      type: 'confirm',
      name: 'git',
      message: 'Initialize Git repository?',
      default: true
    }
  ])

  const config: ProjectConfig = {
    name: projectName,
    template: answers.template,
    features: answers.features,
    packageManager: answers.packageManager as 'npm' | 'yarn' | 'pnpm',
    typescript: answers.typescript,
    git: answers.git
  }

  await createProjectStructure(config, options.directory || '.')
}

async function createProjectStructure(
  config: ProjectConfig,
  targetDir: string
): Promise<void> {
  const projectPath = path.join(targetDir, config.name)

  // Create project directory
  const spinner = ora('Creating project directory...').start()
  
  try {
    await fs.mkdir(projectPath, { recursive: true })
    spinner.succeed('Project directory created')
  } catch (error) {
    spinner.fail('Failed to create directory')
    throw error
  }

  // Create package.json
  spinner.start('Creating package.json...')
  await createPackageJson(projectPath, config)
  spinner.succeed('package.json created')

  // Create project files based on template
  spinner.start('Setting up project files...')
  await createTemplateFiles(projectPath, config)
  spinner.succeed('Project files created')

  // Install dependencies
  if (config.features.length > 0) {
    spinner.start('Installing dependencies...')
    await installDependencies(projectPath, config)
    spinner.succeed('Dependencies installed')
  }

  // Initialize Git
  if (config.git) {
    spinner.start('Initializing Git repository...')
    await initGit(projectPath)
    spinner.succeed('Git repository initialized')
  }

  console.log(chalk.green(`\n✓ Project ${config.name} created successfully!\n`))
  console.log(chalk.blue('Next steps:'))
  console.log(chalk.white(`  cd ${config.name}`))
  console.log(chalk.white(`  ${config.packageManager} start`))
}

async function createPackageJson(
  projectPath: string,
  config: ProjectConfig
): Promise<void> {
  const packageJson = {
    name: config.name,
    version: '1.0.0',
    description: '',
    main: 'index.js',
    scripts: {
      start: 'node index.js',
      build: 'tsc',
      test: 'jest'
    },
    keywords: [],
    author: '',
    license: 'MIT',
    dependencies: {},
    devDependencies: {}
  }

  if (config.typescript) {
    packageJson.devDependencies = {
      ...packageJson.devDependencies,
      'typescript': '^5.0.0',
      '@types/node': '^20.0.0'
    }
  }

  if (config.features.includes('eslint')) {
    packageJson.devDependencies = {
      ...packageJson.devDependencies,
      'eslint': '^8.0.0'
    }
  }

  if (config.features.includes('prettier')) {
    packageJson.devDependencies = {
      ...packageJson.devDependencies,
      'prettier': '^3.0.0'
    }
  }

  if (config.features.includes('jest')) {
    packageJson.devDependencies = {
      ...packageJson.devDependencies,
      'jest': '^29.0.0'
    }
  }

  await fs.writeFile(
    path.join(projectPath, 'package.json'),
    JSON.stringify(packageJson, null, 2)
  )
}

async function createTemplateFiles(
  projectPath: string,
  config: ProjectConfig
): Promise<void> {
  const srcDir = path.join(projectPath, 'src')
  await fs.mkdir(srcDir, { recursive: true })

  if (config.typescript) {
    // Create tsconfig.json
    const tsconfig = {
      compilerOptions: {
        target: 'ES2020',
        module: 'commonjs',
        outDir: './dist',
        rootDir: './src',
        strict: true,
        esModuleInterop: true
      },
      include: ['src/**/*'],
      exclude: ['node_modules']
    }

    await fs.writeFile(
      path.join(projectPath, 'tsconfig.json'),
      JSON.stringify(tsconfig, null, 2)
    )

    // Create index.ts
    await fs.writeFile(
      path.join(srcDir, 'index.ts'),
      `console.log('Hello, TypeScript!');\n`
    )
  } else {
    // Create index.js
    await fs.writeFile(
      path.join(srcDir, 'index.js'),
      `console.log('Hello, JavaScript!');\n`
    )
  }

  // Create .gitignore
  if (config.git) {
    await fs.writeFile(
      path.join(projectPath, '.gitignore'),
      'node_modules/\ndist/\n.env\n'
    )
  }

  // Create README.md
  await fs.writeFile(
    path.join(projectPath, 'README.md'),
    `# ${config.name}\n\nA project created with my-cli\n`
  )
}

async function installDependencies(
  projectPath: string,
  config: ProjectConfig
): Promise<void> {
  const { spawn } = require('child_process')

  return new Promise((resolve, reject) => {
    const child = spawn(config.packageManager, ['install'], {
      cwd: projectPath,
      stdio: 'inherit'
    })

    child.on('close', (code: number) => {
      if (code === 0) {
        resolve()
      } else {
        reject(new Error('Failed to install dependencies'))
      }
    })
  })
}

async function initGit(projectPath: string): Promise<void> {
  const { spawn } = require('child_process')

  return new Promise((resolve, reject) => {
    const child = spawn('git', ['init'], {
      cwd: projectPath,
      stdio: 'inherit'
    })

    child.on('close', (code: number) => {
      if (code === 0) {
        resolve()
      } else {
        reject(new Error('Failed to initialize Git'))
      }
    })
  })
}
```

### File System Operations

```typescript
// src/utils/fs-utils.ts
import * as fs from 'fs/promises'
import * as path from 'path'

export async function copyDirectory(
  source: string,
  destination: string
): Promise<void> {
  await fs.mkdir(destination, { recursive: true })

  const entries = await fs.readdir(source, { withFileTypes: true })

  for (const entry of entries) {
    const sourcePath = path.join(source, entry.name)
    const destPath = path.join(destination, entry.name)

    if (entry.isDirectory()) {
      await copyDirectory(sourcePath, destPath)
    } else {
      await fs.copyFile(sourcePath, destPath)
    }
  }
}

export async function ensureDirectory(dirPath: string): Promise<void> {
  try {
    await fs.access(dirPath)
  } catch {
    await fs.mkdir(dirPath, { recursive: true })
  }
}

export async function readJsonFile<T>(filePath: string): Promise<T> {
  const content = await fs.readFile(filePath, 'utf-8')
  return JSON.parse(content)
}

export async function writeJsonFile(
  filePath: string,
  data: any
): Promise<void> {
  await fs.writeFile(filePath, JSON.stringify(data, null, 2))
}

export async function fileExists(filePath: string): Promise<boolean> {
  try {
    await fs.access(filePath)
    return true
  } catch {
    return false
  }
}

export async function deleteDirectory(dirPath: string): Promise<void> {
  await fs.rm(dirPath, { recursive: true, force: true })
}
```

---

## 7.4 Monorepo Management

### Turborepo Setup

```bash
# Create new monorepo
npx create-turbo@latest

# Or manually
npm install turbo --save-dev
```

**Project Structure:**

```
my-monorepo/
├── apps/
│   ├── web/              # Next.js app
│   ├── admin/            # Admin dashboard
│   ├── mobile/           # React Native app
│   └── api/              # NestJS API
├── packages/
│   ├── ui/               # Shared UI components
│   ├── config/           # Shared configs
│   ├── tsconfig/         # TypeScript configs
│   └── database/         # Database client
├── package.json
├── turbo.json
└── pnpm-workspace.yaml
```

**turbo.json Configuration:**

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts"]
    },
    "lint": {
      "outputs": []
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "deploy": {
      "dependsOn": ["build", "test", "lint"],
      "outputs": []
    }
  }
}
```

**pnpm-workspace.yaml:**

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

### Shared UI Package

```typescript
// packages/ui/package.json
{
  "name": "@repo/ui",
  "version": "0.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch",
    "lint": "eslint src/"
  },
  "devDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "tsup": "^8.0.0",
    "typescript": "^5.0.0"
  },
  "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}

// packages/ui/src/Button.tsx
import React from 'react'

export interface ButtonProps {
  children: React.ReactNode
  onClick?: () => void
  variant?: 'primary' | 'secondary'
  disabled?: boolean
}

export function Button({
  children,
  onClick,
  variant = 'primary',
  disabled = false
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// packages/ui/src/Input.tsx
import React from 'react'

export interface InputProps {
  value: string
  onChange: (value: string) => void
  placeholder?: string
  type?: 'text' | 'email' | 'password'
  disabled?: boolean
  error?: string
}

export function Input({
  value,
  onChange,
  placeholder,
  type = 'text',
  disabled = false,
  error
}: InputProps) {
  return (
    <div className="input-wrapper">
      <input
        type={type}
        value={value}
        onChange={(e) => onChange(e.target.value)}
        placeholder={placeholder}
        disabled={disabled}
        className={error ? 'input-error' : ''}
      />
      {error && <span className="error-message">{error}</span>}
    </div>
  )
}

// packages/ui/src/index.ts
export { Button } from './Button'
export type { ButtonProps } from './Button'
export { Input } from './Input'
export type { InputProps } from './Input'

// packages/ui/tsconfig.json
{
  "extends": "@repo/tsconfig/react-library.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

### Shared Database Package

```typescript
// packages/database/package.json
{
  "name": "@repo/database",
  "version": "0.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch",
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate dev"
  },
  "dependencies": {
    "@prisma/client": "^5.0.0"
  },
  "devDependencies": {
    "prisma": "^5.0.0",
    "tsup": "^8.0.0",
    "typescript": "^5.0.0"
  }
}

// packages/database/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  password  String
  role      Role     @default(USER)
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("users")
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String   @db.Text
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("posts")
}

enum Role {
  USER
  ADMIN
}

// packages/database/src/index.ts
import { PrismaClient } from '@prisma/client'

export * from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error']
  })

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma
}

// Helper functions
export async function getUserById(id: string) {
  return prisma.user.findUnique({
    where: { id },
    include: { posts: true }
  })
}

export async function createUser(data: {
  email: string
  name: string
  password: string
}) {
  return prisma.user.create({ data })
}

export async function updateUser(
  id: string,
  data: { name?: string; email?: string }
) {
  return prisma.user.update({
    where: { id },
    data
  })
}
```

### TypeScript Config Package

```typescript
// packages/tsconfig/package.json
{
  "name": "@repo/tsconfig",
  "version": "0.0.0",
  "files": [
    "base.json",
    "nextjs.json",
    "react-library.json",
    "node.json"
  ]
}

// packages/tsconfig/base.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "Default",
  "compilerOptions": {
    "composite": false,
    "declaration": true,
    "declarationMap": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "inlineSources": false,
    "isolatedModules": true,
    "moduleResolution": "node",
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "preserveWatchOutput": true,
    "skipLibCheck": true,
    "strict": true
  },
  "exclude": ["node_modules"]
}

// packages/tsconfig/nextjs.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "Next.js",
  "extends": "./base.json",
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "jsx": "preserve",
    "module": "esnext",
    "noEmit": true,
    "incremental": true,
    "resolveJsonModule": true,
    "plugins": [{ "name": "next" }]
  },
  "include": ["src", "next-env.d.ts"],
  "exclude": ["node_modules"]
}

// packages/tsconfig/react-library.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "React Library",
  "extends": "./base.json",
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "declaration": true
  }
}

// packages/tsconfig/node.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "Node.js",
  "extends": "./base.json",
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "commonjs",
    "types": ["node"]
  }
}
```

### Using Packages in Apps

```typescript
// apps/web/package.json
{
  "name": "web",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "next": "14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@repo/ui": "workspace:*",
    "@repo/database": "workspace:*"
  },
  "devDependencies": {
    "@repo/tsconfig": "workspace:*",
    "typescript": "^5.0.0"
  }
}

// apps/web/tsconfig.json
{
  "extends": "@repo/tsconfig/nextjs.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}

// apps/web/src/app/page.tsx
import { Button, Input } from '@repo/ui'
import { prisma, getUserById } from '@repo/database'

export default async function Home() {
  const users = await prisma.user.findMany()

  return (
    <main>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
      <Button onClick={() => console.log('Clicked')}>
        Add User
      </Button>
    </main>
  )
}

// apps/api/src/main.ts
import { NestFactory } from '@nestjs/core'
import { AppModule } from './app.module'
import { prisma } from '@repo/database'

async function bootstrap() {
  const app = await NestFactory.create(AppModule)
  
  // Use shared database
  app.enableCors()
  
  await app.listen(3001)
  console.log('API running on http://localhost:3001')
}

bootstrap()
```

### Build Scripts

```bash
# Root package.json
{
  "name": "my-monorepo",
  "private": true,
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "clean": "turbo run clean && rm -rf node_modules",
    "format": "prettier --write \"**/*.{ts,tsx,md}\""
  },
  "devDependencies": {
    "turbo": "latest",
    "prettier": "^3.0.0",
    "eslint": "^8.0.0"
  },
  "packageManager": "pnpm@8.0.0"
}

# Build all packages
pnpm build

# Run all apps in dev mode
pnpm dev

# Build specific app
pnpm --filter web build

# Add dependency to specific package
pnpm --filter @repo/ui add lodash
```

---

## 7.5 E-commerce Application Architecture

### Domain Models

```typescript
// src/domain/models/Product.ts
export interface Product {
  id: string
  name: string
  description: string
  price: number
  stock: number
  category: string
  images: string[]
  specifications: Record<string, string>
  createdAt: Date
  updatedAt: Date
}

export interface ProductVariant {
  id: string
  productId: string
  name: string
  sku: string
  price: number
  stock: number
  attributes: Record<string, string>
}

// src/domain/models/Cart.ts
export interface CartItem {
  productId: string
  variantId?: string
  quantity: number
  price: number
}

export interface Cart {
  id: string
  userId: string
  items: CartItem[]
  subtotal: number
  tax: number
  shipping: number
  total: number
  createdAt: Date
  updatedAt: Date
}

// src/domain/models/Order.ts
export enum OrderStatus {
  Pending = 'pending',
  Confirmed = 'confirmed',
  Processing = 'processing',
  Shipped = 'shipped',
  Delivered = 'delivered',
  Cancelled = 'cancelled',
  Refunded = 'refunded'
}

export enum PaymentStatus {
  Pending = 'pending',
  Authorized = 'authorized',
  Captured = 'captured',
  Failed = 'failed',
  Refunded = 'refunded'
}

export interface OrderItem {
  productId: string
  variantId?: string
  name: string
  quantity: number
  price: number
  total: number
}

export interface ShippingAddress {
  fullName: string
  street: string
  city: string
  state: string
  zipCode: string
  country: string
  phone: string
}

export interface Order {
  id: string
  userId: string
  items: OrderItem[]
  subtotal: number
  tax: number
  shipping: number
  discount: number
  total: number
  status: OrderStatus
  paymentStatus: PaymentStatus
  paymentMethod: string
  shippingAddress: ShippingAddress
  billingAddress: ShippingAddress
  trackingNumber?: string
  notes?: string
  createdAt: Date
  updatedAt: Date
}
```

### Product Service

```typescript
// src/services/ProductService.ts
import { Product, ProductVariant } from '../domain/models/Product'
import { ProductRepository } from '../repositories/ProductRepository'
import { CacheService } from './CacheService'
import { EventEmitter } from 'events'

export interface ProductFilter {
  category?: string
  minPrice?: number
  maxPrice?: number
  inStock?: boolean
  search?: string
}

export interface PaginationOptions {
  page: number
  limit: number
  sortBy?: string
  sortOrder?: 'asc' | 'desc'
}

export class ProductService extends EventEmitter {
  constructor(
    private productRepository: ProductRepository,
    private cacheService: CacheService
  ) {
    super()
  }

  async getProducts(
    filter: ProductFilter,
    pagination: PaginationOptions
  ): Promise<{ products: Product[]; total: number }> {
    const cacheKey = this.buildCacheKey('products', filter, pagination)
    const cached = await this.cacheService.get<{ products: Product[]; total: number }>(
      cacheKey
    )

    if (cached) {
      return cached
    }

    const result = await this.productRepository.findMany(filter, pagination)
    await this.cacheService.set(cacheKey, result, 300) // Cache for 5 minutes

    return result
  }

  async getProductById(id: string): Promise<Product | null> {
    const cacheKey = `product:${id}`
    const cached = await this.cacheService.get<Product>(cacheKey)

    if (cached) {
      return cached
    }

    const product = await this.productRepository.findById(id)
    
    if (product) {
      await this.cacheService.set(cacheKey, product, 600) // Cache for 10 minutes
    }

    return product
  }

  async createProduct(data: Omit<Product, 'id' | 'createdAt' | 'updatedAt'>): Promise<Product> {
    const product = await this.productRepository.create(data)
    
    // Emit event
    this.emit('product:created', product)
    
    // Invalidate cache
    await this.cacheService.deletePattern('products:*')

    return product
  }

  async updateProduct(
    id: string,
    data: Partial<Product>
  ): Promise<Product> {
    const product = await this.productRepository.update(id, data)
    
    // Emit event
    this.emit('product:updated', product)
    
    // Invalidate cache
    await this.cacheService.delete(`product:${id}`)
    await this.cacheService.deletePattern('products:*')

    return product
  }

  async updateStock(id: string, quantity: number): Promise<void> {
    await this.productRepository.updateStock(id, quantity)
    
    // Invalidate cache
    await this.cacheService.delete(`product:${id}`)
  }

  async checkAvailability(
    productId: string,
    quantity: number
  ): Promise<boolean> {
    const product = await this.getProductById(productId)
    return product ? product.stock >= quantity : false
  }

  private buildCacheKey(
    prefix: string,
    filter: ProductFilter,
    pagination: PaginationOptions
  ): string {
    return `${prefix}:${JSON.stringify(filter)}:${JSON.stringify(pagination)}`
  }
}
```

### Cart Service

```typescript
// src/services/CartService.ts
import { Cart, CartItem } from '../domain/models/Cart'
import { CartRepository } from '../repositories/CartRepository'
import { ProductService } from './ProductService'

export class CartService {
  constructor(
    private cartRepository: CartRepository,
    private productService: ProductService
  ) {}

  async getCart(userId: string): Promise<Cart> {
    let cart = await this.cartRepository.findByUserId(userId)

    if (!cart) {
      cart = await this.cartRepository.create({
        userId,
        items: [],
        subtotal: 0,
        tax: 0,
        shipping: 0,
        total: 0
      })
    }

    return cart
  }

  async addItem(
    userId: string,
    productId: string,
    quantity: number,
    variantId?: string
  ): Promise<Cart> {
    // Check product availability
    const available = await this.productService.checkAvailability(
      productId,
      quantity
    )

    if (!available) {
      throw new Error('Product not available in requested quantity')
    }

    const product = await this.productService.getProductById(productId)
    
    if (!product) {
      throw new Error('Product not found')
    }

    const cart = await this.getCart(userId)
    
    // Check if item already exists
    const existingItem = cart.items.find(
      item => item.productId === productId && item.variantId === variantId
    )

    if (existingItem) {
      existingItem.quantity += quantity
    } else {
      cart.items.push({
        productId,
        variantId,
        quantity,
        price: product.price
      })
    }

    return this.updateCartTotals(cart)
  }

  async updateItemQuantity(
    userId: string,
    productId: string,
    quantity: number,
    variantId?: string
  ): Promise<Cart> {
    if (quantity <= 0) {
      return this.removeItem(userId, productId, variantId)
    }

    const cart = await this.getCart(userId)
    const item = cart.items.find(
      i => i.productId === productId && i.variantId === variantId
    )

    if (!item) {
      throw new Error('Item not found in cart')
    }

    // Check availability
    const available = await this.productService.checkAvailability(
      productId,
      quantity
    )

    if (!available) {
      throw new Error('Product not available in requested quantity')
    }

    item.quantity = quantity

    return this.updateCartTotals(cart)
  }

  async removeItem(
    userId: string,
    productId: string,
    variantId?: string
  ): Promise<Cart> {
    const cart = await this.getCart(userId)
    
    cart.items = cart.items.filter(
      item => !(item.productId === productId && item.variantId === variantId)
    )

    return this.updateCartTotals(cart)
  }

  async clearCart(userId: string): Promise<Cart> {
    const cart = await this.getCart(userId)
    cart.items = []
    return this.updateCartTotals(cart)
  }

  private async updateCartTotals(cart: Cart): Promise<Cart> {
    // Calculate subtotal
    cart.subtotal = cart.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    )

    // Calculate tax (example: 10%)
    cart.tax = cart.subtotal * 0.1

    // Calculate shipping (example: flat rate)
    cart.shipping = cart.subtotal > 100 ? 0 : 10

    // Calculate total
    cart.total = cart.subtotal + cart.tax + cart.shipping

    return this.cartRepository.update(cart.id, cart)
  }

  async validateCart(userId: string): Promise<{
    valid: boolean
    errors: string[]
  }> {
    const cart = await this.getCart(userId)
    const errors: string[] = []

    if (cart.items.length === 0) {
      errors.push('Cart is empty')
    }

    // Check each item availability
    for (const item of cart.items) {
      const available = await this.productService.checkAvailability(
        item.productId,
        item.quantity
      )

      if (!available) {
        const product = await this.productService.getProductById(item.productId)
        errors.push(`${product?.name} is not available in requested quantity`)
      }
    }

    return {
      valid: errors.length === 0,
      errors
    }
  }
}
```

### Order Service

```typescript
// src/services/OrderService.ts
import { Order, OrderStatus, PaymentStatus, OrderItem } from '../domain/models/Order'
import { OrderRepository } from '../repositories/OrderRepository'
import { CartService } from './CartService'
import { ProductService } from './ProductService'
import { PaymentService } from './PaymentService'
import { NotificationService } from './NotificationService'
import { EventEmitter } from 'events'

export class OrderService extends EventEmitter {
  constructor(
    private orderRepository: OrderRepository,
    private cartService: CartService,
    private productService: ProductService,
    private paymentService: PaymentService,
    private notificationService: NotificationService
  ) {
    super()
  }

  async createOrder(
    userId: string,
    shippingAddress: ShippingAddress,
    paymentMethod: string
  ): Promise<Order> {
    // Validate cart
    const validation = await this.cartService.validateCart(userId)
    
    if (!validation.valid) {
      throw new Error(`Cart validation failed: ${validation.errors.join(', ')}`)
    }

    // Get cart
    const cart = await this.cartService.getCart(userId)

    // Create order items
    const items: OrderItem[] = await Promise.all(
      cart.items.map(async (cartItem) => {
        const product = await this.productService.getProductById(
          cartItem.productId
        )
        
        return {
          productId: cartItem.productId,
          variantId: cartItem.variantId,
          name: product!.name,
          quantity: cartItem.quantity,
          price: cartItem.price,
          total: cartItem.price * cartItem.quantity
        }
      })
    )

    // Create order
    const order = await this.orderRepository.create({
      userId,
      items,
      subtotal: cart.subtotal,
      tax: cart.tax,
      shipping: cart.shipping,
      discount: 0,
      total: cart.total,
      status: OrderStatus.Pending,
      paymentStatus: PaymentStatus.Pending,
      paymentMethod,
      shippingAddress,
      billingAddress: shippingAddress
    })

    // Process payment
    try {
      const paymentResult = await this.paymentService.processPayment({
        orderId: order.id,
        amount: order.total,
        paymentMethod
      })

      if (paymentResult.success) {
        order.paymentStatus = PaymentStatus.Captured
        order.status = OrderStatus.Confirmed
      } else {
        order.paymentStatus = PaymentStatus.Failed
        order.status = OrderStatus.Cancelled
      }

      await this.orderRepository.update(order.id, order)
    } catch (error) {
      order.paymentStatus = PaymentStatus.Failed
      order.status = OrderStatus.Cancelled
      await this.orderRepository.update(order.id, order)
      throw error
    }

    // Update product stock
    for (const item of items) {
      await this.productService.updateStock(item.productId, -item.quantity)
    }

    // Clear cart
    await this.cartService.clearCart(userId)

    // Send notification
    await this.notificationService.sendOrderConfirmation(order)

    // Emit event
    this.emit('order:created', order)

    return order
  }

  async getOrder(id: string): Promise<Order | null> {
    return this.orderRepository.findById(id)
  }

  async getUserOrders(userId: string): Promise<Order[]> {
    return this.orderRepository.findByUserId(userId)
  }

  async updateOrderStatus(
    orderId: string,
    status: OrderStatus
  ): Promise<Order> {
    const order = await this.orderRepository.findById(orderId)
    
    if (!order) {
      throw new Error('Order not found')
    }

    order.status = status
    await this.orderRepository.update(orderId, order)

    // Send notification based on status
    switch (status) {
      case OrderStatus.Shipped:
        await this.notificationService.sendShippingNotification(order)
        break
      case OrderStatus.Delivered:
        await this.notificationService.sendDeliveryNotification(order)
        break
      case OrderStatus.Cancelled:
        await this.notificationService.sendCancellationNotification(order)
        // Restore stock
        for (const item of order.items) {
          await this.productService.updateStock(item.productId, item.quantity)
        }
        break
    }

    this.emit('order:status_updated', { orderId, status })

    return order
  }

  async cancelOrder(orderId: string): Promise<Order> {
    const order = await this.orderRepository.findById(orderId)
    
    if (!order) {
      throw new Error('Order not found')
    }

    if (order.status !== OrderStatus.Pending && 
        order.status !== OrderStatus.Confirmed) {
      throw new Error('Order cannot be cancelled at this stage')
    }

    // Refund payment if captured
    if (order.paymentStatus === PaymentStatus.Captured) {
      await this.paymentService.refundPayment(orderId)
      order.paymentStatus = PaymentStatus.Refunded
    }

    return this.updateOrderStatus(orderId, OrderStatus.Cancelled)
  }

  async addTrackingNumber(
    orderId: string,
    trackingNumber: string
  ): Promise<Order> {
    const order = await this.orderRepository.findById(orderId)
    
    if (!order) {
      throw new Error('Order not found')
    }

    order.trackingNumber = trackingNumber
    await this.orderRepository.update(orderId, order)

    await this.notificationService.sendTrackingNotification(order)

    return order
  }
}
```

### Payment Integration

```typescript
// src/services/PaymentService.ts
import Stripe from 'stripe'

export interface PaymentRequest {
  orderId: string
  amount: number
  paymentMethod: string
  currency?: string
}

export interface PaymentResult {
  success: boolean
  transactionId?: string
  error?: string
}

export class PaymentService {
  private stripe: Stripe

  constructor(apiKey: string) {
    this.stripe = new Stripe(apiKey, {
      apiVersion: '2023-10-16'
    })
  }

  async processPayment(request: PaymentRequest): Promise<PaymentResult> {
    try {
      const paymentIntent = await this.stripe.paymentIntents.create({
        amount: Math.round(request.amount * 100), // Convert to cents
        currency: request.currency || 'usd',
        payment_method: request.paymentMethod,
        confirm: true,
        metadata: {
          orderId: request.orderId
        }
      })

      if (paymentIntent.status === 'succeeded') {
        return {
          success: true,
          transactionId: paymentIntent.id
        }
      } else {
        return {
          success: false,
          error: 'Payment not completed'
        }
      }
    } catch (error) {
      return {
        success: false,
        error: error instanceof Error ? error.message : 'Payment failed'
      }
    }
  }

  async refundPayment(orderId: string): Promise<PaymentResult> {
    try {
      // Find payment intent by order ID
      const paymentIntents = await this.stripe.paymentIntents.list({
        limit: 1
      })

      const paymentIntent = paymentIntents.data.find(
        pi => pi.metadata.orderId === orderId
      )

      if (!paymentIntent) {
        return {
          success: false,
          error: 'Payment not found'
        }
      }

      const refund = await this.stripe.refunds.create({
        payment_intent: paymentIntent.id
      })

      return {
        success: refund.status === 'succeeded',
        transactionId: refund.id
      }
    } catch (error) {
      return {
        success: false,
        error: error instanceof Error ? error.message : 'Refund failed'
      }
    }
  }

  async createPaymentIntent(amount: number): Promise<{
    clientSecret: string
    paymentIntentId: string
  }> {
    const paymentIntent = await this.stripe.paymentIntents.create({
      amount: Math.round(amount * 100),
      currency: 'usd'
    })

    return {
      clientSecret: paymentIntent.client_secret!,
      paymentIntentId: paymentIntent.id
    }
  }

  async confirmPayment(paymentIntentId: string): Promise<PaymentResult> {
    try {
      const paymentIntent = await this.stripe.paymentIntents.confirm(
        paymentIntentId
      )

      return {
        success: paymentIntent.status === 'succeeded',
        transactionId: paymentIntent.id
      }
    } catch (error) {
      return {
        success: false,
        error: error instanceof Error ? error.message : 'Confirmation failed'
      }
    }
  }
}
```

---

## 7.6 Authentication & Authorization System

### JWT Authentication

```typescript
// src/auth/JwtService.ts
import jwt from 'jsonwebtoken'
import bcrypt from 'bcrypt'

export interface TokenPayload {
  userId: string
  email: string
  role: string
}

export interface AuthTokens {
  accessToken: string
  refreshToken: string
}

export class JwtService {
  private readonly accessTokenSecret: string
  private readonly refreshTokenSecret: string
  private readonly accessTokenExpiry = '15m'
  private readonly refreshTokenExpiry = '7d'

  constructor(accessSecret: string, refreshSecret: string) {
    this.accessTokenSecret = accessSecret
    this.refreshTokenSecret = refreshSecret
  }

  generateTokens(payload: TokenPayload): AuthTokens {
    const accessToken = jwt.sign(payload, this.accessTokenSecret, {
      expiresIn: this.accessTokenExpiry
    })

    const refreshToken = jwt.sign(
      { userId: payload.userId },
      this.refreshTokenSecret,
      { expiresIn: this.refreshTokenExpiry }
    )

    return { accessToken, refreshToken }
  }

  verifyAccessToken(token: string): TokenPayload | null {
    try {
      return jwt.verify(token, this.accessTokenSecret) as TokenPayload
    } catch {
      return null
    }
  }

  verifyRefreshToken(token: string): { userId: string } | null {
    try {
      return jwt.verify(token, this.refreshTokenSecret) as { userId: string }
    } catch {
      return null
    }
  }

  async hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, 10)
  }

  async comparePassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash)
  }
}
```

### Authentication Service

```typescript
// src/auth/AuthService.ts
import { JwtService, TokenPayload, AuthTokens } from './JwtService'
import { UserRepository } from '../repositories/UserRepository'
import { User } from '../domain/models/User'

export interface LoginCredentials {
  email: string
  password: string
}

export interface RegisterData {
  email: string
  password: string
  name: string
}

export class AuthService {
  constructor(
    private jwtService: JwtService,
    private userRepository: UserRepository
  ) {}

  async register(data: RegisterData): Promise<{
    user: User
    tokens: AuthTokens
  }> {
    // Check if user exists
    const existing = await this.userRepository.findByEmail(data.email)
    
    if (existing) {
      throw new Error('Email already in use')
    }

    // Hash password
    const hashedPassword = await this.jwtService.hashPassword(data.password)

    // Create user
    const user = await this.userRepository.create({
      ...data,
      password: hashedPassword,
      role: 'user'
    })

    // Generate tokens
    const tokens = this.jwtService.generateTokens({
      userId: user.id,
      email: user.email,
      role: user.role
    })

    return { user, tokens }
  }

  async login(credentials: LoginCredentials): Promise<{
    user: User
    tokens: AuthTokens
  }> {
    // Find user
    const user = await this.userRepository.findByEmail(credentials.email)
    
    if (!user) {
      throw new Error('Invalid credentials')
    }

    // Verify password
    const valid = await this.jwtService.comparePassword(
      credentials.password,
      user.password
    )

    if (!valid) {
      throw new Error('Invalid credentials')
    }

    // Generate tokens
    const tokens = this.jwtService.generateTokens({
      userId: user.id,
      email: user.email,
      role: user.role
    })

    // Update last login
    await this.userRepository.updateLastLogin(user.id)

    return { user, tokens }
  }

  async refreshToken(refreshToken: string): Promise<AuthTokens> {
    // Verify refresh token
    const payload = this.jwtService.verifyRefreshToken(refreshToken)
    
    if (!payload) {
      throw new Error('Invalid refresh token')
    }

    // Get user
    const user = await this.userRepository.findById(payload.userId)
    
    if (!user) {
      throw new Error('User not found')
    }

    // Generate new tokens
    return this.jwtService.generateTokens({
      userId: user.id,
      email: user.email,
      role: user.role
    })
  }

  async logout(userId: string): Promise<void> {
    // In a production system, you might:
    // - Invalidate refresh tokens in database
    // - Add access token to blacklist (with Redis)
    // - Clear user sessions
    console.log(`User ${userId} logged out`)
  }

  async changePassword(
    userId: string,
    currentPassword: string,
    newPassword: string
  ): Promise<void> {
    const user = await this.userRepository.findById(userId)
    
    if (!user) {
      throw new Error('User not found')
    }

    // Verify current password
    const valid = await this.jwtService.comparePassword(
      currentPassword,
      user.password
    )

    if (!valid) {
      throw new Error('Invalid current password')
    }

    // Hash new password
    const hashedPassword = await this.jwtService.hashPassword(newPassword)

    // Update password
    await this.userRepository.updatePassword(userId, hashedPassword)
  }

  async requestPasswordReset(email: string): Promise<void> {
    const user = await this.userRepository.findByEmail(email)
    
    if (!user) {
      // Don't reveal if email exists
      return
    }

    // Generate reset token
    const resetToken = jwt.sign(
      { userId: user.id, purpose: 'reset' },
      process.env.JWT_SECRET!,
      { expiresIn: '1h' }
    )

    // Save token to database
    await this.userRepository.saveResetToken(user.id, resetToken)

    // Send email (implement email service)
    console.log(`Reset token for ${email}: ${resetToken}`)
  }

  async resetPassword(token: string, newPassword: string): Promise<void> {
    // Verify token
    let payload: any
    try {
      payload = jwt.verify(token, process.env.JWT_SECRET!)
    } catch {
      throw new Error('Invalid or expired reset token')
    }

    if (payload.purpose !== 'reset') {
      throw new Error('Invalid token purpose')
    }

    // Check if token was used
    const user = await this.userRepository.findById(payload.userId)
    
    if (!user || user.resetToken !== token) {
      throw new Error('Invalid reset token')
    }

    // Hash new password
    const hashedPassword = await this.jwtService.hashPassword(newPassword)

    // Update password and clear reset token
    await this.userRepository.updatePassword(payload.userId, hashedPassword)
    await this.userRepository.clearResetToken(payload.userId)
  }
}
```

### Authorization Middleware

```typescript
// src/auth/AuthMiddleware.ts
import { Request, Response, NextFunction } from 'express'
import { JwtService, TokenPayload } from './JwtService'

export interface AuthRequest extends Request {
  user?: TokenPayload
}

export class AuthMiddleware {
  constructor(private jwtService: JwtService) {}

  authenticate = (req: AuthRequest, res: Response, next: NextFunction): void => {
    const token = this.extractToken(req)

    if (!token) {
      res.status(401).json({ error: 'No token provided' })
      return
    }

    const payload = this.jwtService.verifyAccessToken(token)

    if (!payload) {
      res.status(401).json({ error: 'Invalid token' })
      return
    }

    req.user = payload
    next()
  }

  authorize = (...roles: string[]) => {
    return (req: AuthRequest, res: Response, next: NextFunction): void => {
      if (!req.user) {
        res.status(401).json({ error: 'Not authenticated' })
        return
      }

      if (!roles.includes(req.user.role)) {
        res.status(403).json({ error: 'Insufficient permissions' })
        return
      }

      next()
    }
  }

  optionalAuth = (req: AuthRequest, res: Response, next: NextFunction): void => {
    const token = this.extractToken(req)

    if (token) {
      const payload = this.jwtService.verifyAccessToken(token)
      if (payload) {
        req.user = payload
      }
    }

    next()
  }

  private extractToken(req: Request): string | null {
    const authHeader = req.headers.authorization

    if (!authHeader) {
      return null
    }

    const parts = authHeader.split(' ')

    if (parts.length !== 2 || parts[0] !== 'Bearer') {
      return null
    }

    return parts[1]
  }
}

// Usage
import express from 'express'

const app = express()
const authMiddleware = new AuthMiddleware(jwtService)

// Public route
app.get('/api/products', (req, res) => {
  res.json({ products: [] })
})

// Protected route - requires authentication
app.get(
  '/api/profile',
  authMiddleware.authenticate,
  (req: AuthRequest, res) => {
    res.json({ user: req.user })
  }
)

// Admin only route
app.delete(
  '/api/users/:id',
  authMiddleware.authenticate,
  authMiddleware.authorize('admin'),
  (req, res) => {
    res.json({ success: true })
  }
)

// Optional authentication
app.get(
  '/api/products/:id',
  authMiddleware.optionalAuth,
  (req: AuthRequest, res) => {
    // req.user will be present if authenticated
    res.json({ product: {}, isAuthenticated: !!req.user })
  }
)
```

### Role-Based Access Control (RBAC)

```typescript
// src/auth/RbacService.ts
export interface Permission {
  resource: string
  action: string
}

export interface Role {
  name: string
  permissions: Permission[]
}

export class RbacService {
  private roles: Map<string, Role> = new Map()

  defineRole(role: Role): void {
    this.roles.set(role.name, role)
  }

  hasPermission(
    roleName: string,
    resource: string,
    action: string
  ): boolean {
    const role = this.roles.get(roleName)
    
    if (!role) {
      return false
    }

    return role.permissions.some(
      p => p.resource === resource && p.action === action
    )
  }

  checkPermission(
    roleName: string,
    resource: string,
    action: string
  ): void {
    if (!this.hasPermission(roleName, resource, action)) {
      throw new Error('Permission denied')
    }
  }

  getRolePermissions(roleName: string): Permission[] {
    return this.roles.get(roleName)?.permissions || []
  }
}

// Define roles
const rbac = new RbacService()

rbac.defineRole({
  name: 'user',
  permissions: [
    { resource: 'product', action: 'read' },
    { resource: 'order', action: 'read' },
    { resource: 'order', action: 'create' },
    { resource: 'profile', action: 'read' },
    { resource: 'profile', action: 'update' }
  ]
})

rbac.defineRole({
  name: 'admin',
  permissions: [
    { resource: 'product', action: 'read' },
    { resource: 'product', action: 'create' },
    { resource: 'product', action: 'update' },
    { resource: 'product', action: 'delete' },
    { resource: 'order', action: 'read' },
    { resource: 'order', action: 'update' },
    { resource: 'user', action: 'read' },
    { resource: 'user', action: 'update' },
    { resource: 'user', action: 'delete' }
  ]
})

// Middleware
export function requirePermission(resource: string, action: string) {
  return (req: AuthRequest, res: Response, next: NextFunction): void => {
    if (!req.user) {
      res.status(401).json({ error: 'Not authenticated' })
      return
    }

    try {
      rbac.checkPermission(req.user.role, resource, action)
      next()
    } catch (error) {
      res.status(403).json({ error: 'Permission denied' })
    }
  }
}

// Usage
app.delete(
  '/api/products/:id',
  authMiddleware.authenticate,
  requirePermission('product', 'delete'),
  (req, res) => {
    res.json({ success: true })
  }
)
```

---

## 7.7 File Upload and Processing

### File Upload Service

```typescript
// src/services/FileUploadService.ts
import multer from 'multer'
import sharp from 'sharp'
import * as path from 'path'
import * as fs from 'fs/promises'
import { v4 as uuidv4 } from 'uuid'

export interface UploadedFile {
  id: string
  originalName: string
  filename: string
  path: string
  mimetype: string
  size: number
  url: string
}

export interface ImageProcessingOptions {
  resize?: {
    width: number
    height: number
  }
  format?: 'jpeg' | 'png' | 'webp'
  quality?: number
}

export class FileUploadService {
  private readonly uploadDir: string
  private readonly maxFileSize: number

  constructor(uploadDir: string = './uploads', maxFileSize: number = 5 * 1024 * 1024) {
    this.uploadDir = uploadDir
    this.maxFileSize = maxFileSize
    this.ensureUploadDir()
  }

  private async ensureUploadDir(): Promise<void> {
    try {
      await fs.access(this.uploadDir)
    } catch {
      await fs.mkdir(this.uploadDir, { recursive: true })
    }
  }

  getMulterConfig(): multer.Options {
    const storage = multer.diskStorage({
      destination: (req, file, cb) => {
        cb(null, this.uploadDir)
      },
      filename: (req, file, cb) => {
        const uniqueName = `${uuidv4()}${path.extname(file.originalname)}`
        cb(null, uniqueName)
      }
    })

    return {
      storage,
      limits: {
        fileSize: this.maxFileSize
      },
      fileFilter: (req, file, cb) => {
        const allowedTypes = /jpeg|jpg|png|gif|pdf|doc|docx/
        const extname = allowedTypes.test(
          path.extname(file.originalname).toLowerCase()
        )
        const mimetype = allowedTypes.test(file.mimetype)

        if (extname && mimetype) {
          cb(null, true)
        } else {
          cb(new Error('Invalid file type'))
        }
      }
    }
  }

  async processImage(
    filePath: string,
    options: ImageProcessingOptions = {}
  ): Promise<string> {
    const processedPath = filePath.replace(
      path.extname(filePath),
      `-processed${path.extname(filePath)}`
    )

    let pipeline = sharp(filePath)

    if (options.resize) {
      pipeline = pipeline.resize(options.resize.width, options.resize.height, {
        fit: 'cover'
      })
    }

    if (options.format) {
      pipeline = pipeline.toFormat(options.format, {
        quality: options.quality || 80
      })
    }

    await pipeline.toFile(processedPath)

    return processedPath
  }

  async generateThumbnail(
    filePath: string,
    width: number = 200,
    height: number = 200
  ): Promise<string> {
    const thumbnailPath = filePath.replace(
      path.extname(filePath),
      `-thumb${path.extname(filePath)}`
    )

    await sharp(filePath)
      .resize(width, height, { fit: 'cover' })
      .toFile(thumbnailPath)

    return thumbnailPath
  }

  async deleteFile(filePath: string): Promise<void> {
    try {
      await fs.unlink(filePath)
    } catch (error) {
      console.error('Error deleting file:', error)
    }
  }

  getFileUrl(filename: string): string {
    return `/uploads/${filename}`
  }

  async getFileInfo(filePath: string): Promise<{
    size: number
    created: Date
    modified: Date
  }> {
    const stats = await fs.stat(filePath)
    
    return {
      size: stats.size,
      created: stats.birthtime,
      modified: stats.mtime
    }
  }
}
```

### Upload Controller

```typescript
// src/controllers/UploadController.ts
import { Request, Response } from 'express'
import multer from 'multer'
import { FileUploadService } from '../services/FileUploadService'

export class UploadController {
  private uploadService: FileUploadService
  private upload: multer.Multer

  constructor() {
    this.uploadService = new FileUploadService()
    this.upload = multer(this.uploadService.getMulterConfig())
  }

  uploadSingle = this.upload.single('file')

  uploadMultiple = this.upload.array('files', 10)

  async handleSingleUpload(req: Request, res: Response): Promise<void> {
    try {
      if (!req.file) {
        res.status(400).json({ error: 'No file uploaded' })
        return
      }

      const file = req.file
      const url = this.uploadService.getFileUrl(file.filename)

      // Process image if it's an image
      if (file.mimetype.startsWith('image/')) {
        const processedPath = await this.uploadService.processImage(
          file.path,
          {
            resize: { width: 1200, height: 1200 },
            format: 'webp',
            quality: 85
          }
        )

        const thumbnailPath = await this.uploadService.generateThumbnail(
          processedPath
        )

        res.json({
          success: true,
          file: {
            id: file.filename,
            originalName: file.originalname,
            filename: file.filename,
            mimetype: file.mimetype,
            size: file.size,
            url,
            thumbnailUrl: this.uploadService.getFileUrl(
              path.basename(thumbnailPath)
            )
          }
        })
      } else {
        res.json({
          success: true,
          file: {
            id: file.filename,
            originalName: file.originalname,
            filename: file.filename,
            mimetype: file.mimetype,
            size: file.size,
            url
          }
        })
      }
    } catch (error) {
      res.status(500).json({
        error: error instanceof Error ? error.message : 'Upload failed'
      })
    }
  }

  async handleMultipleUpload(req: Request, res: Response): Promise<void> {
    try {
      if (!req.files || !Array.isArray(req.files)) {
        res.status(400).json({ error: 'No files uploaded' })
        return
      }

      const files = req.files.map(file => ({
        id: file.filename,
        originalName: file.originalname,
        filename: file.filename,
        mimetype: file.mimetype,
        size: file.size,
        url: this.uploadService.getFileUrl(file.filename)
      }))

      res.json({
        success: true,
        files
      })
    } catch (error) {
      res.status(500).json({
        error: error instanceof Error ? error.message : 'Upload failed'
      })
    }
  }

  async deleteFile(req: Request, res: Response): Promise<void> {
    try {
      const { filename } = req.params
      const filePath = path.join('./uploads', filename)

      await this.uploadService.deleteFile(filePath)

      res.json({ success: true })
    } catch (error) {
      res.status(500).json({
        error: error instanceof Error ? error.message : 'Delete failed'
      })
    }
  }
}

// Routes
import express from 'express'

const router = express.Router()
const uploadController = new UploadController()

router.post(
  '/upload',
  uploadController.uploadSingle,
  (req, res) => uploadController.handleSingleUpload(req, res)
)

router.post(
  '/upload/multiple',
  uploadController.uploadMultiple,
  (req, res) => uploadController.handleMultipleUpload(req, res)
)

router.delete('/upload/:filename', (req, res) =>
  uploadController.deleteFile(req, res)
)

export default router
```

### S3 Upload Service

```typescript
// src/services/S3UploadService.ts
import { S3Client, PutObjectCommand, DeleteObjectCommand } from '@aws-sdk/client-s3'
import { getSignedUrl } from '@aws-sdk/s3-request-presigner'
import { v4 as uuidv4 } from 'uuid'

export interface S3UploadOptions {
  bucket: string
  key: string
  body: Buffer
  contentType: string
  acl?: string
}

export class S3UploadService {
  private s3Client: S3Client
  private bucket: string

  constructor(region: string, bucket: string) {
    this.s3Client = new S3Client({ region })
    this.bucket = bucket
  }

  async uploadFile(
    file: Express.Multer.File,
    folder: string = 'uploads'
  ): Promise<{ key: string; url: string }> {
    const key = `${folder}/${uuidv4()}-${file.originalname}`

    const command = new PutObjectCommand({
      Bucket: this.bucket,
      Key: key,
      Body: file.buffer,
      ContentType: file.mimetype,
      ACL: 'public-read'
    })

    await this.s3Client.send(command)

    const url = `https://${this.bucket}.s3.amazonaws.com/${key}`

    return { key, url }
  }

  async deleteFile(key: string): Promise<void> {
    const command = new DeleteObjectCommand({
      Bucket: this.bucket,
      Key: key
    })

    await this.s3Client.send(command)
  }

  async getSignedUploadUrl(
    filename: string,
    contentType: string,
    expiresIn: number = 3600
  ): Promise<{ key: string; url: string }> {
    const key = `uploads/${uuidv4()}-${filename}`

    const command = new PutObjectCommand({
      Bucket: this.bucket,
      Key: key,
      ContentType: contentType
    })

    const url = await getSignedUrl(this.s3Client, command, { expiresIn })

    return { key, url }
  }

  async getSignedDownloadUrl(
    key: string,
    expiresIn: number = 3600
  ): Promise<string> {
    const command = new GetObjectCommand({
      Bucket: this.bucket,
      Key: key
    })

    return getSignedUrl(this.s3Client, command, { expiresIn })
  }
}
```

---

## Conclusion of Part 7

This concludes Part 7 of the Complete TypeScript Mastery series!

**What We've Covered:**

1. **Microservices Architecture** - Complete distributed system
2. **Real-Time Applications** - WebSockets and Socket.IO
3. **CLI Tools** - Command-line application development
4. **Monorepo Management** - Turborepo and shared packages
5. **E-commerce Architecture** - Complete online store system
6. **Authentication & Authorization** - JWT, RBAC, security
7. **File Upload & Processing** - Local and S3 integration

---

## Complete Series Summary

### All Seven Parts:

| Part | Focus | Words |
|------|-------|-------|
| Part 1 | TypeScript Fundamentals | 25,231 |
| Part 2 | Advanced Type System | 19,179 |
| Part 3 | Compiler & Tooling | 6,966 |
| Part 4 | React & Node.js | 10,949 |
| Part 5 | Design Patterns | 6,968 |
| Part 6 | Framework Ecosystem | 7,950 |
| Part 7 | Real-World Applications | 11,000+ |
| **Total** | **Complete Mastery** | **~88,000** |

**You now have the most comprehensive TypeScript course available!** 🎉
