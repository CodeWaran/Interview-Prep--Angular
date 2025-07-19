<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# Angular Advanced Interview Q\&A - 100 Additional Questions

This comprehensive collection of 100 advanced Angular interview questions builds upon fundamental concepts with deep technical challenges, modern Angular features, and real-world scenarios for senior frontend developers.

## Modern Angular Features (Angular 16+)

### Q26: Explain Angular Signals and their advantages over traditional reactive patterns

**Answer:**
Angular Signals represent a new reactive programming model introduced in Angular 16+ that provides a more declarative and efficient way to handle state changes.

```typescript
import { signal, computed, effect } from '@angular/core';

@Component({
  template: `
    <div>
      <p>Count: {{count()}}</p>
      <p>Double: {{doubleCount()}}</p>
      <p>Status: {{status()}}</p>
      <button (click)="increment()">+</button>
      <button (click)="decrement()">-</button>
    </div>
  `
})
export class SignalsComponent {
  // Writable signal
  count = signal(0);
  
  // Computed signal - automatically updates when count changes
  doubleCount = computed(() => this.count() * 2);
  
  // Derived state
  status = computed(() => {
    const value = this.count();
    return value > 10 ? 'High' : value < 0 ? 'Negative' : 'Normal';
  });
  
  constructor() {
    // Effect - runs side effects when signals change
    effect(() => {
      console.log(`Count changed to: ${this.count()}`);
      
      // Cleanup function
      return () => console.log('Effect cleanup');
    });
  }
  
  increment() {
    this.count.update(value => value + 1);
  }
  
  decrement() {
    this.count.set(this.count() - 1);
  }
}
```

**Key Advantages:**

- **Fine-grained reactivity**: Only components using changed signals re-render
- **Automatic dependency tracking**: Computed signals track dependencies automatically
- **Synchronous updates**: No async pipe needed, direct value access
- **Better performance**: More efficient change detection
- **Type safety**: Full TypeScript support with proper typing


### Q27: Implement a complex Signal-based state management system

```typescript
// Signal-based store pattern
interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
  loading: boolean;
}

interface Todo {
  id: number;
  text: string;
  completed: boolean;
  createdAt: Date;
}

@Injectable({ providedIn: 'root' })
export class TodoSignalStore {
  // Private state signals
  private _todos = signal<Todo[]>([]);
  private _filter = signal<'all' | 'active' | 'completed'>('all');
  private _loading = signal(false);
  
  // Public readonly signals
  readonly todos = this._todos.asReadonly();
  readonly filter = this._filter.asReadonly();
  readonly loading = this._loading.asReadonly();
  
  // Computed selectors
  readonly filteredTodos = computed(() => {
    const todos = this._todos();
    const filter = this._filter();
    
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed);
      case 'completed':
        return todos.filter(todo => todo.completed);
      default:
        return todos;
    }
  });
  
  readonly todoStats = computed(() => {
    const todos = this._todos();
    return {
      total: todos.length,
      active: todos.filter(t => !t.completed).length,
      completed: todos.filter(t => t.completed).length
    };
  });
  
  constructor(private http: HttpClient) {
    this.loadTodos();
  }
  
  // Actions
  async loadTodos() {
    this._loading.set(true);
    try {
      const todos = await this.http.get<Todo[]>('/api/todos').toPromise();
      this._todos.set(todos || []);
    } catch (error) {
      console.error('Failed to load todos:', error);
    } finally {
      this._loading.set(false);
    }
  }
  
  addTodo(text: string) {
    const newTodo: Todo = {
      id: Date.now(),
      text: text.trim(),
      completed: false,
      createdAt: new Date()
    };
    
    this._todos.update(todos => [...todos, newTodo]);
  }
  
  toggleTodo(id: number) {
    this._todos.update(todos => 
      todos.map(todo => 
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }
  
  deleteTodo(id: number) {
    this._todos.update(todos => todos.filter(todo => todo.id !== id));
  }
  
  setFilter(filter: 'all' | 'active' | 'completed') {
    this._filter.set(filter);
  }
  
  clearCompleted() {
    this._todos.update(todos => todos.filter(todo => !todo.completed));
  }
}

// Component using the signal store
@Component({
  template: `
    <div class="todo-app">
      <div class="stats">
        <span>Total: {{store.todoStats().total}}</span>
        <span>Active: {{store.todoStats().active}}</span>
        <span>Completed: {{store.todoStats().completed}}</span>
      </div>
      
      <div class="filters">
        <button 
          *ngFor="let f of filters"
          [class.active]="store.filter() === f"
          (click)="store.setFilter(f)">
          {{f | titlecase}}
        </button>
      </div>
      
      <div class="todo-list">
        <div 
          *ngFor="let todo of store.filteredTodos(); trackBy: trackByTodoId"
          class="todo-item"
          [class.completed]="todo.completed">
          <input 
            type="checkbox" 
            [checked]="todo.completed"
            (change)="store.toggleTodo(todo.id)">
          <span>{{todo.text}}</span>
          <button (click)="store.deleteTodo(todo.id)">Delete</button>
        </div>
      </div>
      
      <div *ngIf="store.loading()" class="loading">Loading...</div>
    </div>
  `
})
export class TodoAppComponent {
  filters: ('all' | 'active' | 'completed')[] = ['all', 'active', 'completed'];
  
  constructor(public store: TodoSignalStore) {}
  
  trackByTodoId(index: number, todo: Todo): number {
    return todo.id;
  }
}
```


### Q28: What are the differences between Signals and RxJS Observables?

**Answer:**


| Aspect | Signals | RxJS Observables |
| :-- | :-- | :-- |
| **Execution** | Synchronous | Asynchronous |
| **Value Access** | Direct: `signal()` | Stream: `observable.subscribe()` |
| **Change Detection** | Automatic optimization | Manual async pipe |
| **Memory Management** | Automatic cleanup | Manual unsubscription needed |
| **Composition** | `computed()` functions | Operators (`map`, `filter`, etc.) |
| **Side Effects** | `effect()` function | `tap()` operator |
| **Error Handling** | Try/catch in computed | `catchError()` operator |
| **Hot/Cold** | Always "hot" | Can be hot or cold |

**Practical Example:**

```typescript
@Component({
  template: `
    <div>
      <h3>Signals Approach</h3>
      <p>User: {{userSignal()?.name || 'Loading...'}}</p>
      <p>Posts Count: {{userPostsCount()}}</p>
      
      <h3>RxJS Approach</h3>
      <p>User: {{(user$ | async)?.name || 'Loading...'}}</p>
      <p>Posts Count: {{userPostsCount$ | async}}</p>
    </div>
  `
})
export class SignalsVsRxJSComponent implements OnInit, OnDestroy {
  // Signals approach
  userSignal = signal<User | null>(null);
  userPostsCount = computed(() => {
    const user = this.userSignal();
    return user?.posts?.length || 0;
  });
  
  // RxJS approach  
  user$ = new BehaviorSubject<User | null>(null);
  userPostsCount$ = this.user$.pipe(
    map(user => user?.posts?.length || 0)
  );
  
  private destroy$ = new Subject<void>();
  
  constructor(private userService: UserService) {
    // Signals - automatic cleanup
    effect(() => {
      console.log('User changed:', this.userSignal()?.name);
    });
  }
  
  ngOnInit() {
    // Load user data for both approaches
    this.userService.getCurrentUser().subscribe(user => {
      // Update both
      this.userSignal.set(user);
      this.user$.next(user);
    });
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
    // Note: Signals automatically clean up, no manual work needed
  }
}
```


### Q29: Implement Standalone Components with advanced dependency injection

```typescript
// Standalone service
@Injectable()
export class StandaloneDataService {
  private data = signal<any[]>([]);
  
  getData = computed(() => this.data());
  
  loadData() {
    // Simulate API call
    setTimeout(() => {
      this.data.set([
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' }
      ]);
    }, 1000);
  }
}

// Standalone child component
@Component({
  selector: 'app-child',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="child">
      <h4>Child Component</h4>
      <p>Received: {{data}}</p>
      <button (click)="emitEvent()">Emit Event</button>
    </div>
  `,
  providers: [
    // Child-specific providers
    { 
      provide: 'CHILD_CONFIG', 
      useValue: { theme: 'dark', maxItems: 10 } 
    }
  ]
})
export class StandaloneChildComponent {
  @Input() data: string = '';
  @Output() childEvent = new EventEmitter<string>();
  
  constructor(
    @Inject('CHILD_CONFIG') private config: any,
    private dataService: StandaloneDataService
  ) {
    console.log('Child config:', config);
  }
  
  emitEvent() {
    this.childEvent.emit(`Event from child at ${new Date().toISOString()}`);
  }
}

// Main standalone component
@Component({
  selector: 'app-standalone',
  standalone: true,
  imports: [
    CommonModule,
    FormsModule,
    StandaloneChildComponent // Import child component directly
  ],
  template: `
    <div class="standalone-app">
      <h2>Standalone Component App</h2>
      
      <div class="data-section">
        <button (click)="loadData()">Load Data</button>
        <div *ngIf="dataService.getData().length > 0">
          <h3>Data:</h3>
          <ul>
            <li *ngFor="let item of dataService.getData()">
              {{item.name}}
            </li>
          </ul>
        </div>
      </div>
      
      <div class="child-section">
        <app-child 
          [data]="childData"
          (childEvent)="onChildEvent($event)">
        </app-child>
      </div>
      
      <div class="events">
        <h3>Events:</h3>
        <ul>
          <li *ngFor="let event of events">{{event}}</li>
        </ul>
      </div>
    </div>
  `,
  providers: [
    StandaloneDataService,
    {
      provide: 'APP_CONFIG',
      useFactory: () => ({
        apiUrl: 'https://api.example.com',
        version: '1.0.0'
      })
    }
  ]
})
export class StandaloneMainComponent {
  childData = 'Data from parent';
  events: string[] = [];
  
  constructor(
    public dataService: StandaloneDataService,
    @Inject('APP_CONFIG') private appConfig: any
  ) {
    console.log('App config:', appConfig);
  }
  
  loadData() {
    this.dataService.loadData();
  }
  
  onChildEvent(event: string) {
    this.events.push(event);
  }
}

// Bootstrap standalone component
main.ts:
import { bootstrapApplication } from '@angular/platform-browser';
import { importProvidersFrom } from '@angular/core';
import { HttpClientModule } from '@angular/common/http';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';

bootstrapApplication(StandaloneMainComponent, {
  providers: [
    importProvidersFrom(
      HttpClientModule,
      BrowserAnimationsModule
    ),
    // Global providers
    {
      provide: 'GLOBAL_CONFIG',
      useValue: { environment: 'production' }
    }
  ]
});
```


### Q30: Advanced Control Flow with @if, @for, @switch

```typescript
@Component({
  template: `
    <!-- New @if syntax (Angular 17+) -->
    @if (user(); else loadingBlock) {
      <div class="user-profile">
        <h2>{{user()!.name}}</h2>
        
        <!-- Nested @if with complex conditions -->
        @if (user()!.role === 'admin' && user()!.permissions.length > 0) {
          <div class="admin-panel">
            <h3>Admin Controls</h3>
            
            <!-- @for with advanced features -->
            @for (permission of user()!.permissions; track permission.id; let i = $index, first = $first, last = $last) {
              <div class="permission-item" 
                   [class.first]="first"
                   [class.last]="last">
                <span>{{i + 1}}. {{permission.name}}</span>
                
                <!-- @switch for permission types -->
                @switch (permission.type) {
                  @case ('read') {
                    <span class="badge read">👁️ Read</span>
                  }
                  @case ('write') {
                    <span class="badge write">✏️ Write</span>
                  }
                  @case ('delete') {
                    <span class="badge delete">🗑️ Delete</span>
                  }
                  @default {
                    <span class="badge unknown">❓ Unknown</span>
                  }
                }
              </div>
            } @empty {
              <p>No permissions assigned</p>
            }
          </div>
        } @else if (user()!.role === 'user') {
          <div class="user-panel">
            <p>Standard user access</p>
          </div>
        } @else {
          <div class="guest-panel">
            <p>Guest access - limited functionality</p>
          </div>
        }
        
        <!-- @for with complex objects and filtering -->
        @if (user()!.projects && user()!.projects.length > 0) {
          <div class="projects">
            <h3>Active Projects</h3>
            @for (project of getActiveProjects(); track project.id; let count = $count) {
              <div class="project-card">
                <h4>{{project.name}}</h4>
                <p>Status: {{project.status}}</p>
                <small>{{count}} of {{user()!.projects.length}} total projects</small>
                
                <!-- Nested @for for project tasks -->
                @if (project.tasks && project.tasks.length > 0) {
                  <ul class="task-list">
                    @for (task of project.tasks; track task.id) {
                      <li [class]="task.completed ? 'completed' : 'pending'">
                        {{task.title}}
                        @if (task.dueDate && isOverdue(task.dueDate)) {
                          <span class="overdue">⚠️ Overdue</span>
                        }
                      </li>
                    } @empty {
                      <li>No tasks assigned</li>
                    }
                  </ul>
                }
              </div>
            } @empty {
              <p>No active projects</p>
            }
          </div>
        }
      </div>
    } @else {
      <ng-template #loadingBlock>
        <div class="loading">
          @switch (loadingState()) {
            @case ('loading') {
              <p>Loading user data...</p>
            }
            @case ('error') {
              <p>Failed to load user data</p>
              <button (click)="retryLoad()">Retry</button>
            }
            @case ('empty') {
              <p>No user data available</p>
            }
          }
        </div>
      </ng-template>
    }
  `
})
export class AdvancedControlFlowComponent {
  user = signal<User | null>(null);
  loadingState = signal<'loading' | 'error' | 'empty'>('loading');
  
