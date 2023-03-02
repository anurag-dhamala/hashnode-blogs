---
title: "Object.freeze() vs Object.seal() vs Object.defineProperty()"
seoTitle: "Object.seal() vs Object.freeze(). How to restrict JavaScript Objects?"
seoDescription: "This article covers how you can modify and restrict the JavaScript objects."
datePublished: Thu Mar 02 2023 17:43:29 GMT+0000 (Coordinated Universal Time)
cuid: clereawqa000009mpfvjqdyzg
slug: restrict-js-objects
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/eMP4sYPJ9x0/upload/863e1a1dc27090249815cc352eb4020c.jpeg
tags: javascript, javascript-framework, javascript-library, javascript-modules, programming-tips

---

When it comes to objects in JavaScript, we tend to mess up real quick if we do not pay attention to them. So, adding some restrictions to them is probably a good idea. But, how? There are multiple ways to do so, but without proper understanding, they may not match your use cases. In this article, I will explain the various methods available in JavaScript Object through the scenarios that you may encounter in your coding journey.

Let's say that you have the following object:

```javascript
const bio = {
    name: "Anurag"
}
bio.hobby = "Programming";
console.log(bio);
```

The above program will result in:

```javascript
{ name: "Anurag", hobby: "Programmer" }
```

### Scenario 1

**You have a Javascript Object. You want to restrict the addition of new properties to it. You also want to restrict the modification of already existing property.**

To match up with scenario 1, we can use **Object.freeze().**

Applying Object.freeze() to any object disables the addition of new property to it. It also restricts the modification of already existing property.

```javascript
 const bio = {
    name: "Anurag"
}
Object.freeze(bio);
bio.name = "Random";
bio.hobby = "Programming";
console.log(bio);
```

The above program will result in:

```javascript
{ name: "Anurag" }
```

### Scenario 2

**You have a Javascript object. You want to allow the modification of already existing property, but want to restrict the addition of new property.**

Well, here comes the **Object.seal()** method. It is used when you want to allow users to modify the pre-existing properties of an object, but restrict them from adding a new property.

```javascript
 const bio = {
    name: "Anurag"
}
Object.seal(bio);
bio.name = "Random";
bio.hobby = "Programming";
console.log(bio);
```

The above program will result in:

```javascript
{ name: "Random" }
```

See, how Object.seal() played its role? It allowed the "name" to be modified but did not allow for "hobby" to be added to the "bio".

### Scenario 3

**You have a JavaScript object with more than one property. You want to allow the modification of one property while restricting the other one.**

Let's create a new object to see how we can implement scenario 3:

```javascript
const job = {
    title: "Software Engineer",
    salary: "$150K"
}
```

Now, let's say that you want to allow the users to modify the "salary", but not the "title".  
We can achieve this by using **Object.defineProperty()**. It is used to manually define the properties of the objects and specify the property descriptors while doing so.

Here's how you can do it:

```javascript
const job = {
    salary: "$135K"
}
Object.defineProperty(job, "title", {
    value: "Software Engineer",
    writable: false,
    enumerable: true
    }
);

job.salary = "$200K";
job.title = "DevOps Engineer"

console.log(job);
```

The output of the above program is:

```javascript
{ salary: '$200K', title: 'Software Engineer' }
```

Let's see what we did here. We wanted to allow the modification of "salary", but not of the "title". So, the property that we want to restrict is first moved out of the object and is manually created using Object.defineProperty().

Then we say, Hey! in the "job" object, add the property with the name "title". By the way, I want its value to be equal to "Software Engineer", and I want it to be non-writable (writable: false).

Notice that **enumerable: true**? It is added because of node.js. NodeJs chooses not to display the property whose **enumerable** property descriptor is false. In web browsers, it is not needed.

We then modified the salary successfully to $200k, but couldn't modify the title to "DevOps Engineer".

### Summary

We learned how to add restrictions to our JavaScript objects.

**Object.freeze()**: It allows the users to modify the pre-existing properties of an object, but restrict them from adding a new property.

**Object.seal()**: It is more flexible than Object.freeze(). It allows the users to modify the pre-existing properties of an object but restricts them from adding a new property.

**Object.defineProperty()**: It can be used to define property with different restriction levels. It depends on you whether the property you create can be writable or not.

**That's it**! You are now ahead of many JavaScript developers who are not aware of these three awesome JavaScript Object properties. Do support me with your love and share it as much as you can if you like it.