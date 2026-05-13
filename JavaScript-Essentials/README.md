# JavaScript Essentials – Walkthrough

![JavaScript](https://img.shields.io/badge/JavaScript-Essentials-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-Beginner-blue?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge)

---

## 📌 Overview

This repository documents my learning journey through the **JavaScript Essentials** module.  
It covers core JavaScript fundamentals including syntax, functions, DOM manipulation, and event handling.

---

## 🎯 Learning Objectives

By completing this module, I gained practical understanding of:

- JavaScript syntax and structure  
- Variables and data types  
- Functions and reusable logic  
- Arrays and objects  
- DOM manipulation  
- Event handling  
- Debugging using browser developer tools  

---

## 🧠 JavaScript Fundamentals

### 🔹 Variables

JavaScript provides three ways to declare variables:

- `let` → block-scoped, reassignable  
- `const` → block-scoped, constant  
- `var` → function-scoped (legacy usage)

```javascript
let name = "User";
const age = 25;
```

📸 Screenshot Placeholder: Variable declaration and console output

---

### 🔹 Data Types

JavaScript supports multiple data types:

- String  
- Number  
- Boolean  
- Array  
- Object  

```javascript
let isStudent = true;
let score = 95;

console.log(typeof isStudent);
console.log(typeof score);
```

📸 Screenshot Placeholder: Console showing typeof results

---

### 🔹 Functions

Functions allow reusable blocks of logic:

```javascript
function greet(name) {
    return "Hello " + name;
}

console.log(greet("John"));
```

📸 Screenshot Placeholder: Function execution in console

---

### 🔹 Arrays & Objects

#### Arrays

```javascript
let fruits = ["Apple", "Banana", "Mango"];
console.log(fruits);
```

#### Objects

```javascript
let user = {
    name: "Alex",
    age: 30
};

console.log(user.name);
```

📸 Screenshot Placeholder: Array and object output in console

---

### 🔹 DOM Manipulation

JavaScript can dynamically modify HTML content:

```javascript
document.getElementById("title").innerText = "Hello World";
```

📸 Screenshot Placeholder: Before and after DOM update

---

### 🔹 Events

Handling user interactions:

```javascript
document.getElementById("btn").addEventListener("click", function() {
    alert("Button clicked!");
});
```

📸 Screenshot Placeholder: Button click event in action

---

## 📂 Project Structure

```
JavaScript-Essentials/
├── index.html
├── script.js
├── styles.css
├── README.md
└── screenshots/
```

📸 Screenshot Placeholder: Folder structure in VS Code

---

## 🚀 How to Run This Project

1. Open index.html in your browser  
2. Open Developer Tools (F12)  
3. Go to Console tab  
4. View outputs and logs  
5. Interact with UI elements  

---

## 🧪 Key Takeaways

- JavaScript adds interactivity to web pages  
- Functions improve code reusability  
- DOM enables dynamic updates to HTML  
- Events handle user interactions  
- Data structures (arrays/objects) organize information efficiently  

---

## 🏁 Conclusion

This module helped build a strong foundation in JavaScript programming and frontend development concepts.

---

## 📌 Author

Sanjish KC  
GitHub: https://github.com/sanjishkc

---

## ⭐ Support

If you like this project, consider giving it a star on GitHub!
