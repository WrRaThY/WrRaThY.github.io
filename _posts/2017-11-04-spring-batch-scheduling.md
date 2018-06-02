---
layout: post
title:  "Scheduling stuff in Spring Batch"
date:   2017-11-04 17:42:45 +0200
categories: java, spring, cron
---

#### So I needed to schedule something in Spring. 
A batch job, but it's not that important. And I needed it to be externally configurable  
So I started googling around, because this is what devs do 90% of the time anyway. Just this time it was not about cats or weird geeky/science things.  
It turns out that there are numbers of people who enjoy making their life difficult. For example  [this][example1] solution  
And it's one of the recent ones (2016). Check out some older ones. There are literally hundreds of them (like [this][example2] (2013) and [that][example3] (2015))

So I decided that there must be a better way to do that... In [Spring docs][docs] you can find that there is an annotation called `@Scheduled` and it even takes a cron-like 
expression as an argument.

{% highlight java %}

    @Scheduled(cron = "*/15 * * * * *")
    public void justATestScheduledMethod() {
        System.out.println(LocalDateTime.now());
    }
    
{% endhighlight %}

So far so good... This means that my super-complicated method will be executed once every 15 seconds.  

#### But what about configuration? 
So if you paid any attention to your [education][docs2] you must know that...  

{% highlight java %}

    @Scheduled(cron = "${myapp.test-cron-scheduler}")
    public void justATestScheduledMethod() {
        System.out.println(LocalDateTime.now());
    }
    
{% endhighlight %}

You can get any property with `${...}` syntax, so lets see if that works after moving my cron property to the `application.properties` file...

```text
    17:18:47.139 INFO  (...).Application - Started Application in 4.076 seconds (JVM running for 4.649)
    2017-11-03T17:19:00.009
    2017-11-03T17:19:15.002
    2017-11-03T17:19:30.001
```

#### tadaaaam. 
wasn't that hard, was it?

ok, but we didn't run any batch job just yet, right?  
right. but there is any easy way to do that also:

1. inject a `JobOperator`
2. find yourself a jobName (String)
3. connect the facts: `jobOperator.startNextInstance(jobName)`
4. pat yourself on your back.

so my scheduler looks like that:

{% highlight java %}

    @Configuration
    public class Bla {
        private final JobOperator jobOperator;
    
        public Bla(JobOperator jobOperator) {
            this.jobOperator = jobOperator;
        }
      
        @Scheduled(cron = "${myapp.test-cron-scheduler}")
        public void justATestScheduledMethod() throws JobExecutionException {
            Long jobExecutionId = jobOperator.startNextInstance("printTimeJob");
        }
    }
    
{% endhighlight %}

and my batch job looks like that:

{% highlight java %}

    @Bean
    Job printTimeJob() {
        return jobBuilderFactory.get("printTimeJob")
                .incrementer(new RunIdIncrementer())
                .flow(printTimeStep())
                .end()
                .build();
    }

    @Bean
    Step printTimeStep() {
        return stepBuilderFactory.get("printTimeStep")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(LocalDateTime.now());
                    return null;
                })
                .build();

    }
{% endhighlight %}

the output is a little bit more messed up...

    17:39:10.007 INFO  org.springframework.batch.core.launch.support.SimpleJobOperator - Locating parameters for next instance of job with name=printTimeJob
    17:39:10.020 INFO  org.springframework.batch.core.launch.support.SimpleJobOperator - Attempting to launch job with name=printTimeJob and parameters={run.id=1}
    17:39:10.082 INFO  org.springframework.batch.core.launch.support.SimpleJobLauncher - Job: [FlowJob: [name=printTimeJob]] launched with the following parameters: [{run.id=1}]
    17:39:10.137 INFO  org.springframework.batch.core.job.SimpleStepHandler - Executing step: [printTimeStep]
    2017-11-03T17:39:10.156
    17:39:10.188 INFO  org.springframework.batch.core.launch.support.SimpleJobLauncher - Job: [FlowJob: [name=printTimeJob]] completed with the following parameters: [{run.id=1}] and the following status: [COMPLETED]
    17:39:15.001 INFO  org.springframework.batch.core.launch.support.SimpleJobOperator - Locating parameters for next instance of job with name=printTimeJob
    17:39:15.034 INFO  org.springframework.batch.core.launch.support.SimpleJobOperator - Attempting to launch job with name=printTimeJob and parameters={run.id=2}
    17:39:15.045 INFO  org.springframework.batch.core.launch.support.SimpleJobLauncher - Job: [FlowJob: [name=printTimeJob]] launched with the following parameters: [{run.id=2}]
    17:39:15.066 INFO  org.springframework.batch.core.job.SimpleStepHandler - Executing step: [printTimeStep]
    2017-11-03T17:39:15.077
    17:39:15.102 INFO  org.springframework.batch.core.launch.support.SimpleJobLauncher - Job: [FlowJob: [name=printTimeJob]] completed with the following parameters: [{run.id=2}] and the following status: [COMPLETED]
    17:39:20.002 INFO  org.springframework.batch.core.launch.support.SimpleJobOperator - Locating parameters for next instance of job with name=printTimeJob
    17:39:20.019 INFO  org.springframework.batch.core.launch.support.SimpleJobOperator - Attempting to launch job with name=printTimeJob and parameters={run.id=3}
    17:39:20.028 INFO  org.springframework.batch.core.launch.support.SimpleJobLauncher - Job: [FlowJob: [name=printTimeJob]] launched with the following parameters: [{run.id=3}]
    17:39:20.051 INFO  org.springframework.batch.core.job.SimpleStepHandler - Executing step: [printTimeStep]
    2017-11-03T17:39:20.060
    17:39:20.086 INFO  org.springframework.batch.core.launch.support.SimpleJobLauncher - Job: [FlowJob: [name=printTimeJob]] completed with the following parameters: [{run.id=3}] and the following status: [COMPLETED]

and we have some extra time consumed by the framework...  
check millies in printed time. 01-10 vs 60 - 156 

but in the end... it's more or less what we expected.

#### so... that's it.  
I hope that you guys enjoyed.  
see you next time and until then - `happy coding!`

## PS. one more thing!  
don't forget to use those somewhere in your configuration:

{% highlight java %}
    @EnableBatchProcessing
    @EnableScheduling
{% endhighlight %}

[docs]: https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#scheduling
[docs2]: https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#propertysource
[example1]: https://examples.javacodegeeks.com/enterprise-java/spring/batch/quartz-spring-batch-example/
[example2]: https://www.mkyong.com/spring-batch/spring-batch-and-quartz-scheduler-example/
[example3]: https://examples.javacodegeeks.com/enterprise-java/spring/spring-batch-quartz-example/
