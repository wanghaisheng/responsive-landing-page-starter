---
title: Builder Pattern with Inheritance in Java
description: This article demonstrates the Builder pattern within a class
  hierarchy, using the Messages API as an example
author: sina-madani
published: false
published_at: 2022-07-13T11:25:38.518Z
updated_at: 2022-07-13T11:25:38.530Z
category: tutorial
tags:
  - java
  - messages-api
  - design-pattern
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
The typical way we instantiate objects is through a constructor, passing in the parameters required directly. For brevity, we may group related parameters into a class and construct an instance of that class separately, passing it in to the main object as a parameter. This has the advantage of reducing the number of parameters required for object construction by breaking it down into multiple steps. However, there are several downsides to using a constructor. For one, the order in which parameters are passed is significant, which can be daunting when there are many parameters, and even more error-prone when these parameters are of the same type (e.g. String). There may also be optional parameters. Whilst we can work around this by having multiple constructors with varying signatures, this adds a lot of boilerplate and cognitive load for the user and developer. It also becomes infeasible when multiple optional parameters are of the same type.

The Builder pattern is a well-known design pattern in object-oriented languages for controlling object construction. Like many (perhaps most) design patterns, it exists to address a deficiency in the language. In languages with named parameters (such as Kotlin, Scala, Python, C#, Ruby and many others), the need for builders is diminished in many cases. Using the builder pattern is a recommendation in Effective Java, but for languages with named arguments, its value becomes questionable (see for example [this article](https://blog.kotlin-academy.com/effective-java-in-kotlin-item-2-consider-a-builder-when-faced-with-many-constructor-parameters-1927e69608e1), which explores its utility in Kotlin).

Discussions on the extent to which optional parameters obsolete the builder pattern are beyond the scope of this article. Instead, the purpose is to explain the rationale for what appears to be a complicated application of the builder pattern in the recently released [Messages API implementation in the Vonage Java SDK](https://github.com/Vonage/vonage-java-sdk/tree/main/src/main/java/com/vonage/client/messages). 

To model the various types of messages that can be sent via the [Messages API](https://developer.vonage.com/messages/overview), an object-oriented approach is used, where a class is created for every valid combination of message type and service. There is a three level inheritance hierarchy. Take the following example:

- `MessageRequest`
  - `MmsRequest`
    - `MmsVcardRequest`

`MessageRequest` and `MmsRequest` are abstract classes, and `MmsVcardRequest` is the class which represents the combination of sending a vCard over MMS. The base class `MessageRequest` takes as arguments the channel and message type in its constructor, which are set by subclasses. It also takes a Builder as a parameter, which is where the main details of the message are set. Some parameters are optional, such as `clientRef`, and all messages ahve a sender and recipient, hence these are declared in the base `MessageRequest`. However, this inheritance hierarchy is transparent to the user when constructing an `MmsVcardRequest`, which looks like this:
```java
MmsVcardRequest message = MmsVcardRequest.builder()
        .from("447900090000").to("447900090001")
        .url("https://www.example.com/contact.vcf")
        .clientRef("vCard-msg-#1")
        .build();
```
Complete examples are available from the [code samples repo](https://github.com/Vonage/vonage-java-code-snippets/tree/master/src/main/java/com/vonage/quickstart/messages).

This seems somewhat elegant from a user's perspective, given the absence of named and optional method / constructor parameters in Java. Behind the scenes however, making the code above possible with an inheritance hierarchy requires a design that is not immediately obvious or intuitive from the SDK developer's perspective. The complications relate to the nested Builder classes associated with each subclass of [`MessageRequest`](https://github.com/Vonage/vonage-java-sdk/blob/main/src/main/java/com/vonage/client/messages/MessageRequest.java). Let's look at [`MmsRequest`](https://github.com/Vonage/vonage-java-sdk/blob/main/src/main/java/com/vonage/client/messages/mms/MmsRequest.java) as an example. It's still abstract, since we're only setting the Channel. But what about that intimidating Builder declaration?

```java
protected abstract static class Builder<M extends MmsRequest, B extends Builder<? extends M, ? extends B>> extends MessageRequest.Builder<M, B>
```

The base Builder class in MessageRequest takes as parameters the type of MessageRequest to be constructed, and the type of Builder. The former (`M`) is easy to explain: `public abstract M build()` in `MessageRequest.Builder` is what the user calls once they have finished setting the parameters, returning to them the appropriate concrete MessageRequest subclass. Of course, this could be ommitted since Java inheritance supports covariant return types. That is, we could omit `M` if we wanted to and acheive the same outcome from the user's perspective. Then, `MessageRequest.Builder` becomes as follows:

```java
public abstract static class Builder<B extends Builder<? extends B>> {
    protected String from, to, clientRef;

    public B from(String from) {
        this.from = from;
        return (B) this;
    }
    
    public B to(String to) {
        this.to = to;
        return (B) this;
    }

    public B clientRef(String clientRef) {
        this.clientRef = clientRef;
        return (B) this;
    }
    
    public abstract MessageRequest build();
}
```

We then have to remember to override this in subclasses, ideally even in abstract classes like `MmsRequest.Builder`, like so:

```java
protected abstract static class Builder<B extends Builder<? extends B>> extends MessageRequest.Builder<B> {
    String url;

    protected B url(String url) {
        this.url = url;
        return (B) this;
    }

    @Override
    public abstract MmsRequest build();
}
```

The concrete class `MmsVcardRequest.Builder` would look the same, since that's where we actually declare the type:

```java
public static final class Builder extends MmsRequest.Builder<Builder> {
    Builder() {}

    public Builder url(String url) {
        return super.url(url);
    }

    @Override
    public MmsVcardRequest build() {
        return new MmsVcardRequest(this);
    }
}
```

So why are we adding this seemingly redundant parameter if we can use covariant return types? It is simply to ensure that we don't forget to override the return type. By generifying the return type, the compiler ensures that the `build()` method has the correct signature. In the absence of this, the following version of `MmsVcardRequest.Builderwould also be valid, yet inaccurate:

```java
public static final class Builder extends MmsRequest.Builder<Builder> {
    Builder() {}

    public Builder url(String url) {
        return super.url(url);
    }

    @Override
    public MmsRequest build() {
        return new MmsVcardRequest(this);
    }
}
```

Since we remembered to override the return type of `MmsRequest.Builder#build()`, the compiler will catch the following `build()` signature:

```java
@Override
public MessageRequest build() {
    return new MmsVcardRequest(this);
}
```

with error message:

*'build()' in 'com.vonage.client.messages.mms.MmsVcardRequest.Builder' clashes with 'build()' in 'com.vonage.client.messages.mms.MmsRequest.Builder'; attempting to use incompatible return type*.

This is only because we remembered to manually override the signature of the method in `MmsRequest.Builder`. If we didn't do that, there would be no error. By paramterising the return type, we are forced to declare the correct type, and we don't need to override the `build()` method in subclasses of `MessageRequest` - the compiler takes care of that for us.

Let us return to the latter Builder parameter - `B`. If you're familiar with the builder pattern, you'll know that each method call on the builder returns the builder itself, so that you can fluently chain method calls to set parameters easily. This works well when there is no inheritance, but we want the user to be able to set parameters in any order - after all, isn't that one of the main reasons for using builder pattern? Thus, we need to ensure that regardless of which methods are called first, the most specific concrete Builder class is returned. Otherwise, we lose the ability to chain method calls and would have to resort to casting the return value every time - which ruins the fluency we're trying to achieve by using a builder. To make this clearer, let's consider the case where we naively return the current builder:

```java
public abstract static class Builder<M extends MessageRequest> {
    protected String from, to clientRef;

    public Builder<M> from(String from) {
        this.from = from;
        return this;
    }

    public Builder<M> to(String to) {
        this.to = to;
        return this;
    }

    public Builder<M> clientRef(String clientRef) {
        this.clientRef = clientRef;
        return this;
    }

    public abstract M build();
}
```

This compiles fine and is usable. Then `MmsRequest`'s builder becomes the following:

```java
protected abstract static class Builder<M extends MmsRequest> extends MessageRequest.Builder<M> {
    String url;

    protected Builder<M> url(String url) {
        this.url = url;
        return this;
    }
}
```

Finally, the concrete sublcass (sticking with the `MmsVcardRequest` example) becomes as follows:

```java
public static final class Builder extends MmsRequest.Builder<MmsVcardRequest> {
    Builder() {}

    public Builder url(String url) {
        return (Builder) super.url(url);
    }

    @Override
    public MmsVcardRequest build() {
        return new MmsVcardRequest(this);
    }
}
```

Notice that we had to cast the return type of `super.url(url)` to this Builder, since the super method's return type is `com.vonage.messages.mms.MmsRequest.Builder`, not `com.vonage.messages.mms.MmsVcardRequest.Builder`. Note that we only overrode this method to add Javadocs to it, not to change its functionality. But this actually perfectly highlights the problem which the parameterised Builder type attempts to solve. To illustrate, let's attempt to use this builder:

```java
MmsVcardRequest message = MmsVcardRequest.builder()
    .from("447900090000").to("447900090001")
    .url("https://www.example.com/path/to/contact.vcf")
    .build();
```

The compiler gives an error: `Cannot resolve method 'url' in 'Builder'`. By contrast, the following works:

```java
MmsVcardRequest message = MmsVcardRequest.builder()
    .url("https://www.example.com/path/to/contact.vcf")
    .from("447900090000").to("447900090001")
    .build();
```

What gives? It's the exact same information, but the methods are called in a different order. Isn't the whole point of a builder to allow for flexibility in the order in which methods are called? For a good user experience, it is necessary to account for these situations. The user should not have to care about which classes contribute which properties - these are internal implementation details. A verbose solution is to override every method in the concrete subclass of builder. For example, in `MmsVcardRequest`, we would have:

```java
public static final class Builder extends MmsRequest.Builder<MmsVcardRequest> {
    Builder() {}

    public Builder from(String from) {
        return (Builder) super.from(from);
    }

    public Builder to(String to) {
        return (Builder) super.to(to);
    }

    public Builder clientRef(String clientRef) {
        return (Builder) super.clientRef(clientRef);
    }

    public Builder url(String url) {
        return (Builder) super.url(url);
    }

    @Override
    public MmsVcardRequest build() {
        return new MmsVcardRequest(this);
    }
}
```

But this defeats the whole point of inheritance, since we're repeating information! That is why we parameterise the builder type. The compiler ensures that the most concrete subtype is always returned. However, due to the fact that our builder classes can be extended, we need to encode this in the parameter declarations too, hence the bound `B extends Builder<? extends M, ? extends B>` as opposed to `B extends Builder<M, B>`. Unfortunately, we still have to cast the return type of the builder to `B` every time we call `return this`, but from what I can tell, that's a limitation of the compiler. Thankfully the casting only needs to happen in the abstract Builder classes, not the concrete types.

I hope this article has taught you a somewhat useful (although perhaps seemingly convoluted) pattern for using the Builder pattern when there is inheritance and abstract classes involved. Perhaps one day, such patterns will become obsolete when the language adds better ways to instantiate objects. Until then, at least we have generics to help us, as daunting as they can be to work with sometimes!
