# Angular Best Practices

This document outlines widely accepted Angular best practices, focusing on non-obvious and impactful ones.

## 1. Components

*   **Practice:** Use `OnPush` Change Detection Strategy.
    *   **Explanation:** Setting `changeDetection: ChangeDetectionStrategy.OnPush` in your components tells Angular to only run change detection when input properties change, an event originates from the component or one of its children, or when explicitly triggered. This significantly improves performance by reducing the number of change detection cycles.
    *   **Code Snippet:**
        ```typescript
        import { Component, ChangeDetectionStrategy, Input } from '@angular/core';

        interface User { name: string; } // Example User interface

        @Component({
          selector: 'app-user-profile',
          template: `<div>{{ user?.name }}</div>`, // Added safe navigation operator
          changeDetection: ChangeDetectionStrategy.OnPush
        })
        export class UserProfileComponent {
          @Input() user: User | null = null; // Initialize Input
        }
        ```

*   **Practice:** Use `trackBy` Function for `*ngFor` Loops.
    *   **Explanation:** When rendering lists with `*ngFor`, providing a `trackBy` function helps Angular identify which items have been added, removed, or reordered. This prevents the DOM from being re-rendered for unchanged items, boosting performance, especially for large lists or lists with complex items.
    *   **Code Snippet:**
        ```typescript
        import { Component } from '@angular/core';

        interface User {
          id: number;
          name: string;
        }

        @Component({
          selector: 'app-user-list',
          template: `
            <ul>
              <li *ngFor="let user of users; trackBy: trackByUser">
                {{ user.name }}
              </li>
            </ul>
          `
        })
        export class UserListComponent {
          users: User[] = [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }];

          trackByUser(index: number, user: User): number {
            return user.id; // Or any unique identifier
          }
        }
        ```

*   **Practice:** Smart vs. Presentational (Dumb) Components.
    *   **Explanation:** Divide components into two types:
        *   **Smart Components (Containers):** Concerned with how things work. They manage state, fetch data, and pass data down to presentational components.
        *   **Presentational Components (Dumb):** Concerned with how things look. They receive data via `@Input()` and emit events via `@Output()`. They don't have their own state related to business logic and are highly reusable.
    *   **Benefit:** This separation improves reusability, testability, and maintainability.

## 2. Services

*   **Practice:** Provide Services in the Correct Scope.
    *   **Explanation:**
        *   `providedIn: 'root'`: (Recommended default) Creates a single, application-wide singleton instance of the service. Tree-shakeable.
        *   `providedIn: SomeModule`: Scopes the service instance to that specific module.
        *   `providers: [MyService]` (in a component): Creates a new instance of the service for each instance of that component.
    *   **Impact:** Choosing the right scope avoids unintended multiple instances or lack of instance when one is needed, and helps with tree-shaking.
    *   **Code Snippet:**
        ```typescript
        import { Injectable } from '@angular/core';

        @Injectable({
          providedIn: 'root'
        })
        export class LoggingService {
          log(message: string) {
            console.log(message);
          }
        }
        ```

*   **Practice:** Avoid `providedIn: 'any'`.
    *   **Explanation:** While `providedIn: 'any'` provides a service instance for every eagerly loaded module that injects it, it can lead to multiple instances if not carefully managed with lazy loading boundaries, potentially causing unexpected behavior. Prefer `'root'` for singletons or module-specific provision.

## 3. Modules

*   **Practice:** Create Feature Modules.
    *   **Explanation:** Organize your application into feature modules (e.g., `UserModule`, `ProductModule`). Each feature module encapsulates a specific piece of functionality with its own components, services, and routing. This improves organization, scalability, and enables lazy loading.
*   **Practice:** Use Lazy Loading for Feature Modules.
    *   **Explanation:** Configure your router to lazy load feature modules. This means the code for a specific feature is only downloaded and compiled when the user navigates to a route within that feature, significantly reducing initial application load time.
    *   **Code Snippet (Routing Configuration):**
        ```typescript
        // app-routing.module.ts
        import { NgModule } from '@angular/core';
        import { RouterModule, Routes } from '@angular/router';

        const routes: Routes = [
          {
            path: 'customers',
            loadChildren: () => import('./customers/customers.module').then(m => m.CustomersModule)
          },
          // ... other routes
        ];

        @NgModule({
          imports: [RouterModule.forRoot(routes)],
          exports: [RouterModule]
        })
        export class AppRoutingModule { }
        ```
