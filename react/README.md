# React Best Practices

Here's a compilation of widely accepted React best practices, drawn from official documentation and popular style guides.

## 1. Component Design

*   **Practice:** Prefer Functional Components with Hooks over Class Components.
    *   **Explanation:** Functional components with Hooks are generally more concise, easier to read, and promote better code reuse. They avoid the complexities of `this` binding and lifecycle methods in class components.
    *   **Code Snippet:**
        ```javascript
        // Good (Functional Component)
        function MyComponent({ name }) {
          const [message, setMessage] = useState(`Hello, ${name}`);

          return <div>{message}</div>;
        }

        // Less Preferred (Class Component)
        // class MyComponent extends React.Component {
        //   constructor(props) {
        //     super(props);
        //     this.state = { message: `Hello, ${props.name}` };
        //   }
        //   render() {
        //     return <div>{this.state.message}</div>;
        //   }
        // }
        ```

*   **Practice:** Keep Components Small and Focused (Single Responsibility Principle).
    *   **Explanation:** Smaller components are easier to understand, test, and maintain. Each component should ideally do one thing well.
*   **Practice:** Use PascalCase for Component Names and camelCase for Instances.
    *   **Explanation:** This is a convention that helps distinguish React components from regular HTML elements and JavaScript variables.
    *   **Code Snippet:**
        ```javascript
        import MyComponent from './MyComponent'; // Component name is PascalCase

        const myComponentInstance = <MyComponent />; // Instance is camelCase (though often used directly in JSX)
        ```
*   **Practice:** Define `propTypes` for Type Checking and `defaultProps` for Non-Required Props.
    *   **Explanation:** `propTypes` help catch bugs early by ensuring components receive the correct data types. `defaultProps` make components more robust by providing fallback values.
    *   **Code Snippet:**
        ```javascript
        import PropTypes from 'prop-types';

        function Greeting({ name }) {
          return <h1>Hello, {name}</h1>;
        }

        Greeting.propTypes = {
          name: PropTypes.string.isRequired,
        };

        Greeting.defaultProps = {
          name: 'Guest',
        };
        ```

## 2. Props

*   **Practice:** Pass Props Down from Parent to Child.
    *   **Explanation:** This is the standard way data flows in React, making it unidirectional and easier to trace.
*   **Practice:** Use camelCase for Prop Names.
    *   **Explanation:** Consistent with JavaScript conventions.
*   **Practice:** Omit the Value of a Prop When it's Explicitly `true`.
    *   **Explanation:** This is a shorthand that makes JSX cleaner.
    *   **Code Snippet:**
        ```javascript
        // Bad
        // <MyInput disabled={true} />

        // Good
        <MyInput disabled />
        ```
*   **Practice:** Use Spread Props Sparingly.
    *   **Explanation:** While useful for passing down multiple props, excessive use can lead to passing unnecessary or unknown props, making components harder to reason about. Filter props if necessary.
    *   **Code Snippet:**
        ```javascript
        // Good (when props are known and controlled)
        // const buttonProps = {
        //   onClick: handleClick,
        //   label: 'Submit',
        // };
        // <Button {...buttonProps} />

        // Better to be explicit if unsure
        // <Button onClick={handleClick} label="Submit" />
        ```

## 3. State

*   **Practice:** Use the `useState` Hook for Managing Local Component State.
    *   **Explanation:** `useState` is the fundamental Hook for adding state to functional components.
    *   **Code Snippet:**
        ```javascript
        import { useState } from 'react';

        function Counter() {
          const [count, setCount] = useState(0);
          return (
            <button onClick={() => setCount(count + 1)}>
              Clicked {count} times
            </button>
          );
        }
        ```
*   **Practice:** State Updates are Asynchronous.
    *   **Explanation:** When you call a state setter function (e.g., `setCount`), React batches updates for performance. Don't rely on the state being updated immediately in the next line of code. Use the functional update form if the new state depends on the previous state.
    *   **Code Snippet:**
        ```javascript
        // Good (functional update when new state depends on old)
        // setCount(prevCount => prevCount + 1);
        ```
