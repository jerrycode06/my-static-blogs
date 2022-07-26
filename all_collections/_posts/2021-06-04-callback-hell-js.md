---
layout: post
title: Callback Hell and How to Rescue it?
date: 2021-06-04
categories: ["web", "javascript"]
---

![Intro Image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m9phbghld7ihlxgoulzs.jpg)

For understanding the concept of callbacks and callback hell, I think you should know about **Synchronous** and **Asynchronous** programming in JavaScript(or any other language). Let's see a quick view on these topics in context of JavaScript.

## Synchronous Programming

It is a way of programming in which you can perform only one task at a time and when one task is completed we move to another task. This is what we called **Blocking Code** operation because you need to wait for a task to finish to move to the next one.

```javascript
console.log("Program Starts");
let sum = getSum(2, 3);
console.log(sum);
console.log("Program Ends");
```

In the above code snippet, you see code will execute line by line and when an operation on one line is finished then we move to the next line so this is just a simple example of the synchronous way of programming and we do this in our daily life of programming.

## Asynchronous Programming

Asynchronous programming allows you to perform that work without blocking the main process(or thread). It’s often related to parallelization, the art of performing independent tasks in parallel, that is achieved by using asynchronous programming.
In asynchronous operation, you can move to another task before the previous one finishes, and this way you can deal with multiple requests simultaneously.
In JavaScript, a good example of asynchronous programming is `setTimeout` function, let's see a quick example -

```javascript
console.log("Program Starts");
setTimeout(() => {
  console.log("Reading an user from database...");
}, 2000);
console.log("Program Ends");
```

So, the output of this program will look like -

```
Program Starts
Program Ends
Reading an user from database...
```

Pretty cool, right? Our program didn't wait for `setTimeout` to finish, just goes for the next line, then came back to the function and prints the output. This is what we called **Non Blocking** code. You can read more about it [here](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Introducing).
There are three design patterns in javascript to deal with asynchronous programming -

- Callbacks
- Promises
- async/await (just a syntactical sugar of promises)

## Callbacks

Callbacks is a great way of handling asynchronous behavior in javascript. In JavaScript, everything behaves like an object so functions have the type of object and like any other object (strings, arrays, etc) you can pass functions as an argument to other functions and that's the idea of callback.

```javascript
function getUser(id, callback) {
  setTimeout(() => {
    console.log("Reading an user from database...");
    callback({ id: id, githubUsername: "jerrycode06" });
  }, 2000);
}

getUser(1, (user) => {
  console.log("User", user);
});
```

You see, we are passing function as an argument to `getUser` function and it calls inside the `getUser` function, output will look like -

```
Reading an user from database...
User {id: 1, githubUsername: 'jerrycode06'}
```

## Callback Hell

In above code snippet, we are getting user with github username now let's suppose you also want repositories for that username and also commits in the specific repository so what can we do with the callback approach -

```javascript
getUser(1, (user) => {
  console.log("User", user);
  getRepositories(user.githubUsername, (repos) => {
    console.log(repos);
    getCommits(repos[0], (commits) => {
      console.log(commits);
      // Callback Hell ("-_-)
    }
})
```

You are seeing now a nesting of functions here and code also looks scary and this is what we called **Callback Hell**. For a big application it creates more nesting.
![Callback Hell](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/b8euo2n7twvgh3dbuatd.jpeg)
To avoid this, we will see now **Promises**.

## Promises

Promises are the alternative to callbacks for delivering the results of asynchronous computation. They require more effort from implementors of asynchronous functions, but provide several benefits for users of those functions. They are more readable as compared to callbacks and promises has many applications like `fetch` in javascript , `mongoose` operations and so on. Let's see how to implement promises with above example. Actually, promises have four states -

- fulfilled - The action relating to the promise succeeded
- rejected - The action relating to the promise failed
- pending - Hasn't fulfilled or rejected yet
- settled - Has fulfilled or rejected
  ![Promises](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p60413nfwvkuezgtzzcx.png)
  First we have to create promises to understand this -

```javascript
function getUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("Reading from a database....");
      resolve({ id: id, githubUsername: "jerrycode06" });
    }, 2000);
  });
}

function getRepositories(username) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(`Extracting Repositories for ${username}....`);
      resolve(["repo1", "repo2", "repo3"]);
      // reject(new Error("Error occured in repositories"));
    }, 2000);
  });
}

function getCommits(repo) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("Extracting Commits for " + repo + "....");
      resolve(["commits"]);
    }, 2000);
  });
}
```

We created three functions, instead of passing callback function we are now returning a Promise which has two argument **resolve** and **reject**. If everything worked, call `resolve` otherwise call `reject`. Let's see how to use promises -

```javascript
// Replace Callback with Promises to avoid callback hell
getUser(1)
  .then((user) => getRepositories(user.githubUsername))
  .then((repos) => getCommits(repos[0]))
  .then((commits) => console.log("Commits", commits))
  .catch((err) => console.log("Error: ", err.message));
```

More readable, Isn't it? Using arrow functions made this less complex than using simple functions. We have avoided nesting of functions and reduce the complexity of code (callback approach) and that's how promises work. You can deep-dive more about promises [here](https://web.dev/promises).

## async/await

It is supposed to be the better way to write promises and it helps us keep our code simple and clean.
![Aync-Await](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3s7u6tt9iqs9k80dtdkn.png)
All you have to do is write the word `async` before any regular function and it becomes a promise. In other words `async/await` is a syntactical sugar of using promises it means if you want to avoid chaining of `then()` methods in promises, so you can use the `async/await` approach but internally it also uses the chaining.
Let's see how to implement it with above example -

```javascript
// Async- await approach
async function displayCommits() {
  try {
    const user = await getUser(1);
    const repos = await getRepositories(user.githubUsername);
    const commits = await getCommits(repos[0]);
    console.log(commits);
  } catch (err) {
    console.log("Error: ", err.message);
  }
}

displayCommit();
```

Now, it is more readable than using promises above. Every time we use `await`, we need to decorate this with a function with `async`. Like promises, we don't have `catch()` method here so that's why we are using `try-catch` block for the error handling.

## Conclusion

In this article we've seen -

- Synchronous vs Asynchronous
- Callbacks and Callback Hell
- Avoid Callback hell with promises and async/await

I personally like the async/await approach the most but sometimes we should the promises approach to deal with async behaviour.

Thanks for reading this long post! I hope it helped you understand these topics a little better.
