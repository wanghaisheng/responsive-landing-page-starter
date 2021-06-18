---
title: The functions that aren't pure
description: Explore pure functions concept from the functional programming paradigm.
author: igor-wojda
published: true
published_at: 2021-06-18T07:37:41.506Z
updated_at: 2021-06-18T07:37:41.532Z
category: inspiration
tags:
  - functionalprogramming
  - purefunction
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Every day more developers become aware of the functional programming paradigm. This paradigm promises bug-free, efficient code because pure functions are easier to test and parallelize. 

In practice full fledge functional applications are still an abstract thing, however, certain concepts from the functional programming paradigm become more and more often applied to non-functional languages. Also, this "partially functional" approach helps to solve many common "problems" in a better way. Today we will take a closer look at one of such functional concepts - pure/impure functions.

> NOTE: We will use Kotlin based pseudo code to define our functions, but this code is basic, so it will be easily understood.

## Pure Functions

The `pure function` is a function that does not have any side effects - in other words, this function should not retrieve and modify any values other than values passed as params.  This means that each function call with the same arguments will always result in the same output (returned value). Let's start by defining a pure function first:

```
fun max(a: Int, b: Int) {
if (a > b) 
    return a
else
    return b	
}
```

The `max` function takes two arguments, two numbers, and returns larger of them. Notice that this function does not access or modifies any values from outside function scope, so it is a `pure function`.  We can be sure that calling this function with arguments `2` and `7` will ALWAYS return `7`.

To better understand this concept let's take a closer look a closer look at the other side of the same coin - the various ways of breaking function purity.

## Impure Functions

The `impure function` is the function that has side effects - it is modifying or accessing values from outside function scope (outside of function body). 

### Quite obvious side effects

The simplest example is the function that modifies the external property to store a state.

```
val loalScore = 0

val getScore(score: Int): Int {
    loalScore = score
    return loalScore
}
```

In this case, impurity does not manifest itself strongly because subsequent method calls will return the same value:

```
getScore(12) // returns 12
getScore(6) // returns 6
getScore(3) // returns 3
```

This function modifies external value but still returns the same value on each invocation, because of the value assignment. However, this is not always the case. Let's consider another impure function:

```
val addScore = 0

val addScore(score: Int): Int {
    loalScore + score
    return loalScore
}
```

This time function has "stronger impurity" because it modifies external value and returns the different result on each invocation - the state is stored in the variable, outside the function scope:

```
addScore(12) // returns 12
addScore(6) // returns 18
addScore(3) // returns 21
```

The downside of keeping the state is that sometimes testing is more complex, however, in many applications, this cannot be avoided. 

The next function is accessing the value from outside the function scope and this makes it impure:

```
fun getString(length: Int): String {
    return Random().nextString(length)
}
```

This time each function call with the same argument will result in a different value being returned:

```
getString(2) // returns "ab"
getString(2) // returns "hh"
getString(2) // returns "zk"
```

Presented side effects are quite a clear example, however side effects can be also more subtle:

### Not so obvious side effects

An interesting side-effecting scenario is a modification of the object passed as a function argument:

```
fun increaseHeight(person: Person) {
    person.height++
}
```

Calling this function multiple times with the same Person instance will lead to different output because the value outside the function (stored in Person instance) is modified. 

Exception thrown by a function is a great example of the side effects that are hard to spot:

```
fun addDistance (a:Int, b:Int): Int {
    if(a < 0) {
        throw IllegalAccessException("a must be >= 0")
    }
     
    return a + b
}
```

Another interesting way of creating side effects is simply by calling another side-effecting function:

```
fun firstFunction() {
    addDistance(-5, 7)
}

fun addDistance (a:Int, b:Int): Int {
    if(a < 0) {
        throw IllegalAccessException("a must be >= 0")
    }
     
    return a + b
}
```

Another not so obvious side effect is logging. Let's take a look at this real-life sample from our sample [Base Video Chat](https://github.com/opentok/opentok-android-sdk-samples/blob/main/Basic-Video-Chat/app/src/main/java/com/tokbox/sample/basicvideochat/MainActivity.java) application:

```
private PublisherKit.PublisherListener publisherListener = new PublisherKit.PublisherListener() {
    @Override
    public void onStreamCreated(PublisherKit publisherKit, Stream stream) {
        Log.d(TAG, "onStreamCreated: Publisher Stream Created. Own stream " + stream.getStreamId());
    }

    @Override
    public void onStreamDestroyed(PublisherKit publisherKit, Stream stream) {
        Log.d(TAG, "onStreamDestroyed: Publisher Stream Destroyed. Own stream " + stream.getStreamId());
    }

    @Override
    public void onError(PublisherKit publisherKit, OpentokError opentokError) {
        Log.d(TAG, "PublisherKit onError: " + opentokError.getMessage());
    }
};
```

In the above code, logging as a side-effects does not impact application logic but they are used to helps us to understand what's going on in the application. Latter most likley developer will use data returned by callbacks to indroduce more side-effects.

## Determine If Function Is Pure\Impure

There are two clues that a function may be impure - the function does not take any arguments or does not return any value.  Let's look at the first case:

```
list.getItem(): String
```

In the above example, the function does not take, aby params but it returns the value. This means that most likely the value is retrieved from the class state. Let's consider what happens when a function does not return any value:

```
list.setItem("item")
```

Just looking at the function name we cal tell that param will be most likely used to modify class state. 

And finally, we can have a combo where there is no argument and no value returned:

`list.sort()`

These are only clues. This does not always have to be the case but often these clues are a good purity indicator.


## Summary 

In the functional programming paradigm ideally, all functions are pure. However, in many real-world applications, the usage of impure functions cannot be avoided, especially if an application requires external resources like persistence, user input, or network data access. This breaks the purity of the function and whole application. This is not a bad thing, but it is good to be aware of this fact when working with the application code. Typically we will have a mix of pure and impure functions in a single application. It is always good to be aware of purity/impurity because this facilitates application testing and helps to avoid bugs.