  constructor() {
    this.loadUser();
  }
  
  loadUser() {
    this.loadingState.set('loading');
    
    // Simulate API call
    setTimeout(() => {
      const userData: User = {
        id: 1,
        name: 'John Doe',
        role: 'admin',
        permissions: [
          { id: 1, name: 'User Management', type: 'write' },
          { id: 2, name: 'System Settings', type: 'read' },
          { id: 3, name: 'Data Export', type: 'delete' }
        ],
        projects: [
          {
            id: 1,
            name: 'Project Alpha',
            status: 'active',
            tasks: [
              { id: 1, title: 'Setup database', completed: true, dueDate: null },
              { id: 2, title: 'API development', completed: false, dueDate: new Date('2024-01-15') }
            ]
          }
        ]
      };
      
      this.user.set(userData);
    }, 2000);
  }
  
  getActiveProjects() {
    return this.user()?.projects?.filter(p => p.status === 'active') || [];
  }
  
  isOverdue(dueDate: Date): boolean {
    return new Date() > new Date(dueDate);
  }
  
  retryLoad() {
    this.loadUser();
  }
}

interface User {
  id: number;
  name: string;
  role: 'admin' | 'user' | 'guest';
  permissions: Permission[];
  projects: Project[];
}

interface Permission {
  id: number;
  name: string;
  type: 'read' | 'write' | 'delete';
}

interface Project {
  id: number;
  name: string;
  status: string;
  tasks: Task[];
}

interface Task {
  id: number;
  title: string;
  completed: boolean;
  dueDate: Date | null;
}
```


## Advanced RxJS Patterns

### Q31: Implement complex operator chaining with error recovery

```typescript
interface ApiResponse<T> {
  data: T;
  status: 'success' | 'error';
  message?: string;
  retryAfter?: number;
}

@Injectable()
export class RobustApiService {
  private baseUrl = 'https://api.example.com';
  private maxRetries = 3;
  private retryDelay = 1000;
  
  constructor(private http: HttpClient) {}
  
  // Complex API call with retry logic, caching, and fallback
  getData<T>(endpoint: string, options?: any): Observable<T> {
    return this.http.get<ApiResponse<T>>(`${this.baseUrl}/${endpoint}`, options).pipe(
      // Log the request
      tap(() => console.log(`API Request: ${endpoint}`)),
      
      // Transform response
      map(response => {
        if (response.status === 'error') {
          throw new Error(response.message || 'API Error');
        }
        return response.data;
      }),
      
      // Retry with exponential backoff
      retryWhen(errors => 
        errors.pipe(
          scan((retryCount, error) => {
            console.log(`Retry attempt ${retryCount + 1} for ${endpoint}:`, error.message);
            
            if (retryCount >= this.maxRetries) {
              throw error;
            }
            
            return retryCount + 1;
          }, 0),
          delayWhen(retryCount => timer(this.retryDelay * Math.pow(2, retryCount))),
          take(this.maxRetries)
        )
      ),
      
      // Cache successful responses
      shareReplay({ 
        bufferSize: 1, 
        refCount: true,
        windowTime: 300000 // 5 minutes
      }),
      
      // Fallback for critical failures
      catchError(error => {
        console.error(`Final error for ${endpoint}:`, error);
        
        // Try to get cached data
        const cachedData = this.getCachedData<T>(endpoint);
        if (cachedData) {
          console.log(`Using cached data for ${endpoint}`);
          return of(cachedData);
        }
        
        // Return empty result or throw based on endpoint criticality
        if (this.isCriticalEndpoint(endpoint)) {
          return throwError(() => new Error(`Critical endpoint ${endpoint} failed`));
        }
        
        return of(this.getDefaultData<T>(endpoint));
      })
    );
  }
  
  // Complex stream composition with multiple sources
  getDashboardData(): Observable<DashboardData> {
    const user$ = this.getData<User>('user');
    const stats$ = this.getData<Statistics>('stats');
    const notifications$ = this.getData<Notification[]>('notifications');
    const recentActivity$ = this.getData<Activity[]>('recent-activity');
    
    return combineLatest([user$, stats$, notifications$, recentActivity$]).pipe(
      map(([user, stats, notifications, activities]) => ({
        user,
        stats,
        notifications: notifications.slice(0, 5), // Limit to 5 most recent
        recentActivity: activities.slice(0, 10),   // Limit to 10 most recent
        lastUpdated: new Date()
      })),
      
      // Add loading state management
      startWith({
        user: null,
        stats: null,
        notifications: [],
        recentActivity: [],
        lastUpdated: null,
        loading: true
      } as DashboardData),
      
      // Remove loading state when data arrives
      map(data => ({ ...data, loading: false })),
      
      // Handle partial failures gracefully
      catchError(error => {
        console.error('Dashboard data error:', error);
        return of({
          user: null,
          stats: this.getDefaultStats(),
          notifications: [],
          recentActivity: [],
          lastUpdated: new Date(),
          loading: false,
          error: error.message
        } as DashboardData);
      })
    );
  }
  
  // Advanced search with debouncing, distinct queries, and result caching
  search<T>(searchTerm$: Observable<string>): Observable<SearchResult<T>> {
    return searchTerm$.pipe(
      // Debounce user input
      debounceTime(300),
      
      // Only search for non-empty terms
      filter(term => term.trim().length > 0),
      
      // Avoid duplicate searches
      distinctUntilChanged(),
      
      // Switch to new search, canceling previous
      switchMap(term => 
        this.performSearch<T>(term).pipe(
          // Add search metadata
          map(results => ({
            term,
            results,
            timestamp: new Date(),
            resultCount: results.length
          })),
          
          // Handle search errors without breaking the stream
          catchError(error => of({
            term,
            results: [] as T[],
            timestamp: new Date(),
            resultCount: 0,
            error: error.message
          }))
        )
      ),
      
      // Start with empty state
      startWith({
        term: '',
        results: [] as T[],
        timestamp: new Date(),
        resultCount: 0
      } as SearchResult<T>)
    );
  }
  
  private performSearch<T>(term: string): Observable<T[]> {
    return this.http.get<ApiResponse<T[]>>(`${this.baseUrl}/search`, {
      params: { q: term }
    }).pipe(
      map(response => response.data),
      timeout(5000), // 5 second timeout
      retry(2)
    );
  }
  
  private getCachedData<T>(endpoint: string): T | null {
    // Implementation for cache retrieval
    return null;
  }
  
  private isCriticalEndpoint(endpoint: string): boolean {
    const criticalEndpoints = ['user', 'auth', 'security'];
    return criticalEndpoints.some(critical => endpoint.includes(critical));
  }
  
  private getDefaultData<T>(endpoint: string): T {
    // Return sensible defaults based on endpoint
    return {} as T;
  }
  
  private getDefaultStats(): Statistics {
    return {
      totalUsers: 0,
      activeUsers: 0,
      revenue: 0,
      growthRate: 0
    };
  }
}

interface DashboardData {
  user: User | null;
  stats: Statistics | null;
  notifications: Notification[];
  recentActivity: Activity[];
  lastUpdated: Date | null;
  loading?: boolean;
  error?: string;
}

interface SearchResult<T> {
  term: string;
  results: T[];
  timestamp: Date;
  resultCount: number;
  error?: string;
}
```


### Q32: What's the difference between switchMap, mergeMap, concatMap, and exhaustMap?

```typescript
@Component({
  template: `
    <div class="operator-demo">
      <div class="controls">
        <button (click)="triggerSwitchMap()">SwitchMap Request</button>
        <button (click)="triggerMergeMap()">MergeMap Request</button>
        <button (click)="triggerConcatMap()">ConcatMap Request</button>
        <button (click)="triggerExhaustMap()">ExhaustMap Request</button>
        <button (click)="clearResults()">Clear</button>
      </div>
      
      <div class="results">
        <div class="result-section">
          <h3>SwitchMap Results:</h3>
          <div *ngFor="let result of switchMapResults">{{result}}</div>
        </div>
        
        <div class="result-section">
          <h3>MergeMap Results:</h3>
          <div *ngFor="let result of mergeMapResults">{{result}}</div>
        </div>
        
        <div class="result-section">
          <h3>ConcatMap Results:</h3>
          <div *ngFor="let result of concatMapResults">{{result}}</div>
        </div>
        
        <div class="result-section">
          <h3>ExhaustMap Results:</h3>
          <div *ngFor="let result of exhaustMapResults">{{result}}</div>
        </div>
      </div>
    </div>
  `
})
export class RxJSOperatorDemoComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  switchMapTrigger$ = new Subject<number>();
  mergeMapTrigger$ = new Subject<number>();
  concatMapTrigger$ = new Subject<number>();
  exhaustMapTrigger$ = new Subject<number>();
  
  switchMapResults: string[] = [];
  mergeMapResults: string[] = [];
  concatMapResults: string[] = [];
  exhaustMapResults: string[] = [];
  
  private requestCounter = 0;
  
  ngOnInit() {
    this.setupSwitchMapDemo();
    this.setupMergeMapDemo();
    this.setupConcatMapDemo();
    this.setupExhaustMapDemo();
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
  
  private setupSwitchMapDemo() {
    // switchMap: Cancels previous inner observables when new one arrives
    this.switchMapTrigger$.pipe(
      switchMap(id => this.makeRequest(`switchMap-${id}`)),
      takeUntil(this.destroy$)
    ).subscribe(result => {
      this.switchMapResults.push(result);
    });
  }
  
  private setupMergeMapDemo() {
    // mergeMap: Allows concurrent inner observables
    this.mergeMapTrigger$.pipe(
      mergeMap(id => this.makeRequest(`mergeMap-${id}`)),
      takeUntil(this.destroy$)
    ).subscribe(result => {
      this.mergeMapResults.push(result);
    });
  }
  
  private setupConcatMapDemo() {
    // concatMap: Waits for previous inner observable to complete
    this.concatMapTrigger$.pipe(
      concatMap(id => this.makeRequest(`concatMap-${id}`)),
      takeUntil(this.destroy$)
    ).subscribe(result => {
      this.concatMapResults.push(result);
    });
  }
  
  private setupExhaustMapDemo() {
    // exhaustMap: Ignores new emissions while inner observable is active
    this.exhaustMapTrigger$.pipe(
      exhaustMap(id => this.makeRequest(`exhaustMap-${id}`)),
      takeUntil(this.destroy$)
    ).subscribe(result => {
      this.exhaustMapResults.push(result);
    });
  }
  
  private makeRequest(id: string): Observable<string> {
    // Simulate API call with random delay
    const delay = Math.random() * 2000 + 1000; // 1-3 seconds
    
    return timer(delay).pipe(
      map(() => `${id} completed after ${delay.toFixed(0)}ms at ${new Date().toLocaleTimeString()}`)
    );
  }
  
  triggerSwitchMap() {
    this.switchMapTrigger$.next(++this.requestCounter);
  }
  
  triggerMergeMap() {
    this.mergeMapTrigger$.next(++this.requestCounter);
  }
  
  triggerConcatMap() {
    this.concatMapTrigger$.next(++this.requestCounter);
  }
  
  triggerExhaustMap() {
    this.exhaustMapTrigger$.next(++this.requestCounter);
  }
  
  clearResults() {
    this.switchMapResults = [];
    this.mergeMapResults = [];
    this.concatMapResults = [];
    this.exhaustMapResults = [];
  }
}
```

**Key Differences:**


| Operator | Behavior | Use Case |
| :-- | :-- | :-- |
| **switchMap** | Cancels previous inner observables | Search autocomplete, latest data only |
| **mergeMap** | Runs all inner observables concurrently | Independent parallel requests |
| **concatMap** | Queues inner observables sequentially | Order-dependent operations |
| **exhaustMap** | Ignores new emissions until current completes | Prevent double-clicks, login requests |

### Q33: Advanced Subject patterns and multicasting

```typescript
// Custom Subject with replay and filtering capabilities
export class SmartSubject<T> extends Subject<T> {
  private replayBuffer: T[] = [];
  private bufferSize: number;
  
  constructor(bufferSize: number = 1) {
    super();
    this.bufferSize = bufferSize;
  }
  
  next(value: T) {
    // Add to replay buffer
    this.replayBuffer.push(value);
    if (this.replayBuffer.length > this.bufferSize) {
      this.replayBuffer.shift();
    }
    
    super.next(value);
  }
  
  // Get current value (like BehaviorSubject)
  getValue(): T | undefined {
    return this.replayBuffer[this.replayBuffer.length - 1];
  }
  
