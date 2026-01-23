---
title: "10 TypeScript Practice Projects with Requirements"
date: 2026-01-08 19:00:00 +0530
categories: [Programming, TypeScript, Projects]
tags: [typescript, projects, practice, portfolio]
---

# 10 TypeScript Practice Projects

> Hands-on projects to practice and showcase your TypeScript skills

---

## 📋 How to Use This Guide

Each project includes:
- **Difficulty Level:** ⭐ Beginner | ⭐⭐ Intermediate | ⭐⭐⭐ Advanced
- **Time Estimate:** Expected completion time
- **Tech Stack:** Required technologies
- **Learning Objectives:** Skills you'll practice
- **Core Features:** Must-have functionality
- **Bonus Features:** Extra challenges
- **Implementation Tips:** Helpful guidance

---

## 🎯 Beginner Projects (After Parts 1-3)

### **Project 1: TypeScript Todo Application**
**Difficulty:** ⭐ Beginner  
**Time:** 8-12 hours  
**Best After:** Part 1 (Fundamentals)

#### **Tech Stack**
```typescript
// Core
- TypeScript
- HTML/CSS
- Local Storage

// Optional
- Vite for building
- CSS framework (Tailwind/Bootstrap)
```

#### **Learning Objectives**
- ✅ Basic TypeScript syntax
- ✅ Interfaces and types
- ✅ Classes and methods
- ✅ DOM manipulation with types
- ✅ Local storage typing
- ✅ Enums for status

#### **Core Features**
```typescript
// Required functionality
1. Add new todo items
2. Mark todos as complete/incomplete
3. Delete todo items
4. Filter by status (all/active/completed)
5. Persist data to localStorage
6. Edit existing todos

// Data structure
interface Todo {
  id: string
  title: string
  description?: string
  completed: boolean
  createdAt: Date
  priority: Priority
}

enum Priority {
  Low = 'low',
  Medium = 'medium',
  High = 'high'
}
```

#### **Bonus Features**
- 🎁 Due dates with date validation
- 🎁 Categories/tags system
- 🎁 Search functionality
- 🎁 Sort by different criteria
- 🎁 Dark mode toggle
- 🎁 Keyboard shortcuts
- 🎁 Undo/redo functionality

#### **Implementation Tips**
```typescript
// Start with these classes
class TodoManager {
  private todos: Todo[] = []
  
  addTodo(todo: Omit<Todo, 'id' | 'createdAt'>): Todo {
    // Implementation
  }
  
  updateTodo(id: string, updates: Partial<Todo>): void {
    // Implementation
  }
  
  deleteTodo(id: string): void {
    // Implementation
  }
  
  filterTodos(status: 'all' | 'active' | 'completed'): Todo[] {
    // Implementation
  }
}

// Local storage helper
class StorageService<T> {
  constructor(private key: string) {}
  
  save(data: T): void {
    localStorage.setItem(this.key, JSON.stringify(data))
  }
  
  load(): T | null {
    const data = localStorage.getItem(this.key)
    return data ? JSON.parse(data) : null
  }
}
```

#### **Success Criteria**
- ✅ All features working without runtime errors
- ✅ Type-safe throughout
- ✅ No use of `any` type
- ✅ Clean, maintainable code
- ✅ Proper error handling

---

### **Project 2: Weather Dashboard**
**Difficulty:** ⭐ Beginner  
**Time:** 10-15 hours  
**Best After:** Part 1 & Part 2 (Advanced Types)

#### **Tech Stack**
```typescript
// Core
- TypeScript
- Weather API (OpenWeatherMap)
- Fetch API
- Chart.js or similar

// Optional
- React or vanilla TypeScript
- Axios for HTTP
```

#### **Learning Objectives**
- ✅ API integration with types
- ✅ Async/await with TypeScript
- ✅ Response type definitions
- ✅ Error handling
- ✅ Generic functions
- ✅ Union types for weather conditions

#### **Core Features**
```typescript
// Required functionality
1. Search cities by name
2. Display current weather
3. 5-day forecast
4. Temperature unit conversion (C/F)
5. Weather icons/conditions
6. Save favorite cities

// Type definitions
interface WeatherData {
  city: string
  country: string
  temperature: number
  feelsLike: number
  humidity: number
  windSpeed: number
  description: string
  icon: string
  timestamp: Date
}

interface ForecastDay {
  date: Date
  high: number
  low: number
  conditions: WeatherCondition
  precipitation: number
}

type WeatherCondition = 
  | 'Clear'
  | 'Clouds'
  | 'Rain'
  | 'Snow'
  | 'Thunderstorm'
  | 'Drizzle'
  | 'Mist'

interface WeatherResponse {
  coord: { lon: number; lat: number }
  weather: Array<{
    id: number
    main: string
    description: string
    icon: string
  }>
  main: {
    temp: number
    feels_like: number
    temp_min: number
    temp_max: number
    pressure: number
    humidity: number
  }
  wind: {
    speed: number
    deg: number
  }
  // ... more fields
}
```

#### **Bonus Features**
- 🎁 Geolocation support
- 🎁 Weather alerts/warnings
- 🎁 Historical data charts
- 🎁 Air quality index
- 🎁 UV index
- 🎁 Sunrise/sunset times
- 🎁 Multiple language support

#### **Implementation Tips**
```typescript
// API service with generics
class WeatherService {
  private baseUrl = 'https://api.openweathermap.org/data/2.5'
  
  async getCurrentWeather(city: string): Promise<WeatherData> {
    const response = await this.fetchWeather<WeatherResponse>(
      `/weather?q=${city}`
    )
    return this.transformWeatherData(response)
  }
  
  private async fetchWeather<T>(endpoint: string): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}&appid=${API_KEY}`)
    if (!response.ok) {
      throw new WeatherError('Failed to fetch weather data')
    }
    return response.json()
  }
  
  private transformWeatherData(data: WeatherResponse): WeatherData {
    return {
      city: data.name,
      temperature: data.main.temp,
      // ... transform all fields
    }
  }
}

