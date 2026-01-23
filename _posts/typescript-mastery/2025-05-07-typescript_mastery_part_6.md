---
title: "Complete TypeScript Mastery Part 6: Ecosystem & Framework Integration"
date: 2026-01-08 15:00:00 +0530
categories: [Programming, TypeScript, Web Development]
tags: [typescript, vue, angular, nextjs, nestjs, graphql, orm, ecosystem]
---

# Complete TypeScript Mastery Part 6: Ecosystem & Framework Integration

## Introduction

Welcome to Part 6 of the Complete TypeScript Mastery series! In Parts 1-5, we covered TypeScript fundamentals, advanced features, compiler tooling, practical applications, and design patterns. Now we'll explore the rich TypeScript ecosystem and how to integrate TypeScript with popular frameworks and tools.

This part focuses on real-world integrations:
- TypeScript with Vue.js
- TypeScript with Angular
- TypeScript with Next.js
- TypeScript with NestJS
- GraphQL and TypeScript
- Database ORMs (TypeORM, Prisma)
- Testing frameworks
- Build tools and bundlers
- Mobile development with React Native
- Publishing TypeScript libraries

By the end of this part, you'll be equipped to work with TypeScript across the entire modern web development ecosystem.

---

## 6.1 TypeScript with Vue.js

### Vue 3 Setup with TypeScript

```bash
# Create Vue 3 project with TypeScript
npm create vue@latest my-vue-app

# Select TypeScript when prompted
# ✔ Add TypeScript? Yes
# ✔ Add JSX Support? Yes (optional)
# ✔ Add Vue Router? Yes
# ✔ Add Pinia? Yes
# ✔ Add Vitest? Yes

cd my-vue-app
npm install
```

**tsconfig.json for Vue:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "preserve",
    "moduleResolution": "bundler",
    
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### Component Definition with Composition API

```typescript
// Basic component with TypeScript
<script setup lang="ts">
import { ref, computed } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

const user = ref<User>({
  id: 1,
  name: 'Alice',
  email: 'alice@example.com'
})

const greeting = computed(() => `Hello, ${user.value.name}!`)

function updateName(newName: string): void {
  user.value.name = newName
}
</script>

<template>
  <div>
    <h1>{{ greeting }}</h1>
    <p>Email: {{ user.email }}</p>
    <button @click="updateName('Bob')">Change Name</button>
  </div>
</template>

// Component with props
<script setup lang="ts">
interface Props {
  title: string
  count?: number
  items: string[]
  onUpdate?: (value: number) => void
}

const props = withDefaults(defineProps<Props>(), {
  count: 0
})

const emit = defineEmits<{
  update: [value: number]
  delete: []
}>()

function handleClick(): void {
  emit('update', props.count + 1)
}
</script>

<template>
  <div>
    <h2>{{ title }}</h2>
    <p>Count: {{ count }}</p>
    <button @click="handleClick">Increment</button>
  </div>
</template>

// Generic component
<script setup lang="ts" generic="T extends { id: number }">
interface Props {
  items: T[]
  selectedId?: number
}

const props = defineProps<Props>()

const emit = defineEmits<{
  select: [item: T]
}>()

function selectItem(item: T): void {
  emit('select', item)
}
</script>

<template>
  <ul>
    <li
      v-for="item in items"
      :key="item.id"
      @click="selectItem(item)"
      :class="{ selected: item.id === selectedId }"
    >
      <slot :item="item">{{ item.id }}</slot>
    </li>
  </ul>
</template>
```

### Composables (Custom Hooks)

```typescript
// composables/useFetch.ts
import { ref, Ref } from 'vue'

interface UseFetchReturn<T> {
  data: Ref<T | null>
  error: Ref<Error | null>
  loading: Ref<boolean>
  refetch: () => Promise<void>
}

export function useFetch<T>(url: string): UseFetchReturn<T> {
  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)
  const loading = ref(false)

  async function fetchData(): Promise<void> {
    loading.value = true
    error.value = null

    try {
      const response = await fetch(url)
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      
      data.value = await response.json()
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }

  fetchData()

  return {
    data,
    error,
    loading,
    refetch: fetchData
  }
}

// Usage in component
<script setup lang="ts">
import { useFetch } from '@/composables/useFetch'

interface User {
  id: number
  name: string
  email: string
}

const { data: user, error, loading, refetch } = useFetch<User>('/api/user/1')
</script>

<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error">Error: {{ error.message }}</div>
    <div v-else-if="user">
      <h1>{{ user.name }}</h1>
      <p>{{ user.email }}</p>
      <button @click="refetch">Refresh</button>
    </div>
  </div>
</template>

// composables/useLocalStorage.ts
import { ref, watch, Ref } from 'vue'

export function useLocalStorage<T>(
  key: string,
  defaultValue: T
): Ref<T> {
  const storedValue = localStorage.getItem(key)
  const data = ref<T>(
    storedValue ? JSON.parse(storedValue) : defaultValue
  ) as Ref<T>

  watch(
    data,
    (newValue) => {
      localStorage.setItem(key, JSON.stringify(newValue))
    },
    { deep: true }
  )

  return data
}

// Usage
const user = useLocalStorage<User>('user', {
  id: 0,
  name: '',
  email: ''
})
```

### Pinia Store with TypeScript

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface User {
  id: string
  name: string
  email: string
  role: 'admin' | 'user'
}

interface LoginCredentials {
  email: string
  password: string
}

export const useUserStore = defineStore('user', () => {
  // State
  const user = ref<User | null>(null)
  const token = ref<string | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)

  // Getters
  const isAuthenticated = computed(() => !!user.value && !!token.value)
  const isAdmin = computed(() => user.value?.role === 'admin')

  // Actions
  async function login(credentials: LoginCredentials): Promise<void> {
    loading.value = true
    error.value = null

    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      })

      if (!response.ok) {
        throw new Error('Login failed')
      }

      const data = await response.json()
      user.value = data.user
      token.value = data.token
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Login failed'
      throw e
    } finally {
      loading.value = false
    }
  }

  function logout(): void {
    user.value = null
    token.value = null
  }

  async function fetchProfile(): Promise<void> {
    if (!token.value) return

    loading.value = true

    try {
      const response = await fetch('/api/user/profile', {
        headers: { Authorization: `Bearer ${token.value}` }
      })

      if (!response.ok) {
        throw new Error('Failed to fetch profile')
      }

      user.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Failed to fetch profile'
    } finally {
      loading.value = false
    }
  }

  return {
    // State
    user,
    token,
    loading,
    error,
    // Getters
    isAuthenticated,
    isAdmin,
    // Actions
    login,
    logout,
    fetchProfile
  }
})

// Usage in component
<script setup lang="ts">
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

async function handleLogin(): Promise<void> {
  await userStore.login({
    email: 'user@example.com',
    password: 'password123'
  })
}
</script>

<template>
  <div>
    <div v-if="userStore.isAuthenticated">
      <h1>Welcome, {{ userStore.user?.name }}</h1>
      <p v-if="userStore.isAdmin">You are an admin</p>
      <button @click="userStore.logout">Logout</button>
    </div>
    <div v-else>
      <button @click="handleLogin">Login</button>
    </div>
  </div>
</template>
```

### Vue Router with TypeScript

```typescript
// router/index.ts
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'
import { useUserStore } from '@/stores/user'