  // Get replay buffer
  getReplayBuffer(): T[] {
    return [...this.replayBuffer];
  }
  
  // Subscribe with replay
  subscribeWithReplay(observer: Partial<Observer<T>>): Subscription {
    // Emit replay buffer first
    this.replayBuffer.forEach(value => {
      if (observer.next) {
        observer.next(value);
      }
    });
    
    // Then subscribe normally
    return this.subscribe(observer);
  }
}

// Advanced multicasting service
@Injectable({ providedIn: 'root' })
export class MulticastingService {
  private dataSource$ = new BehaviorSubject<any[]>([]);
  private errorSubject$ = new Subject<Error>();
  private loadingSubject$ = new BehaviorSubject<boolean>(false);
  
  // Smart subject for complex state
  private stateSubject = new SmartSubject<AppState>(3); // Keep last 3 states
  
  // Public observables with different multicast strategies
  public data$ = this.dataSource$.asObservable();
  public errors$ = this.errorSubject$.asObservable();
  public loading$ = this.loadingSubject$.asObservable();
  
  // Hot observable that starts immediately
  public realtimeData$ = interval(1000).pipe(
    map(i => ({ id: i, value: Math.random(), timestamp: new Date() })),
    share() // Multicasting operator
  );
  
  // Cold observable that becomes hot when subscribed
  public expensiveData$ = defer(() => {
    console.log('Creating expensive observable');
    return this.http.get('/api/expensive-data');
  }).pipe(
    shareReplay({ bufferSize: 1, refCount: true })
  );
  
  // Multicast with custom subject
  public customMulticast$ = defer(() => 
    this.http.get('/api/data')
  ).pipe(
    multicast(() => new ReplaySubject(2)), // Keep last 2 emissions
    refCount()
  );
  
  constructor(private http: HttpClient) {
    this.setupComplexMulticasting();
  }
  
  private setupComplexMulticasting() {
    // Complex state management with multiple sources
    const userActions$ = new Subject<UserAction>();
    const serverEvents$ = new Subject<ServerEvent>();
    const timerEvents$ = interval(30000); // Every 30 seconds
    
    // Merge different event sources
    const allEvents$ = merge(
      userActions$.pipe(map(action => ({ type: 'user', data: action }))),
      serverEvents$.pipe(map(event => ({ type: 'server', data: event }))),
      timerEvents$.pipe(map(tick => ({ type: 'timer', data: tick })))
    );
    
    // Process events and update state
    allEvents$.pipe(
      scan((state, event) => this.processEvent(state, event), this.getInitialState()),
      tap(state => this.stateSubject.next(state)),
      takeUntil(this.destroy$)
    ).subscribe();
  }
  
  // Advanced publish/subscribe pattern
  createChannel<T>(channelName: string): ChannelSubject<T> {
    return new ChannelSubject<T>(channelName);
  }
  
  // State management with time-travel debugging
  getStateHistory(): AppState[] {
    return this.stateSubject.getReplayBuffer();
  }
  
  getCurrentState(): AppState | undefined {
    return this.stateSubject.getValue();
  }
  
  subscribeToStateChanges(callback: (state: AppState) => void): Subscription {
    return this.stateSubject.subscribeWithReplay({ next: callback });
  }
  
  private processEvent(state: AppState, event: any): AppState {
    // Process different event types
    switch (event.type) {
      case 'user':
        return { ...state, lastUserAction: event.data };
      case 'server':
        return { ...state, lastServerEvent: event.data };
      case 'timer':
        return { ...state, lastHeartbeat: new Date() };
      default:
        return state;
    }
  }
  
  private getInitialState(): AppState {
    return {
      initialized: true,
      lastUserAction: null,
      lastServerEvent: null,
      lastHeartbeat: new Date()
    };
  }
}

// Custom channel subject for pub/sub patterns
export class ChannelSubject<T> extends Subject<T> {
  private subscribers = new Map<string, Observer<T>>();
  
  constructor(private channelName: string) {
    super();
  }
  
  // Subscribe with identifier for targeted messages
  subscribeWithId(id: string, observer: Partial<Observer<T>>): Subscription {
    this.subscribers.set(id, observer as Observer<T>);
    
    const subscription = this.subscribe(observer);
    
    // Clean up on unsubscribe
    subscription.add(() => {
      this.subscribers.delete(id);
    });
    
    return subscription;
  }
  
  // Send to specific subscriber
  sendTo(id: string, value: T) {
    const subscriber = this.subscribers.get(id);
    if (subscriber && subscriber.next) {
      subscriber.next(value);
    }
  }
  
  // Broadcast to all except specific subscribers
  broadcastExcept(excludeIds: string[], value: T) {
    this.subscribers.forEach((observer, id) => {
      if (!excludeIds.includes(id) && observer.next) {
        observer.next(value);
      }
    });
  }
  
  getChannelName(): string {
    return this.channelName;
  }
  
  getSubscriberCount(): number {
    return this.subscribers.size;
  }
}

interface AppState {
  initialized: boolean;
  lastUserAction: any;
  lastServerEvent: any;
  lastHeartbeat: Date;
}

interface UserAction {
  type: string;
  payload: any;
}

interface ServerEvent {
  event: string;
  data: any;
}
```


### Q34: Complex error handling patterns with RxJS

```typescript
@Injectable()
export class RobustErrorHandlingService {
  private errorLog: ErrorEntry[] = [];
  private maxRetries = 3;
  
  constructor(private http: HttpClient, private notification: NotificationService) {}
  
  // Comprehensive error handling with classification
  makeRobustRequest<T>(url: string, options?: any): Observable<T> {
    return this.http.get<T>(url, options).pipe(
      // Add request timing
      timeout(10000),
      
      // Retry with sophisticated logic
      retryWhen(errors => 
        errors.pipe(
          mergeMap((error, index) => {
            const errorType = this.classifyError(error);
            
            // Don't retry certain error types
            if (errorType.retryable === false || index >= this.maxRetries) {
              throw error;
            }
            
            console.log(`Retry attempt ${index + 1} for ${errorType.category} error`);
            
            // Variable delay based on error type
            const delay = this.calculateRetryDelay(errorType, index);
            return timer(delay);
          }),
          take(this.maxRetries)
        )
      ),
      
      // Comprehensive error handling
      catchError(error => this.handleError(error, url))
    );
  }
  
  // Advanced error recovery with fallback strategies
  getDataWithFallback<T>(
    primaryUrl: string, 
    fallbackUrl?: string, 
    defaultValue?: T
  ): Observable<T> {
    return this.makeRobustRequest<T>(primaryUrl).pipe(
      catchError(primaryError => {
        console.warn('Primary request failed:', primaryError.message);
        
        if (fallbackUrl) {
          console.log('Attempting fallback request...');
          return this.makeRobustRequest<T>(fallbackUrl).pipe(
            catchError(fallbackError => {
              console.error('Fallback also failed:', fallbackError.message);
              
              if (defaultValue !== undefined) {
                console.log('Using default value');
                return of(defaultValue);
              }
              
              return throwError(() => new Error(
                `Both primary and fallback failed: ${primaryError.message}`
              ));
            })
          );
        }
        
        if (defaultValue !== undefined) {
          return of(defaultValue);
        }
        
        return throwError(() => primaryError);
      })
    );
  }
  
  // Circuit breaker pattern
  private circuitBreaker = new Map<string, CircuitBreakerState>();
  
  makeRequestWithCircuitBreaker<T>(url: string): Observable<T> {
    const state = this.getCircuitState(url);
    
    if (state.isOpen()) {
      return throwError(() => new Error('Circuit breaker is open'));
    }
    
    return this.http.get<T>(url).pipe(
      tap(() => state.recordSuccess()),
      catchError(error => {
        state.recordFailure();
        
        if (state.shouldOpen()) {
          console.log(`Circuit breaker opened for ${url}`);
          this.notification.showError(`Service temporarily unavailable: ${url}`);
        }
        
        return throwError(() => error);
      })
    );
  }
  
  // Bulk operations with individual error handling
  processBatch<T, R>(
    items: T[], 
    processor: (item: T) => Observable<R>
  ): Observable<BatchResult<R>> {
    return forkJoin(
      items.map((item, index) => 
        processor(item).pipe(
          map(result => ({ success: true, result, item, index })),
          catchError(error => of({ 
            success: false, 
            error: error.message, 
            item, 
            index 
          }))
        )
      )
    ).pipe(
      map(results => ({
        successful: results.filter(r => r.success),
        failed: results.filter(r => !r.success),
        totalProcessed: items.length,
        successRate: results.filter(r => r.success).length / items.length
      }))
    );
  }
  
  // Global error stream for monitoring
  globalError$ = new Subject<GlobalErrorEvent>();
  
  // Error recovery strategies
  recoverFromError<T>(
    source: Observable<T>, 
    recoveryStrategies: RecoveryStrategy<T>[]
  ): Observable<T> {
    return source.pipe(
      catchError(error => {
        console.log('Attempting error recovery for:', error.message);
        
        return from(recoveryStrategies).pipe(
          concatMap(strategy => 
            strategy.canHandle(error) 
              ? strategy.recover(error).pipe(
                  tap(() => console.log(`Recovery successful with: ${strategy.name}`)),
                  take(1)
                )
              : EMPTY
          ),
          defaultIfEmpty(null as T),
          switchMap(recovered => 
            recovered !== null 
              ? of(recovered)
              : throwError(() => error)
          )
        );
      })
    );
  }
  
  private classifyError(error: any): ErrorClassification {
    if (error.name === 'TimeoutError') {
      return { category: 'timeout', retryable: true, severity: 'medium' };
    }
    
    if (error.status) {
      if (error.status >= 500) {
        return { category: 'server', retryable: true, severity: 'high' };
      }
      if (error.status === 429) {
        return { category: 'rate-limit', retryable: true, severity: 'low' };
      }
      if (error.status >= 400) {
        return { category: 'client', retryable: false, severity: 'medium' };
      }
    }
    
    if (error.name === 'NetworkError') {
      return { category: 'network', retryable: true, severity: 'high' };
    }
    
    return { category: 'unknown', retryable: false, severity: 'high' };
  }
  
  private calculateRetryDelay(errorType: ErrorClassification, attempt: number): number {
    const baseDelay = 1000;
    
    switch (errorType.category) {
      case 'rate-limit':
        return baseDelay * Math.pow(2, attempt + 2); // Longer delay for rate limits
      case 'server':
        return baseDelay * Math.pow(2, attempt);
      case 'network':
        return baseDelay * (attempt + 1);
      default:
        return baseDelay;
    }
  }
  
  private handleError<T>(error: any, context: string): Observable<T> {
    const errorEntry: ErrorEntry = {
      timestamp: new Date(),
      context,
      error: error.message,
      stack: error.stack,
      classification: this.classifyError(error)
    };
    
    this.errorLog.push(errorEntry);
    this.globalError$.next({ error, context, classification: errorEntry.classification });
    
    // Show user-friendly message
    this.showUserFriendlyError(errorEntry.classification);
    
    return throwError(() => error);
  }
  
  private showUserFriendlyError(classification: ErrorClassification) {
    const messages = {
      'timeout': 'Request timed out. Please try again.',
      'server': 'Server error occurred. Our team has been notified.',
      'network': 'Network connection issue. Please check your connection.',
      'rate-limit': 'Too many requests. Please wait a moment.',
      'client': 'Invalid request. Please check your input.',
      'unknown': 'An unexpected error occurred.'
    };
    
    this.notification.showError(messages[classification.category] || messages['unknown']);
  }
  
  private getCircuitState(url: string): CircuitBreakerState {
    if (!this.circuitBreaker.has(url)) {
      this.circuitBreaker.set(url, new CircuitBreakerState());
    }
    return this.circuitBreaker.get(url)!;
  }
  
  getErrorLog(): ErrorEntry[] {
    return [...this.errorLog];
  }
  
  clearErrorLog() {
    this.errorLog = [];
  }
}

// Supporting interfaces and classes
interface ErrorClassification {
  category: 'timeout' | 'server' | 'network' | 'rate-limit' | 'client' | 'unknown';
  retryable: boolean;
  severity: 'low' | 'medium' | 'high';
}

interface ErrorEntry {
  timestamp: Date;
  context: string;
  error: string;
  stack?: string;
  classification: ErrorClassification;
}

interface GlobalErrorEvent {
  error: any;
  context: string;
  classification: ErrorClassification;
}

interface BatchResult<T> {
  successful: Array<{ success: true; result: T; item: any; index: number }>;
  failed: Array<{ success: false; error: string; item: any; index: number }>;
  totalProcessed: number;
  successRate: number;
}

interface RecoveryStrategy<T> {
  name: string;
  canHandle(error: any): boolean;
  recover(error: any): Observable<T>;
}

class CircuitBreakerState {
  private failures = 0;
  private lastFailure?: Date;
  private state: 'closed' | 'open' | 'half-open' = 'closed';
  private readonly failureThreshold = 5;
  private readonly openDuration = 60000; // 1 minute
  
