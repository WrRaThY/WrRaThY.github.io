---
layout: post
title:  "Circuit breakers and Spring in 2020"
date:   2020-11-24 17:43:45 +1200
categories: java jvm spring hystrix resilience4j
---

it's been awhile... but without further ado - let's just get into it.
# Circuit breakers - what's that?  
I'm gonna assume that you, my dear reader, know what they are and if not - [there are smarter people than me that can explain that](https://martinfowler.com/bliki/CircuitBreaker.html)  
Resilience in general is a bit broader topic and covers things like retrying, throttling and others. I'll let you google that on your own if you need to.  
Bottom line is - in here you will learn how to use those patterns in Spring and I'll share a trick or two from my current project.

# Which library to choose?
Last time I was dealing with circuit breakers [Hystrix](https://github.com/Netflix/Hystrix) was the shit. 
It had all the bells and whistles that we could dream of.  
Now it seems it was made redundant by [Resilience4j](https://github.com/resilience4j/resilience4j), 
but looks like Spring provides [it's own implementation](https://spring.io/projects/spring-cloud-circuitbreaker) 
(which is based on either [Hystrix](https://spring.io/guides/gs/circuit-breaker/) 
or [Resilience4j](https://spring.io/guides/gs/cloud-circuit-breaker/) \[ u may also check [this](https://cloud.spring.io/spring-cloud-circuitbreaker/reference/html/index.html) \] underneath, but obviously the API is different).  
Anyway, these days nothing is as easy as "just using a lib". Especially one that interferes with your external calls.  
So Resilience4j can be used using one of the following: 
* directly: [io.github.resilience4j.resilience4j-circuitbreaker](https://mvnrepository.com/artifact/io.github.resilience4j/resilience4j-circuitbreaker)
* directly, with spring integration: [io.github.resilience4j:resilience4j-spring-boot2](https://mvnrepository.com/artifact/io.github.resilience4j/resilience4j-spring-boot2) (there is also an older version for spring-boot-1, but who cares about old stuff?)
* indirectly, through spring: [org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-circuitbreaker-resilience4j)
* probably there are other ways, but I couldn't be bothered

After some reading (and having Feign in the project, which is also a factor) I decided to discard the first option, 
because it would introduce too much clutter into the project. 
The third option cought my attention and I prepared a POC with it, 
but integrating with retries (which are also needed in my project) was annoying. 
Log configuration was also... less than ideal. 
So I was left with option two, which proved to be a good fit for our needs.

At the time of this writing `resilience4j` library group is at version `1.6.1`, which may or may not be an important piece of information ;) 
# Give me some code!
Ok, now that we have our library selected we can finally get some coding done.  
But hey, there is a number of [blog posts](https://dev.to/nagarajendra/basics-of-resilience4j-with-spring-boot-4dk8) 
and [tutorials](https://www.baeldung.com/resilience4j) 
(including the [official ones](https://github.com/resilience4j/resilience4j-spring-boot-demo)) 
and [github issues](https://github.com/resilience4j/resilience4j/issues/883) 
(or [this](https://github.com/resilience4j/resilience4j/issues/645) 
or [that](https://github.com/spring-cloud/spring-cloud-circuitbreaker/issues/3)) that cover this.

###Why do I think this is gonna be any better?
Because I needed to dig through all of them to come up with something that was complicated enough to satisfy my needs 
and be simple enough to not annoy the hell out of me.  
So I've done the homework for you, my dear reader. Enjoy :) 

# Additional resources
- maybe you didn't notice, but there are heaps of links in this post. just use them :)  
- repo: https://github.com/WrRaThY/spring-with-resilient-feign