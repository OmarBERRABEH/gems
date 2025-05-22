# JavaScript Best Practices

This document outlines widely accepted JavaScript best practices, drawing from reputable style guides (Airbnb, Google, StandardJS) and the MDN Web Docs.

## 1. Variable Declarations

*   **Practice:** Prefer `const` by Default; Use `let` for Reassignable Variables. Avoid `var`.
    *   **Explanation:** `const` ensures that the variable cannot be reassigned after its initial assignment, which helps prevent accidental modifications and makes code easier to reason about (it does *not* make the value immutable if it's an object or array). `let` allows reassignment and is block-scoped, which is generally safer and more predictable than `var`'s function scoping and hoisting behavior. `var` can lead to hoisting issues and less clear scope, making it error-prone.
    *   **Code Snippet:**
        ```javascript
        // Good
        const PI = 3.14159;
        let count = 0;
        count = 1; // Allowed

        // Avoid
        // var legacyVariable = 'avoid me';
        ```

*   **Practice:** Declare One Variable Per Declaration.
    *   **Explanation:** This improves readability and makes it easier to debug and reorder variable declarations.
    *   **Code Snippet:**
        ```javascript
        // Good
        const user = 'Alice';
        const age = 30;

        // Avoid
        // const user = 'Alice', age = 30;
        ```

*   **Practice:** Initialize Variables When Declared, If Possible.
    *   **Explanation:** This prevents accidental usage of `undefined` variables and makes the code's intent clearer.

## 2. Strict Mode

*   **Practice:** Enable Strict Mode.
    *   **Explanation:** `'use strict';` at the beginning of your scripts or functions enables a stricter parsing and error handling mode in JavaScript. It helps you write more robust and maintainable code by catching common coding bloopers and preventing or throwing errors for unsafe actions (like assigning to undeclared variables or using reserved keywords).
    *   **Code Snippet:**
        ```javascript
        'use strict';

        function doSomething() {
          // undeclaredVariable = 10; // This would throw an error in strict mode
          const declaredVariable = 10;
          console.log(declaredVariable);
        }
        doSomething();
        ```

## 3. Types and Coercion

*   **Practice:** Use Strict Equality (`===` and `!==`).
    *   **Explanation:** Strict equality operators compare both value and type without performing type coercion. This prevents unexpected behavior that can occur with loose equality (`==` and `!=`) due to JavaScript's type coercion rules.
    *   **Code Snippet:**
        ```javascript
        // Good
        console.log(0 === '0'); // false
        console.log(null === undefined); // false

        // Avoid (unless specifically handling null/undefined together)
        // console.log(0 == '0'); // true
        // console.log(null == undefined); // true
        ```
*   **Practice:** Be Explicit About Type Coercion When Necessary.
    *   **Explanation:** While generally avoiding implicit coercion is good, sometimes you need to convert types. Make these conversions explicit and clear.
    *   **Code Snippet:**
        ```javascript
        const stringValue = '123';
        const numberValue = Number(stringValue); // Explicit conversion

        const numericValue = 42;
        const textValue = String(numericValue); // Explicit conversion
        ```
*   **Practice:** Understand Truthy and Falsy Values.
    *   **Explanation:** Be aware of which values coerce to `true` (truthy) and `false` (falsy) in boolean contexts (e.g., `if` statements). Falsy values include `false`, `0`, `''` (empty string), `null`, `undefined`, and `NaN`.
    *   **Code Snippet:**
        ```javascript
        const name = '';
        if (name) {
          // This block will not execute because empty string is falsy
        }
        ```

## 4. Functions

*   **Practice:** Prefer Arrow Functions for Non-Method Functions.
    *   **Explanation:** Arrow functions (`=>`) provide a more concise syntax and lexically bind `this`, meaning `this` refers to the `this` of the enclosing scope. This is often desirable for callbacks and nested functions, avoiding the need for `var self = this;` or `.bind(this)`. For methods that need their own `this` context (e.g., in classes or object literals), traditional function expressions or method shorthand are still appropriate.
    *   **Code Snippet:**
        ```javascript
        // Good for callbacks
        const numbers = [1, 2, 3];
        const squared = numbers.map(n => n * n);

        // Good for object methods (method shorthand)
        const myObject = {
          value: 10,
          getValue() { // `this` refers to myObject
            return this.value;
          }
        };
        ```
*   **Practice:** Use Default Parameters.
    *   **Explanation:** Default parameters allow you to specify default values for function parameters if they are not provided or are `undefined`. This makes functions more robust and reduces boilerplate for checking undefined parameters.
    *   **Code Snippet:**
        ```javascript
        function greet(name = 'Guest', greeting = 'Hello') {
          console.log(`${greeting}, ${name}!`);
        }
        greet(); // Hello, Guest!
        greet('Alice', 'Hi'); // Hi, Alice!
        ```
*   **Practice:** Use Rest Parameters for Variadic Functions.
    *   **Explanation:** Rest parameters (`...paramName`) allow you to represent an indefinite number of arguments as an array. This is cleaner and more explicit than using the `arguments` object.
    *   **Code Snippet:**
        ```javascript
        function sum(...numbers) {
          return numbers.reduce((total, num) => total + num, 0);
        }
        console.log(sum(1, 2, 3)); // 6
        console.log(sum(10, 20, 30, 40)); // 100
        ```
*   **Practice:** Avoid Using the `arguments` Object.
    *   **Explanation:** The `arguments` object is an array-like object (not a true array) that can be less intuitive to work with than rest parameters. Rest parameters are generally preferred for clarity and because they provide a real array.

## 5. Objects and Arrays

*   **Practice:** Use Object and Array Destructuring.
    *   **Explanation:** Destructuring makes it easy to extract values from objects and arrays into distinct variables, leading to more concise and readable code.
    *   **Code Snippet:**
        ```javascript
        // Object destructuring
        const user = { name: 'Bob', age: 25, city: 'New York' };
        const { name, age } = user;
        console.log(name, age); // Bob 25

        // Array destructuring
        const coordinates = [10, 20, 30];
        const [x, y] = coordinates;
        console.log(x, y); // 10 20
        ```
*   **Practice:** Use Spread Syntax for Copying and Merging.
    *   **Explanation:** The spread syntax (`...`) allows an iterable (like an array or object) to be expanded in places where zero or more arguments (for function calls) or elements (for array literals) or key-value pairs (for object literals) are expected. It's very useful for creating shallow copies of arrays and objects, and for merging them.
    *   **Code Snippet:**
        ```javascript
        // Array copy and concatenation
        const originalArray = [1, 2, 3];
        const copiedArray = [...originalArray];
        const combinedArray = [...originalArray, 4, 5];

        // Object copy and merging
        const originalObject = { a: 1, b: 2 };
        const copiedObject = { ...originalObject };
        const mergedObject = { ...originalObject, c: 3, d: 4 };
        ```
*   **Practice:** Strive for Immutability with Objects and Arrays.
    *   **Explanation:** Instead of modifying objects and arrays directly (mutating them), create new objects or arrays with the desired changes. This helps prevent unintended side effects, makes state changes more predictable (especially in larger applications or when using frameworks like React/Redux), and can improve performance by making change detection easier.
    *   **Code Snippet (Array):**
        ```javascript
        const numbers = [1, 2, 3];
        // Bad: Mutating
        // numbers.push(4);

        // Good: Creating a new array
        const newNumbers = [...numbers, 4];
        const updatedNumbers = numbers.map(n => n * 2); // map returns a new array
        ```
    *   **Code Snippet (Object):**
        ```javascript
        const person = { name: 'Alice', age: 30 };
        // Bad: Mutating
        // person.age = 31;

        // Good: Creating a new object
        const updatedPerson = { ...person, age: 31 };
        ```
*   **Practice:** Use Array Helper Methods (`.map()`, `.filter()`, `.reduce()`, etc.).
    *   **Explanation:** These methods provide a declarative and often more readable way to iterate and transform arrays compared to traditional `for` loops. They also promote immutability by typically returning new arrays.
    *   **Code Snippet:**
        ```javascript
        const items = [
          { id: 1, name: 'Book', price: 10 },
          { id: 2, name: 'Pen', price: 2 },
          { id: 3, name: 'Paper', price: 5 },
        ];
        const itemNames = items.map(item => item.name);
        const expensiveItems = items.filter(item => item.price > 5);
        const totalPrice = items.reduce((sum, item) => sum + item.price, 0);
        ```
*   **Practice:** Use Literals for Object and Array Creation.
    *   **Explanation:** Use `{}` for objects and `[]` for arrays instead of `new Object()` or `new Array()`. It's more concise and generally preferred.
    *   **Code Snippet:**
        ```javascript
        // Good
        const myObject = {};
        const myArray = [];

        // Avoid
        // const myObject = new Object();
        // const myArray = new Array();
        ```

## 6. ES6+ Features

*   **Practice:** Use ES Modules (`import`/`export`).
    *   **Explanation:** ES Modules are the standard way to organize and share code in JavaScript. They provide a clean syntax for importing and exporting functions, objects, or primitives from one module to another, promoting modularity and reusability.
    *   **Code Snippet:**
        ```javascript
        // utils.js
        // export const PI = 3.14;
        // export function greet(name) { return `Hello, ${name}`; }

        // main.js
        // import { PI, greet } from './utils.js';
        // console.log(greet('World'), PI);
        ```
*   **Practice:** Use Classes for Object-Oriented Programming Patterns.
    *   **Explanation:** ES6 classes provide a more familiar syntax for creating objects and implementing inheritance, built on JavaScript's existing prototype-based inheritance.
    *   **Code Snippet:**
        ```javascript
        class Person {
          constructor(name, age) {
            this.name = name;
            this.age = age;
          }

          greet() {
            console.log(`Hello, my name is ${this.name}.`);
          }
        }

        const alice = new Person('Alice', 30);
        alice.greet();
        ```
*   **Practice:** Use Promises for Asynchronous Operations.
    *   **Explanation:** Promises provide a cleaner and more robust way to handle asynchronous operations compared to traditional callbacks, helping to avoid "callback hell" and making asynchronous code easier to manage and reason about.
    *   **Code Snippet:**
        ```javascript
        // function fetchData() {
        //   return new Promise((resolve, reject) => {
        //     setTimeout(() => {
        //       const data = { id: 1, message: 'Data fetched' };
        //       if (data) {
        //         resolve(data);
        //       } else {
        //         reject('Error fetching data');
        //       }
        //     }, 1000);
        //   });
        // }

        // fetchData()
        //   .then(data => console.log(data))
        //   .catch(error => console.error(error));
        ```
*   **Practice:** Use `async/await` for Cleaner Asynchronous Code.
    *   **Explanation:** `async/await` builds on top of Promises and allows you to write asynchronous code that looks and behaves a bit more like synchronous code, making it even easier to read and understand. `async` functions always return a Promise.
    *   **Code Snippet:**
        ```javascript
        // async function processData() {
        //   try {
        //     const data = await fetchData(); // fetchData is a function returning a Promise
        //     console.log('Processed:', data.message);
        //   } catch (error) {
        //     console.error('Error processing data:', error);
        //   }
        // }
        // processData();
        ```
*   **Practice:** Use Template Literals for String Interpolation and Multiline Strings.
    *   **Explanation:** Template literals (backticks `` ` ``) provide an easy way to embed expressions within strings and to create multiline strings without needing escape characters.
    *   **Code Snippet:**
        ```javascript
        const name = "World";
        const greeting = `Hello, ${name}!
This is a multiline string.`;
        console.log(greeting);
        ```

## 7. Error Handling

*   **Practice:** Use `try...catch` for Synchronous Error Handling.
    *   **Explanation:** The `try...catch` statement allows you to test a block of code for errors and to handle them gracefully.
    *   **Code Snippet:**
        ```javascript
        try {
          // Code that might throw an error
          // const result = riskyOperation();
          // console.log(result);
        } catch (error) {
          console.error('An error occurred:', error.message);
          // Handle the error, e.g., show a user-friendly message
        } finally {
          // Code that will always execute, regardless of an error
          // console.log('Operation finished.');
        }
        ```
*   **Practice:** Use `.catch()` for Promise Rejections and `try...catch` with `async/await`.
    *   **Explanation:** For Promises, chain a `.catch()` method to handle rejections. When using `async/await`, wrap `await` calls in `try...catch` blocks to handle rejected Promises.
*   **Practice:** Throw `Error` Objects, Not Strings.
    *   **Explanation:** When throwing errors, throw instances of the `Error` object or custom error classes derived from `Error`. This provides more information (like stack traces) than throwing plain strings.
    *   **Code Snippet:**
        ```javascript
        function divide(a, b) {
          if (b === 0) {
            throw new Error('Division by zero is not allowed.');
          }
          return a / b;
        }
        ```
*   **Practice:** Create Custom Error Types for Specific Errors.
    *   **Explanation:** For more specific error handling, you can create custom error classes that extend the base `Error` class. This allows you to differentiate between types of errors in your `catch` blocks.
    *   **Code Snippet:**
        ```javascript
        // class NetworkError extends Error {
        //   constructor(message, status) {
        //     super(message);
        //     this.name = 'NetworkError';
        //     this.status = status;
        //   }
        // }

        // try {
        //   // Simulate a network request
        //   // throw new NetworkError('Failed to fetch data', 500);
        // } catch (error) {
        //   if (error instanceof NetworkError) {
        //     console.error(`Network Error (Status ${error.status}): ${error.message}`);
        //   } else {
        //     console.error('An unexpected error occurred:', error.message);
        //   }
        // }
        ```

## 8. Code Style and Readability

*   **Practice:** Adopt Consistent Naming Conventions.
    *   **Explanation:**
        *   `camelCase` for variables and functions (e.g., `myVariable`, `calculateTotal`).
        *   `PascalCase` (UpperCamelCase) for classes and constructors (e.g., `MyClass`, `UserService`).
        *   `UPPERCASE_SNAKE_CASE` for constants representing truly immutable values (e.g., `API_KEY`, `MAX_USERS`).
*   **Practice:** Write Meaningful Comments.
    *   **Explanation:** Write comments to explain *why* something is done a certain way if it's not obvious, or to clarify complex logic. Avoid comments that just restate what the code clearly does. Use JSDoc for documenting functions, classes, and parameters.
*   **Practice:** Consistent Formatting (Indentation, Spacing, Braces).
    *   **Explanation:** Consistent formatting makes code easier to read and understand. Use a linter (like ESLint) and a formatter (like Prettier) to automate this. Common practices include:
        *   2 or 4 spaces for indentation (be consistent).
        *   Spaces around operators (e.g., `x + y` not `x+y`).
        *   Opening braces on the same line as the statement (e.g., `if (condition) {`).
        *   Meaningful whitespace to group logical blocks of code.
*   **Practice:** Keep Functions Short and Focused.
    *   **Explanation:** Small functions that do one thing well are easier to understand, test, and reuse.

## 9. DOM Manipulation (General)

*   **Practice:** Minimize Direct DOM Manipulation.
    *   **Explanation:** While sometimes necessary, frequent direct manipulation of the DOM can be slow and lead to complex, hard-to-maintain code. When using frameworks (like React, Angular, Vue), let the framework handle DOM updates. If working with vanilla JS, batch DOM changes where possible and be mindful of performance.
*   **Practice:** Cache DOM Elements.
    *   **Explanation:** If you're repeatedly querying the same DOM element, store it in a variable instead of re-querying it every time.
    *   **Code Snippet:**
        ```javascript
        // Good: Cache the element
        // const myButton = document.getElementById('myButton');
        // myButton.addEventListener('click', () => { /* ... */ });
        // myButton.textContent = 'Clicked!';

        // Avoid: Repeated queries
        // document.getElementById('myButton').addEventListener('click', () => { /* ... */ });
        // document.getElementById('myButton').textContent = 'Clicked!';
        ```

## 10. Security Considerations

*   **Practice:** Avoid `eval()` and `new Function(string)`.
    *   **Explanation:** These functions execute arbitrary code from strings, which is a major security risk if the string can be influenced by external input (e.g., user input). It also makes code harder to optimize and debug.
*   **Practice:** Sanitize User Input.
    *   **Explanation:** When displaying user-provided data or using it in queries, always sanitize it to prevent Cross-Site Scripting (XSS) attacks or injection vulnerabilities. How you sanitize depends on the context (e.g., using `textContent` instead of `innerHTML` for displaying text, using parameterized queries for databases).
*   **Practice:** Don't Expose Sensitive Information in Client-Side Code.
    *   **Explanation:** API keys, secret tokens, or other sensitive data should not be hardcoded or directly accessible in client-side JavaScript, as it can be easily viewed by anyone inspecting the code.

This list provides a solid foundation for writing high-quality JavaScript code.
