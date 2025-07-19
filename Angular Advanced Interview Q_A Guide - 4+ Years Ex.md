<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# Angular Advanced Interview Q/A Guide - 4+ Years Experience

This comprehensive guide covers advanced Angular interview questions with detailed explanations, output-based scenarios, and machine coding challenges specifically designed for frontend developers with 4+ years of experience.

## Advanced Angular Concepts

### Component Architecture \& Lifecycle

**Q1: Explain the complete Angular component lifecycle with practical use cases.**

**Answer:**
Angular components follow a specific lifecycle from creation to destruction:

```typescript
export class MyComponent implements OnInit, AfterViewInit, OnDestroy {
  @Input() data: any;
  
  constructor() {
    // Called first, dependency injection happens here
    console.log('Constructor called');
  }
  
  ngOnChanges(changes: SimpleChanges) {
    // Called when @Input properties change
    console.log('Input changes:', changes);
  }
  
  ngOnInit() {
    // Called after constructor and first ngOnChanges
    // Best place for initialization logic
    this.loadData();
  }
  
  ngAfterViewInit() {
    // Called after view initialization
    // Safe to access ViewChild elements
    this.setupViewRelatedLogic();
  }
  
  ngOnDestroy() {
    // Cleanup subscriptions and resources
    this.subscription?.unsubscribe();
  }
}
```

**Key Points:**

- **Constructor**: Dependency injection, avoid heavy logic
- **ngOnInit**: Initialization logic, HTTP calls, form setup
- **ngAfterViewInit**: DOM manipulation, ViewChild access
- **ngOnDestroy**: Memory leak prevention, cleanup

**Q2: What will be the output of this component interaction?**

```typescript
// Parent Component
@Component({
  template: `
    <child-component [counter]="parentCounter" (counterChange)="onCounterChange($event)">
    </child-component>
    <p>Parent counter: {{parentCounter}}</p>
  `
})
export class ParentComponent {
  parentCounter = 0;
  
  onCounterChange(value: number) {
    this.parentCounter = value;
  }
}

// Child Component
@Component({
  template: `
    <button (click)="increment()">Increment</button>
    <p>Child counter: {{counter}}</p>
  `
})
export class ChildComponent {
  @Input() counter = 0;
  @Output() counterChange = new EventEmitter<number>();
  
  increment() {
    this.counter++;
    this.counterChange.emit(this.counter);
  }
}
```

**Output Explanation:**

- Initial state: Both parent and child show 0
- After clicking increment: Child shows 1, Parent shows 1
- This demonstrates proper two-way data binding pattern
- The child maintains its own state while notifying parent of changes


### Change Detection Deep Dive

**Q3: Explain Angular's change detection mechanism and OnPush strategy.**

**Answer:**
Angular's change detection runs after specific events:

```typescript
// Default Change Detection
@Component({
  selector: 'default-component',
  template: `
    <p>{{expensiveCalculation()}}</p>
    <child-component [data]="childData"></child-component>
  `
})
export class DefaultComponent {
  childData = { count: 0 };
  
  expensiveCalculation() {
    console.log('Expensive calculation running...');
    return Math.random();
  }
}

// OnPush Strategy
@Component({
  selector: 'optimized-component',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <p>{{count}}</p>
    <button (click)="increment()">Increment</button>
  `
})
export class OptimizedComponent {
  count = 0;
  
  constructor(private cdr: ChangeDetectorRef) {}
  