  recordSuccess() {
    this.failures = 0;
    this.state = 'closed';
  }
  
  recordFailure() {
    this.failures++;
    this.lastFailure = new Date();
  }
  
  shouldOpen(): boolean {
    return this.failures >= this.failureThreshold;
  }
  
  isOpen(): boolean {
    if (this.state === 'open' && this.lastFailure) {
      const timeSinceFailure = Date.now() - this.lastFailure.getTime();
      if (timeSinceFailure > this.openDuration) {
        this.state = 'half-open';
        return false;
      }
      return true;
    }
    return this.state === 'open';
  }
}
```


### Q35: Advanced Observable creation and custom operators

```typescript
// Custom operators
export function filterAndLog<T>(
  predicate: (value: T) => boolean,
  logMessage?: string
) {
  return (source: Observable<T>) => 
    source.pipe(
      tap(value => {
        if (logMessage) {
          console.log(`${logMessage}:`, value);
        }
      }),
      filter(predicate)
    );
}

export function retryWithBackoff<T>(
  maxRetries: number,
  scalingDuration: number = 1000,
  excludedStatusCodes: number[] = []
) {
  return (source: Observable<T>) =>
    source.pipe(
      retryWhen(errors =>
        errors.pipe(
          mergeMap((error, index) => {
            const retryAttempt = index + 1;
            
            // Don't retry if we've exceeded max retries
            if (retryAttempt > maxRetries) {
              return throwError(error);
            }
            
            // Don't retry certain HTTP status codes
            if (error.status && excludedStatusCodes.includes(error.status)) {
              return throwError(error);
            }
            
            console.log(
              `Retry attempt ${retryAttempt}/${maxRetries} after ${scalingDuration * retryAttempt}ms`
            );
            
            return timer(scalingDuration * retryAttempt);
          })
        )
      )
    );
}

export function cacheWithExpiry<T>(expiryTime: number) {
  return (source: Observable<T>) => {
    let cache: T | undefined;
    let cacheTime = 0;
    
    return defer(() => {
      const now = Date.now();
      
      if (cache !== undefined && (now - cacheTime) < expiryTime) {
        return of(cache);
      }
      
      return source.pipe(
        tap(value => {
          cache = value;
          cacheTime = now;
        })
      );
    });
  };
}

export function debounceAfterFirst<T>(dueTime: number) {
  return (source: Observable<T>) =>
    source.pipe(
      switchMap((value, index) =>
        index === 0 ? of(value) : of(value).pipe(delay(dueTime))
      )
    );
}

// Advanced observable factory functions
export class ObservableFactory {
  
  // Create websocket observable with reconnection
  static createWebSocket<T>(
    url: string,
    protocols?: string | string[]
  ): Observable<T> {
    return new Observable(subscriber => {
      let ws: WebSocket;
      let reconnectAttempts = 0;
      const maxReconnectAttempts = 5;
      const reconnectDelay = 1000;
      
      const connect = () => {
        ws = new WebSocket(url, protocols);
        
        ws.onopen = () => {
          console.log('WebSocket connected');
          reconnectAttempts = 0;
        };
        
        ws.onmessage = (event) => {
          try {
            const data = JSON.parse(event.data);
            subscriber.next(data);
          } catch (error) {
            subscriber.error(error);
          }
        };
        
        ws.onerror = (error) => {
          console.error('WebSocket error:', error);
        };
        
        ws.onclose = (event) => {
          console.log('WebSocket closed:', event.reason);
          
          if (!event.wasClean && reconnectAttempts < maxReconnectAttempts) {
            reconnectAttempts++;
            console.log(`Reconnecting... Attempt ${reconnectAttempts}`);
            setTimeout(connect, reconnectDelay * reconnectAttempts);
          } else {
            subscriber.complete();
          }
        };
      };
      
      connect();
      
      // Cleanup function
      return () => {
        if (ws) {
          ws.close(1000, 'Observable unsubscribed');
        }
      };
    });
  }
  
  // Create polling observable with smart intervals
  static createSmartPolling<T>(
    pollFunction: () => Observable<T>,
    initialInterval: number = 5000,
    options: SmartPollingOptions = {}
  ): Observable<T> {
    const {
      maxInterval = 30000,
      backoffMultiplier = 1.5,
      resetOnActivity = true,
      activityEvents = ['click', 'keypress', 'mousemove']
    } = options;
    
    return new Observable(subscriber => {
      let currentInterval = initialInterval;
      let timeoutId: any;
      let lastActivity = Date.now();
      
      // Track user activity
      const activityHandler = () => {
        lastActivity = Date.now();
        if (currentInterval > initialInterval) {
          currentInterval = initialInterval;
          reschedule();
        }
      };
      
      if (resetOnActivity) {
        activityEvents.forEach(event => {
          document.addEventListener(event, activityHandler, { passive: true });
        });
      }
      
      const poll = () => {
        pollFunction().subscribe({
          next: (value) => {
            subscriber.next(value);
            currentInterval = initialInterval; // Reset on successful poll
            schedule();
          },
          error: (error) => {
            console.warn('Polling error:', error);
            // Increase interval on error
            currentInterval = Math.min(currentInterval * backoffMultiplier, maxInterval);
            schedule();
          }
        });
      };
      
      const schedule = () => {
        timeoutId = setTimeout(poll, currentInterval);
      };
      
      const reschedule = () => {
        clearTimeout(timeoutId);
        schedule();
      };
      
      // Start polling
      poll();
      
      // Cleanup
      return () => {
        clearTimeout(timeoutId);
        if (resetOnActivity) {
          activityEvents.forEach(event => {
            document.removeEventListener(event, activityHandler);
          });
        }
      };
    });
  }
  
  // Create observable from async iterator
  static fromAsyncIterator<T>(iterator: AsyncIterableIterator<T>): Observable<T> {
    return new Observable(subscriber => {
      let cancelled = false;
      
      (async () => {
        try {
          for await (const value of iterator) {
            if (cancelled) break;
            subscriber.next(value);
          }
          if (!cancelled) {
            subscriber.complete();
          }
        } catch (error) {
          if (!cancelled) {
            subscriber.error(error);
          }
        }
      })();
      
      return () => {
        cancelled = true;
      };
    });
  }
  
  // Create observable that emits when element enters viewport
  static createIntersectionObserver(
    element: Element,
    options?: IntersectionObserverInit
  ): Observable<IntersectionObserverEntry> {
    return new Observable(subscriber => {
      const observer = new IntersectionObserver(
        (entries) => {
          entries.forEach(entry => subscriber.next(entry));
        },
        options
      );
      
      observer.observe(element);
      
      return () => observer.disconnect();
    });
  }
  
  // Create observable from Server-Sent Events
  static createEventSource(url: string): Observable<MessageEvent> {
    return new Observable(subscriber => {
      const eventSource = new EventSource(url);
      
      eventSource.onmessage = (event) => subscriber.next(event);
      eventSource.onerror = (error) => subscriber.error(error);
      
      return () => eventSource.close();
    });
  }
}

interface SmartPollingOptions {
  maxInterval?: number;
  backoffMultiplier?: number;
  resetOnActivity?: boolean;
  activityEvents?: string[];
}

// Usage examples
@Component({
  template: `
    <div class="observable-examples">
      <div class="websocket-data">
        <h3>WebSocket Data:</h3>
        <pre>{{wsData | json}}</pre>
      </div>
      
      <div class="polling-data">
        <h3>Smart Polling Data:</h3>
        <pre>{{pollingData | json}}</pre>
      </div>
      
      <div class="intersection-status">
        <h3>Element Visibility:</h3>
        <p>{{isVisible ? 'Visible' : 'Not Visible'}}</p>
      </div>
      
      <div #observedElement class="observed-element">
        This element is being observed
      </div>
    </div>
  `
})
export class ObservableExamplesComponent implements OnInit, OnDestroy {
  wsData: any = null;
  pollingData: any = null;
  isVisible = false;
  
  @ViewChild('observedElement', { static: true }) 
  observedElement!: ElementRef;
  
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    this.setupWebSocket();
    this.setupSmartPolling();
    this.setupIntersectionObserver();
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
  
  private setupWebSocket() {
    ObservableFactory.createWebSocket('ws://localhost:8080/ws')
      .pipe(
        takeUntil(this.destroy$),
        retryWithBackoff(3, 2000, [4000, 4001])
      )
      .subscribe({
        next: data => this.wsData = data,
        error: error => console.error('WebSocket error:', error)
      });
  }
  
  private setupSmartPolling() {
    ObservableFactory.createSmartPolling(
      () => this.http.get('/api/status'),
      5000,
      { maxInterval: 30000, backoffMultiplier: 2 }
    ).pipe(
      takeUntil(this.destroy$),
      cacheWithExpiry(60000) // Cache for 1 minute
    ).subscribe({
      next: data => this.pollingData = data,
      error: error => console.error('Polling error:', error)
    });
  }
  
  private setupIntersectionObserver() {
    ObservableFactory.createIntersectionObserver(
      this.observedElement.nativeElement,
      { threshold: 0.1 }
    ).pipe(
      takeUntil(this.destroy$),
      map(entry => entry.isIntersecting)
    ).subscribe(isVisible => {
      this.isVisible = isVisible;
    });
  }
  
  constructor(private http: HttpClient) {}
}
```


## Performance Optimization Deep Dive

### Q36: Implement advanced OnPush optimization strategies

```typescript
// Performance monitoring service
@Injectable({ providedIn: 'root' })
export class PerformanceMonitorService {
  private measurements: PerformanceMeasurement[] = [];
  
  measureChangeDetection(componentName: string): () => void {
    const startTime = performance.now();
    
    return () => {
      const endTime = performance.now();
      const duration = endTime - startTime;
      
      this.measurements.push({
        component: componentName,
        duration,
        timestamp: new Date(),
        type: 'change-detection'
      });
      
      if (duration > 16.67) { // > 60fps threshold
        console.warn(`Slow change detection in ${componentName}: ${duration.toFixed(2)}ms`);
      }
    };
  }
  
  getMeasurements(): PerformanceMeasurement[] {
    return [...this.measurements];
  }
  
  getAverageDetectionTime(componentName?: string): number {
    const filtered = componentName 
      ? this.measurements.filter(m => m.component === componentName)
      : this.measurements;
    
    if (filtered.length === 0) return 0;
    
    return filtered.reduce((sum, m) => sum + m.duration, 0) / filtered.length;
  }
}

interface PerformanceMeasurement {
  component: string;
  duration: number;
  timestamp: Date;
  type: string;
}