// Custom error class
class WeatherError extends Error {
  constructor(
    message: string,
    public code?: string
  ) {
    super(message)
    this.name = 'WeatherError'
  }
}

// Temperature conversion utility
class TemperatureConverter {
  static celsiusToFahrenheit(celsius: number): number {
    return (celsius * 9/5) + 32
  }
  
  static fahrenheitToCelsius(fahrenheit: number): number {
    return (fahrenheit - 32) * 5/9
  }
  
  static kelvinToCelsius(kelvin: number): number {
    return kelvin - 273.15
  }
}
```

#### **Success Criteria**
- ✅ Successful API integration
- ✅ All API responses properly typed
- ✅ Graceful error handling
- ✅ Loading states
- ✅ Responsive design

---

### **Project 3: TypeScript Calculator**
**Difficulty:** ⭐ Beginner  
**Time:** 6-10 hours  
**Best After:** Part 1 (Fundamentals)

#### **Tech Stack**
```typescript
// Core
- TypeScript
- HTML/CSS
- Optional: React/Vue for UI
```

#### **Learning Objectives**
- ✅ Function overloading
- ✅ Type guards
- ✅ Union types
- ✅ Enums for operations
- ✅ Error handling
- ✅ Design patterns (Strategy pattern)

#### **Core Features**
```typescript
// Required functionality
1. Basic operations (+, -, *, /)
2. Clear and clear entry
3. Decimal point support
4. Percentage calculations
5. Memory functions (M+, M-, MR, MC)
6. Keyboard input support

// Type definitions
enum Operation {
  Add = 'add',
  Subtract = 'subtract',
  Multiply = 'multiply',
  Divide = 'divide',
  Percentage = 'percentage'
}

interface CalculatorState {
  display: string
  memory: number
  previousValue: number | null
  currentOperation: Operation | null
  waitingForOperand: boolean
}

type CalculatorInput = number | Operation | 'clear' | 'equals'
```

#### **Bonus Features**
- 🎁 Scientific calculator mode
- 🎁 History of calculations
- 🎁 Multiple themes
- 🎁 Expression evaluation (parse strings like "2 + 3 * 4")
- 🎁 Unit conversions
- 🎁 Programmable functions

#### **Implementation Tips**
```typescript
// Strategy pattern for operations
interface CalculationStrategy {
  calculate(a: number, b: number): number
}

class AddStrategy implements CalculationStrategy {
  calculate(a: number, b: number): number {
    return a + b
  }
}

class Calculator {
  private state: CalculatorState = {
    display: '0',
    memory: 0,
    previousValue: null,
    currentOperation: null,
    waitingForOperand: false
  }
  
  private strategies: Map<Operation, CalculationStrategy> = new Map([
    [Operation.Add, new AddStrategy()],
    [Operation.Subtract, new SubtractStrategy()],
    // ... more strategies
  ])
  
  input(value: CalculatorInput): void {
    if (typeof value === 'number') {
      this.inputNumber(value)
    } else if (this.isOperation(value)) {
      this.inputOperation(value)
    }
  }
  
  private isOperation(value: CalculatorInput): value is Operation {
    return Object.values(Operation).includes(value as Operation)
  }
  
  calculate(): number {
    if (this.state.previousValue === null || !this.state.currentOperation) {
      throw new Error('Invalid calculation state')
    }
    
    const strategy = this.strategies.get(this.state.currentOperation)
    if (!strategy) {
      throw new Error('Unknown operation')
    }
    
    return strategy.calculate(
      this.state.previousValue,
      parseFloat(this.state.display)
    )
  }
}
```

#### **Success Criteria**
- ✅ Accurate calculations
- ✅ Proper decimal handling
- ✅ No floating-point errors
- ✅ Edge case handling (division by zero, etc.)
- ✅ Clean UI/UX

---

## 🚀 Intermediate Projects (After Parts 4-6)

### **Project 4: E-commerce Store with Cart**
**Difficulty:** ⭐⭐ Intermediate  
**Time:** 25-35 hours  
**Best After:** Part 4 (Practice) & Part 6 (Frameworks)

#### **Tech Stack**
```typescript
// Frontend
- React with TypeScript
- React Router
- Context API or Zustand
- Tailwind CSS

// Backend (Optional)
- Express with TypeScript
- SQLite or PostgreSQL
- Prisma ORM

// Tools
- Vite
- ESLint
- Prettier
```

#### **Learning Objectives**
- ✅ React with TypeScript
- ✅ State management
- ✅ Routing with types
- ✅ Form handling
- ✅ API integration
- ✅ Shopping cart logic
- ✅ Local storage persistence

#### **Core Features**
```typescript
// Required functionality
1. Product listing with filters
2. Product detail pages
3. Shopping cart functionality
4. Add/remove/update quantities
5. Checkout process
6. Order summary
7. Search and filtering
8. Category navigation

// Type definitions
interface Product {
  id: string
  name: string
  description: string
  price: number
  category: Category
  images: string[]
  stock: number
  rating: number
  reviews: Review[]
}

interface CartItem {
  product: Product
  quantity: number
}

interface Cart {
  items: CartItem[]
  subtotal: number
  tax: number
  shipping: number
  total: number
}

interface Order {
  id: string
  items: CartItem[]
  total: number
  status: OrderStatus
  createdAt: Date
  shippingAddress: Address
  paymentMethod: PaymentMethod
}

enum OrderStatus {
  Pending = 'pending',
  Processing = 'processing',
  Shipped = 'shipped',
  Delivered = 'delivered',
  Cancelled = 'cancelled'
}

type Category = 
  | 'Electronics'
  | 'Clothing'
  | 'Books'
  | 'Home'
  | 'Sports'

