---
layout: post
title:  "Feign, Circuit breakers and Spring in 2020"
date:   2020-11-24 17:43:45 +1300
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

## Considerations
Things that I considered when choosing the right one:
1. it was important to not only provide the needed functionality, but also enable proper logging for it. In large-scale systems this is crucial.
2. endpoint configuration should be annotation based - to avoid repetitive code
3. logging configuration should be generic / centralised - to avoid repetitive code
4. library configuration should be external (not in code) to enable easy switching between environments
5. the less code the better

Anyway, these days nothing is as easy as "just using a lib". Especially one that interferes with your external calls.  
So Resilience4j can be used using one of the following: 
1. directly: [io.github.resilience4j.resilience4j-circuitbreaker](https://mvnrepository.com/artifact/io.github.resilience4j/resilience4j-circuitbreaker)
2. directly, with spring integration: [io.github.resilience4j:resilience4j-spring-boot2](https://mvnrepository.com/artifact/io.github.resilience4j/resilience4j-spring-boot2) (there is also an older version for spring-boot-1, but who cares about old stuff?)
3. indirectly, through spring: [org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-circuitbreaker-resilience4j)

probably there are other ways, but I couldn't be bothered

After some reading (and having [Spring-cloud-feign](https://cloud.spring.io/spring-cloud-openfeign/reference/html/) in the project, which is also a factor) I decided to discard the first option, 
because it would introduce too much clutter into the project. Example?
so according to [official resilience4j documentation](https://resilience4j.readme.io/docs/feign)
 we should do the following to wrap feign clients
```java
public interface MyService {
            @RequestLine("GET /greeting")
            String getGreeting();
            
            @RequestLine("POST /greeting")
            String createGreeting();
        }

        CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("backendName");
        RateLimiter rateLimiter = RateLimiter.ofDefaults("backendName");
        FeignDecorators decorators = FeignDecorators.builder()
                                         .withRateLimiter(rateLimiter)
                                         .withCircuitBreaker(circuitBreaker)
                                         .build();
        MyService myService = Resilience4jFeign.builder(decorators).target(MyService.class, "http://localhost:8080/");
```
apart from a simple fact that this does not compile as-is, which I find disturbing, it's just not readable. And it doesn't use spring-cloud-feign

The third option caught my attention and I prepared a POC with it (which I already deleted, so I won't recreate it here) 
but integrating with retries (which are also needed in my project) was annoying. I'll just leave with an example from [the official docs](https://spring.io/guides/gs/cloud-circuit-breaker/):
```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import reactor.core.publisher.Mono;

import org.springframework.cloud.client.circuitbreaker.ReactiveCircuitBreaker;
import org.springframework.cloud.client.circuitbreaker.ReactiveCircuitBreakerFactory;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;

@Service
public class BookService {

  private static final Logger LOG = LoggerFactory.getLogger(BookService.class);


  private final WebClient webClient;
  private final ReactiveCircuitBreaker readingListCircuitBreaker;

  public BookService(ReactiveCircuitBreakerFactory circuitBreakerFactory) {
    this.webClient = WebClient.builder().baseUrl("http://localhost:8090").build();
    this.readingListCircuitBreaker = circuitBreakerFactory.create("recommended");
  }

  public Mono<String> readingList() {
    return readingListCircuitBreaker.run(webClient.get().uri("/recommended").retrieve().bodyToMono(String.class), throwable -> {
      LOG.warn("Error making request to book service", throwable);
      return Mono.just("Cloud Native Java (O'Reilly)");
    });
  }
}
```
Log configuration was also... less than ideal.  
So I was left with option two, which proved to be a good fit for our needs as it satisfies all the considerations.
 
# Give me some code!
Ok, now that we have our library selected we can finally get some coding done.  
But hey, there is a number of [blog posts](https://dev.to/nagarajendra/basics-of-resilience4j-with-spring-boot-4dk8) 
and [tutorials](https://www.baeldung.com/resilience4j) 
(including the [official ones](https://github.com/resilience4j/resilience4j-spring-boot-demo)) 
and [github issues](https://github.com/resilience4j/resilience4j/issues/883) 
(or [this](https://github.com/resilience4j/resilience4j/issues/645) 
or [that](https://github.com/spring-cloud/spring-cloud-circuitbreaker/issues/3)) that cover this.

### Why do I think this is gonna be any better?
Because I needed to dig through all of them to come up with something that was complicated enough to satisfy my needs 
and be simple enough to not annoy the hell out of me.  
So I've done the homework for you, my dear reader. Enjoy :) 

## Ok, here it goes
Let's start with the basics.  
Our test app is super-simple. Important bits:
- `WorkingClient` - a public API being called by a spring-cloud-feign client
- `FailingClient` - a spring-cloud-feign client that always fails
- `CircuitBreakerLoggingConfiguration` - this is where the logging magic happens
- `application.yml` - configures log levels per feign client
- `application-resilience.yml` - configures resilience4j props
- `logback-spring.xml` - uses `the MDC trick`
- the usuals - dummy controller, dummy object, application main

repo is available here: [https://github.com/WrRaThY/spring-with-resilient-feign](https://github.com/WrRaThY/spring-with-resilient-feign)

### FailingClient
`WorkingClient` is kinda boring, so I'll get to the interesting stuff
```java
@FeignClient(value = FailingClient.CLIENT_NAME, url = "http://this.does.not.exist")
public interface FailingClient {
    String CLIENT_NAME = "FailingClient";

    @Retry(name = FailingClient.CLIENT_NAME)
    @CircuitBreaker(name = FailingClient.CLIENT_NAME)
    @GetMapping(value = "/resources/titles/{bookId}")
    Book getById(
            @PathVariable("bookId") String bookId
    );
}
```
as you can see it's really short, but there are heaps of things happening here.
1. with Feign this is it. u don't need to `implement` the client. the declaration is enough. so this is a ready to use http client already.
2. We have `Retry` and `CircuitBreaker` annotations with the client name. this bit is important, because this config name correlates to the config name in the `application-resilience.yml`
3. There is no logging defined here, but we will get logs as configured in `application.yml` and `CircuitBreakerLoggingConfiguration`

### CircuitBreakerLoggingConfiguration
Thanks to [this magnificent, undocumented feature](https://stackoverflow.com/questions/64519894/resilience4j-retry-logging-retry-attempts-from-client/64536776#64536776) we can actually have default logging for all our resilience actions!  
Normally we would have to call a `CircuitBreakerFactory` with a service name and `bla, bla, bla`... we don't have to.  
Now inside there are a couple of things that are worth noting and explaining:
```java
@Override
public void onEntryAddedEvent(EntryAddedEvent<CircuitBreaker> entryAddedEvent) {
    CircuitBreaker.EventPublisher eventPublisher = entryAddedEvent.getAddedEntry().getEventPublisher();

    eventPublisher.onEvent(event -> {
        if (event.getEventType() == CircuitBreakerEvent.Type.SUCCESS && !circuitBreakerState.isDebugEnabled()) {
            return;
        }

        // that little trick adds it to MDC, which can then be used in a log statement
        MDC.put(EVENT_TYPE_LABEL, event.getEventType().name());
        MDC.put(NAME_LABEL, event.getCircuitBreakerName());

        if (event.getEventType() == CircuitBreakerEvent.Type.SUCCESS) {
            circuitBreakerState.debug(event.toString());
        } else {
            circuitBreakerState.error(event.toString());
        }

        MDC.remove(EVENT_TYPE_LABEL);
        MDC.remove(NAME_LABEL);
    });
}
```
1. `EntryAddedEvent<CircuitBreaker> entryAddedEvent` - that's a `CircuitBreaker` creation event. that means that we can configure defaults for all `CircuitBreaker`s here
2. `eventPublisher.onEvent(event -> {})` - that's an event connected with a specific request. that means we can do something low-level here.
3. `if (event.getEventType() == CircuitBreakerEvent.Type.SUCCESS && !circuitBreakerState.isDebugEnabled())` - just an optimisation. obviously not useful here, but kinda useful with thousands of requests per second in our main app ;) (toString methods, which can be quire costly, will not be executed. same goes for MDC-related stuff)
4.  `MDC.put` and `MDC.remove` - adds additional info to log statements. enables `the MDC trick`
5.  `eventPublisher` has useful methods like `onSuccess` or `onError`, but I felt that the above approach is actually shorter / more understandable

### the MDC trick
so those MDC entries are not really useful as-is (because this info is included in the `toString` anyway), 
but in our case - we send logs to ELK (in JSON format, with adding all MDC fields), 
so with those two MDC entries added we can actually create more meaningful visualisations, 
filters and other funky things.

For the sake of this example I just limited myself to providing the following in the console-logger config (`logback-spring.xml`):
```
    [%X{circuitBreakerName}:%X{circuitBreakerEventType}]
```
it's a really stupid usage, but just shows that MDCs are meaningful for the logging library and u can see the result of it in the log itself:
```java
2020-11-24 21:15:19.953 [http-nio-8080-exec-2] DEBUG 49783 --- priv.rdo.WorkingClient                  .log: [:] [WorkingClient#getById] <--- HTTP/1.1 200 OK (1581ms)
2020-11-24 21:15:20.352 [http-nio-8080-exec-2] DEBUG 49783 --- circuitBreakerState                     .lambda$onEntryAddedEvent$0: [WorkingClient:SUCCESS] 2020-11-24T21:15:20.351870+13:00[Pacific/Auckland]: CircuitBreaker 'WorkingClient' recorded a successful call. Elapsed time: 1982 ms
2020-11-24 21:15:26.884 [http-nio-8080-exec-3] ERROR 49783 --- circuitBreakerState                     .lambda$onEntryAddedEvent$0: [FailingClient:ERROR] 2020-11-24T21:15:26.883893+13:00[Pacific/Auckland]: CircuitBreaker 'FailingClient' recorded an error: 'feign.RetryableException: this.does.not.exist executing GET http://this.does.not.exist/resources/titles/9781400079148'. Elapsed time: 1 ms
```

### Config files
I encourage you to check out the config files, because you can play around with things like
- log levels
- different types of retry / circuit breaker configs
- values for those configs (which will result in different logs actually)

example:
```yaml
resilience4j.retry:
  configs:
    default:
      retryExceptions:
        - feign.FeignException
```
this might be a bit too simplistic (u don't really want to retry on 400s in a real system, right?)  
an easy fix could be just changing the exception to FeignServerException (which will catch only 500s)
```yaml
resilience4j.retry:
  configs:
    default:
      retryExceptions:
        - feign.FeignException.FeignServerException
```

If you feel like my configs are too simple... they are :)
check out [this document](https://resilience4j.readme.io/docs/circuitbreaker#create-and-configure-a-circuitbreaker) from official docs for a full list of possible properties 

### Free stuff!
yes, we get some stuff literally for free. that stuff being - metrics. just enable actuator metrics and that's it. everything is handled for ya.  
So try to explore those actuator endpoints, below I'll just give you some basic examples:

`http://localhost:8080/actuator/circuitbreakerevents`
```json
{
    "circuitBreakerEvents": [
        {
            "circuitBreakerName": "WorkingClient",
            "type": "SUCCESS",
            "creationTime": "2020-11-25T17:30:23.464618+13:00[Pacific/Auckland]",
            "errorMessage": null,
            "durationInMs": 2380,
            "stateTransition": null
        },
(...)
        {
            "circuitBreakerName": "FailingClient",
            "type": "FAILURE_RATE_EXCEEDED",
            "creationTime": "2020-11-25T17:30:26.784061+13:00[Pacific/Auckland]",
            "errorMessage": null,
            "durationInMs": null,
            "stateTransition": null
        },
        {
            "circuitBreakerName": "FailingClient",
            "type": "STATE_TRANSITION",
            "creationTime": "2020-11-25T17:30:26.786452+13:00[Pacific/Auckland]",
            "errorMessage": null,
            "durationInMs": null,
            "stateTransition": "CLOSED_TO_OPEN"
        }
    ]
}
```

or `http://localhost:8080/actuator/metrics/resilience4j.circuitbreaker.calls`
```json
{
    "name": "resilience4j.circuitbreaker.calls",
    "description": "Total number of calls which failed but the exception was ignored",
    "baseUnit": "seconds",
    "measurements": [
        {
            "statistic": "COUNT",
            "value": 5.0
        },
        {
            "statistic": "TOTAL_TIME",
            "value": 2.875985408
        },
        {
            "statistic": "MAX",
            "value": 0.0
        }
    ],
    "availableTags": [
        {
            "tag": "kind",
            "values": [
                "ignored",
                "failed",
                "successful"
            ]
        },
        {
            "tag": "name",
            "values": [
                "WorkingClient",
                "FailingClient"
            ]
        }
    ]
}
```

# Additional resources
- maybe you didn't notice, but there are heaps of links in this post. just use them :)  
- repo: [https://github.com/WrRaThY/spring-with-resilient-feign](https://github.com/WrRaThY/spring-with-resilient-feign)

so... that's it.
I hoped it's gonna make someones life easier. cheers! 

ps. At the time of this writing `resilience4j` library group is at version `1.6.1`, which may or may not be an important piece of information ;)

pps. huh. not a lot of ranting. or swearing. or madness in general. I have to say that this particular piece of software just works.  
huh. who would've thought?  
or maybe I'm just getting old? ;) 