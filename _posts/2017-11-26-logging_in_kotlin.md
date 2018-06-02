---
layout: post
title:  "Logging in Kotlin"
date:   26.11.2017 19:10:00 +0200
categories: java, kotlin, logging, slf4j
---


Hello There!  
I've been pretty busy lately. Partially, because I'm learning Kotlin in my spare time.  
I started rewriting one of my coding challenges (maybe I'll write something about it later on. for now you may want to check [this repository][tradeValidatorKotlin] out) and one of my problems was...  
__how to instantiate loggers properly?__  
This may seem stupid, but Kotlin rejects the idea of `static` fields, so you can't really write

{% highlight java %}

    private static final Logger LOG = LoggerFactory.getLogger(SomeClass.class);

{% endhighlight %}

in Kotlin. On the other hand you can use companion objects:

{% highlight kotlin %}

    companion object {
        private val LOG = LoggerFactory.getLogger(TradeValidationEndpoint::class.java)
    }

{% endhighlight %}

But it's not that pretty, right?  
Especially given the fact that Kotlin tries to be as expressive as possible.

If you search for a second you'll find some information about that:
- [pretty long discussion on kotlin forums][kotlinLangDiscuss]  
- [SO question][soQuestion]  
- [blog posts][someBlog]
  
and plenty more.  
And all that is great, but...  A more general approach would be nice.

#### Well... Lucky us! `oshai` already created a [library][kotlinLogging] wrapping SLF4J  
And it's great! But I really enjoyed using SLF4J-ext methods (entry and exit) and I feel like `LOG` is a better name for a static logger than `logger`...  
So I made my own version of this library :) You can find it [here][kotlinLoggingExt]

##### This is how you can declare a logger in Kotlin now:  
```kotlin
    companion object : KLogging()
```
and use it like that:
```kotlin
    LOG.entry(input)
    
    return LOG.exit(output)
```

hopefully you'll like it. I have to think about sending those changes to the guy.  
Maybe his project will benefit from them? :)

cheers!

[kotlinLangDiscuss]: https://discuss.kotlinlang.org/t/best-practices-for-loggers/226
[soQuestion]: https://stackoverflow.com/questions/34416869/idiomatic-way-of-logging-in-kotlin
[someBlog]: https://realjenius.com/2017/08/31/logging-in-kotlin/

[kotlinLogging]: https://github.com/MicroUtils/kotlin-logging
[kotlinLoggingExt]: https://github.com/WrRaThY/kotlin-logging-ext 

[tradeValidatorKotlin]: https://github.com/WrRaThY/trade-validator-kotlin
