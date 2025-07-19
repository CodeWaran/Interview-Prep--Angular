<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# Angular Advanced Tricky Interview Q/A - Additional Challenges

## Memory Management \& Change Detection Gotchas

### Q14: What will be the output and what's wrong with this code?

```typescript
@Component({
  template: `
    <div *ngFor="let item of getItems()">{{item.name}}</div>
    <p>Total calls: {{callCount}}</p>
  `
})
export class TrickyComponent {
  callCount = 0;
  private items = [
    {name: 'Item 1'}, 
    {name: 'Item 2'}, 
    {name: 'Item 3'}
  ];
  
  getItems() {
    this.callCount++;
    console.log('getItems called:', this.callCount);
    return this.items;
  }
}
```

**Answer:**

- **Output**: The `getItems()` method will be called on **every change detection cycle**
- **Problem**: Function calls in templates create performance issues
- **Call count**: Could reach hundreds or thousands depending on user interactions

**Fixed Version:**

```typescript
@Component({
  template: `
    <div *ngFor="let item of items">{{item.name}}</div>
  `
})
export class FixedComponent {
  items = [
    {name: 'Item 1'}, 
    {name: 'Item 2'}, 
    {name: 'Item 3'}
  ];
}
```


### Q15: Predict the sequence of lifecycle hooks for this parent-child scenario:

```typescript
@Component({
  selector: 'parent',
  template: `
    <child *ngIf="showChild" [data]="parentData"></child>
    <button (click)="toggleChild()">Toggle</button>
    <button (click)="updateData()">Update Data</button>
  `
})
export class ParentComponent implements OnInit, AfterViewInit {
  showChild = true;
  parentData = 'initial';
  
  ngOnInit() { console.log('Parent OnInit'); }
  ngAfterViewInit() { console.log('Parent AfterViewInit'); }
  
  toggleChild() {
    this.showChild = !this.showChild;
  }
  
  updateData() {
    this.parentData = 'updated-' + Date.now();
  }
}

@Component({
  selector: 'child',
  template: `<p>{{data}}</p>`
})
export class ChildComponent implements OnInit, OnChanges, OnDestroy {
  @Input() data: string;
  
  ngOnInit() { console.log('Child OnInit'); }
  ngOnChanges(changes) { console.log('Child OnChanges', changes); }
  ngOnDestroy() { console.log('Child OnDestroy'); }
}
```

**Expected Sequence:**

**Initial Load:**

1. Parent OnInit
2. Child OnInit
3. Child OnChanges (first time)
4. Parent AfterViewInit

**Click "Update Data":**

1. Child OnChanges

**Click "Toggle" (hide):**

1. Child OnDestroy

**Click "Toggle" (show):**

1. Child OnInit
2. Child OnChanges

## Advanced RxJS Pitfalls

### Q16: What's the output and memory leak in this service?

```typescript
@Injectable()
export class TrickyService {
  private counter = 0;
  private intervalSub: Subscription;
  
  startCounting() {
    this.intervalSub = interval(1000).pipe(
      tap(() => console.log('Count:', ++this.counter)),
      switchMap(() => this.http.get('/api/data')),
      retry(3)
    ).subscribe();
  }
  
  stopCounting() {
    this.intervalSub?.unsubscribe();
  }
  
  // Called from component
  getData() {
    return this.http.get('/api/user').pipe(
      switchMap(user => this.http.get(`/api/profile/${user.id}`)),
      switchMap(profile => this.http.get(`/api/settings/${profile.settingsId}`))
    );
  }
}

// Component usage
@Component({
  template: `<div>{{result | async}}</div>`
})
export class ComponentA {
  result$ = this.trickyService.getData();
  
  constructor(private trickyService: TrickyService) {}
  
  ngOnInit() {
    this.trickyService.startCounting();
  }
}
```

**Problems Identified:**

1. **Memory Leak**: `startCounting()` creates subscription without proper cleanup
2. **Multiple HTTP calls**: Each `getData()` call creates new chain of requests
3. **No error handling**: Failed requests aren't handled gracefully

**Issues:**

- Service is singleton, but subscription isn't managed properly
- `switchMap` will cancel previous HTTP requests if new ones start
- Component doesn't stop the counting service

**Fixed Version:**