// Route meta typing
declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    requiresAdmin?: boolean
    title?: string
  }
}

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue'),
    meta: { title: 'Home' }
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/Login.vue'),
    meta: { title: 'Login' }
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('@/views/Dashboard.vue'),
    meta: { requiresAuth: true, title: 'Dashboard' }
  },
  {
    path: '/admin',
    name: 'Admin',
    component: () => import('@/views/Admin.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Admin' }
  },
  {
    path: '/users/:id',
    name: 'UserProfile',
    component: () => import('@/views/UserProfile.vue'),
    props: (route) => ({ userId: Number(route.params.id) }),
    meta: { requiresAuth: true, title: 'User Profile' }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// Navigation guards
router.beforeEach((to, from, next) => {
  const userStore = useUserStore()

  // Set page title
  document.title = to.meta.title || 'My App'

  // Check authentication
  if (to.meta.requiresAuth && !userStore.isAuthenticated) {
    next({ name: 'Login', query: { redirect: to.fullPath } })
    return
  }

  // Check admin access
  if (to.meta.requiresAdmin && !userStore.isAdmin) {
    next({ name: 'Home' })
    return
  }

  next()
})

export default router

// Type-safe navigation
import { useRouter } from 'vue-router'

export function useTypedRouter() {
  const router = useRouter()

  function navigateToUser(userId: number): void {
    router.push({ name: 'UserProfile', params: { id: userId } })
  }

  function navigateToLogin(redirect?: string): void {
    router.push({
      name: 'Login',
      query: redirect ? { redirect } : undefined
    })
  }

  return {
    navigateToUser,
    navigateToLogin
  }
}
```

### Testing Vue Components

```typescript
// UserProfile.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import { createPinia, setActivePinia } from 'pinia'
import UserProfile from '@/components/UserProfile.vue'
import { useUserStore } from '@/stores/user'

describe('UserProfile', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('renders user information', () => {
    const wrapper = mount(UserProfile, {
      props: {
        userId: 1
      }
    })

    expect(wrapper.find('h1').text()).toBe('User Profile')
  })

  it('calls store action on mount', async () => {
    const userStore = useUserStore()
    const fetchSpy = vi.spyOn(userStore, 'fetchProfile')

    mount(UserProfile, {
      props: { userId: 1 }
    })

    expect(fetchSpy).toHaveBeenCalled()
  })

  it('emits update event', async () => {
    const wrapper = mount(UserProfile, {
      props: { userId: 1 }
    })

    await wrapper.find('button').trigger('click')

    expect(wrapper.emitted('update')).toBeTruthy()
    expect(wrapper.emitted('update')?.[0]).toEqual([1])
  })
})
```

---

## 6.2 TypeScript with Angular

### Angular Project Setup

```bash
# Create new Angular project with TypeScript (default)
ng new my-angular-app

# Angular CLI automatically configures TypeScript
cd my-angular-app
```

**tsconfig.json for Angular:**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "lib": ["ES2022", "dom"],
    "moduleResolution": "node",
    
    "strict": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "strictPropertyInitialization": true,
    
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    
    "baseUrl": "./",
    "paths": {
      "@app/*": ["src/app/*"],
      "@environments/*": ["src/environments/*"]
    }
  }
}
```

### Component Definition

```typescript
// user-profile.component.ts
import { Component, Input, Output, EventEmitter, OnInit } from '@angular/core'
import { CommonModule } from '@angular/common'

interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user'
}

@Component({
  selector: 'app-user-profile',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="user-profile">
      <h2>{{ user?.name }}</h2>
      <p>{{ user?.email }}</p>
      <span [class.admin]="isAdmin">{{ user?.role }}</span>
      <button (click)="handleEdit()">Edit</button>
    </div>
  `,
  styles: [`
    .user-profile {
      padding: 1rem;
      border: 1px solid #ccc;
    }
    .admin {
      color: red;
      font-weight: bold;
    }
  `]
})
export class UserProfileComponent implements OnInit {
  @Input() user: User | null = null
  @Output() edit = new EventEmitter<User>()

  get isAdmin(): boolean {
    return this.user?.role === 'admin'
  }

  ngOnInit(): void {
    console.log('Component initialized')
  }

  handleEdit(): void {
    if (this.user) {
      this.edit.emit(this.user)
    }
  }
}

// Generic component
@Component({
  selector: 'app-list',
  standalone: true,
  imports: [CommonModule],
  template: `
    <ul>
      <li *ngFor="let item of items; trackBy: trackByFn" (click)="onSelect(item)">
        <ng-content [ngTemplateOutlet]="itemTemplate" [ngTemplateOutletContext]="{ $implicit: item }"></ng-content>
      </li>
    </ul>
  `
})
export class ListComponent<T extends { id: number | string }> {
  @Input() items: T[] = []
  @Input() itemTemplate: any
  @Output() select = new EventEmitter<T>()

  trackByFn(index: number, item: T): number | string {
    return item.id
  }

  onSelect(item: T): void {
    this.select.emit(item)
  }
}
```

### Services with Dependency Injection

```typescript
// services/user.service.ts
import { Injectable } from '@angular/core'
import { HttpClient, HttpHeaders } from '@angular/common/http'
import { Observable, BehaviorSubject, throwError } from 'rxjs'
import { catchError, map, tap } from 'rxjs/operators'

interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user'
}

interface LoginCredentials {
  email: string
  password: string
}

interface AuthResponse {
  user: User
  token: string
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private readonly apiUrl = 'https://api.example.com'
  private userSubject = new BehaviorSubject<User | null>(null)
  private tokenSubject = new BehaviorSubject<string | null>(null)

  public user$ = this.userSubject.asObservable()
  public token$ = this.tokenSubject.asObservable()

  constructor(private http: HttpClient) {
    // Load from localStorage on init
    const token = localStorage.getItem('token')
    if (token) {
      this.tokenSubject.next(token)
      this.fetchProfile().subscribe()
    }
  }

  login(credentials: LoginCredentials): Observable<AuthResponse> {
    return this.http
      .post<AuthResponse>(`${this.apiUrl}/auth/login`, credentials)
      .pipe(
        tap(response => {
          this.userSubject.next(response.user)
          this.tokenSubject.next(response.token)
          localStorage.setItem('token', response.token)
        }),
        catchError(this.handleError)
      )
  }

  logout(): void {
    this.userSubject.next(null)
    this.tokenSubject.next(null)
    localStorage.removeItem('token')
  }

  fetchProfile(): Observable<User> {
    const headers = new HttpHeaders({
      Authorization: `Bearer ${this.tokenSubject.value}`
    })

    return this.http
      .get<User>(`${this.apiUrl}/user/profile`, { headers })
      .pipe(
        tap(user => this.userSubject.next(user)),
        catchError(this.handleError)
      )
  }

  getUser(id: number): Observable<User> {
    return this.http
      .get<User>(`${this.apiUrl}/users/${id}`)
      .pipe(catchError(this.handleError))
  }

  updateUser(id: number, updates: Partial<User>): Observable<User> {
    const headers = new HttpHeaders({
      Authorization: `Bearer ${this.tokenSubject.value}`
    })

    return this.http
      .put<User>(`${this.apiUrl}/users/${id}`, updates, { headers })
      .pipe(
        tap(user => {
          if (user.id === this.userSubject.value?.id) {
            this.userSubject.next(user)
          }
        }),
        catchError(this.handleError)
      )
  }

  private handleError(error: any): Observable<never> {
    console.error('An error occurred:', error)
    return throwError(() => new Error(error.message || 'Server error'))
  }

  get currentUser(): User | null {
    return this.userSubject.value
  }

  get isAuthenticated(): boolean {
    return !!this.tokenSubject.value
  }
}

// Usage in component
import { Component, OnInit } from '@angular/core'
import { UserService } from '@app/services/user.service'

@Component({
  selector: 'app-dashboard',
  template: `
    <div *ngIf="user$ | async as user">
      <h1>Welcome, {{ user.name }}</h1>
    </div>
  `
})
export class DashboardComponent implements OnInit {
  user$ = this.userService.user$

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    // Component logic
  }
}
```

### RxJS Operators with TypeScript

```typescript
// operators/custom-operators.ts
import { Observable, OperatorFunction } from 'rxjs'
import { map, filter, catchError, retry, debounceTime } from 'rxjs/operators'

// Type-safe filter operator
export function filterByType<T, S extends T>(
  predicate: (value: T) => value is S
): OperatorFunction<T, S> {
  return (source: Observable<T>) => source.pipe(filter(predicate))
}