*   **Practice:** Lift State Up to the Nearest Common Ancestor.
    *   **Explanation:** When multiple components need to share and manipulate the same state, move that state to their closest common parent component. The parent then passes the state and update functions down as props.
*   **Practice:** Choose the State Structure Wisely.
    *   **Explanation:** Group related state. Avoid redundant or derived state; calculate it during rendering if possible.

## 4. Hooks

*   **Practice:** Only Call Hooks at the Top Level.
    *   **Explanation:** Do not call Hooks inside loops, conditions, or nested functions. This ensures Hooks are called in the same order each render.
*   **Practice:** Only Call Hooks from React Functions.
    *   **Explanation:** Call Hooks from React functional components or custom Hooks.
*   **Practice:** Use `useEffect` for Side Effects.
    *   **Explanation:** `useEffect` allows you to perform side effects after rendering, such as data fetching, subscriptions, or manually changing the DOM.
    *   **Code Snippet (Data Fetching Example):**
        ```javascript
        import { useState, useEffect } from 'react';

        function UserProfile({ userId }) {
          const [user, setUser] = useState(null);

          useEffect(() => {
            async function fetchUser() {
              const response = await fetch(`/api/users/${userId}`);
              const data = await response.json();
              setUser(data);
            }
            fetchUser();
          }, [userId]); // Dependency array: re-run effect if userId changes

          if (!user) return <p>Loading...</p>;
          return <div>{user.name}</div>;
        }
        ```
*   **Practice:** Provide a Dependency Array for `useEffect`.
    *   **Explanation:** The dependency array tells React when to re-run the effect. An empty array `[]` means the effect runs once after the initial render and cleans up on unmount. Omitting it causes the effect to run after every render.
*   **Practice:** Create Custom Hooks to Reuse State Logic.
    *   **Explanation:** If you find yourself repeating stateful logic across multiple components, extract it into a custom Hook. Custom Hooks are functions whose names start with `use`.
    *   **Code Snippet:**
        ```javascript
        // Custom Hook
        // function useFormInput(initialValue) {
        //   const [value, setValue] = useState(initialValue);
        //   function handleChange(e) {
        //     setValue(e.target.value);
        //   }
        //   return { value, onChange: handleChange };
        // }

        // Usage in a component
        // function MyForm() {
        //   const nameInput = useFormInput('');
        //   return <input type="text" {...nameInput} />;
        // }
        ```

## 5. State Management

*   **Practice:** Use the Context API for Passing Data Deeply.
    *   **Explanation:** For global state or data that needs to be accessed by many components at different nesting levels without prop drilling, Context is a good solution.
    *   **Code Snippet:**
        ```javascript
        // import { createContext, useContext } from 'react';

        // const ThemeContext = createContext('light');

        // function MyApp() {
        //   return (
        //     <ThemeContext.Provider value="dark">
        //       <MyComponent />
        //     </ThemeContext.Provider>
        //   );
        // }

        // function MyComponent() {
        //   const theme = useContext(ThemeContext);
        //   return <div className={`theme-${theme}`}>...</div>;
        // }
        ```
*   **Practice:** Consider Reducers (`useReducer`) for Complex State Logic.
    *   **Explanation:** For components with complex state transitions or when the next state depends on multiple sub-values, `useReducer` can be more manageable than multiple `useState` calls. It's often used with Context for global state management.

## 6. Event Handling

*   **Practice:** Pass Event Handlers as Functions (References).
    *   **Explanation:** Instead of calling the function directly in JSX (which would execute it on render), pass a reference to the function.
    *   **Code Snippet:**
        ```javascript
        // function MyButton() {
        //   function handleClick() {
        //     console.log('Button clicked!');
        //   }
        //   // Good: Pass function reference
        //   return <button onClick={handleClick}>Click Me</button>;

        //   // Bad: Calls function on render
        //   // return <button onClick={handleClick()}>Click Me</button>;
        // }
        ```