  increment() {
    this.count++;
    this.cdr.markForCheck(); // Manual trigger needed
  }
}
```

**Key Differences:**

- **Default**: Checks every component on every change detection cycle
- **OnPush**: Only checks when:
    - Input references change
    - Event occurs in component
    - Async pipe receives new value
    - Manual trigger via ChangeDetectorRef


### Reactive Forms Advanced Scenarios

**Q4: Create a dynamic form with custom validators and cross-field validation.**

```typescript
@Component({
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <div formArrayName="addresses">
        <div *ngFor="let address of addresses.controls; let i = index" [formGroupName]="i">
          <input formControlName="street" placeholder="Street">
          <input formControlName="city" placeholder="City">
          <button (click)="removeAddress(i)">Remove</button>
        </div>
      </div>
      <button (click)="addAddress()">Add Address</button>
      
      <input formControlName="email" placeholder="Email">
      <input formControlName="confirmEmail" placeholder="Confirm Email">
      
      <div *ngIf="userForm.errors?.['emailMismatch']" class="error">
        Emails don't match
      </div>
    </form>
  `
})
export class DynamicFormComponent implements OnInit {
  userForm!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.userForm = this.fb.group({
      addresses: this.fb.array([this.createAddress()]),
      email: ['', [Validators.required, Validators.email]],
      confirmEmail: ['', Validators.required]
    }, { validators: this.emailMatchValidator });
  }
  
  get addresses() {
    return this.userForm.get('addresses') as FormArray;
  }
  
  createAddress(): FormGroup {
    return this.fb.group({
      street: ['', [Validators.required, this.noSpecialCharsValidator]],
      city: ['', Validators.required]
    });
  }
  
  addAddress() {
    this.addresses.push(this.createAddress());
  }
  
  removeAddress(index: number) {
    this.addresses.removeAt(index);
  }
  
  // Custom Validator
  noSpecialCharsValidator(control: AbstractControl): ValidationErrors | null {
    const hasSpecialChars = /[!@#$%^&*(),.?":{}|<>]/.test(control.value);
    return hasSpecialChars ? { specialChars: true } : null;
  }
  
  // Cross-field Validator
  emailMatchValidator(group: AbstractControl): ValidationErrors | null {
    const email = group.get('email')?.value;
    const confirmEmail = group.get('confirmEmail')?.value;
    return email === confirmEmail ? null : { emailMismatch: true };
  }
}
```


## Machine Coding Questions

### Q5: Build a Custom Autocomplete Component

```typescript
// Autocomplete Component
@Component({
  selector: 'app-autocomplete',
  template: `
    <div class="autocomplete-container">
      <input 
        #searchInput
        [value]="displayValue"
        (input)="onInputChange($event)"
        (focus)="onFocus()"
        (blur)="onBlur()"
        (keydown)="onKeyDown($event)"
        placeholder="Search...">
      
      <ul *ngIf="showSuggestions && filteredItems.length > 0" class="suggestions">
        <li 
          *ngFor="let item of filteredItems; let i = index"
          [class.active]="i === activeIndex"
          (click)="selectItem(item)"
          (mouseenter)="activeIndex = i">
          {{displayFn(item)}}
        </li>
      </ul>
    </div>
  `,
  styleUrls: ['./autocomplete.component.css']
})
export class AutocompleteComponent implements OnInit, OnDestroy, ControlValueAccessor {
  @Input() items: any[] = [];
  @Input() displayFn: (item: any) => string = (item) => item;
  @Input() filterFn: (items: any[], query: string) => any[] = this.defaultFilter;
  @Input() minLength = 1;
  @Input() debounceTime = 300;
  
  displayValue = '';
  filteredItems: any[] = [];
  showSuggestions = false;
  activeIndex = -1;
  
  private searchSubject = new Subject<string>();
  private subscription = new Subscription();
  
  // ControlValueAccessor implementation
  private onChange = (value: any) => {};
  private onTouched = () => {};
  
  ngOnInit() {
    this.setupSearch();
  }
  
  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
  
  private setupSearch() {
    this.subscription.add(
      this.searchSubject.pipe(
        debounceTime(this.debounceTime),
        distinctUntilChanged(),
        switchMap(query => {
          if (query.length >= this.minLength) {
            return of(this.filterFn(this.items, query));
          }
          return of([]);
        })
      ).subscribe(results => {
        this.filteredItems = results;
        this.showSuggestions = results.length > 0;
        this.activeIndex = -1;
      })
    );
  }
  
  onInputChange(event: Event) {
    const value = (event.target as HTMLInputElement).value;
    this.displayValue = value;
    this.searchSubject.next(value);
  }
  
  onKeyDown(event: KeyboardEvent) {
    switch (event.key) {
      case 'ArrowDown':
        event.preventDefault();
        this.activeIndex = Math.min(this.activeIndex + 1, this.filteredItems.length - 1);
        break;
      case 'ArrowUp':
        event.preventDefault();
        this.activeIndex = Math.max(this.activeIndex - 1, -1);
        break;
      case 'Enter':
        if (this.activeIndex >= 0 && this.filteredItems[this.activeIndex]) {
          this.selectItem(this.filteredItems[this.activeIndex]);
        }
        break;
      case 'Escape':
        this.showSuggestions = false;
        break;
    }
  }
  
  selectItem(item: any) {
    this.displayValue = this.displayFn(item);
    this.showSuggestions = false;
    this.onChange(item);
    this.onTouched();
  }
  
  private defaultFilter(items: any[], query: string): any[] {
    return items.filter(item => 
      this.displayFn(item).toLowerCase().includes(query.toLowerCase())
    );
  }
  
  // ControlValueAccessor methods
  writeValue(value: any): void {
    this.displayValue = value ? this.displayFn(value) : '';
  }
  
  registerOnChange(fn: any): void {
    this.onChange = fn;
  }
  
  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
}
```


### Q6: Implement a Virtual Scrolling List

```typescript
@Component({
  selector: 'virtual-scroll-list',
  template: `
    <div class="viewport" #viewport (scroll)="onScroll()" [style.height.px]="viewportHeight">
      <div class="spacer" [style.height.px]="totalHeight"></div>
      <div class="content" [style.transform]="'translateY(' + startIndex * itemHeight + 'px)'">
        <div 
          *ngFor="let item of visibleItems; trackBy: trackByFn" 
          class="list-item"
          [style.height.px]="itemHeight">
          <ng-container *ngTemplateOutlet="itemTemplate; context: { $implicit: item }"></ng-container>
        </div>
      </div>
    </div>
  `
})
export class VirtualScrollListComponent implements OnInit {
  @Input() items: any[] = [];
  @Input() itemHeight = 50;
  @Input() viewportHeight = 400;
  @Input() itemTemplate!: TemplateRef<any>;
  @Input() trackByFn: TrackByFunction<any> = (index, item) => item.id || index;
  
  @ViewChild('viewport', { static: true }) viewport!: ElementRef;
  
  visibleItems: any[] = [];
  startIndex = 0;
  endIndex = 0;
  totalHeight = 0;
  
  private itemsPerScreen = 0;
  private buffer = 5;
  
  ngOnInit() {
    this.calculateDimensions();
    this.updateVisibleItems();
  }
  
  @HostListener('window:resize')
  onResize() {
    this.calculateDimensions();
    this.updateVisibleItems();
  }
  
  private calculateDimensions() {
    this.totalHeight = this.items.length * this.itemHeight;
    this.itemsPerScreen = Math.ceil(this.viewportHeight / this.itemHeight);
  }
  
  onScroll() {
    const scrollTop = this.viewport.nativeElement.scrollTop;
    const newStartIndex = Math.floor(scrollTop / this.itemHeight);
    
    if (newStartIndex !== this.startIndex) {
      this.startIndex = Math.max(0, newStartIndex - this.buffer);
      this.updateVisibleItems();
    }
  }
  
  private updateVisibleItems() {
    this.endIndex = Math.min(
      this.items.length,
      this.startIndex + this.itemsPerScreen + (this.buffer * 2)
    );
    
    this.visibleItems = this.items.slice(this.startIndex, this.endIndex);
  }
}

// Usage Example
@Component({
  template: `
    <virtual-scroll-list 
      [items]="largeDataset" 
      [itemHeight]="60"
      [viewportHeight]="500"
      [itemTemplate]="itemTpl">
    </virtual-scroll-list>
    
    <ng-template #itemTpl let-item>
      <div class="user-item">
        <img [src]="item.avatar" alt="avatar">
        <span>{{item.name}}</span>
      </div>
    </ng-template>
  `
})
export class AppComponent {
  largeDataset = Array.from({length: 10000}, (_, i) => ({
    id: i,
    name: `User ${i}`,
    avatar: `https://api.dicebear.com/7.x/avataaars/svg?seed=${i}`
  }));
}
```


## Advanced RxJS and State Management

### Q7: What will be the output of this RxJS chain?

```typescript
import { of, interval, merge, combineLatest } from 'rxjs';
import { map, filter, take, delay, switchMap, startWith } from 'rxjs/operators';