// Usage
interface Animal {
  type: string
}

interface Dog extends Animal {
  type: 'dog'
  bark: () => void
}

const animals$ = new Observable<Animal>()
const dogs$ = animals$.pipe(
  filterByType<Animal, Dog>((animal): animal is Dog => animal.type === 'dog')
)

// Type-safe pluck operator
export function pluck<T, K extends keyof T>(key: K): OperatorFunction<T, T[K]> {
  return (source: Observable<T>) => source.pipe(map(value => value[key]))
}

// Usage
interface User {
  id: number
  name: string
  email: string
}

const users$ = new Observable<User>()
const names$ = users$.pipe(pluck('name'))  // Observable<string>

// Retry with exponential backoff
export function retryWithBackoff(
  maxRetries: number = 3,
  delayMs: number = 1000
): OperatorFunction<any, any> {
  return (source: Observable<any>) =>
    source.pipe(
      retry({
        count: maxRetries,
        delay: (error, retryCount) => {
          console.log(`Retry attempt ${retryCount}`)
          return new Observable(subscriber => {
            setTimeout(() => {
              subscriber.next()
              subscriber.complete()
            }, delayMs * Math.pow(2, retryCount - 1))
          })
        }
      })
    )
}

// Type-safe search operator
export function search<T>(
  getSearchTerm: (value: T) => string,
  searchFn: (term: string) => Observable<T[]>
): OperatorFunction<T, T[]> {
  return (source: Observable<T>) =>
    source.pipe(
      debounceTime(300),
      map(getSearchTerm),
      filter(term => term.length > 2),
      switchMap(searchFn)
    )
}
```

### Angular Routing with Type Safety

```typescript
// app.routes.ts
import { Routes } from '@angular/router'
import { inject } from '@angular/core'
import { Router } from '@angular/router'
import { UserService } from '@app/services/user.service'
import { map } from 'rxjs/operators'

// Route guards
export const authGuard = () => {
  const userService = inject(UserService)
  const router = inject(Router)

  return userService.user$.pipe(
    map(user => {
      if (user) {
        return true
      }
      router.navigate(['/login'])
      return false
    })
  )
}

export const adminGuard = () => {
  const userService = inject(UserService)
  const router = inject(Router)

  return userService.user$.pipe(
    map(user => {
      if (user && user.role === 'admin') {
        return true
      }
      router.navigate(['/'])
      return false
    })
  )
}

export const routes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./pages/home/home.component').then(m => m.HomeComponent)
  },
  {
    path: 'login',
    loadComponent: () =>
      import('./pages/login/login.component').then(m => m.LoginComponent)
  },
  {
    path: 'dashboard',
    loadComponent: () =>
      import('./pages/dashboard/dashboard.component').then(m => m.DashboardComponent),
    canActivate: [authGuard]
  },
  {
    path: 'admin',
    loadComponent: () =>
      import('./pages/admin/admin.component').then(m => m.AdminComponent),
    canActivate: [authGuard, adminGuard]
  },
  {
    path: 'users/:id',
    loadComponent: () =>
      import('./pages/user-profile/user-profile.component').then(m => m.UserProfileComponent),
    canActivate: [authGuard]
  },
  {
    path: '**',
    redirectTo: ''
  }
]

// Type-safe navigation service
import { Injectable } from '@angular/core'
import { Router } from '@angular/router'

@Injectable({
  providedIn: 'root'
})
export class NavigationService {
  constructor(private router: Router) {}

  navigateToHome(): Promise<boolean> {
    return this.router.navigate(['/'])
  }

  navigateToLogin(returnUrl?: string): Promise<boolean> {
    return this.router.navigate(['/login'], {
      queryParams: returnUrl ? { returnUrl } : {}
    })
  }

  navigateToUser(userId: number): Promise<boolean> {
    return this.router.navigate(['/users', userId])
  }

  navigateToDashboard(): Promise<boolean> {
    return this.router.navigate(['/dashboard'])
  }
}
```

### Testing Angular Components

```typescript
// user-profile.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing'
import { UserProfileComponent } from './user-profile.component'
import { UserService } from '@app/services/user.service'
import { of } from 'rxjs'

describe('UserProfileComponent', () => {
  let component: UserProfileComponent
  let fixture: ComponentFixture<UserProfileComponent>
  let userService: jasmine.SpyObj<UserService>

  beforeEach(async () => {
    const userServiceSpy = jasmine.createSpyObj('UserService', [
      'getUser',
      'updateUser'
    ])

    await TestBed.configureTestingModule({
      imports: [UserProfileComponent],
      providers: [{ provide: UserService, useValue: userServiceSpy }]
    }).compileComponents()

    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>
    fixture = TestBed.createComponent(UserProfileComponent)
    component = fixture.componentInstance
  })

  it('should create', () => {
    expect(component).toBeTruthy()
  })

  it('should load user on init', () => {
    const mockUser = {
      id: 1,
      name: 'Alice',
      email: 'alice@example.com',
      role: 'user' as const
    }

    userService.getUser.and.returnValue(of(mockUser))
    component.user = mockUser

    fixture.detectChanges()

    expect(component.user).toEqual(mockUser)
  })

  it('should emit edit event', () => {
    const mockUser = {
      id: 1,
      name: 'Alice',
      email: 'alice@example.com',
      role: 'user' as const
    }

    component.user = mockUser

    spyOn(component.edit, 'emit')

    component.handleEdit()

    expect(component.edit.emit).toHaveBeenCalledWith(mockUser)
  })

  it('should display admin badge for admin users', () => {
    component.user = {
      id: 1,
      name: 'Admin',
      email: 'admin@example.com',
      role: 'admin'
    }

    fixture.detectChanges()

    expect(component.isAdmin).toBe(true)
  })
})
```

---

## 6.3 TypeScript with Next.js

### Next.js Project Setup

```bash
# Create Next.js project with TypeScript
npx create-next-app@latest my-next-app --typescript

# Or with App Router
npx create-next-app@latest my-next-app --typescript --app

cd my-next-app
npm run dev
```

**tsconfig.json for Next.js:**

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### Pages with TypeScript (Pages Router)

```typescript
// pages/index.tsx
import { GetServerSideProps, GetStaticProps, NextPage } from 'next'
import Head from 'next/head'
import Link from 'next/link'

interface HomeProps {
  users: User[]
  timestamp: string
}

interface User {
  id: number
  name: string
  email: string
}

const Home: NextPage<HomeProps> = ({ users, timestamp }) => {
  return (
    <>
      <Head>
        <title>Home Page</title>
        <meta name="description" content="Home page" />
      </Head>

      <main>
        <h1>Users</h1>
        <p>Page generated at: {timestamp}</p>
        
        <ul>
          {users.map(user => (
            <li key={user.id}>
              <Link href={`/users/${user.id}`}>
                {user.name}
              </Link>
            </li>
          ))}
        </ul>
      </main>
    </>
  )
}

// Server-side rendering
export const getServerSideProps: GetServerSideProps<HomeProps> = async (context) => {
  const response = await fetch('https://api.example.com/users')
  const users: User[] = await response.json()

  return {
    props: {
      users,
      timestamp: new Date().toISOString()
    }
  }
}

export default Home

// pages/users/[id].tsx - Dynamic route
import { GetStaticPaths, GetStaticProps } from 'next'

interface UserPageProps {
  user: User
}

const UserPage: NextPage<UserPageProps> = ({ user }) => {
  return (
    <>
      <Head>
        <title>{user.name}</title>
      </Head>

      <main>
        <h1>{user.name}</h1>
        <p>{user.email}</p>
      </main>
    </>
  )
}

export const getStaticPaths: GetStaticPaths = async () => {
  const response = await fetch('https://api.example.com/users')
  const users: User[] = await response.json()

  const paths = users.map(user => ({
    params: { id: user.id.toString() }
  }))

  return {
    paths,
    fallback: 'blocking'
  }
}