*   **Practice:** Bind `this` in Class Components (if not using arrow functions for handlers) or Use Arrow Functions.
    *   **Explanation:** In class components, ensure event handlers have the correct `this` context. This is less of an issue with functional components and Hooks. The Airbnb style guide recommends binding in the constructor for class components.
    *   **Code Snippet (Class Component - Constructor Binding):**
        ```javascript
        // class MyClassComponent extends React.Component {
        //   constructor(props) {
        //     super(props);
        //     this.handleClick = this.handleClick.bind(this);
        //   }
        //   handleClick() { /* ... */ }
        //   render() {
        //     return <button onClick={this.handleClick}>Click</button>;
        //   }
        // }
        ```

## 7. Conditional Rendering

*   **Practice:** Use JavaScript Expressions like `if/else`, Ternary Operators, or Logical `&&`.
    *   **Explanation:** React doesn't have special syntax for conditions; use standard JavaScript.
    *   **Code Snippet:**
        ```javascript
        // function MyConditionalComponent({ isLoggedIn }) {
        //   if (isLoggedIn) {
        //     return <AdminPanel />;
        //   } else {
        //     return <LoginForm />;
        //   }

        //   // Or using ternary operator
        //   // return isLoggedIn ? <AdminPanel /> : <LoginForm />;

        //   // Or using logical && for rendering only if true
        //   // return isLoggedIn && <AdminPanel />;
        // }
        ```

## 8. Keys in Lists

*   **Practice:** Always Provide a Unique `key` Prop for Items in a List.
    *   **Explanation:** Keys help React identify which items have changed, are added, or are removed. This is crucial for efficient updates and maintaining component state within lists. Use stable, unique IDs from your data as keys. Avoid using array indices as keys if the list can be reordered, added to, or filtered.
    *   **Code Snippet:**
        ```javascript
        // function MyList({ items }) {
        //   return (
        //     <ul>
        //       {items.map(item => (
        //         <li key={item.id}>{item.name}</li>
        //       ))}
        //     </ul>
        //   );
        // }
        ```

## 9. Code Organization and Structure

*   **Practice:** One Component Per File.
    *   **Explanation:** Generally, each React component should reside in its own file for better organization. Multiple stateless functional components (SFCs) or pure components can be in one file if they are small and closely related.
*   **Practice:** Use PascalCase for Filenames (e.g., `MyComponent.jsx`).
*   **Practice:** Group Files by Feature or Route.
    *   **Explanation:** Organize your project by features (e.g., `components/User/UserProfile.jsx`, `components/Product/ProductDetails.jsx`) rather than just by type (e.g., all components in one folder, all services in another).
*   **Practice:** Use `index.jsx` for Root Components of a Directory.
    *   **Explanation:** This allows for cleaner imports (e.g., `import Footer from './Footer';` instead of `import Footer from './Footer/Footer.jsx';`).

## 10. Performance Considerations

*   **Practice:** Memoization with `React.memo` and `useMemo`.
    *   **Explanation:**
        *   `React.memo`: A higher-order component that memoizes your component, preventing re-renders if props haven't changed. Useful for optimizing functional components.
        *   `useMemo`: A Hook that memoizes the result of a calculation between renders. Use it to avoid expensive recalculations if dependencies haven't changed.
    *   **Code Snippet (`React.memo`):**
        ```javascript
        // const MyMemoizedComponent = React.memo(function MyComponent(props) {
        //   /* render using props */
        // });
        ```
*   **Practice:** Use `useCallback` to Memoize Callbacks.
    *   **Explanation:** If you pass callbacks to optimized child components that rely on reference equality (e.g., memoized with `React.memo`), use `useCallback` to prevent creating new function instances on every render, which would break memoization.