*   **Practice:** Use a `SharedModule` for Commonly Used Declarables.
    *   **Explanation:** Create a `SharedModule` to declare and export components, directives, and pipes that are used across multiple feature modules. Import this `SharedModule` into the feature modules that need these common items. Do *not* provide services in the `SharedModule` (provide them in `'root'` or feature modules).
*   **Practice:** Use a `CoreModule` for Application-Wide Singleton Services and Core Components.
    *   **Explanation:** Create a `CoreModule` (imported *only* once in `AppModule`) to provide application-wide singleton services (though `'root'` is now preferred for services), and to house components that are used only once in your application shell (e.g., navigation bar, footer). This helps prevent accidental multiple instantiations of services.

## 4. Performance

*   **Practice:** Use `APP_INITIALIZER` for Application Initialization Tasks.
    *   **Explanation:** If you have tasks that must complete before the application starts (e.g., loading configuration), use the `APP_INITIALIZER` token. This ensures these tasks run during the app bootstrap process.
    *   **Code Snippet:**
        ```typescript
        import { APP_INITIALIZER, NgModule } from '@angular/core';
        import { Injectable } from '@angular/core'; // Import Injectable
        import { Observable, of } from 'rxjs'; // Import Observable and of

        @Injectable({ providedIn: 'root' }) // Make ConfigService injectable
        export class ConfigService {
          loadConfig(): Observable<any> { // Return type can be Observable or Promise
            // Simulate loading config
            console.log('Config loaded');
            return of({ setting: 'initial' });
          }
        }

        export function initializeApp(configService: ConfigService) {
          return () => configService.loadConfig();
        }

        @NgModule({
          providers: [
            ConfigService,
            {
              provide: APP_INITIALIZER,
              useFactory: initializeApp,
              deps: [ConfigService],
              multi: true
            }
          ],
          // ...
        })
        export class AppModule { }
        ```
*   **Practice:** Detach Change Detector for Long-Running Background Tasks.
    *   **Explanation:** If a component performs long-running tasks or frequent updates not affecting the UI directly (e.g., WebSocket updates handled internally), detach its change detector (`ChangeDetectorRef.detach()`) and only reattach (`reattach()`) or explicitly trigger change detection (`detectChanges()`) when necessary.
*   **Practice:** Optimize Template Expressions.
    *   **Explanation:** Keep template expressions simple and fast. Avoid complex calculations or method calls directly in templates, as they execute on every change detection cycle. Pre-calculate values in your component class.

## 5. RxJS

*   **Practice:** Use the `async` Pipe to Manage Subscriptions.
    *   **Explanation:** The `async` pipe automatically subscribes to an Observable or Promise and returns the latest value. Crucially, it also automatically unsubscribes when the component is destroyed, preventing memory leaks.
    *   **Code Snippet:**
        ```html
        <!-- user-display.component.html -->
        <div *ngIf="user$ | async as user">
          Hello, {{ user.name }}
        </div>
        ```
        ```typescript
        // user-display.component.ts
        import { Component } from '@angular/core';
        import { Observable, of } from 'rxjs'; // Assuming UserService provides user$

        interface User { name: string; } // Example User interface

        // Mock UserService for standalone example
        class UserService {
          getUser(): Observable<User> {
            return of({ name: 'Default User' });
          }
        }


        @Component({
          selector: 'app-user-display',
          templateUrl: './user-display.component.html',
          providers: [UserService] // Provide UserService if not root
        })
        export class UserDisplayComponent {
          user$: Observable<User>;

          constructor(private userService: UserService) {
            this.user$ = this.userService.getUser();
          }
        }
        ```