export const getStaticProps: GetStaticProps<UserPageProps> = async (context) => {
  const id = context.params?.id as string
  const response = await fetch(`https://api.example.com/users/${id}`)
  const user: User = await response.json()

  return {
    props: {
      user
    },
    revalidate: 60 // Revalidate every 60 seconds
  }
}

export default UserPage
```

### App Router with TypeScript (Next.js 13+)

```typescript
// app/page.tsx
import { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Home',
  description: 'Home page'
}

async function getUsers(): Promise<User[]> {
  const response = await fetch('https://api.example.com/users', {
    next: { revalidate: 60 }
  })
  
  if (!response.ok) {
    throw new Error('Failed to fetch users')
  }
  
  return response.json()
}

export default async function Home() {
  const users = await getUsers()

  return (
    <main>
      <h1>Users</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            <a href={`/users/${user.id}`}>{user.name}</a>
          </li>
        ))}
      </ul>
    </main>
  )
}

// app/users/[id]/page.tsx - Dynamic route
interface UserPageProps {
  params: {
    id: string
  }
}

export async function generateMetadata(
  { params }: UserPageProps
): Promise<Metadata> {
  const user = await fetch(`https://api.example.com/users/${params.id}`)
    .then(res => res.json())

  return {
    title: user.name,
    description: `Profile of ${user.name}`
  }
}

export async function generateStaticParams() {
  const users: User[] = await fetch('https://api.example.com/users')
    .then(res => res.json())

  return users.map(user => ({
    id: user.id.toString()
  }))
}

export default async function UserPage({ params }: UserPageProps) {
  const user: User = await fetch(`https://api.example.com/users/${params.id}`)
    .then(res => res.json())

  return (
    <main>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </main>
  )
}
```

### API Routes with TypeScript

```typescript
// pages/api/users/index.ts
import type { NextApiRequest, NextApiResponse } from 'next'

interface User {
  id: number
  name: string
  email: string
}

type UsersResponse = User[]
type ErrorResponse = { error: string }

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<UsersResponse | ErrorResponse>
) {
  if (req.method === 'GET') {
    try {
      const users: User[] = await fetchUsers()
      res.status(200).json(users)
    } catch (error) {
      res.status(500).json({ error: 'Failed to fetch users' })
    }
  } else {
    res.status(405).json({ error: 'Method not allowed' })
  }
}

async function fetchUsers(): Promise<User[]> {
  // Database query
  return []
}

// pages/api/users/[id].ts
import type { NextApiRequest, NextApiResponse } from 'next'

type UserResponse = User | { error: string }

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<UserResponse>
) {
  const { id } = req.query

  if (typeof id !== 'string') {
    return res.status(400).json({ error: 'Invalid ID' })
  }

  switch (req.method) {
    case 'GET':
      return handleGet(id, res)
    case 'PUT':
      return handlePut(id, req, res)
    case 'DELETE':
      return handleDelete(id, res)
    default:
      return res.status(405).json({ error: 'Method not allowed' })
  }
}

async function handleGet(
  id: string,
  res: NextApiResponse<UserResponse>
): Promise<void> {
  try {
    const user = await fetchUserById(id)
    
    if (!user) {
      res.status(404).json({ error: 'User not found' })
      return
    }

    res.status(200).json(user)
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' })
  }
}

async function handlePut(
  id: string,
  req: NextApiRequest,
  res: NextApiResponse<UserResponse>
): Promise<void> {
  try {
    const updates = req.body
    const user = await updateUser(id, updates)
    res.status(200).json(user)
  } catch (error) {
    res.status(500).json({ error: 'Failed to update user' })
  }
}

async function handleDelete(
  id: string,
  res: NextApiResponse<UserResponse>
): Promise<void> {
  try {
    await deleteUser(id)
    res.status(204).end()
  } catch (error) {
    res.status(500).json({ error: 'Failed to delete user' })
  }
}
```

### Server Actions (App Router)

```typescript
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

interface CreateUserInput {
  name: string
  email: string
}

interface ActionResult<T> {
  success: boolean
  data?: T
  error?: string
}

export async function createUser(
  input: CreateUserInput
): Promise<ActionResult<User>> {
  try {
    // Validation
    if (!input.name || !input.email) {
      return {
        success: false,
        error: 'Name and email are required'
      }
    }

    // Database operation
    const user = await db.users.create({
      data: input
    })

    // Revalidate cache
    revalidatePath('/users')

    return {
      success: true,
      data: user
    }
  } catch (error) {
    return {
      success: false,
      error: 'Failed to create user'
    }
  }
}

export async function deleteUser(id: number): Promise<ActionResult<void>> {
  try {
    await db.users.delete({
      where: { id }
    })

    revalidatePath('/users')

    return { success: true }
  } catch (error) {
    return {
      success: false,
      error: 'Failed to delete user'
    }
  }
}

// Usage in component
'use client'

import { useTransition } from 'react'
import { createUser } from './actions'

export function CreateUserForm() {
  const [isPending, startTransition] = useTransition()

  async function handleSubmit(formData: FormData) {
    startTransition(async () => {
      const result = await createUser({
        name: formData.get('name') as string,
        email: formData.get('email') as string
      })

      if (result.success) {
        console.log('User created:', result.data)
      } else {
        console.error('Error:', result.error)
      }
    })
  }

  return (
    <form action={handleSubmit}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>
    </form>
  )
}
```

---

## 6.4 TypeScript with NestJS

### NestJS Project Setup

```bash
# Install NestJS CLI
npm i -g @nestjs/cli

# Create new project
nest new my-nest-app

cd my-nest-app
npm run start:dev
```

**tsconfig.json for NestJS:**

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2021",
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "strictBindCallApply": false,
    "forceConsistentCasingInFileNames": false,
    "noFallthroughCasesInSwitch": false
  }
}
```

### Controllers with TypeScript

```typescript
// users.controller.ts
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Body,
  Param,
  Query,
  HttpCode,
  HttpStatus,
  ParseIntPipe,
  ValidationPipe
} from '@nestjs/common'
import { UsersService } from './users.service'
import { CreateUserDto, UpdateUserDto, UserResponseDto } from './dto'
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger'

@ApiTags('users')
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  @ApiOperation({ summary: 'Get all users' })
  @ApiResponse({ status: 200, description: 'List of users', type: [UserResponseDto] })
  async findAll(
    @Query('page', ParseIntPipe) page: number = 1,
    @Query('limit', ParseIntPipe) limit: number = 10
  ): Promise<UserResponseDto[]> {
    return this.usersService.findAll({ page, limit })
  }

  @Get(':id')
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiResponse({ status: 200, description: 'User found', type: UserResponseDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  async findOne(
    @Param('id', ParseIntPipe) id: number
  ): Promise<UserResponseDto> {
    return this.usersService.findOne(id)
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  @ApiOperation({ summary: 'Create new user' })
  @ApiResponse({ status: 201, description: 'User created', type: UserResponseDto })
  @ApiResponse({ status: 400, description: 'Invalid input' })
  async create(
    @Body(ValidationPipe) createUserDto: CreateUserDto
  ): Promise<UserResponseDto> {
    return this.usersService.create(createUserDto)
  }

  @Put(':id')
  @ApiOperation({ summary: 'Update user' })
  @ApiResponse({ status: 200, description: 'User updated', type: UserResponseDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  async update(
    @Param('id', ParseIntPipe) id: number,
    @Body(ValidationPipe) updateUserDto: UpdateUserDto
  ): Promise<UserResponseDto> {
    return this.usersService.update(id, updateUserDto)
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  @ApiOperation({ summary: 'Delete user' })
  @ApiResponse({ status: 204, description: 'User deleted' })
  @ApiResponse({ status: 404, description: 'User not found' })
  async remove(@Param('id', ParseIntPipe) id: number): Promise<void> {
    await this.usersService.remove(id)
  }
}
```

### DTOs with Validation