```typescript
@Injectable()
export class FixedService implements OnDestroy {
  private destroy$ = new Subject<void>();
  private counter = 0;
  
  startCounting() {
    return interval(1000).pipe(
      tap(() => console.log('Count:', ++this.counter)),
      switchMap(() => this.http.get('/api/data').pipe(
        catchError(error => {
          console.error('API Error:', error);
          return of(null);
        })
      )),
      retry(3),
      takeUntil(this.destroy$)
    );
  }
  
  getData() {
    return this.http.get('/api/user').pipe(
      switchMap(user => this.http.get(`/api/profile/${user.id}`)),
      switchMap(profile => this.http.get(`/api/settings/${profile.settingsId}`)),
      catchError(error => {
        console.error('Chain failed:', error);
        return throwError(error);
      })
    );
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```


### Q17: Complex RxJS Timing Question

```typescript
const source1$ = timer(0, 1000).pipe(take(3)); // 0, 1, 2 at 0s, 1s, 2s
const source2$ = timer(500, 1000).pipe(take(3)); // 0, 1, 2 at 0.5s, 1.5s, 2.5s

// Question: What values and when?
const result$ = combineLatest([source1$, source2$]).pipe(
  map(([a, b]) => a + b),
  distinctUntilChanged(),
  debounceTime(300)
);

result$.subscribe(console.log);
```

**Answer \& Timeline:**

- **0s**: source1\$ emits 0, source2\$ hasn't emitted yet → No combineLatest emission
- **0.5s**: source2\$ emits 0 → combineLatest emits  → map to 0 → debounce starts
- **0.8s**: debounce completes → **Output: 0**
- **1s**: source1\$ emits 1 →  → map to 1 → debounce starts
- **1.3s**: debounce completes → **Output: 1**
- **1.5s**: source2\$ emits 1 →  → map to 2 → debounce starts
- **1.8s**: debounce completes → **Output: 2**
- **2s**: source1\$ emits 2 →  → map to 3 → debounce starts
- **2.3s**: debounce completes → **Output: 3**
- **2.5s**: source2\$ emits 2 →  → map to 4 → debounce starts
- **2.8s**: debounce completes → **Output: 4**


## Dependency Injection Edge Cases

### Q18: What will happen with these injection configurations?

```typescript
// Service definitions
@Injectable()
class ServiceA {
  constructor(public serviceB: ServiceB) {
    console.log('ServiceA created');
  }
}

@Injectable()
class ServiceB {
  constructor(public serviceC: ServiceC) {
    console.log('ServiceB created');
  }
}

@Injectable()
class ServiceC {
  constructor(@Optional() public serviceA?: ServiceA) {
    console.log('ServiceC created');
  }
}

// Module configuration
@NgModule({
  providers: [
    ServiceA,
    ServiceB,
    ServiceC
  ]
})
export class AppModule {}

// Component
@Component({
  template: '<div>Test</div>'
})
export class TestComponent {
  constructor(private serviceA: ServiceA) {}
}
```

**Answer:**
This creates a **circular dependency** that Angular can resolve due to the `@Optional()` decorator:

**Creation Order:**

1. ServiceC created (with serviceA = null)
2. ServiceB created
3. ServiceA created

**Key Points:**

- `@Optional()` breaks the circular dependency
- ServiceC gets `undefined` for serviceA initially
- Angular creates services in dependency order when possible


### Q19: Tricky Hierarchical Injection Scenario

```typescript
@Injectable({ providedIn: 'root' })
class GlobalService {
  value = 'global';
}

@Component({
  selector: 'parent',
  template: '<child></child>',
  providers: [
    { provide: GlobalService, useValue: { value: 'parent' } }
  ]
})
export class ParentComponent {
  constructor(public service: GlobalService) {
    console.log('Parent service:', service.value);
  }
}

@Component({
  selector: 'child',
  template: '<grandchild></grandchild>',
  providers: [GlobalService] // Creates new instance
})
export class ChildComponent {
  constructor(public service: GlobalService) {
    console.log('Child service:', service.value);
  }
}

@Component({
  selector: 'grandchild',
  template: 'Grandchild'
})
export class GrandchildComponent {
  constructor(public service: GlobalService) {
    console.log('Grandchild service:', service.value);
  }
}
```

**Output:**

- Parent service: parent
- Child service: global (new instance)
- Grandchild service: global (inherits from child)