*   **Practice:** Use Higher-Order Mapping Operators (e.g., `switchMap`, `mergeMap`, `concatMap`, `exhaustMap`).
    *   **Explanation:** Choose the correct higher-order mapping operator based on the desired behavior for handling new emissions from the source Observable when an inner Observable is active.
        *   `switchMap`: Cancels the previous inner Observable when the source emits. Good for type-ahead searches or data fetching based on changing inputs.
        *   `mergeMap`: Subscribes to all inner Observables concurrently. Good for parallel operations.
        *   `concatMap`: Subscribes to inner Observables sequentially, waiting for the current one to complete before starting the next. Good for ordered operations.
        *   `exhaustMap`: Ignores new source emissions while the current inner Observable is active. Good for preventing multiple submissions (e.g., button clicks).
*   **Practice:** Unsubscribe from Observables Manually (When Not Using `async` Pipe).
    *   **Explanation:** If you manually subscribe to Observables in your component, you must unsubscribe in `ngOnDestroy` to prevent memory leaks. Common patterns include using a `Subject` with `takeUntil`.
    *   **Code Snippet:**
        ```typescript
        import { Component, OnDestroy } from '@angular/core';
        import { Subject, Observable, of } from 'rxjs';
        import { takeUntil } from 'rxjs/operators';

        // Mock DataService for standalone example
        class DataService {
          getData(): Observable<any> {
            return of('some data');
          }
        }

        @Component({
          selector: 'app-my-component',
          template: `<p>Check console for logs.</p>`,
          providers: [DataService] // Provide DataService if not root
        })
        export class MyComponent implements OnDestroy {
          private destroy$ = new Subject<void>();

          constructor(private dataService: DataService) {
            this.dataService.getData()
              .pipe(takeUntil(this.destroy$))
              .subscribe(data => { console.log(data); });
          }

          ngOnDestroy() {
            this.destroy$.next();
            this.destroy$.complete();
          }
        }
        ```
*   **Practice:** Use `shareReplay` for Sharing Observable Results.
    *   **Explanation:** If multiple subscribers need the result of an Observable (especially one that makes an HTTP request), use `shareReplay({ bufferSize: 1, refCount: true })` to share the source and replay the last emitted value(s) to new subscribers, preventing redundant work.

## 6. State Management

*   **Practice:** For Simple Local State, Use RxJS `BehaviorSubject` in Services.
    *   **Explanation:** For managing shared state that doesn't warrant a full NgRx/NGXS setup, a `BehaviorSubject` within a service can be a lightweight and effective solution. Components can subscribe to the `BehaviorSubject` to get state updates.
    *   **Code Snippet:**
        ```typescript
        // data-store.service.ts
        import { Injectable } from '@angular/core';
        import { BehaviorSubject, Observable } from 'rxjs';

        @Injectable({ providedIn: 'root' })
        export class DataStoreService {
          private myData = new BehaviorSubject<string>('Initial Data');
          public readonly myData$: Observable<string> = this.myData.asObservable();

          updateData(newData: string) {
            this.myData.next(newData);
          }
        }
        ```
*   **Practice:** Consider Dedicated State Management Libraries (NgRx, NGXS, Akita) for Complex Applications.
    *   **Explanation:** For applications with complex state interactions, many sources of state changes, or a need for robust developer tools (like time-travel debugging), consider using a dedicated state management library. These libraries enforce unidirectional data flow and provide patterns for managing state in a predictable way.
*   **Practice:** Keep State Immutable.
    *   **Explanation:** Regardless of the state management approach, treat state as immutable. When updating state, always produce a new state object/array rather than modifying the existing one directly. This is crucial for change detection, performance, and predictability.

## 7. General / Other

*   **Practice:** Use Route Guards for Protecting Routes.
    *   **Explanation:** Implement `CanActivate`, `CanActivateChild`, `CanDeactivate`, `CanLoad`, and `Resolve` guards to control access to routes based on authentication, authorization, unsaved changes, or pre-fetching data.
*   **Practice:** Use HTTP Interceptors for Global HTTP Request/Response Handling.
    *   **Explanation:** Interceptors allow you to globally transform HTTP requests (e.g., adding authentication headers) and responses (e.g., global error handling, logging).
*   **Practice:** Follow Angular File Naming Conventions Consistently.
    *   **Explanation:** E.g., `feature.component.ts`, `feature.service.ts`, `feature.module.ts`, `feature.pipe.ts`, `feature.directive.ts`. This improves project organization and predictability.