```typescript
// dto/create-user.dto.ts
import { IsEmail, IsString, IsInt, MinLength, Min, IsOptional } from 'class-validator'
import { ApiProperty } from '@nestjs/swagger'

export class CreateUserDto {
  @ApiProperty({ example: 'Alice', description: 'User name' })
  @IsString()
  @MinLength(2)
  name: string

  @ApiProperty({ example: 'alice@example.com', description: 'User email' })
  @IsEmail()
  email: string

  @ApiProperty({ example: 30, description: 'User age' })
  @IsInt()
  @Min(18)
  age: number

  @ApiProperty({ example: 'password123', description: 'User password' })
  @IsString()
  @MinLength(8)
  password: string
}

// dto/update-user.dto.ts
import { PartialType } from '@nestjs/swagger'
import { CreateUserDto } from './create-user.dto'

export class UpdateUserDto extends PartialType(CreateUserDto) {}

// dto/user-response.dto.ts
import { Exclude, Expose } from 'class-transformer'
import { ApiProperty } from '@nestjs/swagger'

export class UserResponseDto {
  @ApiProperty()
  @Expose()
  id: number

  @ApiProperty()
  @Expose()
  name: string

  @ApiProperty()
  @Expose()
  email: string

  @ApiProperty()
  @Expose()
  age: number

  @Exclude()
  password: string

  @ApiProperty()
  @Expose()
  createdAt: Date

  @ApiProperty()
  @Expose()
  updatedAt: Date
}
```

### Services with Dependency Injection

```typescript
// users.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common'
import { InjectRepository } from '@nestjs/typeorm'
import { Repository } from 'typeorm'
import { User } from './entities/user.entity'
import { CreateUserDto, UpdateUserDto, UserResponseDto } from './dto'
import * as bcrypt from 'bcrypt'
import { plainToClass } from 'class-transformer'

interface FindAllOptions {
  page: number
  limit: number
}

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>
  ) {}

  async findAll(options: FindAllOptions): Promise<UserResponseDto[]> {
    const { page, limit } = options
    const skip = (page - 1) * limit

    const users = await this.userRepository.find({
      skip,
      take: limit,
      order: { createdAt: 'DESC' }
    })

    return users.map(user => plainToClass(UserResponseDto, user))
  }

  async findOne(id: number): Promise<UserResponseDto> {
    const user = await this.userRepository.findOne({ where: { id } })

    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`)
    }

    return plainToClass(UserResponseDto, user)
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOne({ where: { email } })
  }

  async create(createUserDto: CreateUserDto): Promise<UserResponseDto> {
    const existingUser = await this.findByEmail(createUserDto.email)

    if (existingUser) {
      throw new ConflictException('Email already exists')
    }

    const hashedPassword = await bcrypt.hash(createUserDto.password, 10)

    const user = this.userRepository.create({
      ...createUserDto,
      password: hashedPassword
    })

    const savedUser = await this.userRepository.save(user)

    return plainToClass(UserResponseDto, savedUser)
  }

  async update(id: number, updateUserDto: UpdateUserDto): Promise<UserResponseDto> {
    const user = await this.userRepository.findOne({ where: { id } })

    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`)
    }

    if (updateUserDto.email && updateUserDto.email !== user.email) {
      const existingUser = await this.findByEmail(updateUserDto.email)
      if (existingUser) {
        throw new ConflictException('Email already exists')
      }
    }

    if (updateUserDto.password) {
      updateUserDto.password = await bcrypt.hash(updateUserDto.password, 10)
    }

    Object.assign(user, updateUserDto)
    const updatedUser = await this.userRepository.save(user)

    return plainToClass(UserResponseDto, updatedUser)
  }

  async remove(id: number): Promise<void> {
    const result = await this.userRepository.delete(id)

    if (result.affected === 0) {
      throw new NotFoundException(`User with ID ${id} not found`)
    }
  }
}
```

### Guards and Interceptors

```typescript
// guards/jwt-auth.guard.ts
import { Injectable, ExecutionContext, UnauthorizedException } from '@nestjs/common'
import { AuthGuard } from '@nestjs/passport'
import { Observable } from 'rxjs'

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    return super.canActivate(context)
  }

  handleRequest<TUser = any>(err: any, user: any): TUser {
    if (err || !user) {
      throw err || new UnauthorizedException()
    }
    return user
  }
}

// guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common'
import { Reflector } from '@nestjs/core'

export enum Role {
  User = 'user',
  Admin = 'admin'
}

export const ROLES_KEY = 'roles'

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass()
    ])

    if (!requiredRoles) {
      return true
    }

    const { user } = context.switchToHttp().getRequest()
    return requiredRoles.some((role) => user.roles?.includes(role))
  }
}

// decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common'
import { Role, ROLES_KEY } from '../guards/roles.guard'

export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles)

// Usage in controller
import { UseGuards } from '@nestjs/common'
import { JwtAuthGuard } from './guards/jwt-auth.guard'
import { RolesGuard } from './guards/roles.guard'
import { Roles } from './decorators/roles.decorator'
import { Role } from './guards/roles.guard'

@Controller('admin')
@UseGuards(JwtAuthGuard, RolesGuard)
export class AdminController {
  @Get('users')
  @Roles(Role.Admin)
  async getUsers() {
    return []
  }
}

// interceptors/transform.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common'
import { Observable } from 'rxjs'
import { map } from 'rxjs/operators'

export interface Response<T> {
  success: boolean
  data: T
  timestamp: string
}

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString()
      }))
    )
  }
}
```

### Testing NestJS Services

```typescript
// users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing'
import { getRepositoryToken } from '@nestjs/typeorm'
import { Repository } from 'typeorm'
import { UsersService } from './users.service'
import { User } from './entities/user.entity'
import { NotFoundException, ConflictException } from '@nestjs/common'

type MockRepository<T = any> = Partial<Record<keyof Repository<T>, jest.Mock>>

const createMockRepository = <T = any>(): MockRepository<T> => ({
  find: jest.fn(),
  findOne: jest.fn(),
  create: jest.fn(),
  save: jest.fn(),
  delete: jest.fn()
})

describe('UsersService', () => {
  let service: UsersService
  let repository: MockRepository<User>

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: createMockRepository()
        }
      ]
    }).compile()

    service = module.get<UsersService>(UsersService)
    repository = module.get<MockRepository<User>>(getRepositoryToken(User))
  })

  describe('findOne', () => {
    it('should return a user if found', async () => {
      const mockUser = {
        id: 1,
        name: 'Alice',
        email: 'alice@example.com',
        age: 30
      } as User

      repository.findOne?.mockResolvedValue(mockUser)

      const result = await service.findOne(1)

      expect(result).toEqual(mockUser)
      expect(repository.findOne).toHaveBeenCalledWith({ where: { id: 1 } })
    })

    it('should throw NotFoundException if user not found', async () => {
      repository.findOne?.mockResolvedValue(null)

      await expect(service.findOne(999)).rejects.toThrow(NotFoundException)
    })
  })

  describe('create', () => {
    it('should create a new user', async () => {
      const createUserDto = {
        name: 'Alice',
        email: 'alice@example.com',
        age: 30,
        password: 'password123'
      }

      const mockUser = {
        id: 1,
        ...createUserDto,
        createdAt: new Date(),
        updatedAt: new Date()
      } as User

      repository.findOne?.mockResolvedValue(null)
      repository.create?.mockReturnValue(mockUser)
      repository.save?.mockResolvedValue(mockUser)

      const result = await service.create(createUserDto)

      expect(result).toBeDefined()
      expect(repository.create).toHaveBeenCalled()
      expect(repository.save).toHaveBeenCalled()
    })

    it('should throw ConflictException if email exists', async () => {
      const createUserDto = {
        name: 'Alice',
        email: 'alice@example.com',
        age: 30,
        password: 'password123'
      }

      repository.findOne?.mockResolvedValue({ id: 1 } as User)

      await expect(service.create(createUserDto)).rejects.toThrow(ConflictException)
    })
  })
})
```

---

## 6.5 GraphQL with TypeScript

### GraphQL Setup with Apollo

```bash
# Install dependencies
npm install @nestjs/graphql @nestjs/apollo @apollo/server graphql
npm install class-validator class-transformer
```

**Configuration:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common'
import { GraphQLModule } from '@nestjs/graphql'
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo'
import { join } from 'path'

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
      sortSchema: true,
      playground: true,
      context: ({ req }) => ({ req })
    })
  ]
})
export class AppModule {}
```

### Object Types and Resolvers

```typescript
// models/user.model.ts
import { ObjectType, Field, Int, ID } from '@nestjs/graphql'