const source1$ = interval(1000).pipe(take(3)); // 0, 1, 2
const source2$ = of('A', 'B', 'C').pipe(delay(500));

// Question: What will be emitted and when?
const result$ = merge(
  source1$.pipe(map(x => `Number: ${x}`)),
  source2$.pipe(map(x => `Letter: ${x}`))
).pipe(
  filter(value => !value.includes('1')),
  take(4)
);

result$.subscribe(console.log);
```

**Answer \& Timeline:**

- **0.5s**: "Letter: A", "Letter: B", "Letter: C" (all at once due to delay)
- **1s**: "Number: 0"
- **2s**: (Nothing - "Number: 1" filtered out)
- **3s**: "Number: 2"

**Final Output:** 4 values total, then completes due to `take(4)`

### Q8: Implement Advanced State Management Pattern

```typescript
// State Management Service
export interface AppState {
  users: User[];
  loading: boolean;
  error: string | null;
  filters: UserFilters;
}

interface UserFilters {
  searchTerm: string;
  department: string;
  active: boolean;
}

@Injectable({ providedIn: 'root' })
export class UserStateService {
  private stateSubject = new BehaviorSubject<AppState>({
    users: [],
    loading: false,
    error: null,
    filters: { searchTerm: '', department: '', active: true }
  });
  