// Advanced OnPush component with fine-grained control
@Component({
  selector: 'optimized-component',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="optimized-component">
      <h2>{{title}}</h2>
      
      <!-- Immutable data display -->
      <div class="user-info" *ngIf="user">
        <p>Name: {{user.name}}</p>
        <p>Email: {{user.email}}</p>
        <p>Last Updated: {{user.lastUpdated | date:'medium'}}</p>
      </div>
      
      <!-- OnPush-optimized list -->
      <div class="items-list">
        <div 
          *ngFor="let item of items; trackBy: trackByItemId"
          class="item"
          [class.highlighted]="item.highlighted">
          
          <!-- Child components with OnPush -->
          <optimized-item 
            [item]="item"
            [config]="itemConfig"
            (itemChange)="onItemChange($event)">
          </optimized-item>
        </div>
      </div>
      
      <!-- Conditional rendering with OnPush -->
      <div *ngIf="showDetails$ | async" class="details">
        <expensive-detail-component 
          [data]="detailData$ | async"
          [options]="detailOptions">
        </expensive-detail-component>
      </div>
      
      <!-- Performance metrics -->
      <div class="performance-info" *ngIf="showPerformanceInfo">
        <small>
          CD Count: {{changeDetectionCount}} | 
          Avg Time: {{averageDetectionTime}}ms |
          Last Update: {{lastUpdateTime | date:'HH:mm:ss.SSS'}}
        </small>
      </div>
    </div>
  `
})
export class OptimizedComponent implements OnInit, OnDestroy {
  @Input() title = 'Optimized Component';
  @Input() user: User | null = null;
  @Input() items: readonly Item[] = [];
  @Input() showPerformanceInfo = false;
  
  // Immutable configuration object
  readonly itemConfig: Readonly<ItemConfig> = Object.freeze({
    showActions: true,
    allowEdit: true,
    theme: 'default'
  });
  
  // Immutable detail options
  readonly detailOptions: Readonly<DetailOptions> = Object.freeze({
    expandable: true,
    showTimestamps: true,
    maxItems: 100
  });
  
  // Observables for async data
  showDetails$ = new BehaviorSubject<boolean>(false);
  detailData$ = new BehaviorSubject<DetailData | null>(null);
  
  // Performance tracking
  changeDetectionCount = 0;
  averageDetectionTime = 0;
  lastUpdateTime = new Date();
  
  private destroy$ = new Subject<void>();
  
  constructor(
    private cdr: ChangeDetectorRef,
    private performance: PerformanceMonitorService,
    private ngZone: NgZone
  ) {}
  
  ngOnInit() {
    // Setup performance monitoring
    this.setupPerformanceMonitoring();
    
    // Setup data streams
    this.setupDataStreams();
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
  
  // Efficient tracking function
  trackByItemId(index: number, item: Item): number {
    return item.id;
  }
  
  // Optimized event handling
  onItemChange(event: ItemChangeEvent) {
    // Create new immutable array to trigger OnPush
    const updatedItems = this.items.map(item =>
      item.id === event.itemId
        ? { ...item, ...event.changes, lastModified: new Date() }
        : item
    );
    
    // Emit change to parent (if needed)
    // this.itemsChange.emit(updatedItems);
    
    // Manual change detection trigger if necessary
    this.markForCheck();
  }
  
  // Manual change detection control
  private markForCheck() {
    if (!this.cdr['destroyed']) { // Check if not destroyed
      this.cdr.markForCheck();
    }
  }
  
  // Detach from change detection temporarily
  pauseChangeDetection() {
    this.cdr.detach();
  }
  
  resumeChangeDetection() {
    this.cdr.reattach();
    this.markForCheck();
  }
  
  // Run expensive operations outside Angular zone
  runExpensiveOperation(data: any[]) {
    this.ngZone.runOutsideAngular(() => {
      // Expensive computation
      const result = this.processLargeDataset(data);
      
      // Update UI in Angular zone
      this.ngZone.run(() => {
        this.detailData$.next(result);
        this.markForCheck();
      });
    });
  }
  
  private setupPerformanceMonitoring() {
    if (this.showPerformanceInfo) {
      // Override ngDoCheck to measure performance
      const originalDoCheck = this.ngDoCheck?.bind(this);
      
      this.ngDoCheck = () => {
        const endMeasurement = this.performance.measureChangeDetection('OptimizedComponent');
        
        this.changeDetectionCount++;
        this.lastUpdateTime = new Date();
        
        if (originalDoCheck) {
          originalDoCheck();
        }
        
        endMeasurement();
        
        // Update average (throttled)
        if (this.changeDetectionCount % 10 === 0) {
          this.averageDetectionTime = Math.round(
            this.performance.getAverageDetectionTime('OptimizedComponent') * 100
          ) / 100;
        }
      };
    }
  }
  
  private setupDataStreams() {
    // Efficient observable composition
    const dataStream$ = combineLatest([
      this.showDetails$,
      interval(30000) // Refresh every 30 seconds
    ]).pipe(
      switchMap(([showDetails]) => 
        showDetails ? this.loadDetailData() : of(null)
      ),
      shareReplay(1),
      takeUntil(this.destroy$)
    );
    
    dataStream$.subscribe(data => {
      this.detailData$.next(data);
      this.markForCheck();
    });
  }
  
  private loadDetailData(): Observable<DetailData> {
    // Simulate API call
    return of({
      id: Math.random(),
      content: 'Detailed content',
      timestamp: new Date()
    }).pipe(delay(100));
  }
  
  private processLargeDataset(data: any[]): DetailData {
    // Simulate expensive computation
    const start = performance.now();
    
    // Heavy processing simulation
    let result = '';
    for (let i = 0; i < 100000; i++) {
      result += Math.random().toString(36).substr(2, 9);
    }
    
    const duration = performance.now() - start;
    console.log(`Processed dataset in ${duration.toFixed(2)}ms`);
    
    return {
      id: Date.now(),
      content: `Processed ${data.length} items`,
      timestamp: new Date()
    };
  }
  
  ngDoCheck?: () => void;
}

// Child component optimized for OnPush
@Component({
  selector: 'optimized-item',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="optimized-item" [class.editing]="isEditing">
      <span *ngIf="!isEditing" (dblclick)="startEdit()">
        {{item.name}}
      </span>
      
      <input 
        *ngIf="isEditing"
        [(ngModel)]="editValue"
        (blur)="saveEdit()"
        (keyup.enter)="saveEdit()"
        (keyup.escape)="cancelEdit()"
        #editInput>
      
      <div class="actions" *ngIf="config.showActions">
        <button (click)="toggleHighlight()">Highlight</button>
        <button (click)="deleteItem()">Delete</button>
      </div>
    </div>
  `
})
export class OptimizedItemComponent implements OnChanges {
  @Input() item!: Item;
  @Input() config!: Readonly<ItemConfig>;
  @Output() itemChange = new EventEmitter<ItemChangeEvent>();
  
  isEditing = false;
  editValue = '';
  
  constructor(private cdr: ChangeDetectorRef) {}
  
  ngOnChanges(changes: SimpleChanges) {
    // Only react to actual item changes
    if (changes['item'] && changes['item'].currentValue !== changes['item'].previousValue) {
      // Reset editing state if item changed
      if (this.isEditing) {
        this.cancelEdit();
      }
    }
  }
  
  startEdit() {
    if (!this.config.allowEdit) return;
    
    this.isEditing = true;
    this.editValue = this.item.name;
    this.cdr.markForCheck();
    
    // Focus input after view update
    setTimeout(() => {
      const input = document.querySelector('.optimized-item input') as HTMLInputElement;
      input?.focus();
    });
  }
  
  saveEdit() {
    if (this.editValue.trim() && this.editValue !== this.item.name) {
      this.itemChange.emit({
        itemId: this.item.id,
        changes: { name: this.editValue.trim() }
      });
    }
    
    this.isEditing = false;
    this.cdr.markForCheck();
  }
  
  cancelEdit() {
    this.isEditing = false;
    this.editValue = '';
    this.cdr.markForCheck();
  }
  
  toggleHighlight() {
    this.itemChange.emit({
      itemId: this.item.id,
      changes: { highlighted: !this.item.highlighted }
    });
  }
  
  deleteItem() {
    this.itemChange.emit({
      itemId: this.item.id,
      changes: { deleted: true }
    });
  }
}

// Supporting interfaces
interface Item {
  id: number;
  name: string;
  highlighted: boolean;
  lastModified: Date;
  deleted?: boolean;
}

interface ItemConfig {
  showActions: boolean;
  allowEdit: boolean;
  theme: string;
}

interface DetailOptions {
  expandable: boolean;
  showTimestamps: boolean;
  maxItems: number;
}

interface DetailData {
  id: number;
  content: string;
  timestamp: Date;
}

interface User {
  name: string;
  email: string;
  lastUpdated: Date;
}

interface ItemChangeEvent {
  itemId: number;
  changes: Partial<Item>;
}
```


### Q37: Advanced lazy loading and code splitting strategies

```typescript
// Lazy loading service with preloading strategies
@Injectable({ providedIn: 'root' })
export class LazyLoadingService {
  private preloadedModules = new Set<string>();
  private loadingModules = new Map<string, Promise<any>>();
  
  constructor(@Inject(DOCUMENT) private document: Document) {}
  
  // Dynamic module loading with caching
  async loadModule(modulePath: string, preload = false): Promise<any> {
    // Check if already loaded
    if (this.preloadedModules.has(modulePath)) {
      console.log(`Module ${modulePath} already preloaded`);
      return Promise.resolve();
    }
    
    // Check if currently loading
    if (this.loadingModules.has(modulePath)) {
      return this.loadingModules.get(modulePath);
    }
    
    // Start loading
    const loadPromise = this.performModuleLoad(modulePath);
    this.loadingModules.set(modulePath, loadPromise);
    
    try {
      const result = await loadPromise;
      
      if (preload) {
        this.preloadedModules.add(modulePath);
      }
      
      this.loadingModules.delete(modulePath);
      return result;
    } catch (error) {
      this.loadingModules.delete(modulePath);
      throw error;
    }
  }
  
  private async performModuleLoad(modulePath: string): Promise<any> {
    const startTime = performance.now();
    
    try {
      const module = await import(modulePath);
      const loadTime = performance.now() - startTime;
      
      console.log(`Loaded module ${modulePath} in ${loadTime.toFixed(2)}ms`);
      return module;
    } catch (error) {
      console.error(`Failed to load module ${modulePath}:`, error);
      throw error;
    }
  }
  
  // Preload modules based on user behavior
  preloadModulesOnIdle(modules: string[]) {
    if ('requestIdleCallback' in window) {
      (window as any).requestIdleCallback(() => {
        this.preloadModules(modules);
      });
    } else {
      setTimeout(() => this.preloadModules(modules), 1000);
    }
  }
  
  private async preloadModules(modules: string[]) {
    for (const module of modules) {
      try {
        await this.loadModule(module, true);
      } catch (error) {
        console.warn(`Failed to preload module ${module}:`, error);
      }
    }
  }
  
  // Smart preloading based on route patterns
  preloadRelatedModules(currentRoute: string) {
    const preloadMap: Record<string, string[]> = {
      '/dashboard': ['./reports/reports.module', './analytics/analytics.module'],
      '/users': ['./user-details/user-details.module', './permissions/permissions.module'],
      '/products': ['./inventory/inventory.module', './orders/orders.module']
    };
    
    const modulesToPreload = preloadMap[currentRoute];
    if (modulesToPreload) {
      this.preloadModulesOnIdle(modulesToPreload);
    }
  }
  
  getLoadingStats(): LoadingStats {
    return {
      preloadedCount: this.preloadedModules.size,
      currentlyLoading: this.loadingModules.size,
      preloadedModules: Array.from(this.preloadedModules)
    };
  }
}

interface LoadingStats {
  preloadedCount: number;
  currentlyLoading: number;
  preloadedModules: string[];
}

// Advanced component lazy loading
@Injectable()
export class ComponentLazyLoader {
  private componentCache = new Map<string, ComponentFactory<any>>();
  
  constructor(
    private cfr: ComponentFactoryResolver,
    private vcr: ViewContainerRef
  ) {}
  
  // Load component dynamically
  async loadComponent<T>(
    componentPath: string,
    componentName: string,
    container: ViewContainerRef,
    inputs?: any
  ): Promise<ComponentRef<T>> {
    const cacheKey = `${componentPath}#${componentName}`;
    
    // Check cache first
    let factory = this.componentCache.get(cacheKey);
    
    if (!factory) {
      // Load module dynamically
      const module = await import(componentPath);
      const component = module[componentName];
      
      if (!component) {
        throw new Error(`Component ${componentName} not found in ${componentPath}`);
      }
      
      factory = this.cfr.resolveComponentFactory(component);
      this.componentCache.set(cacheKey, factory);
    }
    
    // Create component instance
    const componentRef = container.createComponent(factory);
    
    // Set inputs if provided
    if (inputs) {
      Object.keys(inputs).forEach(key => {
        componentRef.instance[key] = inputs[key];
      });
    }
    
    return componentRef;
  }
  
  // Clear component cache
  clearCache() {
    this.componentCache.clear();
  }
}

// Lazy loading directive for images and content
@Directive({
  selector: '[appLazyLoad]'
})
export class LazyLoadDirective implements OnInit, OnDestroy {
  @Input('appLazyLoad') src: string = '';
  @Input() placeholder = 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjAwIiBoZWlnaHQ9IjIwMCIgdmlld0JveD0iMCAwIDIwMCAyMDAiIGZpbGw9Im5vbmUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+CjxyZWN0IHdpZHRoPSIyMDAiIGhlaWdodD0iMjAwIiBmaWxsPSIjRjNGNEY2Ii8+Cjx0ZXh0IHg9IjUwJSIgeT0iNTAlIiBkb21pbmFudC1iYXNlbGluZT0ibWlkZGxlIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBmaWxsPSIjOUM5RkE2Ij5Mb2FkaW5nLi4uPC90ZXh0Pgo8L3N2Zz4K';
  @Input() threshold = 0.1;
  @Input() rootMargin = '50px';
  
  private observer?: IntersectionObserver;
  private isLoaded = false;
  
  constructor(private el: ElementRef) {}
  
  ngOnInit() {
    // Set placeholder initially
    if (this.el.nativeElement.tagName === 'IMG') {
      this.el.nativeElement.src = this.placeholder;
    }
    
    // Create intersection observer
    this.observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting && !this.isLoaded) {
            this.loadContent();
          }
        });
      },
      {
        threshold: this.threshold,
        rootMargin: this.rootMargin
      }
    );
    
    this.observer.observe(this.el.nativeElement);
  }
  
  ngOnDestroy() {
    if (this.observer) {
      this.observer.disconnect();
    }
  }
  
  private async loadContent() {
    if (this.isLoaded) return;
    
    this.isLoaded = true;
    
    if (this.el.nativeElement.tagName === 'IMG') {
      await this.loadImage();
    } else {
      // Load content for other elements
      await this.loadDynamicContent();
    }
    
    // Disconnect observer after loading
    if (this.observer) {
      this.observer.disconnect();
    }
  }
  
  private loadImage(): Promise<void> {
    return new Promise((resolve, reject) => {
      const img = new Image();
      
      img.onload = () => {
        this.el.nativeElement.src = this.src;
        this.el.nativeElement.classList.add('loaded');
        resolve();
      };
      
      img.onerror = () => {
        this.el.nativeElement.classList.add('error');
        reject(new Error(`Failed to load image: ${this.src}`));
      };
      
      img.src = this.src;
    });
  }
  
  private async loadDynamicContent() {
    try {
      // Simulate loading dynamic content
      this.el.nativeElement.classList.add('loading');
      
      // Load content (could be from API, file, etc.)
      await new Promise(resolve => setTimeout(resolve, 500));
      
      this.el.nativeElement.textContent = 'Content loaded!';
      this.el.nativeElement.classList.remove('loading');
      this.el.nativeElement.classList.add('loaded');
    } catch (error) {
      this.el.nativeElement.classList.remove('loading');
      this.el.nativeElement.classList.add('error');
      this.el.nativeElement.textContent = 'Failed to load content';
    }
  }
}