@ObjectType()
export class User {
  @Field(() => ID)
  id: string

  @Field()
  name: string

  @Field()
  email: string

  @Field(() => Int)
  age: number

  @Field()
  role: string

  @Field(() => [Post], { nullable: true })
  posts?: Post[]

  @Field()
  createdAt: Date
}

@ObjectType()
export class Post {
  @Field(() => ID)
  id: string

  @Field()
  title: string

  @Field()
  content: string

  @Field(() => User)
  author: User

  @Field()
  createdAt: Date
}

// inputs/create-user.input.ts
import { InputType, Field, Int } from '@nestjs/graphql'
import { IsEmail, IsString, IsInt, MinLength, Min } from 'class-validator'

@InputType()
export class CreateUserInput {
  @Field()
  @IsString()
  @MinLength(2)
  name: string

  @Field()
  @IsEmail()
  email: string

  @Field(() => Int)
  @IsInt()
  @Min(18)
  age: number

  @Field()
  @IsString()
  @MinLength(8)
  password: string
}

@InputType()
export class UpdateUserInput {
  @Field({ nullable: true })
  @IsString()
  @MinLength(2)
  name?: string

  @Field({ nullable: true })
  @IsEmail()
  email?: string

  @Field(() => Int, { nullable: true })
  @IsInt()
  @Min(18)
  age?: number
}

// resolvers/users.resolver.ts
import { Resolver, Query, Mutation, Args, ResolveField, Parent, ID, Int } from '@nestjs/graphql'
import { UseGuards } from '@nestjs/common'
import { User, Post } from '../models'
import { CreateUserInput, UpdateUserInput } from '../inputs'
import { UsersService } from '../services/users.service'
import { PostsService } from '../services/posts.service'
import { GqlAuthGuard } from '../guards/gql-auth.guard'
import { CurrentUser } from '../decorators/current-user.decorator'

@Resolver(() => User)
export class UsersResolver {
  constructor(
    private readonly usersService: UsersService,
    private readonly postsService: PostsService
  ) {}

  @Query(() => [User], { name: 'users' })
  async getUsers(
    @Args('limit', { type: () => Int, nullable: true, defaultValue: 10 }) limit: number,
    @Args('offset', { type: () => Int, nullable: true, defaultValue: 0 }) offset: number
  ): Promise<User[]> {
    return this.usersService.findAll({ limit, offset })
  }

  @Query(() => User, { name: 'user', nullable: true })
  async getUser(
    @Args('id', { type: () => ID }) id: string
  ): Promise<User | null> {
    return this.usersService.findOne(id)
  }

  @Mutation(() => User)
  @UseGuards(GqlAuthGuard)
  async createUser(
    @Args('input') input: CreateUserInput
  ): Promise<User> {
    return this.usersService.create(input)
  }

  @Mutation(() => User)
  @UseGuards(GqlAuthGuard)
  async updateUser(
    @Args('id', { type: () => ID }) id: string,
    @Args('input') input: UpdateUserInput,
    @CurrentUser() currentUser: User
  ): Promise<User> {
    return this.usersService.update(id, input)
  }

  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard)
  async deleteUser(
    @Args('id', { type: () => ID }) id: string
  ): Promise<boolean> {
    await this.usersService.remove(id)
    return true
  }

  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    return this.postsService.findByUserId(user.id)
  }
}
```

### Subscriptions

```typescript
// resolvers/posts.resolver.ts
import { Resolver, Mutation, Subscription, Args } from '@nestjs/graphql'
import { PubSub } from 'graphql-subscriptions'
import { Post } from '../models/post.model'
import { CreatePostInput } from '../inputs/create-post.input'

const pubSub = new PubSub()

@Resolver(() => Post)
export class PostsResolver {
  @Mutation(() => Post)
  async createPost(
    @Args('input') input: CreatePostInput
  ): Promise<Post> {
    const post = await this.postsService.create(input)
    
    // Publish to subscribers
    pubSub.publish('postCreated', { postCreated: post })
    
    return post
  }

  @Subscription(() => Post, {
    name: 'postCreated',
    filter: (payload, variables) => {
      return payload.postCreated.authorId === variables.authorId
    }
  })
  postCreated(@Args('authorId') authorId: string) {
    return pubSub.asyncIterator('postCreated')
  }
}

// Client-side subscription usage
import { gql } from '@apollo/client'

const POST_CREATED_SUBSCRIPTION = gql`
  subscription OnPostCreated($authorId: ID!) {
    postCreated(authorId: $authorId) {
      id
      title
      content
      createdAt
    }
  }
`
```

### DataLoader for N+1 Problem

```typescript
// loaders/users.loader.ts
import * as DataLoader from 'dataloader'
import { Injectable } from '@nestjs/common'
import { UsersService } from '../services/users.service'
import { User } from '../models/user.model'

@Injectable()
export class UsersLoader {
  constructor(private usersService: UsersService) {}

  createLoader(): DataLoader<string, User> {
    return new DataLoader<string, User>(async (userIds: readonly string[]) => {
      const users = await this.usersService.findByIds(Array.from(userIds))
      
      const userMap = new Map(users.map(user => [user.id, user]))
      
      return userIds.map(id => userMap.get(id) || new Error(`User ${id} not found`))
    })
  }
}

// Usage in resolver
@Resolver(() => Post)
export class PostsResolver {
  @ResolveField(() => User)
  async author(
    @Parent() post: Post,
    @Context() { usersLoader }: { usersLoader: DataLoader<string, User> }
  ): Promise<User> {
    return usersLoader.load(post.authorId)
  }
}
```

---

## 6.6 Database ORMs with TypeScript

### TypeORM

```bash
# Install TypeORM
npm install @nestjs/typeorm typeorm mysql2
```

**Configuration:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common'
import { TypeOrmModule } from '@nestjs/typeorm'

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'password',
      database: 'mydb',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true, // Don't use in production
      logging: true
    })
  ]
})
export class AppModule {}
```

**Entity Definition:**

```typescript
// entities/user.entity.ts
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  OneToMany,
  ManyToMany,
  JoinTable
} from 'typeorm'
import { Post } from './post.entity'
import { Role } from './role.entity'

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number

  @Column({ length: 100 })
  name: string

  @Column({ unique: true })
  email: string

  @Column()
  password: string

  @Column()
  age: number

  @Column({ default: true })
  isActive: boolean

  @OneToMany(() => Post, post => post.author)
  posts: Post[]

  @ManyToMany(() => Role)
  @JoinTable()
  roles: Role[]

  @CreateDateColumn()
  createdAt: Date

  @UpdateDateColumn()
  updatedAt: Date
}

// entities/post.entity.ts
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  ManyToOne,
  CreateDateColumn
} from 'typeorm'
import { User } from './user.entity'

@Entity('posts')
export class Post {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  title: string

  @Column('text')
  content: string

  @ManyToOne(() => User, user => user.posts)
  author: User

  @Column()
  authorId: number

  @CreateDateColumn()
  createdAt: Date
}
```

**Repository Pattern:**