*   **Practice:** Lazy Loading Components with `React.lazy` and `Suspense`.
    *   **Explanation:** For larger applications, split your code into smaller chunks and load components only when they are needed. This improves initial load time.
    *   **Code Snippet:**
        ```javascript
        // import React, { Suspense, lazy } from 'react';

        // const OtherComponent = lazy(() => import('./OtherComponent'));

        // function MyPage() {
        //   return (
        //     <div>
        //       <Suspense fallback={<div>Loading...</div>}>
        //         <OtherComponent />
        //       </Suspense>
        //     </div>
        //   );
        // }
        ```
*   **Practice:** Virtualize Long Lists.
    *   **Explanation:** For rendering very long lists of data, use libraries like `react-window` or `react-virtualized` to only render the items currently visible in the viewport.

## 11. Immutability

*   **Practice:** Do Not Mutate State or Props Directly.
    *   **Explanation:** React relies on immutable data structures to efficiently detect changes. When updating state (especially objects or arrays), always create a new copy with the changes instead of modifying the original.
    *   **Code Snippet (Updating an array in state):**
        ```javascript
        // // Bad: Mutating state
        // // this.state.items.push(newItem);

        // // Good: Creating a new array
        // // In a class component:
        // // this.setState(prevState => ({
        // //   items: [...prevState.items, newItem],
        // // }));

        // // In a functional component with useState:
        // // setItems(prevItems => [...prevItems, newItem]);
        ```

## 12. Error Handling

*   **Practice:** Use Error Boundaries.
    *   **Explanation:** Error Boundaries are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of the component tree that crashed.
    *   **Code Snippet:**
        ```javascript
        // class ErrorBoundary extends React.Component {
        //   constructor(props) {
        //     super(props);
        //     this.state = { hasError: false };
        //   }

        //   static getDerivedStateFromError(error) {
        //     return { hasError: true };
        //   }

        //   componentDidCatch(error, errorInfo) {
        //     // logErrorToMyService(error, errorInfo);
        //   }

        //   render() {
        //     if (this.state.hasError) {
        //       return <h1>Something went wrong.</h1>;
        //     }
        //     return this.props.children;
        //   }
        // }

        // // Usage:
        // // <ErrorBoundary>
        // //   <MyWidget />
        // // </ErrorBoundary>
        ```
    *   **Note:** Error Boundaries do not catch errors in event handlers (use try/catch), async code, server-side rendering, or errors thrown in the Error Boundary itself.

## 13. Basic Accessibility (a11y) Considerations

*   **Practice:** Use Semantic HTML.
    *   **Explanation:** Use appropriate HTML tags for their intended purpose (e.g., `<button>` for buttons, `<nav>` for navigation). This provides inherent accessibility features.
*   **Practice:** Provide `alt` Text for Images.
    *   **Explanation:** Essential for screen readers. If an image is purely decorative, use `alt=""`. Avoid redundant words like "image" or "picture" in alt text.
    *   **Code Snippet:**
        ```javascript
        <img src="profile.jpg" alt="User's profile picture" />
        <img src="decorative-swirl.png" alt="" />
        ```
*   **Practice:** Ensure Keyboard Navigability.
    *   **Explanation:** All interactive elements should be focusable and operable via keyboard.
*   **Practice:** Use ARIA Attributes When Necessary.
    *   **Explanation:** Use [Accessible Rich Internet Applications (ARIA)](https://www.w3.org/WAI/ARIA/apg/) attributes to enhance accessibility for custom components or dynamic content, but prioritize semantic HTML. Use valid, non-abstract ARIA roles.
*   **Practice:** Manage Focus.
    *   **Explanation:** In dynamic applications, programmatically manage focus when appropriate (e.g., after opening a modal dialog, move focus into the modal).

This list covers many core best practices. The React ecosystem is always evolving, so continuous learning is key.