// Advanced route preloading strategy
@Injectable()
export class IntelligentPreloadingStrategy implements PreloadingStrategy {
  private preloadedRoutes = new Set<string>();
  
  constructor(
    private router: Router,
    private lazyLoader: LazyLoadingService
  ) {}
  
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    const routePath = route.path || '';
    
    // Skip if already preloaded
    if (this.preloadedRoutes.has(routePath)) {
      return of(null);
    }
    
    // Determine if route should be preloaded
    if (this.shouldPreload(route)) {
      console.log('Preloading route:', routePath);
      this.preloadedRoutes.add(routePath);
      
      return load().pipe(
        catchError(error => {
          console.error(`Failed to preload route ${routePath}:`, error);
          return of(null);
        })
      );
    }
    
    return of(null);
  }
  
  private shouldPreload(route: Route): boolean {
    // Preload routes marked as high priority
    if (route.data?.['preload'] === 'high') {
      return true;
    }
    
    // Preload based on user behavior
    if (route.data?.['userFrequent']) {
      return true;
    }
    
    // Don't preload admin routes for regular users
    if (route.path?.includes('admin') && !this.isAdmin()) {
      return false;
    }
    
    // Preload small modules
    if (route.data?.['size'] === 'small') {
      return true;
    }
    
    // Don't preload on slow connections
    if (this.isSlowConnection()) {
      return route.data?.['preload'] === 'critical';
    }
    
    return route.data?.['preload'] === true;
  }
  
  private isAdmin(): boolean {
    // Check user permissions
    return false; // Implement based on your auth system
  }
  
  private isSlowConnection(): boolean {
    const connection = (navigator as any).connection;
    if (connection) {
      return connection.effectiveType === 'slow-2g' || connection.effectiveType === '2g';
    }
    return false;
  }
}

// Usage in app routing
const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module').then(m => m.DashboardModule),
    data: { preload: 'high', size: 'small' }
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    data: { preload: false }
  },
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.module').then(m => m.ReportsModule),
    data: { preload: true, userFrequent: true }
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes, {
    preloadingStrategy: IntelligentPreloadingStrategy
  })],
  providers: [IntelligentPreloadingStrategy],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```


### Q38: Memory leak detection and prevention

```typescript
// Memory leak detection service
@Injectable({ providedIn: 'root' })
export class MemoryLeakDetectionService {
  private subscriptionTracker = new Map<string, SubscriptionInfo[]>();
  private componentInstances = new WeakMap<object, ComponentMemoryInfo>();
  private memorySnapshots: MemorySnapshot[] = [];
  
  constructor() {
    this.startMemoryMonitoring();
  }
  
  // Track component subscriptions
  trackSubscription(
    componentName: string, 
    subscription: Subscription, 
    source: string
  ): Subscription {
    const info: SubscriptionInfo = {
      subscription,
      source,
      createdAt: new Date(),
      componentName
    };
    
    if (!this.subscriptionTracker.has(componentName)) {
      this.subscriptionTracker.set(componentName, []);
    }
    
    this.subscriptionTracker.get(componentName)!.push(info);
    
    // Wrap unsubscribe to track cleanup
    const originalUnsubscribe = subscription.unsubscribe.bind(subscription);
    subscription.unsubscribe = () => {
      this.removeSubscriptionTracking(componentName, subscription);
      originalUnsubscribe();
    };
    
    return subscription;
  }
  
  // Register component instance for memory tracking
  registerComponent(component: object, name: string) {
    const info: ComponentMemoryInfo = {
      name,
      createdAt: new Date(),
      subscriptions: new Set(),
      timers: new Set(),
      eventListeners: new Set()
    };
    
    this.componentInstances.set(component, info);
  }
  
  // Unregister component and check for leaks
  unregisterComponent(component: object): MemoryLeakReport {
    const info = this.componentInstances.get(component);
    if (!info) {
      return { hasLeaks: false, leaks: [] };
    }
    
    const leaks: MemoryLeak[] = [];
    
    // Check for active subscriptions
    if (info.subscriptions.size > 0) {
      leaks.push({
        type: 'subscription',
        count: info.subscriptions.size,
        details: Array.from(info.subscriptions)
      });
    }
    
    // Check for active timers
    if (info.timers.size > 0) {
      leaks.push({
        type: 'timer',
        count: info.timers.size,
        details: Array.from(info.timers)
      });
    }
    
    // Check for event listeners
    if (info.eventListeners.size > 0) {
      leaks.push({
        type: 'eventListener',
        count: info.eventListeners.size,
        details: Array.from(info.eventListeners)
      });
    }
    
    this.componentInstances.delete(component);
    
    const report: MemoryLeakReport = {
      hasLeaks: leaks.length > 0,
      leaks,
      componentName: info.name
    };
    
    if (report.hasLeaks) {
      console.warn('Memory leaks detected in component:', report);
    }
    
    return report;
  }
  
  // Check for memory leaks across all tracked components
  performLeakCheck(): GlobalLeakReport {
    const activeSubscriptions = new Map<string, number>();
    let totalLeaks = 0;
    
    this.subscriptionTracker.forEach((subscriptions, componentName) => {
      const activeCount = subscriptions.filter(s => !s.subscription.closed).length;
      if (activeCount > 0) {
        activeSubscriptions.set(componentName, activeCount);
        totalLeaks += activeCount;
      }
    });
    
    return {
      totalActiveSubscriptions: totalLeaks,
      componentLeaks: Array.from(activeSubscriptions.entries()).map(([component, count]) => ({
        component,
        activeSubscriptions: count
      })),
      memoryUsage: this.getCurrentMemoryUsage()
    };
  }
  
  // Monitor memory usage over time
  private startMemoryMonitoring() {
    setInterval(() => {
      const snapshot: MemorySnapshot = {
        timestamp: new Date(),
        heapUsed: this.getCurrentMemoryUsage(),
        componentCount: this.componentInstances.size,
        activeSubscriptions: this.getTotalActiveSubscriptions()
      };
      
      this.memorySnapshots.push(snapshot);
      
      // Keep only last 100 snapshots
      if (this.memorySnapshots.length > 100) {
        this.memorySnapshots.shift();
      }
      
      // Check for memory growth
      this.detectMemoryGrowth();
    }, 30000); // Every 30 seconds
  }
  
  private detectMemoryGrowth() {
    if (this.memorySnapshots.length < 5) return;
    
    const recent = this.memorySnapshots.slice(-5);
    const oldest = recent[0];
    const newest = recent[recent.length - 1];
    
    const growthRate = (newest.heapUsed - oldest.heapUsed) / oldest.heapUsed;
    
    if (growthRate > 0.1) { // 10% growth
      console.warn('Potential memory leak detected - memory growth:', 
        `${(growthRate * 100).toFixed(2)}%`);
    }
  }
  
  private removeSubscriptionTracking(componentName: string, subscription: Subscription) {
    const subscriptions = this.subscriptionTracker.get(componentName);
    if (subscriptions) {
      const index = subscriptions.findIndex(s => s.subscription === subscription);
      if (index !== -1) {
        subscriptions.splice(index, 1);
        
        if (subscriptions.length === 0) {
          this.subscriptionTracker.delete(componentName);
        }
      }
    }
  }
  
  private getCurrentMemoryUsage(): number {
    return (performance as any).memory?.usedJSHeapSize || 0;
  }
  
  private getTotalActiveSubscriptions(): number {
    let total = 0;
    this.subscriptionTracker.forEach(subscriptions => {
      total += subscriptions.filter(s => !s.subscription.closed).length;
    });
    return total;
  }
  
  getMemorySnapshots(): MemorySnapshot[] {
    return [...this.memorySnapshots];
  }
}

// Memory-safe base component
export abstract class MemorySafeComponent implements OnDestroy {
  protected destroy$ = new Subject<void>();
  private subscriptions: Subscription[] = [];
  private timers: number[] = [];
  private intervalIds: number[] = [];
  
  constructor(
    private memoryDetection: MemoryLeakDetectionService,
    componentName: string
  ) {
    this.memoryDetection.registerComponent(this, componentName);
  }
  
  ngOnDestroy() {
    // Complete destroy subject first
    this.destroy$.next();
    this.destroy$.complete();
    
    // Clean up tracked subscriptions
    this.subscriptions.forEach(sub => sub.unsubscribe());
    
    // Clear timers
    this.timers.forEach(timer => clearTimeout(timer));
    this.intervalIds.forEach(interval => clearInterval(interval));
    
    // Check for memory leaks
    const leakReport = this.memoryDetection.unregisterComponent(this);
    
    if (leakReport.hasLeaks) {
      console.error('Memory leaks detected on component destruction:', leakReport);
    }
  }
  
  // Safe subscription wrapper
  protected safeSubscribe<T>(
    observable: Observable<T>,
    observer?: Partial<Observer<T>>
  ): Subscription {
    const subscription = observable.pipe(
      takeUntil(this.destroy$)
    ).subscribe(observer);
    
    this.subscriptions.push(subscription);
    this.memoryDetection.trackSubscription(
      this.constructor.name,
      subscription,
      'safeSubscribe'
    );
    
    return subscription;
  }
  
  // Safe timer wrapper
  protected safeSetTimeout(callback: () => void, delay: number): number {
    const timerId = setTimeout(() => {
      callback();
      // Remove from tracking when executed
      const index = this.timers.indexOf(timerId);
      if (index !== -1) {
        this.timers.splice(index, 1);
      }
    }, delay);
    
    this.timers.push(timerId);
    return timerId;
  }
  
  // Safe interval wrapper
  protected safeSetInterval(callback: () => void, interval: number): number {
    const intervalId = setInterval(callback, interval);
    this.intervalIds.push(intervalId);
    return intervalId;
  }
  
  // Manual cleanup for specific resources
  protected cleanupResource(cleanup: () => void) {
    // Execute cleanup immediately and on destroy
    this.destroy$.subscribe({ 
      complete: cleanup,
      error: cleanup
    });
  }
}

// Usage example
@Component({
  selector: 'memory-safe-example',
  template: `
    <div>
      <p>Memory safe component example</p>
      <p>Active subscriptions: {{activeSubscriptions}}</p>
      <button (click)="createSubscription()">Add Subscription</button>
      <button (click)="performLeakCheck()">Check Leaks</button>
    </div>
  `
})
export class MemorySafeExampleComponent extends MemorySafeComponent implements OnInit {
  activeSubscriptions = 0;
  
  constructor(
    memoryDetection: MemoryLeakDetectionService,
    private http: HttpClient
  ) {
    super(memoryDetection, 'MemorySafeExampleComponent');
  }
  
  ngOnInit() {
    // Safe subscription - automatically cleaned up
    this.safeSubscribe(
      interval(1000),
      {
        next: (value) => console.log('Timer tick:', value),
        error: (error) => console.error('Timer error:', error)
      }
    );
    
    // Safe HTTP request
    this.safeSubscribe(
      this.http.get('/api/data'),
      {
        next: (data) => console.log('Data received:', data),
        error: (error) => console.error('HTTP error:', error)
      }
    );
    
    // Safe timer
    this.safeSetTimeout(() => {
      console.log('Timeout executed');
    }, 5000);
    
    // Manual cleanup example
    const eventListener = () => console.log('Window resized');
    window.addEventListener('resize', eventListener);
    
    this.cleanupResource(() => {
      window.removeEventListener('resize', eventListener);
    });
  }
  
  createSubscription() {
    this.safeSubscribe(
      timer(0, 1000),
      { next: () => this.activeSubscriptions++ }
    );
  }
  
  performLeakCheck() {
    const report = this.memoryDetection.performLeakCheck();
    console.log('Leak check report:', report);
  }
}

// Supporting interfaces
interface SubscriptionInfo {
  subscription: Subscription;
  source: string;
  createdAt: Date;
  componentName: string;
}

interface ComponentMemoryInfo {
  name: string;
  createdAt: Date;
  subscriptions: Set<string>;
  timers: Set<number>;
  eventListeners: Set<string>;
}

interface MemoryLeak {
  type: 'subscription' | 'timer' | 'eventListener';
  count: number;
  details: any[];
}

interface MemoryLeakReport {
  hasLeaks: boolean;
  leaks: MemoryLeak[];
  componentName?: string;
}

interface GlobalLeakReport {
  totalActiveSubscriptions: number;
  componentLeaks: Array<{
    component: string;
    activeSubscriptions: number;
  }>;
  memoryUsage: number;
}