```typescript
// repositories/users.repository.ts
import { Injectable } from '@nestjs/common'
import { InjectRepository } from '@nestjs/typeorm'
import { Repository } from 'typeorm'
import { User } from '../entities/user.entity'

@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>
  ) {}

  async findAll(options: { limit: number; offset: number }): Promise<User[]> {
    return this.repository.find({
      take: options.limit,
      skip: options.offset,
      relations: ['posts', 'roles']
    })
  }

  async findById(id: number): Promise<User | null> {
    return this.repository.findOne({
      where: { id },
      relations: ['posts', 'roles']
    })
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repository.findOne({ where: { email } })
  }

  async create(userData: Partial<User>): Promise<User> {
    const user = this.repository.create(userData)
    return this.repository.save(user)
  }

  async update(id: number, userData: Partial<User>): Promise<User> {
    await this.repository.update(id, userData)
    return this.findById(id) as Promise<User>
  }

  async delete(id: number): Promise<void> {
    await this.repository.delete(id)
  }

  async findUsersWithPosts(): Promise<User[]> {
    return this.repository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'post')
      .where('post.createdAt > :date', { date: new Date('2024-01-01') })
      .orderBy('user.name', 'ASC')
      .getMany()
  }
}
```

### Prisma

```bash
# Install Prisma
npm install @prisma/client
npm install -D prisma

# Initialize Prisma
npx prisma init
```

**Schema Definition:**

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  password  String
  age       Int
  isActive  Boolean  @default(true)
  posts     Post[]
  roles     Role[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("users")
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String   @db.Text
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId  Int
  createdAt DateTime @default(now())

  @@map("posts")
  @@index([authorId])
}

model Role {
  id    Int    @id @default(autoincrement())
  name  String @unique
  users User[]

  @@map("roles")
}
```

**Prisma Service:**

```typescript
// prisma.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common'
import { PrismaClient } from '@prisma/client'

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect()
  }

  async onModuleDestroy() {
    await this.$disconnect()
  }
}

// users.service.ts with Prisma
import { Injectable } from '@nestjs/common'
import { PrismaService } from './prisma.service'
import { User, Prisma } from '@prisma/client'

@Injectable()
export class UsersService {
  constructor(private prisma: PrismaService) {}

  async findAll(params: {
    skip?: number
    take?: number
    where?: Prisma.UserWhereInput
    orderBy?: Prisma.UserOrderByWithRelationInput
  }): Promise<User[]> {
    const { skip, take, where, orderBy } = params
    return this.prisma.user.findMany({
      skip,
      take,
      where,
      orderBy,
      include: {
        posts: true,
        roles: true
      }
    })
  }

  async findOne(id: number): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: { id },
      include: {
        posts: {
          orderBy: { createdAt: 'desc' }
        },
        roles: true
      }
    })
  }

  async create(data: Prisma.UserCreateInput): Promise<User> {
    return this.prisma.user.create({
      data,
      include: {
        posts: true,
        roles: true
      }
    })
  }

  async update(id: number, data: Prisma.UserUpdateInput): Promise<User> {
    return this.prisma.user.update({
      where: { id },
      data,
      include: {
        posts: true,
        roles: true
      }
    })
  }

  async delete(id: number): Promise<User> {
    return this.prisma.user.delete({
      where: { id }
    })
  }

  async searchUsers(searchTerm: string): Promise<User[]> {
    return this.prisma.user.findMany({
      where: {
        OR: [
          { name: { contains: searchTerm, mode: 'insensitive' } },
          { email: { contains: searchTerm, mode: 'insensitive' } }
        ]
      }
    })
  }

  async getUsersWithPostCount(): Promise<Array<User & { _count: { posts: number } }>> {
    return this.prisma.user.findMany({
      include: {
        _count: {
          select: { posts: true }
        }
      }
    })
  }

  // Transaction example
  async createUserWithPost(
    userData: Prisma.UserCreateInput,
    postData: Omit<Prisma.PostCreateInput, 'author'>
  ): Promise<User> {
    return this.prisma.$transaction(async (prisma) => {
      const user = await prisma.user.create({
        data: userData
      })

      await prisma.post.create({
        data: {
          ...postData,
          author: {
            connect: { id: user.id }
          }
        }
      })

      return prisma.user.findUnique({
        where: { id: user.id },
        include: { posts: true }
      }) as Promise<User>
    })
  }
}
```

---

## 6.7 React Native with TypeScript

### Project Setup

```bash
# Create new React Native project with TypeScript
npx react-native init MyApp --template react-native-template-typescript

cd MyApp
npm run ios  # or npm run android
```

**tsconfig.json for React Native:**

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "commonjs",
    "lib": ["es2019", "es2020.promise", "es2020.bigint", "es2020.string"],
    "jsx": "react-native",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "baseUrl": "./src",
    "paths": {
      "@components/*": ["components/*"],
      "@screens/*": ["screens/*"],
      "@utils/*": ["utils/*"]
    }
  },
  "exclude": ["node_modules"]
}
```

### Components with TypeScript

```typescript
// components/Button.tsx
import React from 'react'
import {
  TouchableOpacity,
  Text,
  StyleSheet,
  TouchableOpacityProps,
  ViewStyle,
  TextStyle
} from 'react-native'

interface ButtonProps extends TouchableOpacityProps {
  title: string
  variant?: 'primary' | 'secondary'
  onPress: () => void
}

export const Button: React.FC<ButtonProps> = ({
  title,
  variant = 'primary',
  onPress,
  style,
  ...props
}) => {
  return (
    <TouchableOpacity
      style={[styles.button, styles[variant], style]}
      onPress={onPress}
      {...props}
    >
      <Text style={styles.text}>{title}</Text>
    </TouchableOpacity>
  )
}

const styles = StyleSheet.create({
  button: {
    paddingVertical: 12,
    paddingHorizontal: 24,
    borderRadius: 8,
    alignItems: 'center'
  } as ViewStyle,
  primary: {
    backgroundColor: '#007AFF'
  } as ViewStyle,
  secondary: {
    backgroundColor: '#8E8E93'
  } as ViewStyle,
  text: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600'
  } as TextStyle
})

// components/UserCard.tsx
import React from 'react'
import { View, Text, Image, StyleSheet } from 'react-native'

interface User {
  id: string
  name: string
  email: string
  avatar?: string
}

interface UserCardProps {
  user: User
  onPress?: (user: User) => void
}

export const UserCard: React.FC<UserCardProps> = ({ user, onPress }) => {
  return (
    <TouchableOpacity
      style={styles.container}
      onPress={() => onPress?.(user)}
    >
      {user.avatar && (
        <Image source={{ uri: user.avatar }} style={styles.avatar} />
      )}
      <View style={styles.info}>
        <Text style={styles.name}>{user.name}</Text>
        <Text style={styles.email}>{user.email}</Text>
      </View>
    </TouchableOpacity>
  )
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    padding: 16,
    backgroundColor: '#FFFFFF',
    borderRadius: 8,
    marginBottom: 8
  },
  avatar: {
    width: 48,
    height: 48,
    borderRadius: 24
  },
  info: {
    marginLeft: 12,
    justifyContent: 'center'
  },
  name: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 4
  },
  email: {
    fontSize: 14,
    color: '#8E8E93'
  }
})
```

### Navigation with TypeScript

```bash
npm install @react-navigation/native @react-navigation/stack
npm install react-native-screens react-native-safe-area-context
```