**Explanation:**

- Parent overrides with `useValue`
- Child provides new instance of original class (uses default constructor)
- Grandchild inherits from child's injector


## Advanced Template Scenarios

### Q20: Template Reference Variable Gotcha

```typescript
@Component({
  template: `
    <div #container>
      <input #userInput (keyup)="processInput(userInput.value)">
      <button (click)="focusInput(userInput)">Focus</button>
      
      <!-- What happens here? -->
      <div *ngIf="showDiv">
        <input #userInput placeholder="Second input">
      </div>
      
      <button (click)="logInputs(userInput)">Log Input</button>
    </div>
  `
})
export class TemplateRefComponent {
  showDiv = false;
  
  processInput(value: string) {
    console.log('Processing:', value);
  }
  
  focusInput(input: HTMLInputElement) {
    input.focus();
  }
  
  logInputs(input: HTMLInputElement) {
    console.log('Input element:', input);
    console.log('Input value:', input.value);
  }
}
```

**Problem:**

- **Duplicate template reference names** `#userInput`
- When `showDiv = true`, which input does `userInput` refer to?

**Answer:**

- Angular will reference the **last** `#userInput` in DOM order
- If `showDiv = false`: References first input
- If `showDiv = true`: References second input (inside *ngIf)
- This can cause unexpected behavior when toggling

**Better Approach:**

```typescript
@Component({
  template: `
    <input #primaryInput (keyup)="processInput(primaryInput.value)">
    <button (click)="focusInput(primaryInput)">Focus Primary</button>
    
    <div *ngIf="showDiv">
      <input #secondaryInput placeholder="Second input">
      <button (click)="focusInput(secondaryInput)">Focus Secondary</button>
    </div>
  `
})
export class FixedTemplateRefComponent {}
```


### Q21: Structural Directive Execution Order

```typescript
@Component({
  template: `
    <div 
      *ngFor="let item of items; trackBy: trackByFn" 
      *ngIf="item.visible"
      [class.active]="item.active">
      {{item.name}}
    </div>
  `
})
export class StructuralDirectiveComponent {
  items = [
    { id: 1, name: 'Item 1', visible: true, active: false },
    { id: 2, name: 'Item 2', visible: false, active: true }
  ];
  
  trackByFn(index: number, item: any) {
    return item.id;
  }
}
```

**Problem:**
**You cannot use two structural directives on the same element!**

**Error:**

```
Can't have multiple template bindings on one element
```

**Solutions:**

**Option 1: Nested Elements**

```typescript
template: `
  <ng-container *ngFor="let item of items; trackBy: trackByFn">
    <div *ngIf="item.visible" [class.active]="item.active">
      {{item.name}}
    </div>
  </ng-container>
`
```

**Option 2: Filter in Component**

```typescript
get visibleItems() {
  return this.items.filter(item => item.visible);
}

template: `
  <div 
    *ngFor="let item of visibleItems; trackBy: trackByFn"
    [class.active]="item.active">
    {{item.name}}
  </div>
`
```


## Advanced Form Validation Edge Cases

### Q22: Complex Cross-Field Validation with Async Validators