interface MemorySnapshot {
  timestamp: Date;
  heapUsed: number;
  componentCount: number;
  activeSubscriptions: number;
}
```


## Security and Best Practices

### Q39: Implement Content Security Policy and XSS prevention

```typescript
// Security service for XSS prevention and CSP management
@Injectable({ providedIn: 'root' })
export class SecurityService {
  private trustedDomains: Set<string>;
  private cspViolations: CSPViolation[] = [];
  
  constructor(
    private sanitizer: DomSanitizer,
    @Inject(DOCUMENT) private document: Document
  ) {
    this.trustedDomains = new Set([
      'your-api-domain.com',
      'trusted-cdn.com',
      'analytics-provider.com'
    ]);
    
    this.setupCSPViolationReporting();
  }
  
  // Comprehensive input sanitization
  sanitizeInput(input: string, context: SecurityContext = SecurityContext.HTML): string {
    if (!input) return '';
    
    // Remove dangerous patterns
    let sanitized = input
      .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '') // Remove script tags
      .replace(/javascript:/gi, '') // Remove javascript: URLs
      .replace(/on\w+\s*=/gi, ''); // Remove event handlers
    
    // Use Angular's built-in sanitizer
    const angularSanitized = this.sanitizer.sanitize(context, sanitized);
    
    return angularSanitized || '';
  }
  
  // Safe HTML rendering with whitelist
  renderSafeHTML(html: string, allowedTags: string[] = []): SafeHtml {
    const defaultAllowedTags = ['p', 'br', 'strong', 'em', 'u', 'h1', 'h2', 'h3', 'ul', 'ol', 'li'];
    const allowedTagsSet = new Set([...defaultAllowedTags, ...allowedTags]);
    
    // Parse HTML and filter allowed tags
    const parser = new DOMParser();
    const doc = parser.parseFromString(html, 'text/html');
    
    this.filterElements(doc.body, allowedTagsSet);
    
    const safeHTML = doc.body.innerHTML;
    return this.sanitizer.bypassSecurityTrustHtml(safeHTML);
  }
  
  private filterElements(element: Element, allowedTags: Set<string>) {
    const children = Array.from(element.children);
    
    children.forEach(child => {
      if (!allowedTags.has(child.tagName.toLowerCase())) {
        // Remove disallowed elements but keep text content
        const textNode = this.document.createTextNode(child.textContent || '');
        child.parentNode?.replaceChild(textNode, child);
      } else {
        // Remove all attributes except safe ones
        const safeAttributes = ['href', 'title', 'alt'];
        const attributesToRemove: string[] = [];
        
        for (let i = 0; i < child.attributes.length; i++) {
          const attr = child.attributes[i];
          if (!safeAttributes.includes(attr.name)) {
            attributesToRemove.push(attr.name);
          }
        }
        
        attributesToRemove.forEach(attr => child.removeAttribute(attr));
        
        // Recursively filter children
        this.filterElements(child, allowedTags);
      }
    });
  }
  
  // Validate URLs for safety
  validateURL(url: string): ValidationResult {
    if (!url) return { valid: false, reason: 'Empty URL' };
    
    try {
      const urlObj = new URL(url);
      
      // Check protocol
      if (!['http:', 'https:', 'mailto:', 'tel:'].includes(urlObj.protocol)) {
        return { valid: false, reason: 'Unsafe protocol' };
      }
      
      // Check for dangerous patterns
      if (url.includes('javascript:') || url.includes('data:')) {
        return { valid: false, reason: 'Dangerous URL pattern' };
      }
      
      // Validate domain if external link
      if (urlObj.protocol.startsWith('http')) {
        const domain = urlObj.hostname;
        if (!this.isDomainTrusted(domain) && !this.isInternalDomain(domain)) {
          return { 
            valid: true, 
            reason: 'External domain', 
            requiresWarning: true,
            domain 
          };
        }
      }
      
      return { valid: true };
    } catch (error) {
      return { valid: false, reason: 'Invalid URL format' };
    }
  }
  
  // CSP header generation
  generateCSPHeader(options: CSPOptions = {}): string {
    const defaultCSP = {
      'default-src': ["'self'"],
      'script-src': ["'self'", "'unsafe-inline'"],
      'style-src': ["'self'", "'unsafe-inline'"],
      'img-src': ["'self'", 'data:', 'https:'],
      'font-src': ["'self'"],
      'connect-src': ["'self'"],
      'frame-src': ["'none'"],
      'object-src': ["'none'"],
      'base-uri': ["'self'"],
      'form-action': ["'self'"]
    };
    
    const mergedCSP = { ...defaultCSP, ...options };
    
    return Object.entries(mergedCSP)
      .map(([directive, sources]) => `${directive} ${sources.join(' ')}`)
      .join('; ');
  }
  
  // Input validation patterns
  private validationPatterns = {
    email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    phone: /^\+?[\d\s\-\(\)]+$/,
    alphanumeric: /^[a-zA-Z0-9]+$/,
    noScript: /^(?!.*<script).*$/i,
    sqlSafe: /^(?!.*(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER)\b)).*$/i
  };
  
  validateInput(input: string, pattern: keyof typeof this.validationPatterns): boolean {
    return this.validationPatterns[pattern].test(input);
  }
  
  // Rate limiting for API calls
  private requestCounts = new Map<string, RateLimitInfo>();
  
  checkRateLimit(identifier: string, limit: number = 100, windowMs: number = 60000): boolean {
    const now = Date.now();
    const windowStart = now - windowMs;
    
    let rateLimitInfo = this.requestCounts.get(identifier);
    
    if (!rateLimitInfo) {
      rateLimitInfo = { count: 0, windowStart: now };
      this.requestCounts.set(identifier, rateLimitInfo);
    }
    
    // Reset window if expired
    if (rateLimitInfo.windowStart < windowStart) {
      rateLimitInfo.count = 0;
      rateLimitInfo.windowStart = now;
    }
    
    rateLimitInfo.count++;
    
    return rateLimitInfo.count <= limit;
  }
  
  // Session security
  validateSession(sessionToken: string): SessionValidation {
    if (!sessionToken) return { valid: false, reason: 'No session token' };
    
    try {
      // Decode JWT token (simplified - use proper JWT library)
      const parts = sessionToken.split('.');
      if (parts.length !== 3) return { valid: false, reason: 'Invalid token format' };
      
      const payload = JSON.parse(atob(parts[1]));
      const now = Math.floor(Date.now() / 1000);
      
      // Check expiration
      if (payload.exp && payload.exp < now) {
        return { valid: false, reason: 'Token expired' };
      }
      
      // Check issued time
      if (payload.iat && payload.iat > now) {
        return { valid: false, reason: 'Token issued in future' };
      }
      
      return { valid: true, payload };
    } catch (error) {
      return { valid: false, reason: 'Token decode error' };
    }
  }
  
  private setupCSPViolationReporting() {
    // Listen for CSP violations
    this.document.addEventListener('securitypolicyviolation', (event) => {
      const violation: CSPViolation = {
        blockedURI: event.blockedURI,
        documentURI: event.documentURI,
        effectiveDirective: event.effectiveDirective,
        originalPolicy: event.originalPolicy,
        referrer: event.referrer,
        statusCode: event.statusCode,
        violatedDirective: event.violatedDirective,
        timestamp: new Date()
      };
      
      this.cspViolations.push(violation);
      this.reportCSPViolation(violation);
    });
  }
  
  private reportCSPViolation(violation: CSPViolation) {
    console.warn('CSP Violation detected:', violation);
    
    // Report to monitoring service
    // this.http.post('/api/security/csp-violation', violation).subscribe();
  }
  
  private isDomainTrusted(domain: string): boolean {
    return this.trustedDomains.has(domain);
  }
  
  private isInternalDomain(domain: string): boolean {
    const currentDomain = window.location.hostname;
    return domain === currentDomain || domain.endsWith(`.${currentDomain}`);
  }
  
  getSecurityReport(): SecurityReport {
    return {
      cspViolations: this.cspViolations.length,
      recentViolations: this.cspViolations.slice(-10),
      rateLimitHits: this.requestCounts.size,
      trustedDomains: Array.from(this.trustedDomains)
    };
  }
}

// Security-focused HTTP interceptor
@Injectable()
export class SecurityInterceptor implements HttpInterceptor {
  constructor(private security: SecurityService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Add security headers
    const secureRequest = req.clone({
      setHeaders: {
        'X-Content-Type-Options': 'nosniff',
        'X-Frame-Options': 'DENY',
        'X-XSS-Protection': '1; mode=block',
        'Referrer-Policy': 'strict-origin-when-cross-origin'
      }
    });
    
    // Validate request URL
    const urlValidation = this.security.validateURL(secureRequest.url);
    if (!urlValidation.valid) {
      return throwError(() => new Error(`Unsafe URL: ${urlValidation.reason}`));
    }
    
    // Check rate limiting for the request
    const identifier = this.getRequestIdentifier(secureRequest);
    if (!this.security.checkRateLimit(identifier, 1000, 60000)) {
      return throwError(() => new Error('Rate limit exceeded'));
    }
    
    return next.handle(secureRequest).pipe(
      catchError((error: HttpErrorResponse) => {
        // Log security-related errors
        if (error.status === 401 || error.status === 403) {
          console.warn('Security-related HTTP error:', error);
        }
        
        return throwError(() => error);
      })
    );
  }
  
  private getRequestIdentifier(req: HttpRequest<any>): string {
    // Use IP or user ID in real application
    return req.url + '_' + (req.headers.get('User-ID') || 'anonymous');
  }
}

// Secure component for handling user input
@Component({
  selector: 'secure-input',
  template: `
    <div class="secure-input">
      <label>{{label}}</label>
      
      <input
        #inputEl
        [type]="inputType"
        [(ngModel)]="inputValue"
        (blur)="validateInput()"
        (input)="onInput($event)"
        [class.invalid]="!isValid"
        [maxlength]="maxLength">
      
      <div class="validation-feedback" *ngIf="!isValid">
        {{validationMessage}}
      </div>
      
      <div class="security-info" *ngIf="showSecurityInfo">
        <small>Input is automatically sanitized for security</small>
      </div>
    </div>
  `,
  styles: [`
    .invalid { border-color: #dc3545; }
    .validation-feedback { color: #dc3545; font-size: 0.875rem; }
    .security-info { color: #6c757d; font-size: 0.75rem; margin-top: 4px; }
  `]
})
export class SecureInputComponent implements OnInit {
  @Input() label = '';
  @Input() inputType = 'text';
  @Input() maxLength = 255;
  @Input() validationType: 'email' | 'phone' | 'alphanumeric' | 'noScript' | 'sqlSafe' = 'noScript';
  @Input() showSecurityInfo = true;
  @Input() value = '';
  @Output() valueChange = new EventEmitter<string>();
  @Output() validationChange = new EventEmitter<boolean>();
  
  inputValue = '';
  isValid = true;
  validationMessage = '';
  
  constructor(private security: SecurityService) {}
  
  ngOnInit() {
    this.inputValue = this.value;
    this.validateInput();
  }
  
  onInput(event: Event) {
    const target = event.target as HTMLInputElement;
    const rawValue = target.value;
    
    // Sanitize input in real-time
    const sanitizedValue = this.security.sanitizeInput(rawValue, SecurityContext.NONE);
    
    if (sanitizedValue !== rawValue) {
      target.value = sanitizedValue;
      this.inputValue = sanitizedValue;
    }
    
    this.validateInput();
  }
  
  validateInput() {
    this.isValid = true;
    this.validationMessage = '';
    
    if (!this.inputValue) {
      this.emitChanges();
      return;
    }
    
    // Length validation
    if (this.inputValue.length > this.maxLength) {
      this.isValid = false;
      this.validationMessage = `Maximum length is ${this.maxLength} characters`;
      this.emitChanges();
      return;
    }
    
    // Pattern validation
    if (!this.security.validateInput(this.inputValue, this.validationType)) {
      this.isValid = false;
      this.validationMessage = this.getValidationMessage();
      this.emitChanges();
      return;
    }
    
    this.emitChanges();
  }
  
  private getValidationMessage(): string {
    const messages = {
      email: 'Please enter a valid email address',
      phone: 'Please enter a valid phone number',
      alphanumeric: 'Only letters and numbers are allowed',
      noScript: 'Script tags are not allowed',
      sqlSafe: 'SQL keywords are not allowed'
    };
    
    return messages[this.validationType] || 'Invalid input';
  }
  
  private emitChanges() {
    this.valueChange.emit(this.inputValue);
    this.validationChange.emit(this.isValid);
  }
}

// Supporting interfaces
interface ValidationResult {
  valid: boolean;
  reason?: string;
  requiresWarning?: boolean;
  domain?: string;
}

interface CSPOptions {
  [directive: string]: string[];
}

interface CSPViolation {
  blockedURI: string;
  documentURI: string;
  effectiveDirective: string;
  originalPolicy: string;
  referrer: string;
  statusCode: number;
  violatedDirective: string;
  timestamp: Date;
}

interface RateLimitInfo {
  count: number;
  windowStart: number;
}

interface SessionValidation {
  valid: boolean;
  reason?: string;
  payload?: any;
}