interface Address {
  street: string
  city: string
  state: string
  zipCode: string
  country: string
}
```

#### **Bonus Features**
- 🎁 User authentication
- 🎁 Wishlist functionality
- 🎁 Product reviews and ratings
- 🎁 Related products
- 🎁 Discount codes
- 🎁 Order history
- 🎁 Payment integration (Stripe)
- 🎁 Inventory management
- 🎁 Admin dashboard

#### **Implementation Tips**
```typescript
// Cart context with TypeScript
interface CartContextType {
  cart: Cart
  addItem: (product: Product, quantity?: number) => void
  removeItem: (productId: string) => void
  updateQuantity: (productId: string, quantity: number) => void
  clearCart: () => void
  calculateTotals: () => void
}

const CartContext = createContext<CartContextType | undefined>(undefined)

export const CartProvider: FC<{ children: ReactNode }> = ({ children }) => {
  const [cart, setCart] = useState<Cart>({
    items: [],
    subtotal: 0,
    tax: 0,
    shipping: 0,
    total: 0
  })
  
  const addItem = (product: Product, quantity: number = 1) => {
    setCart(prev => {
      const existingItem = prev.items.find(item => item.product.id === product.id)
      
      if (existingItem) {
        return {
          ...prev,
          items: prev.items.map(item =>
            item.product.id === product.id
              ? { ...item, quantity: item.quantity + quantity }
              : item
          )
        }
      }
      
      return {
        ...prev,
        items: [...prev.items, { product, quantity }]
      }
    })
  }
  
  // ... more methods
  
  return (
    <CartContext.Provider value={{ cart, addItem, removeItem, updateQuantity, clearCart, calculateTotals }}>
      {children}
    </CartContext.Provider>
  )
}

// Custom hook
export const useCart = () => {
  const context = useContext(CartContext)
  if (!context) {
    throw new Error('useCart must be used within CartProvider')
  }
  return context
}

// Product service
class ProductService {
  async getProducts(filters?: ProductFilters): Promise<Product[]> {
    const query = this.buildQuery(filters)
    const response = await fetch(`/api/products${query}`)
    return response.json()
  }
  
  async getProductById(id: string): Promise<Product> {
    const response = await fetch(`/api/products/${id}`)
    if (!response.ok) {
      throw new Error('Product not found')
    }
    return response.json()
  }
  
  private buildQuery(filters?: ProductFilters): string {
    if (!filters) return ''
    
    const params = new URLSearchParams()
    if (filters.category) params.set('category', filters.category)
    if (filters.minPrice) params.set('minPrice', filters.minPrice.toString())
    if (filters.maxPrice) params.set('maxPrice', filters.maxPrice.toString())
    if (filters.search) params.set('search', filters.search)
    
    return `?${params.toString()}`
  }
}
```

#### **Success Criteria**
- ✅ Fully functional shopping cart
- ✅ Persistent state
- ✅ Responsive design
- ✅ Type-safe throughout
- ✅ Good UX (loading states, error handling)
- ✅ Clean code architecture

---

### **Project 5: Social Media Dashboard (Twitter/X Clone)**
**Difficulty:** ⭐⭐ Intermediate  
**Time:** 30-40 hours  
**Best After:** Part 4 (Practice) & Part 6 (Frameworks)

#### **Tech Stack**
```typescript
// Frontend
- React with TypeScript
- React Query for data fetching
- Socket.io for real-time updates
- TailwindCSS

// Backend
- Express with TypeScript
- PostgreSQL with Prisma
- WebSocket server
- JWT authentication

// Additional
- Redis for caching
- AWS S3 for images
```

#### **Learning Objectives**
- ✅ Full-stack TypeScript
- ✅ Real-time features
- ✅ Authentication & authorization
- ✅ File uploads
- ✅ Infinite scrolling
- ✅ Optimistic updates
- ✅ Complex state management

#### **Core Features**
```typescript
// Required functionality
1. User authentication (signup/login)
2. Create/edit/delete posts
3. Like and retweet posts
4. Comment on posts
5. Follow/unfollow users
6. User profiles
7. Timeline feed
8. Real-time notifications
9. Search users and posts

// Type definitions
interface User {
  id: string
  username: string
  displayName: string
  email: string
  bio?: string
  avatar?: string
  coverPhoto?: string
  followers: number
  following: number
  createdAt: Date
}

interface Post {
  id: string
  content: string
  author: User
  images?: string[]
  likes: number
  retweets: number
  comments: number
  createdAt: Date
  isLiked: boolean
  isRetweeted: boolean
}

interface Comment {
  id: string
  content: string
  author: User
  postId: string
  createdAt: Date
  likes: number
}

interface Notification {
  id: string
  type: NotificationType
  actor: User
  post?: Post
  createdAt: Date
  read: boolean
}

enum NotificationType {
  Like = 'like',
  Retweet = 'retweet',
  Comment = 'comment',
  Follow = 'follow',
  Mention = 'mention'
}

interface Timeline {
  posts: Post[]
  hasMore: boolean
  nextCursor?: string
}
```

#### **Bonus Features**
- 🎁 Direct messaging
- 🎁 Trending topics
- 🎁 Hashtags
- 🎁 Mentions (@username)
- 🎁 Bookmarks
- 🎁 Lists
- 🎁 Advanced search
- 🎁 Analytics dashboard
- 🎁 Thread posts
- 🎁 Poll creation

#### **Implementation Tips**
```typescript
// Post service with optimistic updates
class PostService {
  async createPost(content: string, images?: File[]): Promise<Post> {
    const formData = new FormData()
    formData.append('content', content)
    images?.forEach(image => formData.append('images', image))
    
    const response = await fetch('/api/posts', {
      method: 'POST',
      body: formData,
      headers: {
        'Authorization': `Bearer ${getToken()}`
      }
    })
    
    if (!response.ok) {
      throw new Error('Failed to create post')
    }
    
    return response.json()
  }
  