  public state$ = this.stateSubject.asObservable();
  
  // Selectors
  public users$ = this.state$.pipe(
    map(state => state.users),
    distinctUntilChanged()
  );
  
  public filteredUsers$ = combineLatest([
    this.users$,
    this.state$.pipe(map(state => state.filters))
  ]).pipe(
    map(([users, filters]) => this.applyFilters(users, filters))
  );
  
  public loading$ = this.state$.pipe(
    map(state => state.loading),
    distinctUntilChanged()
  );
  
  public error$ = this.state$.pipe(
    map(state => state.error),
    distinctUntilChanged()
  );
  
  constructor(private http: HttpClient) {}
  
  // Actions
  loadUsers(): Observable<User[]> {
    this.updateState({ loading: true, error: null });
    
    return this.http.get<User[]>('/api/users').pipe(
      tap(users => this.updateState({ users, loading: false })),
      catchError(error => {
        this.updateState({ loading: false, error: error.message });
        return throwError(error);
      })
    );
  }
  
  updateFilters(filters: Partial<UserFilters>) {
    const currentFilters = this.stateSubject.value.filters;
    this.updateState({ 
      filters: { ...currentFilters, ...filters }
    });
  }
  
  addUser(user: User) {
    const currentUsers = this.stateSubject.value.users;
    this.updateState({ 
      users: [...currentUsers, { ...user, id: Date.now() }]
    });
  }
  
  updateUser(id: number, updates: Partial<User>) {
    const currentUsers = this.stateSubject.value.users;
    const updatedUsers = currentUsers.map(user => 
      user.id === id ? { ...user, ...updates } : user
    );
    this.updateState({ users: updatedUsers });
  }
  
  deleteUser(id: number) {
    const currentUsers = this.stateSubject.value.users;
    const filteredUsers = currentUsers.filter(user => user.id !== id);
    this.updateState({ users: filteredUsers });
  }
  
  private updateState(partialState: Partial<AppState>) {
    const currentState = this.stateSubject.value;
    this.stateSubject.next({ ...currentState, ...partialState });
  }
  
  private applyFilters(users: User[], filters: UserFilters): User[] {
    return users.filter(user => {
      const matchesSearch = !filters.searchTerm || 
        user.name.toLowerCase().includes(filters.searchTerm.toLowerCase());
      const matchesDepartment = !filters.department || 
        user.department === filters.department;
      const matchesActive = user.active === filters.active;
      
      return matchesSearch && matchesDepartment && matchesActive;
    });
  }
}

// Component Usage
@Component({
  template: `
    <div class="user-management">
      <div class="filters">
        <input 
          [value]="(userState.state$ | async)?.filters.searchTerm"
          (input)="onSearchChange($event)"
          placeholder="Search users...">
        
        <select 
          [value]="(userState.state$ | async)?.filters.department"
          (change)="onDepartmentChange($event)">
          <option value="">All Departments</option>
          <option value="IT">IT</option>
          <option value="HR">HR</option>
          <option value="Sales">Sales</option>
        </select>
      </div>
      
      <div *ngIf="userState.loading$ | async" class="loading">
        Loading users...
      </div>
      
      <div *ngIf="userState.error$ | async as error" class="error">
        {{error}}
      </div>
      
      <div class="user-list">
        <div 
          *ngFor="let user of userState.filteredUsers$ | async; trackBy: trackByUserId"
          class="user-card">
          <h3>{{user.name}}</h3>
          <p>{{user.department}}</p>
          <button (click)="toggleUserStatus(user)">
            {{user.active ? 'Deactivate' : 'Activate'}}
          </button>
          <button (click)="deleteUser(user.id)">Delete</button>
        </div>
      </div>
    </div>
  `
})
export class UserManagementComponent implements OnInit {
  constructor(public userState: UserStateService) {}
  