```typescript
@Component({
  template: `
    <form [formGroup]="userForm">
      <input formControlName="username" placeholder="Username">
      <div *ngIf="userForm.get('username')?.pending">Checking...</div>
      
      <input formControlName="email" placeholder="Email">
      <input formControlName="confirmEmail" placeholder="Confirm Email">
      
      <div *ngIf="userForm.errors?.['emailMismatch']">
        Emails don't match
      </div>
      
      <div *ngIf="userForm.errors?.['usernameEmailConflict']">
        Username and email domain conflict
      </div>
    </form>
  `
})
export class ComplexValidationComponent implements OnInit {
  userForm: FormGroup;
  
  constructor(private fb: FormBuilder, private http: HttpClient) {}
  
  ngOnInit() {
    this.userForm = this.fb.group({
      username: ['', 
        [Validators.required],
        [this.usernameAsyncValidator.bind(this)]
      ],
      email: ['', [Validators.required, Validators.email]],
      confirmEmail: ['', Validators.required]
    }, { 
      validators: [this.emailMatchValidator, this.usernameEmailConflictValidator],
      updateOn: 'blur' // Important for performance
    });
  }
  
  // Async validator that checks server
  usernameAsyncValidator(control: AbstractControl): Observable<ValidationErrors | null> {
    if (!control.value) return of(null);
    
    return timer(500).pipe( // Debounce
      switchMap(() => 
        this.http.get<{available: boolean}>(`/api/check-username/${control.value}`)
      ),
      map(result => result.available ? null : { usernameTaken: true }),
      catchError(() => of({ usernameCheckError: true }))
    );
  }
  
  // Cross-field validator
  emailMatchValidator(group: AbstractControl): ValidationErrors | null {
    const email = group.get('email')?.value;
    const confirmEmail = group.get('confirmEmail')?.value;
    
    if (!email || !confirmEmail) return null;
    
    return email === confirmEmail ? null : { emailMismatch: true };
  }
  
  // Complex business logic validator
  usernameEmailConflictValidator(group: AbstractControl): ValidationErrors | null {
    const username = group.get('username')?.value;
    const email = group.get('email')?.value;
    
    if (!username || !email) return null;
    
    const emailDomain = email.split('@')[1];
    const forbiddenCombinations = [
      { username: 'admin', domains: ['gmail.com', 'yahoo.com'] },
      { username: 'test', domains: ['company.com'] }
    ];
    
    const conflict = forbiddenCombinations.find(combo => 
      combo.username === username && combo.domains.includes(emailDomain)
    );
    
    return conflict ? { usernameEmailConflict: true } : null;
  }
}
```

**Tricky Points:**

1. **Async validator timing**: Uses debounce to prevent excessive API calls
2. **updateOn: 'blur'**: Improves performance by reducing validation frequency
3. **Error handling**: Async validator handles API failures
4. **Multiple validators**: Form-level and field-level validators interact

### Q23: What's wrong with this validation scenario?

```typescript
@Component({
  template: `
    <form [formGroup]="form">
      <input formControlName="email" (blur)="checkEmail()">
      <button [disabled]="form.invalid" (click)="submit()">Submit</button>
    </form>
  `
})
export class ProblematicValidationComponent {
  form = this.fb.group({
    email: ['', Validators.email]
  });
  
  constructor(private fb: FormBuilder) {}
  
  checkEmail() {
    const email = this.form.get('email')?.value;
    if (email && email.endsWith('@spam.com')) {
      // Problem: Setting error this way doesn't work properly
      this.form.get('email')?.setErrors({spamDomain: true});
    }
  }
  
  submit() {
    if (this.form.valid) {
      console.log('Submitting:', this.form.value);
    }
  }
}
```

**Problems:**

1. **Manual error setting conflicts** with built-in validators
2. **Race condition**: Built-in validators might override manual errors
3. **No error clearing**: Manual errors persist even when corrected

**Fixed Version:**

```typescript
export class FixedValidationComponent {
  form = this.fb.group({
    email: ['', [Validators.email, this.spamDomainValidator]]
  });
  
  // Custom validator instead of manual error setting
  spamDomainValidator(control: AbstractControl): ValidationErrors | null {
    const email = control.value;
    if (email && email.endsWith('@spam.com')) {
      return { spamDomain: true };
    }
    return null;
  }
  
  // Or use async validator for server-side checking
  spamDomainAsyncValidator(control: AbstractControl): Observable<ValidationErrors | null> {
    if (!control.value) return of(null);
    
    return this.http.post<{isSpam: boolean}>('/api/check-spam-domain', {
      email: control.value
    }).pipe(
      map(result => result.isSpam ? { spamDomain: true } : null),
      catchError(() => of(null))
    );
  }
}
```


## Security and Performance Traps

### Q24: Security Vulnerability - What's dangerous here?

```typescript
@Component({
  template: `
    <div [innerHTML]="userContent"></div>
    <div>{{userMessage}}</div>
    
    <!-- User search results -->
    <div *ngFor="let result of searchResults">
      <h3 [innerHTML]="result.title"></h3>
      <p [innerHTML]="result.description"></p>
    </div>
  `
})
export class SecurityIssueComponent {
  userContent = '<script>alert("XSS!")</script><b>Bold text</b>';
  userMessage = '<script>alert("XSS2!")</script>Hello';
  
  searchResults = [
    {
      title: '<img src="x" onerror="alert(\'XSS in title\')">Product 1',
      description: 'Safe description'
    }
  ];
}
```

