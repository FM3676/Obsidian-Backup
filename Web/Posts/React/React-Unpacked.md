---
title: "React Unpacked"
date: 2023-12-23T23:08:14Z
lastmod: "2023-12-23"
tags: ["React", "Behind The Scene", "Tutorial"]
draft: false
summary: "This article unpack react"
layout: PostSimple
bibliography: references-data.bib
---

# React Unpacked

## Understanding Frontend Frameworks

### What is a Frontend Framework?

React and Vue are often at the center of the debate between being a `library` and a `framework`. Fundamentally, they are libraries focused on building UIs with features like:

- Declarative rendering based on state
- Component-based development

### Evolution into Frameworks

As applications evolve into Single Page Applications (SPAs), additional tools integrate with React and Vue to handle routing (React-router for React, Vue-router for Vue), and for managing increasingly complex states (redux/react-redux for React, vuex/pinia for Vue).

### Beyond the Core

React and Vue do not directly support other functionalities such as build tools, data flow solutions, or documentation utilities. These, combined with the core library, form what is referred to as a `framework` or a `full-stack solution`. Examples include:

- `UmiJS`: A framework based on React with built-in routing, building, and deployment features.
- `Next.JS`: A framework based on React, supporting Server-Side Rendering (SSR) and Static Site Generation (SSG).

### Conventional Terminology

The terms "React framework" and "Vue framework" often refer to the entirety of the ecosystem surrounding React and Vue, not just the core libraries.

### Core Aspects of Modern Frontend Frameworks

A modern frontend framework typically includes:

- Declarative rendering based on state
- Component-based development for reusability
- Frontend routing solutions
- State management solutions

### Interview Question Answer

With the shift from Multi-Page Applications (MPA) using jQuery to Single Page Applications (SPA), the volume of client-side JS code surged. Managing this complexity without a structured approach became burdensome.

React and Vue emerged as libraries facilitating declarative state-based rendering and componentization, which are essential as applications grow. Alongside their supporting ecosystems including routers and state management tools, they form the modern frontend framework.

Today, React and Vue are colloquially termed as `frameworks`, reflecting their combined utility with their respective ecosystems to address:

- Declarative state-based rendering
- Component-based development
- Client-side routing
- State management

## React vs. Vue: Differences in Describing Pages

React and Vue are two prevalent frameworks in the front-end development landscape, each with its unique approach to describing User Interfaces (UI). Here is a summary of how each framework accomplishes this task:

### React's JSX

- **Origin & Nature**: React's JSX is a JavaScript syntax extension that resembles XML. It originated from the React team's philosophy of coupling UI with logic, as they are inherently related (e.g., event handling, dynamic styling changes). By using JavaScript to describe the UI, which front-end engineers are most familiar with, React aims for a more cohesive pairing of UI and logic.

- **Conversion Process**: JSX is transformed into JavaScript objects representing virtual DOM nodes by using compilers like Babel.

- **Flexibility**: Because JSX is essentially JavaScript, it can be very flexible, allowing JSX to be used within conditional statements, assigned to variables, passed as arguments, and returned from functions.

- **Example**:

  ```jsx
  function App({ isLoading }) {
    if (isLoading) {
      return <h1>Loading...</h1>;
    }
    return <h1>Hello World</h1>;
  }
  ```

- **Community Impact**: The flexible nature of JSX has allowed React developers to construct complex UI libraries and enrich the ecosystem, which in turn has attracted more contributors.

### Vue's Template Syntax

- **Historical Context**: Vue's template syntax has its roots in the traditional back-end templating engines. These engines allowed for the creation of HTML with placeholders, which would be filled with data server-side before sending the HTML to the client.

- **Template Philosophy**: Vue's template syntax is an extension of HTML designed to express logic directly within the markup. It caters to developers who prefer the separation of concerns, where HTML is augmented to handle a certain level of logic.

- **Back-end Familiarity**: The template syntax of Vue is quite similar to many server-side templating engines, making it more approachable for those with a background in back-end development.