  async likePost(postId: string): Promise<void> {
    await fetch(`/api/posts/${postId}/like`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${getToken()}`
      }
    })
  }
}

// Real-time updates with Socket.io
class RealtimeService {
  private socket: Socket
  
  constructor() {
    this.socket = io('http://localhost:3000', {
      auth: {
        token: getToken()
      }
    })
    
    this.setupListeners()
  }
  
  private setupListeners() {
    this.socket.on('notification', (notification: Notification) => {
      // Handle new notification
      this.onNotification?.(notification)
    })
    
    this.socket.on('post:liked', (data: { postId: string; userId: string }) => {
      // Handle post liked event
      this.onPostLiked?.(data)
    })
  }
  
  onNotification?: (notification: Notification) => void
  onPostLiked?: (data: { postId: string; userId: string }) => void
}

// Infinite scroll hook
function useInfiniteTimeline() {
  return useInfiniteQuery({
    queryKey: ['timeline'],
    queryFn: ({ pageParam = null }) => 
      fetchTimeline({ cursor: pageParam }),
    getNextPageParam: (lastPage) => 
      lastPage.hasMore ? lastPage.nextCursor : undefined
  })
}

// Backend API route
router.post('/posts', authenticate, async (req: Request, res: Response) => {
  const { content } = req.body
  const userId = req.user!.id
  
  const post = await prisma.post.create({
    data: {
      content,
      authorId: userId
    },
    include: {
      author: true
    }
  })
  
  // Emit real-time event
  io.emit('post:created', post)
  
  res.json(post)
})
```

#### **Success Criteria**
- ✅ Real-time updates working
- ✅ Secure authentication
- ✅ Optimistic UI updates
- ✅ Infinite scrolling
- ✅ File upload working
- ✅ Responsive design
- ✅ Performance optimized

---

### **Project 6: Task Management System (Trello Clone)**
**Difficulty:** ⭐⭐ Intermediate  
**Time:** 25-30 hours  
**Best After:** Part 4 & Part 5 (Patterns)

#### **Tech Stack**
```typescript
// Frontend
- React with TypeScript
- React DnD (Drag and Drop)
- React Router
- Zustand for state

// Backend
- Express with TypeScript
- MongoDB with Mongoose
- Socket.io

// Optional
- Next.js for SSR
```

#### **Learning Objectives**
- ✅ Drag and drop functionality
- ✅ Complex state management
- ✅ Real-time collaboration
- ✅ Design patterns (Observer, Command)
- ✅ Undo/redo functionality
- ✅ Optimistic updates

#### **Core Features**
```typescript
// Required functionality
1. Create/edit/delete boards
2. Create/edit/delete lists
3. Create/edit/delete cards
4. Drag and drop cards between lists
5. Drag and drop lists
6. Add labels to cards
7. Add due dates
8. Add descriptions and checklists
9. Assign members to cards
10. Card comments

// Type definitions
interface Board {
  id: string
  title: string
  description?: string
  lists: List[]
  members: User[]
  backgroundColor: string
  createdAt: Date
  updatedAt: Date
}

interface List {
  id: string
  title: string
  boardId: string
  cards: Card[]
  position: number
}

interface Card {
  id: string
  title: string
  description?: string
  listId: string
  position: number
  labels: Label[]
  assignees: User[]
  dueDate?: Date
  checklist: ChecklistItem[]
  comments: Comment[]
  attachments: Attachment[]
  createdAt: Date
  updatedAt: Date
}

interface Label {
  id: string
  name: string
  color: string
}

interface ChecklistItem {
  id: string
  text: string
  completed: boolean
}

// Drag and drop types
interface DragItem {
  type: 'CARD' | 'LIST'
  id: string
  index: number
}

interface DropResult {
  listId: string
  index: number
}
```

#### **Bonus Features**
- 🎁 Board templates
- 🎁 Activity log
- 🎁 Search and filter
- 🎁 Card covers
- 🎁 Power-ups/plugins
- 🎁 Calendar view
- 🎁 Archive/restore
- 🎁 Export board data
- 🎁 Keyboard shortcuts

#### **Implementation Tips**
```typescript
// Board store with Zustand
interface BoardStore {
  boards: Board[]
  activeBoard: Board | null
  
  // Board actions
  createBoard: (title: string) => void
  updateBoard: (id: string, updates: Partial<Board>) => void
  deleteBoard: (id: string) => void
  setActiveBoard: (id: string) => void
  
  // List actions
  createList: (boardId: string, title: string) => void
  updateList: (id: string, updates: Partial<List>) => void
  deleteList: (id: string) => void
  moveList: (listId: string, newPosition: number) => void
  
  // Card actions
  createCard: (listId: string, title: string) => void
  updateCard: (id: string, updates: Partial<Card>) => void
  deleteCard: (id: string) => void
  moveCard: (cardId: string, targetListId: string, position: number) => void
}

const useBoardStore = create<BoardStore>((set, get) => ({
  boards: [],
  activeBoard: null,
  
  createList: (boardId, title) => set(state => ({
    boards: state.boards.map(board =>
      board.id === boardId
        ? {
            ...board,
            lists: [...board.lists, {
              id: generateId(),
              title,
              boardId,
              cards: [],
              position: board.lists.length
            }]
          }
        : board
    )
  })),
  
  moveCard: (cardId, targetListId, position) => {
    const { activeBoard } = get()
    if (!activeBoard) return
    
    set(state => {
      // Implementation of card movement logic
      // This involves removing card from source list
      // and adding to target list at specified position
    })
  }
  
  // ... more actions
}))

// Drag and Drop component
const DraggableCard: FC<{ card: Card; index: number }> = ({ card, index }) => {
  const [{ isDragging }, drag] = useDrag({
    type: 'CARD',
    item: { type: 'CARD', id: card.id, index },
    collect: monitor => ({
      isDragging: monitor.isDragging()
    })
  })
  
  return (
    <div
      ref={drag}
      style={{ opacity: isDragging ? 0.5 : 1 }}
      className="card"
    >
      {card.title}
    </div>
  )
}

const DroppableList: FC<{ list: List }> = ({ list }) => {
  const moveCard = useBoardStore(state => state.moveCard)
  
  const [{ isOver }, drop] = useDrop({
    accept: 'CARD',
    drop: (item: DragItem, monitor) => {
      if (!monitor.didDrop()) {
        moveCard(item.id, list.id, list.cards.length)
      }
    },
    collect: monitor => ({
      isOver: monitor.isOver()
    })
  })
  
  return (
    <div ref={drop} className={isOver ? 'highlight' : ''}>
      {list.cards.map((card, index) => (
        <DraggableCard key={card.id} card={card} index={index} />
      ))}
    </div>
  )
}

// Undo/Redo with Command pattern
interface Command {
  execute(): void
  undo(): void
}

class MoveCardCommand implements Command {
  constructor(
    private cardId: string,
    private sourceListId: string,
    private targetListId: string,
    private position: number,
    private store: BoardStore
  ) {}
  
  execute(): void {
    this.store.moveCard(this.cardId, this.targetListId, this.position)
  }
  
  undo(): void {
    this.store.moveCard(this.cardId, this.sourceListId, this.position)
  }
}

class CommandHistory {
  private history: Command[] = []
  private currentIndex = -1
  
  execute(command: Command): void {
    command.execute()
    this.history = this.history.slice(0, this.currentIndex + 1)
    this.history.push(command)
    this.currentIndex++
  }
  
  undo(): void {
    if (this.currentIndex >= 0) {
      this.history[this.currentIndex].undo()
      this.currentIndex--
    }
  }
  
  redo(): void {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++
      this.history[this.currentIndex].execute()
    }
  }
}
```

#### **Success Criteria**
- ✅ Smooth drag and drop
- ✅ Real-time collaboration
- ✅ Undo/redo functionality
- ✅ Persistent data
- ✅ Responsive design
- ✅ Performance with many cards

---

## 🌟 Advanced Projects (After Parts 7-8)

### **Project 7: Microservices Platform**
**Difficulty:** ⭐⭐⭐ Advanced  
**Time:** 50-70 hours  
**Best After:** Part 7 (Real-World) & Part 8 (Production)

#### **Tech Stack**
```typescript
// Services
- User Service (Express + TypeScript)
- Product Service (NestJS + TypeScript)
- Order Service (Express + TypeScript)
- Notification Service (Node.js + TypeScript)

// Infrastructure
- Docker & Docker Compose
- Kubernetes (optional)
- RabbitMQ or Kafka
- Redis for caching
- PostgreSQL databases

// API Gateway
- Express with TypeScript
- JWT authentication
- Rate limiting

// Monitoring
- Prometheus
- Grafana
- Winston logging
```

#### **Learning Objectives**
- ✅ Microservices architecture
- ✅ Service-to-service communication
- ✅ Message queues
- ✅ API Gateway pattern
- ✅ Service discovery
- ✅ Distributed tracing
- ✅ Event-driven architecture
- ✅ Container orchestration

#### **Core Features**
```typescript
// Service architecture
1. User Service
   - User registration/authentication
   - Profile management
   - Role-based access control
   
2. Product Service
   - Product CRUD
   - Inventory management
   - Categories and search
   
3. Order Service
   - Order creation
   - Order status tracking
   - Payment processing (mock)
   
4. Notification Service
   - Email notifications
   - Real-time notifications
   - Event subscribers

5. API Gateway
   - Request routing
   - Authentication
   - Rate limiting
   - Load balancing

// Shared types (in a common package)
interface ServiceEvent<T = any> {
  eventType: string
  timestamp: Date
  data: T
  correlationId: string
}

interface UserCreatedEvent extends ServiceEvent<User> {
  eventType: 'user.created'
}

interface OrderPlacedEvent extends ServiceEvent<Order> {
  eventType: 'order.placed'
}

// Message broker interface
interface MessageBroker {
  publish(topic: string, event: ServiceEvent): Promise<void>
  subscribe(topic: string, handler: (event: ServiceEvent) => void): void
}
```

#### **Bonus Features**
- 🎁 Service mesh (Istio)
- 🎁 Circuit breaker pattern
- 🎁 Saga pattern for distributed transactions
- 🎁 CQRS implementation
- 🎁 Event sourcing
- 🎁 Health checks
- 🎁 Distributed caching
- 🎁 API versioning

#### **Implementation Tips**
```typescript
// Shared message broker implementation
class RabbitMQBroker implements MessageBroker {
  private connection: Connection
  private channel: Channel
  
  async connect(): Promise<void> {
    this.connection = await amqp.connect(process.env.RABBITMQ_URL!)
    this.channel = await this.connection.createChannel()
  }
  
  async publish(topic: string, event: ServiceEvent): Promise<void> {
    await this.channel.assertExchange(topic, 'fanout', { durable: true })
    this.channel.publish(
      topic,
      '',
      Buffer.from(JSON.stringify(event)),
      { persistent: true }
    )
  }
  
  async subscribe(topic: string, handler: (event: ServiceEvent) => void): Promise<void> {
    await this.channel.assertExchange(topic, 'fanout', { durable: true })
    const { queue } = await this.channel.assertQueue('', { exclusive: true })
    await this.channel.bindQueue(queue, topic, '')
    
    this.channel.consume(queue, (msg) => {
      if (msg) {
        const event = JSON.parse(msg.content.toString())
        handler(event)
        this.channel.ack(msg)
      }
    })
  }
}

// Service base class
abstract class BaseService {
  protected broker: MessageBroker
  protected logger: Logger
  
  constructor(broker: MessageBroker, logger: Logger) {
    this.broker = broker
    this.logger = logger
  }
  
  protected async publishEvent(event: ServiceEvent): Promise<void> {
    try {
      await this.broker.publish(event.eventType, event)
      this.logger.info(`Event published: ${event.eventType}`)
    } catch (error) {
      this.logger.error(`Failed to publish event: ${event.eventType}`, error)
      throw error
    }
  }
}

// Docker Compose example
version: '3.8'
services:
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - rabbitmq
      
  user-service:
    build: ./user-service
    environment:
      - DATABASE_URL=postgresql://user:pass@user-db:5432/users
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - user-db
      - rabbitmq
      
  product-service:
    build: ./product-service
    environment:
      - DATABASE_URL=postgresql://user:pass@product-db:5432/products
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - product-db
      - rabbitmq
      
  order-service:
    build: ./order-service
    environment:
      - DATABASE_URL=postgresql://user:pass@order-db:5432/orders
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - order-db
      - rabbitmq
      
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
      
  user-db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=users
      
  # ... more databases
```

#### **Success Criteria**
- ✅ All services communicating properly
- ✅ Event-driven architecture working
- ✅ API Gateway routing correctly
- ✅ Containerized and orchestrated
- ✅ Logging and monitoring setup
- ✅ Error handling and resilience

---

### **Project 8: Real-Time Collaboration Tool (Google Docs Clone)**
**Difficulty:** ⭐⭐⭐ Advanced  
**Time:** 60-80 hours  
**Best After:** Part 7 (Real-World)

#### **Tech Stack**
```typescript
// Frontend
- React with TypeScript
- Slate.js or Draft.js (rich text editor)
- Socket.io client
- Yjs or Automerge (CRDT)

// Backend
- Express with TypeScript
- Socket.io server
- Redis for presence
- MongoDB for document storage

// Real-time
- Operational Transformation or CRDT
- WebRTC for cursors
```

#### **Learning Objectives**
- ✅ Real-time collaboration algorithms
- ✅ Operational Transformation or CRDT
- ✅ WebSocket at scale
- ✅ Presence awareness
- ✅ Conflict resolution
- ✅ Document versioning
- ✅ Performance optimization for large documents

#### **Core Features**
```typescript
// Required functionality
1. Real-time collaborative editing
2. Multiple cursors/selections
3. Document creation/sharing
4. Rich text formatting
5. Comments and suggestions
6. Version history
7. Offline editing with sync
8. Presence indicators
9. Document permissions

// Type definitions
interface Document {
  id: string
  title: string
  content: any // Editor-specific format
  owner: User
  collaborators: Collaborator[]
  permissions: DocumentPermissions
  version: number
  history: DocumentVersion[]
  createdAt: Date
  updatedAt: Date
}

interface Collaborator {
  user: User
  role: 'viewer' | 'commenter' | 'editor'
  cursor?: CursorPosition
  selection?: SelectionRange
  color: string
  isOnline: boolean
}

interface Operation {
  type: 'insert' | 'delete' | 'format'
  position: number
  content?: string
  attributes?: TextAttributes
  userId: string
  timestamp: number
}

interface CursorPosition {
  line: number
  column: number
}

interface SelectionRange {
  start: CursorPosition
  end: CursorPosition
}

interface TextAttributes {
  bold?: boolean
  italic?: boolean
  underline?: boolean
  color?: string
  fontSize?: number
}
```

#### **Bonus Features**
- 🎁 Voice/video chat
- 🎁 Templates
- 🎁 Export to PDF/Word
- 🎁 Advanced formatting (tables, images)
- 🎁 Spell check
- 🎁 Find and replace
- 🎁 Document outline
- 🎁 Mobile app

#### **Implementation Tips**
```typescript
// Operational Transformation
class OperationalTransform {
  static transform(op1: Operation, op2: Operation): [Operation, Operation] {
    // Transform operations to handle conflicts
    if (op1.type === 'insert' && op2.type === 'insert') {
      if (op1.position < op2.position) {
        return [op1, { ...op2, position: op2.position + op1.content!.length }]
      } else if (op1.position > op2.position) {
        return [{ ...op1, position: op1.position + op2.content!.length }, op2]
      }
      // Handle same position - use timestamp or userId
    }
    // ... handle other operation combinations
    return [op1, op2]
  }
  
  static apply(content: string, operation: Operation): string {
    switch (operation.type) {
      case 'insert':
        return (
          content.slice(0, operation.position) +
          operation.content +
          content.slice(operation.position)
        )
      case 'delete':
        return (
          content.slice(0, operation.position) +
          content.slice(operation.position + (operation.content?.length || 0))
        )
      default:
        return content
    }
  }
}

// Real-time collaboration server
class CollaborationServer {
  private io: Server
  private documents: Map<string, DocumentState> = new Map()
  
  constructor(io: Server) {
    this.io = io
    this.setupHandlers()
  }
  
  private setupHandlers() {
    this.io.on('connection', (socket: Socket) => {
      socket.on('join-document', async (documentId: string) => {
        await this.handleJoinDocument(socket, documentId)
      })
      
      socket.on('operation', async (data: { documentId: string; operation: Operation }) => {
        await this.handleOperation(socket, data)
      })
      
      socket.on('cursor-move', (data: { documentId: string; position: CursorPosition }) => {
        this.broadcastCursorMove(socket, data)
      })
    })
  }
  
  private async handleOperation(
    socket: Socket,
    data: { documentId: string; operation: Operation }
  ) {
    const state = this.documents.get(data.documentId)
    if (!state) return
    
    // Transform operation against concurrent operations
    const transformed = this.transformOperation(data.operation, state.pending)
    
    // Apply to document
    state.content = OperationalTransform.apply(state.content, transformed)
    state.version++
    
    // Broadcast to other clients
    socket.to(data.documentId).emit('operation', {
      operation: transformed,
      version: state.version
    })
    
    // Save to database
    await this.saveDocument(data.documentId, state)
  }
}

// Client-side editor integration
class CollaborativeEditor {
  private socket: Socket
  private editor: Editor
  private pendingOperations: Operation[] = []
  
  constructor(documentId: string, editor: Editor) {
    this.editor = editor
    this.socket = io('http://localhost:3000')
    this.setupListeners(documentId)
  }
  
  private setupListeners(documentId: string) {
    this.socket.emit('join-document', documentId)
    
    this.socket.on('operation', (data: { operation: Operation; version: number }) => {
      this.applyRemoteOperation(data.operation)
    })
    
    this.editor.onChange = (operation: Operation) => {
      this.socket.emit('operation', { documentId, operation })
    }
  }
  
  private applyRemoteOperation(operation: Operation) {
    // Transform against pending operations
    const transformed = this.transformAgainstPending(operation)
    
    // Apply to editor
    this.editor.applyOperation(transformed)
  }
}
```

#### **Success Criteria**
- ✅ Smooth real-time collaboration
- ✅ No conflicts or data loss
- ✅ Performant with multiple users
- ✅ Offline support
- ✅ Version history working
- ✅ Cursor presence accurate

---

### **Project 9: DevOps Monitoring Dashboard**
**Difficulty:** ⭐⭐⭐ Advanced  
**Time:** 40-50 hours  
**Best After:** Part 8 (Production)

#### **Tech Stack**
```typescript
// Frontend
- React with TypeScript
- Recharts or Chart.js
- Socket.io for real-time updates
- TailwindCSS

// Backend
- Express with TypeScript
- Prometheus client
- Node exporter
- Docker SDK

// Infrastructure
- Prometheus
- Grafana (optional)
- Docker & Docker Compose
```

#### **Learning Objectives**
- ✅ Metrics collection
- ✅ Real-time monitoring
- ✅ Alerting systems
- ✅ Docker container monitoring
- ✅ Log aggregation
- ✅ Performance metrics
- ✅ Custom metrics

#### **Core Features**
```typescript
// Required functionality
1. System metrics dashboard
   - CPU usage
   - Memory usage
   - Disk I/O
   - Network traffic
   
2. Container monitoring
   - Running containers
   - Container stats
   - Logs viewer
   
3. Application metrics
   - Request rate
   - Response time
   - Error rate
   - Active users
   
4. Alerts and notifications
   - Threshold-based alerts
   - Email/Slack notifications
   - Alert history
   
5. Log viewer
   - Real-time logs
   - Search and filter
   - Log levels

// Type definitions
interface SystemMetrics {
  timestamp: Date
  cpu: {
    usage: number
    cores: number
  }
  memory: {
    total: number
    used: number
    free: number
    percentage: number
  }
  disk: {
    total: number
    used: number
    free: number
    percentage: number
  }
  network: {
    rxBytes: number
    txBytes: number
  }
}

interface ContainerStats {
  id: string
  name: string
  status: 'running' | 'stopped' | 'paused'
  cpu: number
  memory: number
  network: {
    rx: number
    tx: number
  }
  uptime: number
}

interface ApplicationMetric {
  name: string
  value: number
  type: 'counter' | 'gauge' | 'histogram'
  labels: Record<string, string>
  timestamp: Date
}

interface Alert {
  id: string
  name: string
  condition: AlertCondition
  status: 'active' | 'resolved'
  severity: 'low' | 'medium' | 'high' | 'critical'
  triggeredAt: Date
  resolvedAt?: Date
  message: string
}

interface AlertCondition {
  metric: string
  operator: '>' | '<' | '>=' | '<=' | '=='
  threshold: number
  duration: number // seconds
}
```

#### **Bonus Features**
- 🎁 Custom dashboards
- 🎁 Metric correlations
- 🎁 Anomaly detection (ML)
- 🎁 Cost tracking (AWS/GCP)
- 🎁 SLA tracking
- 🎁 Incident management
- 🎁 Runbook automation

#### **Implementation Tips**
```typescript
// Metrics collector
import * as promClient from 'prom-client'
import * as os from 'os'
import * as si from 'systeminformation'

class MetricsCollector {
  private register = new promClient.Registry()
  
  // System metrics
  private cpuGauge = new promClient.Gauge({
    name: 'system_cpu_usage',
    help: 'CPU usage percentage',
    registers: [this.register]
  })
  
  private memoryGauge = new promClient.Gauge({
    name: 'system_memory_usage',
    help: 'Memory usage in bytes',
    labelNames: ['type'],
    registers: [this.register]
  })
  
  // Application metrics
  private httpRequestCounter = new promClient.Counter({
    name: 'http_requests_total',
    help: 'Total HTTP requests',
    labelNames: ['method', 'route', 'status'],
    registers: [this.register]
  })
  
  private httpRequestDuration = new promClient.Histogram({
    name: 'http_request_duration_seconds',
    help: 'HTTP request duration',
    labelNames: ['method', 'route'],
    buckets: [0.1, 0.5, 1, 2, 5],
    registers: [this.register]
  })
  
  constructor() {
    this.startCollection()
  }
  
  private startCollection() {
    setInterval(async () => {
      await this.collectSystemMetrics()
    }, 5000)
  }
  
  private async collectSystemMetrics() {
    const cpu = await si.currentLoad()
    this.cpuGauge.set(cpu.currentLoad)
    
    const mem = await si.mem()
    this.memoryGauge.set({ type: 'used' }, mem.used)
    this.memoryGauge.set({ type: 'free' }, mem.free)
  }
  
  recordHttpRequest(method: string, route: string, status: number, duration: number) {
    this.httpRequestCounter.inc({ method, route, status: status.toString() })
    this.httpRequestDuration.observe({ method, route }, duration / 1000)
  }
  
  getMetrics(): Promise<string> {
    return this.register.metrics()
  }
}

// Alert manager
class AlertManager {
  private alerts: Map<string, Alert> = new Map()
  private conditions: AlertCondition[] = []
  
  addCondition(condition: AlertCondition): void {
    this.conditions.push(condition)
  }
  
  async checkConditions(metrics: ApplicationMetric[]): Promise<Alert[]> {
    const triggeredAlerts: Alert[] = []
    
    for (const condition of this.conditions) {
      const metric = metrics.find(m => m.name === condition.metric)
      if (!metric) continue
      
      const triggered = this.evaluateCondition(metric.value, condition)
      
      if (triggered) {
        const alert = this.createAlert(condition, metric)
        this.alerts.set(alert.id, alert)
        triggeredAlerts.push(alert)
        
        await this.sendNotification(alert)
      }
    }
    
    return triggeredAlerts
  }
  
  private evaluateCondition(value: number, condition: AlertCondition): boolean {
    switch (condition.operator) {
      case '>': return value > condition.threshold
      case '<': return value < condition.threshold
      case '>=': return value >= condition.threshold
      case '<=': return value <= condition.threshold
      case '==': return value === condition.threshold
      default: return false
    }
  }
  
  private async sendNotification(alert: Alert): Promise<void> {
    // Send to Slack, email, etc.
    console.log(`Alert triggered: ${alert.message}`)
  }
}

// Docker monitoring
import Docker from 'dockerode'

class DockerMonitor {
  private docker = new Docker()
  
  async getContainerStats(): Promise<ContainerStats[]> {
    const containers = await this.docker.listContainers()
    
    const stats = await Promise.all(
      containers.map(async (container) => {
        const containerObj = this.docker.getContainer(container.Id)
        const stats = await containerObj.stats({ stream: false })
        
        return this.parseStats(container, stats)
      })
    )
    
    return stats
  }
  
  private parseStats(container: any, stats: any): ContainerStats {
    const cpuDelta = stats.cpu_stats.cpu_usage.total_usage - 
                     stats.precpu_stats.cpu_usage.total_usage
    const systemDelta = stats.cpu_stats.system_cpu_usage - 
                        stats.precpu_stats.system_cpu_usage
    const cpuPercent = (cpuDelta / systemDelta) * 100
    
    const memoryUsage = stats.memory_stats.usage
    const memoryLimit = stats.memory_stats.limit
    const memoryPercent = (memoryUsage / memoryLimit) * 100
    
    return {
      id: container.Id,
      name: container.Names[0],
      status: container.State as any,
      cpu: cpuPercent,
      memory: memoryPercent,
      network: {
        rx: stats.networks?.eth0?.rx_bytes || 0,
        tx: stats.networks?.eth0?.tx_bytes || 0
      },
      uptime: Date.now() - (container.Created * 1000)
    }
  }
  
  async streamLogs(containerId: string): Promise<NodeJS.ReadableStream> {
    const container = this.docker.getContainer(containerId)
    return container.logs({
      follow: true,
      stdout: true,
      stderr: true,
      timestamps: true
    })
  }
}

// Real-time dashboard updates
class DashboardServer {
  private io: Server
  private metricsCollector: MetricsCollector
  private dockerMonitor: DockerMonitor
  
  constructor(server: http.Server) {
    this.io = new Server(server)
    this.metricsCollector = new MetricsCollector()
    this.dockerMonitor = new DockerMonitor()
    
    this.setupHandlers()
    this.startBroadcasting()
  }
  
  private setupHandlers() {
    this.io.on('connection', (socket) => {
      socket.on('subscribe-metrics', () => {
        socket.join('metrics')
      })
      
      socket.on('subscribe-containers', () => {
        socket.join('containers')
      })
    })
  }
  
  private startBroadcasting() {
    setInterval(async () => {
      const metrics = await this.collectMetrics()
      this.io.to('metrics').emit('metrics-update', metrics)
    }, 1000)
    
    setInterval(async () => {
      const containers = await this.dockerMonitor.getContainerStats()
      this.io.to('containers').emit('containers-update', containers)
    }, 2000)
  }
}
```

#### **Success Criteria**
- ✅ Real-time metrics updating
- ✅ Accurate system monitoring
- ✅ Container stats working
- ✅ Alerts triggering correctly
- ✅ Performance optimized
- ✅ Intuitive dashboard UI

---

### **Project 10: Full-Stack SaaS Application**
**Difficulty:** ⭐⭐⭐ Advanced  
**Time:** 80-100 hours  
**Best After:** Complete Series

#### **Summary**
Build a complete Software-as-a-Service application combining all learned concepts:

**Features:**
- Multi-tenant architecture
- Subscription/billing (Stripe)
- Team collaboration
- Role-based access control
- Real-time features
- File storage
- Email system
- Analytics dashboard
- Admin panel
- API with rate limiting
- Microservices (optional)
- Full CI/CD pipeline
- Kubernetes deployment
- Monitoring and logging

**This is your capstone project!** 🎓

---

## 📊 Project Difficulty Matrix

| Project | TypeScript | React | Backend | DevOps | Time |
|---------|-----------|-------|---------|--------|------|
| Todo App | ⭐ | ⭐ | - | - | 8-12h |
| Weather | ⭐ | ⭐ | ⭐ | - | 10-15h |
| Calculator | ⭐ | ⭐ | - | - | 6-10h |
| E-commerce | ⭐⭐ | ⭐⭐ | ⭐⭐ | - | 25-35h |
| Social Media | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐ | 30-40h |
| Task Manager | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ | - | 25-30h |
| Microservices | ⭐⭐⭐ | ⭐ | ⭐⭐⭐ | ⭐⭐⭐ | 50-70h |
| Collab Tool | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐ | 60-80h |
| Monitoring | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | 40-50h |
| SaaS | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | 80-100h |

---

## 🎯 Recommended Project Sequence

### **For Complete Beginners:**
1. Todo App (Week 1)
2. Calculator (Week 2)
3. Weather Dashboard (Week 3)

### **For Intermediate Learners:**
4. E-commerce Store (Weeks 4-5)
5. Task Manager (Week 6)
6. Social Media Dashboard (Weeks 7-8)

### **For Advanced Learners:**
7. Microservices Platform (Weeks 9-11)
8. Real-Time Collaboration (Weeks 12-14)
9. Monitoring Dashboard (Weeks 15-16)
10. Full SaaS Application (Weeks 17-24)

---

## 💡 Project Tips

### **Before Starting**
- ✅ Read relevant parts of the series
- ✅ Plan the architecture
- ✅ Create a task list
- ✅ Setup version control (Git)
- ✅ Choose tech stack

### **During Development**
- ✅ Commit frequently
- ✅ Write tests as you go
- ✅ Document as you build
- ✅ Refactor regularly
- ✅ Ask for help when stuck

### **After Completion**
- ✅ Deploy to production
- ✅ Add to portfolio
- ✅ Write a blog post
- ✅ Share on GitHub
- ✅ Get feedback

---

**Start building and showcase your TypeScript mastery!** 🚀

*Project Ideas v1.0 - Complete TypeScript Mastery Series*