  ngOnInit() {
    this.userState.loadUsers().subscribe();
  }
  
  onSearchChange(event: Event) {
    const searchTerm = (event.target as HTMLInputElement).value;
    this.userState.updateFilters({ searchTerm });
  }
  
  onDepartmentChange(event: Event) {
    const department = (event.target as HTMLSelectElement).value;
    this.userState.updateFilters({ department });
  }
  
  toggleUserStatus(user: User) {
    this.userState.updateUser(user.id, { active: !user.active });
  }
  
  deleteUser(id: number) {
    this.userState.deleteUser(id);
  }
  
  trackByUserId(index: number, user: User): number {
    return user.id;
  }
}
```


## Performance Optimization

### Q9: Implement Lazy Loading with Preloading Strategy

```typescript
// Custom Preloading Strategy
@Injectable()
export class CustomPreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Preload routes marked with preload: true
    if (route.data && route.data['preload']) {
      console.log('Preloading:', route.path);
      return load();
    }
    return of(null);
  }
}

// Route Configuration
const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module').then(m => m.DashboardModule),
    data: { preload: true } // Will be preloaded
  },
  {
    path: 'users',
    loadChildren: () => import('./users/users.module').then(m => m.UsersModule),
    canLoad: [AuthGuard]
  },
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.module').then(m => m.ReportsModule)
    // Will not be preloaded
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes, {
    preloadingStrategy: CustomPreloadingStrategy,
    enableTracing: false // Set to true for debugging
  })],
  providers: [CustomPreloadingStrategy],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```


### Q10: Memory Leak Prevention Patterns

```typescript
@Component({
  template: `
    <div *ngIf="data$ | async as data">
      {{data.message}}
    </div>
    <button (click)="loadData()">Load Data</button>
  `
})
export class LeakPreventionComponent implements OnInit, OnDestroy {
  data$ = new BehaviorSubject<any>(null);
  
  private destroy$ = new Subject<void>();
  
  constructor(
    private http: HttpClient,
    private router: Router
  ) {}
  
  ngOnInit() {
    // Pattern 1: Using takeUntil
    this.router.events.pipe(
      filter(event => event instanceof NavigationEnd),
      takeUntil(this.destroy$)
    ).subscribe(event => {
      console.log('Navigation:', event);
    });
    
    // Pattern 2: Async pipe handles subscription automatically
    this.data$ = this.http.get('/api/data').pipe(
      retry(3),
      catchError(error => of({ message: 'Error loading data' }))
    );
  }
  
  ngOnDestroy() {
    // Trigger completion for all takeUntil operators
    this.destroy$.next();
    this.destroy$.complete();
  }
  
  loadData() {
    // Pattern 3: Single-shot operations don't need unsubscription
    this.http.get('/api/fresh-data').pipe(
      take(1)
    ).subscribe(data => {
      this.data$.next(data);
    });
  }
}

// Alternative using DestroyRef (Angular 16+)
@Component({
  template: `<div>{{message}}</div>`
})
export class ModernLeakPreventionComponent implements OnInit {
  message = '';
  
  constructor(
    private http: HttpClient,
    private destroyRef: DestroyRef
  ) {}
  