```typescript
// navigation/types.ts
import { StackNavigationProp } from '@react-navigation/stack'
import { RouteProp } from '@react-navigation/native'

export type RootStackParamList = {
  Home: undefined
  UserProfile: { userId: string }
  Settings: { initialTab?: string }
  CreatePost: undefined
}

export type HomeScreenNavigationProp = StackNavigationProp<
  RootStackParamList,
  'Home'
>

export type UserProfileScreenRouteProp = RouteProp<
  RootStackParamList,
  'UserProfile'
>

export type UserProfileScreenNavigationProp = StackNavigationProp<
  RootStackParamList,
  'UserProfile'
>

// navigation/RootNavigator.tsx
import React from 'react'
import { NavigationContainer } from '@react-navigation/native'
import { createStackNavigator } from '@react-navigation/stack'
import { RootStackParamList } from './types'
import HomeScreen from '../screens/HomeScreen'
import UserProfileScreen from '../screens/UserProfileScreen'
import SettingsScreen from '../screens/SettingsScreen'
import CreatePostScreen from '../screens/CreatePostScreen'

const Stack = createStackNavigator<RootStackParamList>()

export const RootNavigator: React.FC = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="UserProfile" component={UserProfileScreen} />
        <Stack.Screen name="Settings" component={SettingsScreen} />
        <Stack.Screen name="CreatePost" component={CreatePostScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  )
}

// screens/HomeScreen.tsx
import React from 'react'
import { View, Text, Button } from 'react-native'
import { HomeScreenNavigationProp } from '../navigation/types'

interface Props {
  navigation: HomeScreenNavigationProp
}

const HomeScreen: React.FC<Props> = ({ navigation }) => {
  return (
    <View>
      <Text>Home Screen</Text>
      <Button
        title="Go to User Profile"
        onPress={() => navigation.navigate('UserProfile', { userId: '123' })}
      />
    </View>
  )
}

export default HomeScreen

// screens/UserProfileScreen.tsx
import React from 'react'
import { View, Text } from 'react-native'
import {
  UserProfileScreenRouteProp,
  UserProfileScreenNavigationProp
} from '../navigation/types'

interface Props {
  route: UserProfileScreenRouteProp
  navigation: UserProfileScreenNavigationProp
}

const UserProfileScreen: React.FC<Props> = ({ route, navigation }) => {
  const { userId } = route.params

  return (
    <View>
      <Text>User Profile: {userId}</Text>
    </View>
  )
}

export default UserProfileScreen
```

### Hooks and State Management

```typescript
// hooks/useApi.ts
import { useState, useEffect, useCallback } from 'react'

interface UseApiResult<T> {
  data: T | null
  loading: boolean
  error: Error | null
  refetch: () => Promise<void>
}

export function useApi<T>(url: string): UseApiResult<T> {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  const fetchData = useCallback(async () => {
    setLoading(true)
    setError(null)

    try {
      const response = await fetch(url)
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      
      const json = await response.json()
      setData(json)
    } catch (e) {
      setError(e as Error)
    } finally {
      setLoading(false)
    }
  }, [url])

  useEffect(() => {
    fetchData()
  }, [fetchData])

  return { data, loading, error, refetch: fetchData }
}

// Usage
const UserListScreen: React.FC = () => {
  const { data: users, loading, error, refetch } = useApi<User[]>('/api/users')

  if (loading) return <ActivityIndicator />
  if (error) return <Text>Error: {error.message}</Text>

  return (
    <FlatList
      data={users}
      renderItem={({ item }) => <UserCard user={item} />}
      refreshing={loading}
      onRefresh={refetch}
    />
  )
}
```

---

## 6.8 Publishing TypeScript Libraries

### Package Setup

```json
// package.json
{
  "name": "my-typescript-library",
  "version": "1.0.0",
  "description": "A TypeScript library",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "require": "./dist/index.js",
      "import": "./dist/index.mjs",
      "types": "./dist/index.d.ts"
    }
  },
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "test": "jest",
    "lint": "eslint src/**/*.ts",
    "prepublishOnly": "npm run build && npm test"
  },
  "keywords": ["typescript", "library"],
  "author": "Your Name",
  "license": "MIT",
  "devDependencies": {
    "typescript": "^5.0.0",
    "tsup": "^8.0.0",
    "@types/node": "^20.0.0"
  }
}
```

**tsconfig.json for Library:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020"],
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "resolveJsonModule": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### Library Code

```typescript
// src/index.ts
export * from './utils'
export * from './types'
export { createClient } from './client'

// src/types.ts
export interface User {
  id: string
  name: string
  email: string
}

export interface ApiResponse<T> {
  data: T
  status: number
  message?: string
}

export type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE'

// src/client.ts
import { ApiResponse, HttpMethod } from './types'

export interface ClientConfig {
  baseUrl: string
  headers?: Record<string, string>
  timeout?: number
}

export class ApiClient {
  private config: ClientConfig

  constructor(config: ClientConfig) {
    this.config = config
  }

  async request<T>(
    method: HttpMethod,
    endpoint: string,
    data?: any
  ): Promise<ApiResponse<T>> {
    const url = `${this.config.baseUrl}${endpoint}`
    
    const response = await fetch(url, {
      method,
      headers: {
        'Content-Type': 'application/json',
        ...this.config.headers
      },
      body: data ? JSON.stringify(data) : undefined
    })

    const json = await response.json()

    return {
      data: json,
      status: response.status
    }
  }

  get<T>(endpoint: string): Promise<ApiResponse<T>> {
    return this.request<T>('GET', endpoint)
  }

  post<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
    return this.request<T>('POST', endpoint, data)
  }

  put<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
    return this.request<T>('PUT', endpoint, data)
  }

  delete<T>(endpoint: string): Promise<ApiResponse<T>> {
    return this.request<T>('DELETE', endpoint)
  }
}

export function createClient(config: ClientConfig): ApiClient {
  return new ApiClient(config)
}

// src/utils.ts
export function formatDate(date: Date): string {
  return date.toISOString()
}

export function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timeoutId: NodeJS.Timeout

  return function (...args: Parameters<T>) {
    clearTimeout(timeoutId)
    timeoutId = setTimeout(() => fn(...args), delay)
  }
}
```

### Documentation

```typescript
// src/client.ts with JSDoc
/**
 * Configuration options for the API client
 */
export interface ClientConfig {
  /** Base URL for all API requests */
  baseUrl: string
  /** Optional headers to include in all requests */
  headers?: Record<string, string>
  /** Request timeout in milliseconds */
  timeout?: number
}

/**
 * HTTP client for making API requests
 * 
 * @example
 * ```typescript
 * const client = createClient({
 *   baseUrl: 'https://api.example.com',
 *   headers: { 'Authorization': 'Bearer token' }
 * })
 * 
 * const response = await client.get<User>('/users/123')
 * console.log(response.data)
 * ```
 */
export class ApiClient {
  /**
   * Creates a new API client instance
   * @param config - Client configuration options
   */
  constructor(config: ClientConfig) {
    this.config = config
  }

  /**
   * Makes a GET request to the specified endpoint
   * @param endpoint - API endpoint path
   * @returns Promise resolving to the API response
   * 
   * @example
   * ```typescript
   * const users = await client.get<User[]>('/users')
   * ```
   */
  async get<T>(endpoint: string): Promise<ApiResponse<T>> {
    return this.request<T>('GET', endpoint)
  }
}
```

---

## Conclusion of Part 6

This concludes Part 6 of the Complete TypeScript Mastery series!

**What We've Covered:**

1. **Vue.js with TypeScript** - Composition API, Pinia, Router
2. **Angular with TypeScript** - Components, Services, RxJS
3. **Next.js with TypeScript** - App Router, API Routes, Server Actions
4. **NestJS** - Backend framework with decorators and DI
5. **GraphQL** - Type-safe APIs with resolvers and subscriptions
6. **Database ORMs** - TypeORM and Prisma
7. **React Native** - Mobile development with TypeScript
8. **Publishing Libraries** - Creating and distributing packages

---

## Complete Series Summary

### All Six Parts:

| Part | Focus | Words |
|------|-------|-------|
| Part 1 | TypeScript Fundamentals | 25,231 |
| Part 2 | Advanced Type System | 19,179 |
| Part 3 | Compiler & Tooling | 6,966 |
| Part 4 | React & Node.js | 10,949 |
| Part 5 | Design Patterns | 6,968 |
| Part 6 | Ecosystem | 10,000+ |
| **Total** | **Complete Mastery** | **~79,000** |

**You now have a complete TypeScript ecosystem guide!** 🎉