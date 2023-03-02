---
title: "You write JavaScript. But do you know it?"
seoTitle: "JavaScript under the hood"
datePublished: Sun Dec 25 2022 04:38:19 GMT+0000 (Coordinated Universal Time)
cuid: clc2vr4f5000508maaizq9rz7
slug: you-write-javascript-but-do-you-know-it
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/6fb2d0ede7521f357c0dc54a31f2d140.jpeg
tags: programming-blogs, javascript, javascript-framework, programming-ciovqvfcb008mb253jrczo9ye, programming-languages

---

How does Javascript even work? How on earth does it allow calling a function before defining it? And that too only on some scenarios? What's the deal with multiple ways to define a variable? And, why am I banging my fist on the desk now and then while working with Javascript? (At least, I was, one and a half years ago).

Have any of these questions ever crossed your mind? If yes, then this is the right place for you. Even if not, what's wrong with learning something new or refreshing the concept?

Enough with the questions! Let's jump right into today's agenda- to learn how JavaScript works.

Let's take a piece of code:

```javascript
var name = "Anurag";
printName();
function printName(){
    console.log("My name is", name);
}
```

How is the above code working? To answer that, we've to move to the beginning. JavaScript is a synchronous single-threaded language. Basically, it means that JavaScript can execute only one statement at a time in a specific order (top-down). The main thread where the JavaScript executes is known as the **Call Stack**.

Inside the call stack, **Execution Context** is created before our code is executed. Execution Context is the container in which our JavaScript is executed. Execution Context has two components:

1. **Memory Component**: It is the place where all variables and functions are stored as a key-value pair. It is also known as a **variable environment**.
    
2. **Code Component**: It is the place where our code is executed. It is also known as a **thread of execution**.
    

Execution Context is created in two phases: The memory creation phase and the Code execution phase. To simplify it, the components of the execution context are created in two steps.

**Memory Creation Phase**

At first, the Memory Creation phase occurs, where the JavaScript engine goes through our line by line from top to bottom. Whenever it encounters variables and functions, it allocates memory for them:  
For variables, it reserves memory and assigns a special value **undefined.**  
For functions, it copies the entire function definition in the memory space.

Till now, it would look something like this if I were to represent it pictorially:

| Memory Creation Phase | Code Execution Phase |
| --- | --- |
| name: undefined |  |
| nothing to allocate memory in line no. 2 |  |
| printName: {......} |  |

**Code Execution Phase**

After the memory has been assigned to all the variables and functions, the code execution finally starts (from the beginning again).  
The first line is var name = "Anurag". So, finally "Anurag" is assigned to "name" in the code execution phase. Till now, the "name" was undefined as you can see in the table above.

The second line is the execution of the "printName" function. There is a little surprise for you guys if you don't know it. Functions in JavaScript are completely different than in other languages. They are like mini-programs in themselves. Since they are like a program in themselves, **a new execution context is created on top of the previous execution context whenever a function call happens**. The previous execution context is called **Global Execution Context** whereas the newly created execution context for the function is called **Function Execution Context**. Then, the program flows to the new execution context and the same process starts i.e. memory creation phase occurs, and code execution phase and so on.

Till now, if we were to visualize:

| Memory Creation Phase | Code Execution Phase |
| --- | --- |
| Nothing to allocate memory as there are no variable assignments and function definitions. |  |

| Memory Creation Phase | Code Execution Phase |
| --- | --- |
| name: undefined | "Anurag" gets assigned to "name" variable. |
| nothing to allocate memory in line no. 2 | New execution is created (see above table) when printName() function is called. |
| printName: {......} |  |

The first table (Function execution context) is created on top of the second table (Global Execution Context).

Now that we know how the execution context of a function is created, let's see what happens inside. There is simply a console.log statement and no variable assignments and function definitions. So, nothing happens in the memory creation phase.

In the code execution phase, the function tries to execute  
console.log("My name is", name).

How does it know where to look for "name" ? At first, the function will look in its own execution context to check if there is a variable named "name" ? If there is not, then it checks the parent's execution context (Global Execution Context in this case). There it finds the variable named "name". As it already has value "Anurag", it prints:

```bash
My name is Anurag
```

The above process of looking for values into parent's scope is known as **scope chaining**. It also involves other concepts such as **lexical environment,** but lets move forward for now.

Once, everything inside function printName() is executed, its execution context gets deleted.

And then, finally the program flow comes back to the global execution context. Since there is nothing to execute for function definition (note: function definition and function call are two different things), the Global Execution Context is also deleted and the Call Stack becomes empty. Finally, the program terminates.

Phew ! That was a lot. But, it was necessary. Many other Javascript fundamentals such as hoisting, scope chaining, closures are based on this core concept.

If you read the whole article and learned something new, congrats to you! You are now a better JavaScript developer who can now explain how JS works. Of course, there are many other core concepts I left intentionally because the article is getting too long. I may cover them in other separate articles. But, please do provide me a feedback on this if you read it ! Thank you !