interface SecurityReport {
  cspViolations: number;
  recentViolations: CSPViolation[];
  rateLimitHits: number;
  trustedDomains: string[];
}
```


### Q40: Advanced Authentication and Authorization patterns

```typescript
// Advanced authentication service with JWT, refresh tokens, and RBAC
@Injectable({ providedIn: 'root' })
export class AuthenticationService {
  private readonly TOKEN_KEY = 'auth_token';
  private readonly REFRESH_TOKEN_KEY = 'refresh_token';
  private readonly USER_KEY = 'user_profile';
  
  private authState$ = new BehaviorSubject<AuthState>({
    isAuthenticated: false,
    user: null,
    permissions: [],
    loading: true
  });
  
  private tokenRefreshSubject$ = new BehaviorSubject<string | null>(null);
  private refreshTokenInProgress = false;
  
  public readonly authState = this.authState$.asObservable();
  public readonly isAuthenticated$ = this.authState.pipe(
    map(state => state.isAuthenticated),
    distinctUntilChanged()
  );
  
  constructor(
    private http: HttpClient,
    private router: Router,
    @Inject(DOCUMENT) private document: Document
  ) {
    this.initializeAuth();
  }
  
  private async initializeAuth() {
    const token = this.getStoredToken();
    const refreshToken = this.getStoredRefreshToken();
    
    if (token && refreshToken) {
      try {
        const isValid = await this.validateToken(token);
        if (isValid) {
          await this.loadUserProfile();
        } else {
          await this.refreshTokens();
        }
      } catch (error) {
        console.error('Auth initialization error:', error);
        this.logout();
      }
    } else {
      this.updateAuthState({ 
        isAuthenticated: false, 
        user: null, 
        permissions: [], 
        loading: false 
      });
    }
  }
  
  // Multi-factor authentication
  async login(credentials: LoginCredentials): Promise<AuthResult> {
    try {
      this.updateAuthState({ loading: true });
      
      const response = await this.http.post<LoginResponse>('/api/auth/login', credentials)
        .toPromise();
      
      if (!response) throw new Error('No response from server');
      
      // Handle MFA requirement
      if (response.requiresMFA) {
        return {
          success: false,
          requiresMFA: true,
          mfaToken: response.mfaToken,
          mfaMethods: response.availableMfaMethods
        };
      }
      
      await this.handleSuccessfulAuth(response);
      return { success: true };
      
    } catch (error: any) {
      this.updateAuthState({ loading: false });
      return { 
        success: false, 
        error: error.message || 'Login failed' 
      };
    }
  }
  
  // Complete MFA process
  async completeMFA(mfaToken: string, mfaCode: string): Promise<AuthResult> {
    try {
      const response = await this.http.post<LoginResponse>('/api/auth/mfa/verify', {
        mfaToken,
        code: mfaCode
      }).toPromise();
      
      if (!response) throw new Error('MFA verification failed');
      
      await this.handleSuccessfulAuth(response);
      return { success: true };
      
    } catch (error: any) {
      return { 
        success: false, 
        error: error.message || 'MFA verification failed' 
      };
    }
  }
  
  private async handleSuccessfulAuth(response: LoginResponse) {
    // Store tokens securely
    this.storeTokens(response.accessToken, response.refreshToken);
    
    // Load user profile and permissions
    await this.loadUserProfile();
    
    // Setup automatic token refresh
    this.setupTokenRefresh(response.accessToken);
    
    // Update auth state
    this.updateAuthState({
      isAuthenticated: true,
      loading: false
    });
  }
  
  private async loadUserProfile() {
    try {
      const user = await this.http.get<UserProfile>('/api/auth/profile').toPromise();
      const permissions = await this.loadUserPermissions(user!.id);
      
      this.storeUserProfile(user!);
      this.updateAuthState({ user, permissions });
      
    } catch (error) {
      console.error('Failed to load user profile:', error);
      throw error;
    }
  }
  
  private async loadUserPermissions(userId: number): Promise<Permission[]> {
    try {
      const permissions = await this.http.get<Permission[]>(`/api/auth/permissions/${userId}`)
        .toPromise();
      return permissions || [];
    } catch (error) {
      console.error('Failed to load permissions:', error);
      return [];
    }
  }
  
  // Token refresh with queue management
  refreshTokens(): Observable<string> {
    if (this.refreshTokenInProgress) {
      return this.tokenRefreshSubject$.pipe(
        filter(token => token !== null),
        take(1)
      );
    }
    
    this.refreshTokenInProgress = true;
    
    const refreshToken = this.getStoredRefreshToken();
    if (!refreshToken) {
      this.logout();
      return throwError(() => new Error('No refresh token available'));
    }
    
    return this.http.post<TokenResponse>('/api/auth/refresh', { refreshToken }).pipe(
      tap(response => {
        this.storeTokens(response.accessToken, response.refreshToken);
        this.setupTokenRefresh(response.accessToken);
        this.tokenRefreshSubject$.next(response.accessToken);
        this.refreshTokenInProgress = false;
      }),
      map(response => response.accessToken),
      catchError(error => {
        this.refreshTokenInProgress = false;
        this.tokenRefreshSubject$.next(null);
        this.logout();
        return throwError(() => error);
      })
    );
  }
  
  private setupTokenRefresh(token: string) {
    try {
      const payload = this.decodeToken(token);
      const expirationTime = payload.exp * 1000; // Convert to milliseconds
      const currentTime = Date.now();
      const refreshTime = expirationTime - (5 * 60 * 1000); // 5 minutes before expiry
      
      if (refreshTime > currentTime) {
        setTimeout(() => {
          this.refreshTokens().subscribe({
            error: (error) => console.error('Auto refresh failed:', error)
          });
        }, refreshTime - currentTime);
      }
    } catch (error) {
      console.error('Token refresh setup error:', error);
    }
  }
  
  logout() {
    // Clear storage
    this.clearTokens();
    this.clearUserProfile();
    
    // Update state
    this.updateAuthState({
      isAuthenticated: false,
      user: null,
      permissions: [],
      loading: false
    });
    
    // Redirect to login
    this.router.navigate(['/login']);
  }
  
  // Permission checking methods
  hasPermission(permission: string): boolean {
    const currentState = this.authState$.value;
    return currentState.permissions.some(p => p.name === permission);
  }
  
  hasAnyPermission(permissions: string[]): boolean {
    return permissions.some(permission => this.hasPermission(permission));
  }
  
  hasAllPermissions(permissions: string[]): boolean {
    return permissions.every(permission => this.hasPermission(permission));
  }
  
  hasRole(role: string): boolean {
    const currentState = this.authState$.value;
    return currentState.user?.roles.includes(role) || false;
  }
  
  canAccessResource(resource: string, action: string): boolean {
    const currentState = this.authState$.value;
    return currentState.permissions.some(p => 
      p.resource === resource && p.actions.includes(action)
    );
  }
  
  // Token management
  getToken(): string | null {
    return this.getStoredToken();
  }
  
  private validateToken(token: string): Promise<boolean> {
    return this.http.post<{valid: boolean}>('/api/auth/validate', { token })
      .toPromise()
      .then(response => response?.valid || false)
      .catch(() => false);
  }
  
  private decodeToken(token: string): any {
    try {
      const payload = token.split('.')[1];
      return JSON.parse(atob(payload));
    } catch (error) {
      throw new Error('Invalid token format');
    }
  }
  
  // Storage methods (consider encryption for sensitive data)
  private storeTokens(accessToken: string, refreshToken: string) {
    localStorage.setItem(this.TOKEN_KEY, accessToken);
    localStorage.setItem(this.REFRESH_TOKEN_KEY, refreshToken);
  }
  
  private getStoredToken(): string | null {
    return localStorage.getItem(this.TOKEN_KEY);
  }
  
  private getStoredRefreshToken(): string | null {
    return localStorage.getItem(this.REFRESH_TOKEN_KEY);
  }
  
  private storeUserProfile(user: UserProfile) {
    localStorage.setItem(this.USER_KEY, JSON.stringify(user));
  }
  
  private clearTokens() {
    localStorage.removeItem(this.TOKEN_KEY);
    localStorage.removeItem(this.REFRESH_TOKEN_KEY);
  }
  
  private clearUserProfile() {
    localStorage.removeItem(this.USER_KEY);
  }
  
  private updateAuthState(updates: Partial<AuthState>) {
    const currentState = this.authState$.value;
    this.authState$.next({ ...currentState, ...updates });
  }
}

// Authorization guard with advanced role-based access control
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate, CanActivateChild {
  constructor(
    private auth: AuthenticationService,
    private router: Router
  ) {}
  
  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> {
    return this.checkAccess(route, state);
  }
  
  canActivateChild(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> {
    return this.checkAccess(route, state);
  }
  
  private checkAccess(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> {
    return this.auth.authState.pipe(
      filter(authState => !authState.loading), // Wait for auth to initialize
      take(1),
      map(authState => {
        if (!authState.isAuthenticated) {
          this.router.navigate(['/login'], { 
            queryParams: { returnUrl: state.url } 
          });
          return false;
        }
        
        // Check route-specific permissions
        const requiredPermissions = route.data['permissions'] as string[];
        const requiredRoles = route.data['roles'] as string[];
        const requireAll = route.data['requireAll'] as boolean || false;
        
        if (requiredPermissions && requiredPermissions.length > 0) {
          const hasPermission = requireAll 
            ? this.auth.hasAllPermissions(requiredPermissions)
            : this.auth.hasAnyPermission(requiredPermissions);
            
          if (!hasPermission) {
            this.router.navigate(['/access-denied']);
            return false;
          }
        }
        
        if (requiredRoles && requiredRoles.length > 0) {
          const hasRole = requiredRoles.some(role => this.auth.hasRole(role));
          if (!hasRole) {
            this.router.navigate(['/access-denied']);
            return false;
          }
        }
        
        return true;
      })
    );
  }
}

// HTTP interceptor for automatic token handling
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private auth: AuthenticationService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Skip token for login/refresh endpoints
    if (this.isAuthEndpoint(req.url)) {
      return next.handle(req);
    }
    
    const token = this.auth.getToken();
    
    if (token) {
      req = this.addToken(req, token);
    }
    
    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        // Handle 401 errors with token refresh
        if (error.status === 401 && token) {
          return this.handle401Error(req, next);
        }
        
        return throwError(() => error);
      })
    );
  }
  
  private handle401Error(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return this.auth.refreshTokens().pipe(
      switchMap(newToken => {
        const newRequest = this.addToken(req, newToken);
        return next.handle(newRequest);
      }),
      catchError(error => {
        this.auth.logout();
        return throwError(() => error);
      })
    );
  }
  
  private addToken(req: HttpRequest<any>, token: string): HttpRequest<any> {
    return req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }
  
  private isAuthEndpoint(url: string): boolean {
    const authEndpoints = ['/api/auth/login', '/api/auth/refresh', '/api/auth/register'];
    return authEndpoints.some(endpoint => url.includes(endpoint));
  }
}

// Permission-based directive
@Directive({
  selector: '[hasPermission]'
})
export class HasPermissionDirective implements OnInit, OnDestroy {
  @Input() hasPermission: string | string[] = '';
  @Input() requireAll = false;
  
  private subscription?: Subscription;
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef,
    private auth: AuthenticationService
  ) {}
  
  ngOnInit() {
    this.subscription = this.auth.authState.subscribe(authState => {
      this.updateView(authState);
    });
  }
  
  ngOnDestroy() {
    this.subscription?.unsubscribe();
  }
  
  private updateView(authState: AuthState) {
    const permissions = Array.isArray(this.hasPermission) 
      ? this.hasPermission 
      : [this.hasPermission];
      
    let hasAccess = false;
    
    if (authState.isAuthenticated) {
      hasAccess = this.requireAll
        ? this.auth.hasAllPermissions(permissions)
        : this.auth.hasAnyPermission(permissions);
    }
    
    if (hasAccess) {
      this.viewContainer.createEmbeddedView(this.templateRef);
    } else {
      this.viewContainer.clear();
    }
  }
}

// Supporting interfaces
interface AuthState {
  isAuthenticated: boolean;
  user: UserProfile | null;
  permissions: Permission[];
  loading: boolean;
}

interface LoginCredentials {
  username: string;
  password: string;
  rememberMe?: boolean;
}

interface LoginResponse {
  accessToken: string;
  refreshToken: string;
  requiresMFA?: boolean;
  mfaToken?: string;
  availableMfaMethods?: string[];
}

interface TokenResponse {
  accessToken: string;
  refreshToken: string;
}

interface UserProfile {
  id: number;
  username: string;
  email: string;
  firstName: string;
  lastName: string;
  roles: string[];
  isActive: boolean;
}

interface Permission {
  id: number;
  name: string;
  resource: string;
  actions: string[];
}

interface AuthResult {
  success: boolean;
  error?: string;
  requiresMFA?: boolean;
  mfaToken?: string;
  mfaMethods?: string[];
}
```

This comprehensive collection continues the advanced Angular interview questions, covering modern features, complex RxJS patterns, performance optimization, and security best practices. Each question includes detailed explanations and practical code examples that demonstrate deep understanding of Angular's capabilities and real-world application patterns.