- **Example**:

  ````html
  <template>
    <div>
      <h1 v-if="isLoading">Loading...</h1>
      <h1 v-else>Hello World</h1>
    </div>
  </template>
  ```
  ````

- **Diversity of Templating Engines**: There's a wide variety of templating engines across different back-end technologies, such as JSP, Thymeleaf, and Jinja2, among others, which inspired Vue's template syntax.

- **Code Snippets from Various Templating Engines**:

  - Twig:
    ```twig
    {% for user in users %}
      * {{ user.name }}
    {% else %}
      No users have been found.
    {% endfor %}
    ```
  - Blade:

    ```blade
    <html>
      <head>
        <title>Application Name - @yield('title')</title>
      </head>
      <body>
        @section('sidebar')
          This is the master sidebar.
        @show

        <div class="container">
          @yield('content')
        </div>
      </body>
    </html>
    ```

  - EJS:
    ```ejs
    <h1>
      <%= title %>
    </h1>
    <ul>
      <% for (var i=0; i<supplies.length; i++) { %>
        <li>
          <a href='supplies/<%= supplies[i] %>'>
            <%= supplies[i] %>
          </a>
        </li>
      <% } %>
    </ul>
    ```

### Conclusion

React and Vue offer distinct approaches for developers to describe UI elements.

React uses JSX to leverage the full power of JavaScript in crafting the UI, which can be a more natural fit for those with strong JavaScript skills.

Vue offers a template syntax that is reminiscent of traditional HTML, providing an easier transition for developers with a background in HTML and back-end templating engines.

Both methods have their own strengths and trade-offs, and the choice between them often comes down to personal preference or specific project requirements.

## Frontend Framework Classification Summary

### Declarative Rendering Based on State

Modern frontend frameworks are characterized by their declarative rendering approach, which is driven by the application state. This relationship can be summarized by the formula:

```plaintext
UI = f(state)
```

- `state`: The current state of the view.
- `f`: The internal mechanism of the framework.
- `UI`: The view description in the host environment.

This relationship implies that changes in the state (`state`) lead to changes in the UI (`UI`), making `state` the independent variable and `UI` the dependent variable.

### React's Approach with Hooks

In React, hooks like `useState` are used to define state variables:

```javascript
const [x, setX] = useState(0);
```

The `useState` hook allows us to define a state variable (`x`), changes to which will trigger updates to the dependent UI.

Dependent variables in React can be without side effects or with side effects:

- **Without Side Effects**: Using `useMemo` in React defines a computed value that only changes when its dependencies change.

  ```javascript
  const y = useMemo(() => x * 2 + 1, [x]);
  ```

- **With Side Effects**: Using `useEffect` in React to perform side effects based on state changes.

  ```javascript
  useEffect(() => (document.title = x), [x]);
  ```

### Framework Classification

Different frameworks handle the computation of the UI from state (`step one`) and the subsequent manipulation of host environment APIs (`step two`) in varying ways. The primary distinction among frameworks lies in how they implement the first step.

### Component State Example

An application with components A, B, and C showcases how state variables (`a`, `b`, `c`) and their changes can affect the UI and propagate through components:

- Component A has state variable `a` and dependent variable `b`.
- Component B receives `b` as a prop and has its own state variable `c`.
- Component C receives `a` as a prop, making it a state variable for C.

### Understanding Variable Relationships

The relationships between state variables (`a`, `b`, `c`) and the UI can be organized into three dimensions:

1. **State Variables and UI Correspondence**: How changes in state variables affect UI updates.
2. **State Variables and Component Correspondence**: How state changes affect component updates.
3. **State Variables and Application Correspondence**: How state changes affect the overall application UI.

## Virtual DOM

### What is the Virtual DOM?

The virtual DOM is essentially a plain JavaScript object that represents the structure of a UI. Introduced by the React team, it is a programming concept where the UI is maintained in memory in an idealized or "virtual" representation.

### Advantages of the Virtual DOM

#### Performance and Size Benefits Over the Real DOM

- Calculations at the JS level are faster than at the DOM level due to the browser's rendering process and the vast number of attributes associated with DOM objects.
- The virtual DOM shows its real advantages during updates, not the initial rendering. While updating, the virtual DOM only needs to modify the necessary DOM nodes instead of destroying and recreating all nodes as with methods like `innerHTML`.

#### Multi-Platform Rendering Abstraction

The virtual DOM provides a way to describe UIs that can be rendered across different environments:

- Web browsers and Node.js environments use the ReactDOM package.
- Native environments use the ReactNative package.
- Canvas, SVG, or VML (for IE8) environments use the ReactArt package.
- The ReactTest package is used to render JavaScript objects for easily testing "universal features" that do not belong to any host environment.

### Virtual DOM in React

In React, UI is described with JSX, which is transformed into calls to a `createElement` function. This function returns the virtual DOM object, which is used to describe the DOM structure. React elements are the objects returned from this function, which the React team prefers to call instead of virtual DOM objects.

### Answering a Common Interview Question

**Question:** What is the Virtual DOM and what are its advantages?

**Answer:** The virtual DOM is a concept introduced by the React team. It is a programming idea that refers to the description capacity of the real UI DOM. The advantages include:

- Faster computation at the JS level compared to the real DOM, especially during updates.
- The ability to abstract rendering across multiple platforms, allowing for the same description of the UI to be rendered in different environments using appropriate packages.