  ngOnInit() {
    interval(1000).pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe(count => {
      this.message = `Count: ${count}`;
    });
  }
}
```


## Testing Advanced Scenarios

### Q11: Complex Component Testing with Mocks

```typescript
describe('UserManagementComponent', () => {
  let component: UserManagementComponent;
  let fixture: ComponentFixture<UserManagementComponent>;
  let userService: jasmine.SpyObj<UserService>;
  let router: jasmine.SpyObj<Router>;
  
  const mockUsers: User[] = [
    { id: 1, name: 'John Doe', department: 'IT', active: true },
    { id: 2, name: 'Jane Smith', department: 'HR', active: false }
  ];
  
  beforeEach(async () => {
    const userServiceSpy = jasmine.createSpyObj('UserService', 
      ['getUsers', 'updateUser', 'deleteUser']);
    const routerSpy = jasmine.createSpyObj('Router', ['navigate']);
    
    await TestBed.configureTestingModule({
      declarations: [UserManagementComponent],
      imports: [HttpClientTestingModule, ReactiveFormsModule],
      providers: [
        { provide: UserService, useValue: userServiceSpy },
        { provide: Router, useValue: routerSpy }
      ]
    }).compileComponents();
    
    fixture = TestBed.createComponent(UserManagementComponent);
    component = fixture.componentInstance;
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
    router = TestBed.inject(Router) as jasmine.SpyObj<Router>;
  });
  
  it('should load and display users on init', fakeAsync(() => {
    // Arrange
    userService.getUsers.and.returnValue(of(mockUsers));
    
    // Act
    component.ngOnInit();
    tick();
    fixture.detectChanges();
    
    // Assert
    expect(userService.getUsers).toHaveBeenCalled();
    expect(component.users.length).toBe(2);
    
    const userElements = fixture.debugElement.queryAll(By.css('.user-card'));
    expect(userElements.length).toBe(2);
    expect(userElements[0].nativeElement.textContent).toContain('John Doe');
  }));
  
  it('should handle user update with error', fakeAsync(() => {
    // Arrange
    const errorMessage = 'Update failed';
    userService.updateUser.and.returnValue(throwError(errorMessage));
    component.users = [...mockUsers];
    
    // Act
    component.updateUser(1, { active: false });
    tick();
    fixture.detectChanges();
    
    // Assert
    expect(userService.updateUser).toHaveBeenCalledWith(1, { active: false });
    expect(component.errorMessage).toBe(errorMessage);
    
    const errorElement = fixture.debugElement.query(By.css('.error-message'));
    expect(errorElement.nativeElement.textContent).toContain(errorMessage);
  }));
  
  it('should filter users based on search input', fakeAsync(() => {
    // Arrange
    component.users = [...mockUsers];
    fixture.detectChanges();
    
    const searchInput = fixture.debugElement.query(By.css('input[name="search"]'));
    
    // Act
    searchInput.nativeElement.value = 'John';
    searchInput.nativeElement.dispatchEvent(new Event('input'));
    tick(300); // Debounce time
    fixture.detectChanges();
    
    // Assert
    const userElements = fixture.debugElement.queryAll(By.css('.user-card'));
    expect(userElements.length).toBe(1);
    expect(userElements[0].nativeElement.textContent).toContain('John Doe');
  }));
});

// Integration Test Example
describe('UserManagementComponent Integration', () => {
  let httpMock: HttpTestingController;
  
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [UserManagementComponent],
      imports: [HttpClientTestingModule],
      providers: [UserService]
    }).compileComponents();
    
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  afterEach(() => {
    httpMock.verify();
  });
  
  it('should make correct HTTP requests', () => {
    const fixture = TestBed.createComponent(UserManagementComponent);
    const component = fixture.componentInstance;
    
    component.ngOnInit();
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    
    req.flush(mockUsers);
    expect(component.users).toEqual(mockUsers);
  });
});
```


## Architecture and Best Practices

### Q12: Implement Feature Module with Barrel Exports

```typescript
// feature/user/models/index.ts
export interface User {
  id: number;
  name: string;
  email: string;
  department: string;
  active: boolean;
}

export interface UserFilter {
  searchTerm?: string;
  department?: string;
  active?: boolean;
}

// feature/user/services/user.service.ts
@Injectable()
export class UserService {
  private apiUrl = '/api/users';
  
  constructor(private http: HttpClient) {}
  
  getUsers(filter?: UserFilter): Observable<User[]> {
    let params = new HttpParams();
    
    if (filter) {
      Object.entries(filter).forEach(([key, value]) => {
        if (value !== undefined && value !== '') {
          params = params.set(key, value.toString());
        }
      });
    }
    
    return this.http.get<User[]>(this.apiUrl, { params }).pipe(
      retry(2),
      catchError(this.handleError)
    );
  }
  
  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`).pipe(
      catchError(this.handleError)
    );
  }
  
  createUser(user: Omit<User, 'id'>): Observable<User> {
    return this.http.post<User>(this.apiUrl, user).pipe(
      catchError(this.handleError)
    );
  }
  
  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user).pipe(
      catchError(this.handleError)
    );
  }
  
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`).pipe(
      catchError(this.handleError)
    );
  }
  
  private handleError = (error: HttpErrorResponse) => {
    let errorMessage = 'An unknown error occurred';
    
    if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = error.error.message;
    } else {
      // Server-side error
      switch (error.status) {
        case 400:
          errorMessage = 'Bad request';
          break;
        case 401:
          errorMessage = 'Unauthorized';
          break;
        case 404:
          errorMessage = 'User not found';
          break;
        case 500:
          errorMessage = 'Internal server error';
          break;
        default:
          errorMessage = `Error Code: ${error.status}`;
      }
    }
    
    return throwError(() => new Error(errorMessage));
  };
}