**Security Issues:**

1. **XSS via innerHTML**: Direct HTML injection without sanitization
2. **User-generated content**: Search results contain malicious HTML
3. **Script injection**: Although `<script>` tags are removed, event handlers (`onerror`) still work

**Angular's Protection:**

- **Interpolation `{{}}`** is safe - automatically escapes HTML
- **innerHTML** is sanitized but not completely safe

**Secure Solutions:**

```typescript
import { DomSanitizer, SafeHtml } from '@angular/platform-browser';

@Component({
  template: `
    <!-- Safe: Use sanitizer -->
    <div [innerHTML]="sanitizedContent"></div>
    
    <!-- Safe: Interpolation automatically escapes -->
    <div>{{userMessage}}</div>
    
    <!-- Safe: Display as text, not HTML -->
    <div *ngFor="let result of searchResults">
      <h3>{{result.title}}</h3>
      <p>{{result.description}}</p>
    </div>
  `
})
export class SecureComponent {
  sanitizedContent: SafeHtml;
  userMessage = '<script>alert("XSS2!")</script>Hello'; // Will display as text
  
  constructor(private sanitizer: DomSanitizer) {
    // Only sanitize trusted content
    this.sanitizedContent = this.sanitizer.sanitize(
      SecurityContext.HTML, 
      '<b>Bold text</b>'
    );
  }
  
  searchResults = [
    {
      title: 'Product 1', // Store as plain text
      description: 'Safe description'
    }
  ];
}
```


### Q25: Performance Anti-Pattern Detection

```typescript
@Component({
  template: `
    <!-- Anti-pattern 1: Function calls in template -->
    <div *ngFor="let item of getFilteredItems()">
      {{formatDate(item.date)}}
    </div>
    
    <!-- Anti-pattern 2: Complex expressions -->
    <div *ngFor="let user of users">
      <span *ngIf="user.roles.some(role => role.permissions.includes('admin'))">
        Admin User
      </span>
    </div>
    
    <!-- Anti-pattern 3: No trackBy -->
    <div *ngFor="let item of dynamicItems">
      {{item.name}}
    </div>
  `
})
export class PerformanceIssueComponent {
  users = [];
  dynamicItems = [];
  
  getFilteredItems() {
    console.log('Filtering items...'); // Called on every CD cycle
    return this.dynamicItems.filter(item => item.active);
  }
  
  formatDate(date: Date) {
    console.log('Formatting date...'); // Called for every item on every CD cycle
    return new Intl.DateTimeFormat().format(date);
  }
}
```

**Performance Issues:**

1. **Function calls**: Execute on every change detection
2. **Complex template logic**: Heavy computation in template
3. **Missing trackBy**: Causes DOM recreation on array changes

**Optimized Version:**

```typescript
@Component({
  template: `
    <div *ngFor="let item of filteredItems; trackBy: trackByItemId">
      {{item.formattedDate}}
    </div>
    
    <div *ngFor="let user of users; trackBy: trackByUserId">
      <span *ngIf="user.isAdmin">Admin User</span>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OptimizedComponent implements OnInit {
  filteredItems: any[] = [];
  processedUsers: any[] = [];
  
  private allItems: any[] = [];
  
  ngOnInit() {
    this.processItems();
    this.processUsers();
  }
  
  // Pre-process data instead of template functions
  private processItems() {
    this.filteredItems = this.allItems
      .filter(item => item.active)
      .map(item => ({
        ...item,
        formattedDate: new Intl.DateTimeFormat().format(item.date)
      }));
  }
  
  private processUsers() {
    this.processedUsers = this.users.map(user => ({
      ...user,
      isAdmin: user.roles.some(role => role.permissions.includes('admin'))
    }));
  }
  
  trackByItemId(index: number, item: any): any {
    return item.id;
  }
  
  trackByUserId(index: number, user: any): any {
    return user.id;
  }
  
  // Update methods that refresh processed data
  updateItems(newItems: any[]) {
    this.allItems = newItems;
    this.processItems();
  }
}
```

These tricky questions test deep understanding of Angular's internals, common pitfalls, and advanced scenarios that experienced developers encounter in real-world applications. They cover edge cases in change detection, RxJS timing, dependency injection hierarchies, template gotchas, security vulnerabilities, and performance anti-patterns.

