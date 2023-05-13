---
title: "De-Proxifying JavaScript Proxy"
seoTitle: "Proxy in JavaScript."
seoDescription: "This article explains all that you need to know to get started with JavaScript proxy object. It also explains what object internal methods and traps."
datePublished: Sun May 07 2023 15:37:21 GMT+0000 (Coordinated Universal Time)
cuid: clhdkuxgh00010akz9pojgibk
slug: de-proxifying-javascript-proxy
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/ieic5Tq8YMk/upload/00d4ab30b1b0238271888fa98a09a4bb.jpeg
tags: programming-blogs, javascript, web-development, reactjs, frontend-development

---

What's so interesting about JavaScript Proxy? Why should I learn it? What is even a proxy in JavaScript? I've never heard about it.

If these are some thoughts popping into your mind after reading the not-so-interesting title, you're on the right track. In this article, I'll try to cover the fundamentals of proxy objects in JavaScript.

Proxy is nothing but a JavaScript object with the capability of modifying the behavior of another object. As simple as that. Proxy constructor takes the object as a first argument (which we'll see later on in the article) and returns its proxy object. This proxy object can be used in place of the original object. But keep in mind that this proxy object can redefine the behavior of the original object such as getting, setting, deleting etc.

How is it possible to do that? Well, it's all due to the handler object. The handler object is passed as a second argument to the Proxy constructor which consists of functions to modify the behavior of the target object passed as the first argument. Enough talk, let's see proxy in action:

```javascript
const object1 = { 
    name: "John Doe",
    role: "SUPERUSER"
}

const handler = {
    get(target, property, receiver) {
       return "hello";
    }
}

const proxy = new Proxy(object1, handler); 

console.log(proxy.name); // output: "hello"
console.log(object1.name); // output: "John Doe"
```

As seen in the code above, Proxy object is created with the target object and its handler object. When we call **proxy.name**, the **get()** method defined inside the handler object is triggered and it returns "hello". The get() method is corresponding to the ***object internal method* ( i.e. \[\[GET\]\])**. These methods inside handler object are often termed ***traps***\*.\* Let's quickly learn what they are as they are an important part of the working mechanism of Proxy objects.

### Object Internal Methods

An object in JavaScript is a collection of key-value pairs which are known as the properties of the object. Each object has some internal methods that define how to interact with the object. These methods are known as object internal methods. For example, when you try to access object1.name, the following steps occur:

1. "name" property is searched in object1 through the prototype chain.
    
2. The getter is invoked to get the value of "name".
    

And where is this getter defined? It is in the \[\[Get\]\] internal method. **object1.name** simply invokes whatever is defined in the \[\[Get\]\] internal method of that object. So, you cannot always be sure what value you will get when you get the property value of an object. As you have already noticed, when we try to get **proxy.name** in the code above, we get "hello" instead of "John Doe". It is because we've modified the \[\[Get\]\] internal method of the proxy object through the trap: **get().**

### Traps

They are what their names suggest. They are the traps that intercept the actual object's internal method calls. They define the behavior of the corresponding internal methods. For example, the **get() trap** defines the behavior of **\[\[Get\]\]**. **set() trap** defines the behavior of **\[\[Set\]\]** internal method and so on. Handler object in Proxy consists of these traps that redefine the actual internal methods. Let's take a closer look at the get() method for example:

```javascript
const handler = {
    get(target, property, receiver) {
       return "hello";
    }
}
```

You can see that get() method takes three arguments: target, property and receiver:

1. target: It is the original object which is passed as the first argument in the Proxy constructor. In our case, it is the **object1**
    
2. property: It is the key that was invoked during the call from proxy object. In our case, when we invoke proxy.name, the **name** becomes the property.
    
3. receiver: It is the proxy object itself. If you call the try to invoke the name again inside get() trap using the receiver (e.g. receiver.name) you will enter the infinite recursion.
    

Similarly, if you look at the set() trap it also takes three properties: target, property and value. Value is the value to be set for the property. For example:

```javascript
object1.name = "Micheal"
```

Here, object1 is the target, name is the property and "Micheal" is the value. You can find more about set() trap in [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/set).

Anyways, you got the idea. By allowing to modify the object internal methods through the help of traps, Proxy object in JavaScript can modify pretty much any behaviors of the object. Anything that is not changed while implementing custom operations are known as **invariants.**

You can find the list of object internal methods and their corresponding traps in the image below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683472826971/db30c0b9-ee39-4ab8-acb9-7da04b950e86.png align="center")

Another thing you should know is that **Proxy** provides a static method: **revocable().** As the method name suggests, it returns the **proxy object** along with **revoke()** method wrapped in the single object.

```javascript
const obj = {
    name: "Programmer",
    isAdmin: false
}

const handler = {
    
    get(target, property, receiver) {
        if(property === 'isAdmin') {
            return false;
        }
        return Reflect.get(...arguments);
    }
}

const proxyWrapper = Proxy.revocable(obj, handler);

const proxyObject = proxyWrapper.proxy;

// when you want to revoke the proxy
proxyWrapper.revoke();
```

Once the proxy is revoked, it becomes unusable. Any call to the internal methods through proxy object will throw TypeError.

That's it! You can now securely add a new weapon into your toolkit: **Proxy in JavaScript.** These are only the basics though. The article would be too long if I try to squeeze all the topics on Proxy in this article, and it would be too boring. You can explore more on Proxy [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy).

That's it from my side. If you love this article, do share it with your fellow JavaScripters. Also, feedback is always welcome, anywhere, anytime. Just let me know!!!