// feature/user/index.ts (Barrel Export)
export * from './models';
export * from './services/user.service';
export * from './components/user-list/user-list.component';
export * from './components/user-form/user-form.component';
export * from './user.module';
```


### Q13: Advanced Error Handling and Logging

```typescript
// Global Error Handler
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  constructor(
    @Inject(DOCUMENT) private document: Document,
    private logger: LoggingService,
    private notification: NotificationService
  ) {}
  
  handleError(error: any): void {
    console.error('Global error:', error);
    
    // Log the error
    this.logger.logError(error);
    
    // Determine error type and show appropriate message
    if (error instanceof HttpErrorResponse) {
      this.handleHttpError(error);
    } else if (error instanceof Error) {
      this.handleClientError(error);
    } else {
      this.handleUnknownError(error);
    }
  }
  
  private handleHttpError(error: HttpErrorResponse) {
    let message = 'A network error occurred';
    
    switch (error.status) {
      case 401:
        message = 'Your session has expired. Please log in again.';
        // Redirect to login
        this.document.location.href = '/login';
        return;
      case 403:
        message = 'You do not have permission to perform this action.';
        break;
      case 404:
        message = 'The requested resource was not found.';
        break;
      case 500:
        message = 'A server error occurred. Please try again later.';
        break;
    }
    
    this.notification.showError(message);
  }
  
  private handleClientError(error: Error) {
    const message = error.message || 'An unexpected error occurred.';
    this.notification.showError(message);
  }
  
  private handleUnknownError(error: any) {
    this.notification.showError('An unexpected error occurred.');
  }
}

// HTTP Error Interceptor
@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  constructor(private router: Router, private auth: AuthService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        if (error.status === 401) {
          this.auth.logout();
          this.router.navigate(['/login']);
        }
        
        return throwError(() => error);
      })
    );
  }
}

// Logging Service
@Injectable({ providedIn: 'root' })
export class LoggingService {
  private logLevel = environment.production ? LogLevel.Error : LogLevel.Debug;
  
  logDebug(message: string, data?: any) {
    if (this.logLevel <= LogLevel.Debug) {
      console.log(`[DEBUG] ${new Date().toISOString()}: ${message}`, data);
    }
  }
  
  logInfo(message: string, data?: any) {
    if (this.logLevel <= LogLevel.Info) {
      console.info(`[INFO] ${new Date().toISOString()}: ${message}`, data);
    }
  }
  
  logWarning(message: string, data?: any) {
    if (this.logLevel <= LogLevel.Warning) {
      console.warn(`[WARNING] ${new Date().toISOString()}: ${message}`, data);
    }
  }
  
  logError(error: any) {
    console.error(`[ERROR] ${new Date().toISOString()}:`, error);
    
    // In production, send to logging service
    if (environment.production) {
      this.sendToLoggingService({
        timestamp: new Date().toISOString(),
        level: 'ERROR',
        message: error.message || 'Unknown error',
        stack: error.stack,
        userAgent: navigator.userAgent,
        url: window.location.href
      });
    }
  }
  
  private sendToLoggingService(logEntry: any) {
    // Implementation to send logs to external service
    // e.g., Sentry, LogRocket, etc.
  }
}

enum LogLevel {
  Debug = 0,
  Info = 1,
  Warning = 2,
  Error = 3
}
```

This comprehensive guide covers the essential advanced Angular concepts that a frontend developer with 4+ years of experience should master. Each section includes practical examples, output-based questions, and real-world scenarios that demonstrate deep understanding of Angular's architecture and best practices.

The key areas covered include component lifecycle management, change detection optimization, reactive forms with custom validation, advanced RxJS patterns, state management, performance optimization, testing strategies, and architectural best practices. These topics reflect the complexity and depth expected at a senior level and provide both theoretical understanding and practical implementation skills.

