# posts

## Laravel Queue based Concurrency Handling
Write

Get unlimited access to all of Medium for less than $1/week.
Become a member
A rare image of Beloved Laravel, Redis and Queue 😐
Concurrency Attack and Queue in Laravel — Batman Returns!
Farhat Shahir Zim

Farhat Shahir Zim
·

8 min read
·
Mar 8, 2020

Prevention is better than cure.

    “Put on the full armor of God, so that you can take your stand against the devil’s schemes.” — Ephesians 6:11

The Lost World!

The world (tech world) is as dark as exciting. There are foul creatures (unethical hackers) everywhere in this dark world. Anytime any kind of foul creature can harm us and our application if we are not careful and smart enough. They have many ways to harm mankind (software engineers). Concurrency attack is one of them.

Today we have Batman with us. In our previous blog, Deadpool developed a nice wallet application. He started it. Batman is gonna end it with perfection and security. Batman is the senior software engineer in Gotham city. Security and prevention are his responsibility.
A horror scenario of concurrency attack!

Let’s think about a scenario. Which will lead us to the thought why we should take concurrency attack seriously. In our previous wallet application, we added the functionality of transferring the balance from one wallet to another. Now let's have some information in our mind.

    Joker has only 1,000,000$ in his wallet.
    He wants to transfer 1,000,000$ to Harley Quinn’s wallet.

Now instead of just clicking on the “send ” button, Joker did something odd.
He logged in to his wallet application account from two different browsers. Both browsers were on different machines. And then,

Joker clicked on the send buttons from two different browsers with his two foul hands, at the same time. With good timing and good perception.
A moment of silence

Harley Quinn got 2,000,000$ in her account.

Where the extra 1,000,000$ came from? From nowhere. This is loss Deadpool must bear now!
We are fried
Now, This is the“I am Batman” Moment!

But wait. What I was telling you is just a scenario. We have Batman remember? He thought about this scenario before launching the application. Now Let’s secure our application from Joker with Batman.
Batman’s Solution for Concurrency Attack In Laravel

The Queue. Yes, Laravel’s Job & Queue. And we are gonna secure our wallet application that way. Now let’s see what else we need to install for our application.

prerequisite

    PHP >= 7.1.3
    Laravel ≥ 5.8
    Redis
    Mysql

Batman’s Important Notes to Take!

    Now the reason we are using Redis because it's fast. Although we have some drawbacks in Redis. You need to stop Redis to remove the previous queues.without clearing cache, restarting the redis will resume those queue process again!

    Redis installation link: https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04

Let’s Begin and save Gotham City…

Before we dive into the main part I request you to read the previous blog where we demonstrate a simple wallet application with only one feature called balance-transfer. If you have read that already then let’s proceed.

First of all, install Laravel’s Horizon in our wallet application.

    “Horizon provides a beautiful dashboard and code-driven configuration for your Laravel powered Redis queues. Horizon allows you to easily monitor key metrics of your queue system such as job throughput, runtime, and job failures.

    All of your worker configuration is stored in a single, simple configuration file, allowing your configuration to stay in source control where your entire team can collaborate.” — From The Beautiful Laravel Doc

Step[00] — Installation of The Weapon Called Horizon:

Install Laravel’s Horizon with this command.

composer require laravel/horizon v3.7.1

Now publish its assets using the horizon:install Artisan command:

php artisan horizon:install

Now let's make migration to create the failed_jobs table to store any failed queue jobs.

php artisan queue:failed-table

php artisan migrate

Now let’s change QUEUE_CONNECTION value “sync” to “redis” in .env file. As we will use the Redis database for our queue.

QUEUE_CONNECTION=syncto QUEUE_CONNECTION=redis

Nice. Installation complete. Now start our application and Horizon with this command.

// Start application
php artisan serve// Start horizon
php artisan horizon

Now if you go to this link (http://127.0.0.1:8000/horizon/dashboard), you will see your horizon is active, Like this.
Step [01] — Configure Your New Weapon Called Horizon:

In our config/horizon.php file, Let’s add another queue just beside the default queue.
config/horizon.php

You can manage multiple environments with multiple supervisors. We will talk more about it later.

Wait, can everyone see our horizon dashboard this way? Yes…But we can’t let it happen. Let’s secure our horizon dashboard.

    “Horizon exposes a dashboard at /horizon. By default, you will only be able to access this dashboard in the local environment. Within your app/Providers/HorizonServiceProvider.php file, there is a gate method. This authorization gate controls access to Horizon in non-local environments. You are free to modify this gate as needed to restrict access to your Horizon installation”
    — From our beloved Laravel Doc

So Let’s add Batman’s email in the gate() method.
Step [02] — Create a Job in our Laravel application:

We will now create a Job called BalanceTransferJob. Which will be created inside the app/Jobs directory

php artisan make:job BalanceTransferJob// Now inside of app/Jobs you will see this
// BalanceTransferJob.php

Step [03] — Recall Our WalletService And WalletController From The Previous Blog:

Let’s take a look again at our WalletService.php and WalletController.php.
app/Http/Controllers/WalletController.php
app/Services/WalletService.php

If you read our previous blog, you will understand what's happening here. But I will explain again.

We have 2 methods here.
1. transfer()
2. balanceTransferValidation()

Our balanceTransferValidation() method will validate

    If all the wallets (sender and receiver wallet) exist.
    And the sender has enough balance to transfer.
    If anything goes missing here, this method is responsible for sending the exception messages according to the validation.

Our transfer() method will check the validation method’s response and

    If everything is ok this method will transfer the balance from the sender’s wallet to the receiver’s wallet.
    If not this will send a response accordingly.

Step [04] — Job & Queue Implementation:

Now, which part should we send to the queue? Well, the main part. Where we are actually transferring the balance. This part below.
Balance transfer

Now we will take this part to your newly created Job called BalanceTransferJob.php. Let's have a look at our Job now.
Let’s break it down our BalanceTransferJob!

    In our Job, we have a constructor and a handle() method.
    We have initialized $senderWalletId, $receiverWalletId, $transferAmount and WalletService in our constructor.
    In our handle() method’s try-block, We started the database transaction first.
    Then checked if all wallets exist and the sender has enough balance to transfer. If not we rolled back the transaction.
    If validation is successful then we decreased the balance from the sender’s wallet.
    And increased the balance to the receiver’s wallet and committed the transaction.
    In our handle() method’s catch-block, We rolled back the transaction.

And that's all. This is all our BalanceTransferJob will do.
Step [05] — Dispatch The Job In Queue With Delay:

Well, we wrote the Job now we need to dispatch it in the transfer() method. A way of dispatching a Job with delay is like this:

dispatch(new YourJob($param1, $param2, ...))
    ->onQueue('your-desired-queue')
    ->delay(Carbon::now()->addSeconds(10));

You can avoid delays in many cases where it is not necessary. But to prevent concurrency attack we need delay here. You can add any value of seconds in your delay according to your requirements. After dispatching the Job in our wallet service, our transfer() method will be like this:

    Please!… don’t forget to import necessary classes.

Now, let's start our horizon again as we did some changes.

php artisan horizon

Now let’s hit on our API route http://127.0.0.1:8000/api/transfer-balance following the below image. I have used phpStrom’s API Debugger, but you can use postman as well.

Now let's go to our horizon dashboard And check our current workload (http://127.0.0.1:8000/horizon/dashboard)

We have a Job in our current workload. Now Let’s check our recent jobs in the dashboard.

Well, our job has been dispatched in the queue successfully.
Summary of our procedures:

    Installed Horizon.
    Created a new Job called BalanceTransferJob.
    Moved our balance transfer code into the Job
    Dispatched the job with 10 seconds delay in transfer() method of our WalletService.
    Restarted our horizon after these changes.
    Make a POST request to our API endpoint.
    And finally, we checked the horizon dashboard to see everything worked as we planned.

Joker will fail to make concurrency attacks now

Now, how Batman prevented concurrency attack? Well, after implementing the job in the queue with a delay, If joker tries to make a concurrent hit both requests will go to the queue in a FIFO (First in first out) manner. So one of the transactions will happen first, then when the 2nd transaction takes place, Our balanceTransferValidation() method will prevent that from happening. And Harly will never get the extra 1M dollar from Joker.

And this is how Gotham city has been secured again. Thanks to Dear God and his warrior of dark, THE BATMAN.

You can try the concurrency attacks by yourself. Ha, ha…you will also fail now !!!!

    But the Lord is faithful, and he will strengthen you and protect you from the evil one. — Thessalonians 3:3

I pray again to dear Lord to keep you guys away from every unexpected and unholy bug. May God bless your application in production.

Ameen!
Laravel
Redis
Concurrency
Laravel Horizon
Laravel Queue

Farhat Shahir Zim
Written by Farhat Shahir Zim
56 Followers

[Embedded Software Engineer] — | C/C++ | Assembly | RISC-V | SystemC | Microcontroller | Computer Architecture | Self taught programmer |
More from Farhat Shahir Zim
Farhat Shahir Zim

Farhat Shahir Zim
Exception Handling and Database Transaction in Laravel — Part 2
Hello good folks! let’s get a machine gun and handle sum bugs. Also, our Deadpool is killing bugs instead of bad guys.
5 min read·Mar 1, 2020

Farhat Shahir Zim

Farhat Shahir Zim
Exception handling and Database Transaction in Laravel  — Part 1
Let's say you are going to insert data into multiple tables or update a table after inserting some data into another table. In the…
5 min read·Mar 1, 2020

See all from Farhat Shahir Zim
Recommended from Medium
Jacob Bennett

Jacob Bennett

in

Level Up Coding
Use Git like a senior engineer
Git is a powerful tool that feels great to use when you know how to use it.
·4 min read·Nov 15, 2022

The PyCoach

The PyCoach

in

Artificial Corner
You’re Using ChatGPT Wrong! Here’s How to Be Ahead of 99% of ChatGPT Users
Master ChatGPT by learning prompt engineering.
·7 min read·Mar 17

Lists
General Coding Knowledge
20 stories·29 saves
Now in AI: Handpicked by Better Programming
248 stories·14 saves
The Coding Diaries

The Coding Diaries

in

The Coding Diaries
Why Experienced Programmers Fail Coding Interviews
A friend of mine recently joined a FAANG company as an engineering manager, and found themselves in the position of recruiting for…
·5 min read·Nov 2, 2022

Arslan Ahmad

Arslan Ahmad

in

Level Up Coding
System Design Interview Survival Guide (2023): Preparation Strategies and Practical Tips
System Design Interview Preparation: Mastering the Art of System Design.
·14 min read·Jan 19

John Raines

John Raines
Be an Engineer, not a Frameworker
Time to level up.
·10 min read·Mar 8, 2022

Tiexin Guo

Tiexin Guo

in

4th Coffee
10 New DevOps Tools to Watch in 2023
With less than two months left in 2022, today, I’d like to look at some new (relatively) DevOps tools we might want to follow in 2023.
·9 min read·Nov 2, 2022


## Does Laravel Scale?

The internet is full of lies about whether Laravel can scale. Here's the truth.
Does Laravel Scale?
Jack Ellis
Written by Jack Ellis
May 16, 2022

In this blog post, I will explore whether you can use Laravel at hyper-scale and whether it could be used to power something like Twitter, Facebook or various other huge applications.
What brought me here

We're all getting tired of the "Does Laravel scale?" questions. Not because people are asking questions but because of the ignorant responses to the question. It has happened numerous times now, and the storyline is always the same.

    Someone posts on Twitter or a forum asking if Laravel can scale
    Two groups of people appear
    Group one says, "of course, it can scale; the web framework won't be your bottleneck for a long, long time."
    Group two says "no" and is mad because how dare you suggest that a PHP framework can perform at hyper-scale.

In this post, I'm going to address this question once and for all: Does Laravel scale?
Why are you worried?

Before we get into details about whether Laravel can scale, you must understand that most people are worried about a situation they will never experience. You are not building the next Google, Facebook or YouTube. I don't write that as an insult, and I'm not a pessimist (I'm an entrepreneur who quit his job to work on a startup!), but it's mentally healthy for us to keep our expectations based on reality.

So when people ask the question, "Does Laravel scale?" because they're starting a company or about to build an application for a multi-billion dollar company, they need to realize that they just won't hit the scale of Facebook.
How many requests does Wikipedia handle?

Wikipedia is one of the biggest websites in the world, and, guess what, it runs on PHP. According to a TechCrunch article from 2020, Wikipedia was processing 255 million pageviews per day on average. So how does that look per second and per month?

    225 million / 86400 = 2,604 pageviews per second
    225 million x 30 = 6.75 billion pageviews per month

Do we consider Wikipedia to be a big website?
How many requests does Facebook handle?

They state in a 2010 blog post that they were processing 100 billion "hits" per day at 500 million users. That's gigantic. The word "hit" is vague and could correspond to anything, but let's assume it's web requests. Let's see what it looks like per second and per month:

    100 billion / 86400 = 1.15 million requests per second
    100 billion x 30 = 3 trillion requests per month

So that's a staggering amount of traffic, beyond comprehension. And that was in 2010!

Could Laravel be scaled to handle 3 trillion requests per month? Sure.

Would you likely bring in other frameworks for certain parts of the app that made sense? Yes.

Will we ever be architecting an application processing 3 trillion requests per month? Statistically, no. Again, that's not me being a pessimist.

Realistically, you're going to be scaling up to between 1 million requests and 100 billion requests each month. Everyone has a different idea about a "big" application, so I've given a huge range here.

Now we've addressed the fact that we're worrying about something that statistically will never happen, let's move on to scaling Laravel in the real world.
Your benchmarks are nonsense

Benchmarks are constantly thrown around whenever you discuss whether something can scale. You'll end up seeing some random framework at the top of the list that nobody has ever heard of, has awful developer experience, zero community and an awful ecosystem. But because it can make many requests on a single server per second, it's apparently the best.

Despite my eagerness to debate whether Laravel could run at 3 trillion requests per month, I will concede that once you reach a certain point where costs are through the roof and you're looking for something more budget-friendly at scale, you will probably switch from Laravel to something in C++ or Rust, at least for parts of the application. But what kind of traffic would you need to be doing to switch?

Recently, someone on Twitter linked to TechEmpower Framework Benchmarks. Laravel sits at position #388 with 4,833 requests per second performance compared with the #1 position held by drogon-core achieving 666,737 req/s. 666, eh?

Well, wait a minute, that means drogon-core can perform 1,752,184,836,000 requests per month. And in 2010, Facebook did 3,000,000,000,000 requests per month…

3,000,000,000,000 / 1,752,184,836,000 = 1.71

Hey, someone should tell Facebook they should move all of their code to drogon-core, and they'll be able to power their entire application layer across two web servers. They'll be so pleased to hear the news.

My point is that these benchmarks are worthless when you're having a conversation about whether Laravel can scale. You look at this Laravel benchmark in the above results, and you think, "huh, something seems off" then you scroll down and see the following note:

    In this test, the framework's ORM is used to fetch all rows from a database table containing an unknown number of Unix fortune cookie messages (the table has 12 rows, but the code cannot have foreknowledge of the table's size)

Well, duh, Laravel doesn't use connection pooling out of the box, no persistent connections, and each of those requests is:

    Bootstrapping the entire framework
    Connecting to the MySQL database
    Running the query
    Disconnecting from the MySQL database

They're using PHP FPM… What about Laravel Octane? What about persistent connections? What about some kind of connection pooling layer?

These benchmarks are just absolute nonsense. Why does anybody take them seriously? And who produced the idea that generic benchmarks were this authority that mattered? Because they're not. I conceded above that, at scale, you'd probably want something faster than PHP, but for 99.99994% of businesses (and more I'd guess), you're going to smash it with Laravel. And you're going to benefit from excellent documentation, an incredible community, and an ecosystem Mark Zuckerberg would've killed for when he first built Facebook.

When we started seeing incredible growth in our software, Fathom Analytics (which is built on Laravel), our problems were all database-related. We never had moments of "does the framework do enough requests per second?". And even then, we're an outlier. Most people aren't doing as much traffic as we are, so they're not even worried about optimizing the HTTP layer and are running comfortably with a relatively simple setup.

What's also funny is when people throw around the term "enterprise" to try and discredit Laravel. Suddenly, because a business is doing millions/billions of dollars in revenue, apparently Laravel isn't fit for the course? Nonsense.

I've worked with enterprise companies using Laravel to power their entire business, and companies such as Twitch, Disney, New York Times, WWE and Warner Bros are using Laravel for various projects they run.

Laravel can handle your application at scale.
What should I be worried about?

Now that I've addressed some of the broken mental models some of us are guilty of, I want to get into the nitty-gritty of scaling a Laravel application. Before I co-founded a website analytics platform, I never had any experience with scaling applications. My experience up until running this business had been building applications for 100 - 10,000 users, and I'd certainly never had to worry about dealing with billions of page views.

So if Laravel isn't the area we should be worried about when it comes to scaling, what is?
Database, cache and sessions.

Your database is going to be the bottleneck as you scale. I'm assuming a traditional MySQL/PostgreSQL setup here. You're not going to run into problems if you're using DynamoDB or SingleStore from the start, as these solutions are built for gigantic scale with minimal configuration. Anyway, database performance is a whole career, and many things can be tweaked outside of the queries you write.

Our database journey was:

    One SQLite file for every customer (this was horrible, don't do it)
    Managed PostgreSQL on Heroku (lack of familiarity with Postgress led to the desire to move to MySQL)
    Managed MySQL on Heroku (this was fine, but we ditched Heroku and moved to Laravel Vapor)
    RDS for MySQL (we started running into performance issues because we needed ad-hoc analytics and hyper scalable ingest. MySQL fell over constantly, despite upgrades and IOPS increases)
    SingleStore (the database of our dreams)

We used Redis as our cache for a few years and then moved to DynamoDB (so we didn't have to worry about sizing Redis or NAT gateways on AWS). Nowadays, we use SingleStore's in-memory tables for caching because it's rapid.

We only worked with managed services because we are not database engineers. If you're a database engineer, you'll have no trouble scaling your database, and you already know how to scale it horizontally, tweak it, etc. But for the rest of us, be ready for database headaches as you grow.
Queue system

The Laravel queue system has multiple drivers such as Amazon SQS, Redis, database, etc. The purpose of the queue system, as you know, is to offload time-consuming jobs into the background to ensure fast page response time for your users.

We are big fans of SQS because it has unlimited throughput and excellent security, and it stores jobs across multiple availability zones. Now that we use SingleStore for our database, they could power our queue, but we like that our queue system is separate from the database (for fault tolerance reasons).

One of my concerns, when we used Redis for our queue was, "what happens if we had a queue backlog, Redis filled up, and we ran out of space." And that was a legitimate concern because we came close in the early days. In addition to that, there's a bunch of stuff you can do to scale Redis, which most people won't get to in their careers, but it is possible. You can configure Redis to run at scale on Laravel queues, and many people have. Or you can use SQS or SingleStore. Easy.
Other services

If you were using PHP-FPM, you'd want to watch that, load test it, and configure it in the most optimal way possible. But away from that, your "external services," such as your database, queue system, etc., will become your primary bottleneck before PHP-FPM does (most times, I'm sure there'll be exceptions, and I'm sure the reply guys will let me know). In addition to the above, don't forget the following:

    Make sure your email API (Postmark, SES, etc.) can handle the rates you might send it. The same goes for SMS notifications and everything else. Heck, check the rate limits on every external service you use and monitor them, as you may need to raise them as you scale
    Use a CDN. You shouldn't be serving static assets from your web server; you should be all-in on a CDN like bunny.net, Cloudfront or Cloudflare

Writing code for scale

When running at scale, you need to think about things you might not have thought about before. I'll share a few thoughts on this, but this isn't a comprehensive coding guide :)

One of my earliest experiences of a "WTF" moment was when I used database transactions for inserts, and I would run into deadlock errors over the auto-increment column in tables. Long story short, I effectively had multiple, multi-query, multi-table transactions fighting to get a lock on auto increment to write a row. The initial solution to this problem, which worked well, was to have the transaction retry 50 times, but that's not going to scale and is highly inefficient.

In the end, we moved away from using transactions, and we fixed the problem. But my point is that you won't notice this issue at a low scale. But once things start ramping up, you'll start seeing things like this.

So with that said, here are a few thoughts I have on writing code for scale:

    Make sure your queries are consistent. Engineers from Facebook had some great advice: "It's OK if a query is slow as long as it is always slow." I agree, and this means you need to know your queries well and write them efficiently.
    Know when to offload functionality. We're talking about running Laravel at scale, but I'm not saying you should use Laravel to process videos. If Amazon has a service that can do this for you, use it. Once you're doing billions of dollars a year in revenue, you can build it in-house using some cutting-edge C++ framework. That'll be so much fun!
    Cache expensive queries. This applies to all projects but becomes crucial as you scale. If you're running a complex/slow SQL query and it's displayed for multiple people or displayed numerous times, cache the result. This way, you only need to do a key/value cache lookup on subsequent requests, which will pay huge dividends. We did this for our current visitors' box on the Fathom dashboard. If someone shared their public Fathom dashboard to a huge audience, you suddenly had thousands of people hitting us every 10 seconds with the same query, looking for real-time current visitors, potentially for hours. So we started caching the response for 10 seconds for non-authenticated users (aka random people) viewing a public dashboard. This change reduced the load on our database significantly because only one of the requests would run the expensive query while the rest would hit the cache. And these spikes never happened again because key/value queries (cache) are much more efficient.
    AWS service limits. If you're using AWS, they have limits on various services, and they can bite you. An excellent example in Laravel Vapor is that, when using secrets, you may need to increase your Parameter Store throughput. Vapor uses these when creating a container (note: the container gets re-used multiple times before restarting itself, so it doesn't hit Parameter Store every request). By default, you get access to 40 transactions per second, which is perfect for many use cases. But as we grew, we ran into errors, and we had to move to higher throughput (3,000 transactions per second). So take a look at the quotas for all of your services.

How we deploy for hyper-scale

Now that I've covered areas to "worry" about and a few tips on writing code at scale, I will talk about how we scale our application. We run a privacy-first analytics service, and our traffic is the aggregate of all of our customers' website traffic. So if we land ten new websites as customers and they each process 50 million website requests each month, our analytics service needs to process an extra 500 million requests per month. As we start to get hundreds of thousands of websites using our service, you can see why we are obsessed with scaling our service.

Before I get into how we deploy things, I should be very clear that you can scale Laravel with EC2 (with autoscaling), ECS, Heroku, DigitalOcean, etc. We don't use those services for Fathom, so I'm not going to cover them. I will talk specifically about how we set up our infrastructure to handle hyper-scale as we grow rapidly.

As a lot of you will know, I am a big fan of serverless infrastructure. I teach the Serverless Laravel course, where I have taught over a thousand people how to run their applications at scale without worrying about managing servers. That course came after we deployed our software to Laravel Vapor because I fell in love.

I'm going to cover our analytics collector application, not our dashboard. Although our dashboard receives millions of requests each month, our analytics collector (separate deployment) is the hyper-scale part of our business.

Firstly, we use a CDN as the core entry point to our infrastructure. We use bunny.net, which processes just under 1 trillion requests each month across all of its customers. They comfortably handle our traffic, provide us with basic security settings (rate limiting, etc.) and allows us to route traffic as we need (e.g. routing EU traffic to our EU infrastructure via EU isolation).

Once it gets through Bunny, it's routed to one of the following places.

    If the traffic is coming from the EU, it's routed to our multi-region EU proxy hosted on Hetzner. This setup uses Go (I didn't write it!) and is built to handle significant scale across multiple regions, but this is a small portion of our overall traffic.
    If the traffic is coming from outside the EU, it's routed to our core AWS infrastructure.

The proxy hits the same point that our "outside EU" traffic hits: our Application Load Balancer (ALB), a managed load balancer service by Amazon that can handle incredible throughput. Of course, you could roll this yourself if you have the expertise (or can hire the expertise), but we are big fans of managed services. Before the traffic gets routed past the ALB, it passes through our Web Application Firewall (WAF), which rate limits and protects against malicious traffic.

Once it's passed through the WAF & ALB, it heads to Lambda. We have a throughput limit of 60,000 requests per second burst and then we can push beyond that. At the time of writing, this equates to roughly 157 billion requests per month.

Lambda passes the request to SQS, which triggers another Lambda to process the request. We plan to replace SQS with a direct database write soon (to save money and improve performance), but we'll still keep SQS as an availability backup. We wouldn't dream of doing this if we were running RDS for MySQL, but we're now running a (database built for hyper-scale).

Once it's in Lambda being processed, we hit our database a few times (via key/value lookup in memory) for some spam evaluation (we call this our 3rd firewall) and then we run a series of reads & inserts. And this runs beautifully at scale because we're using the correct database for the job.

One of our most significant issues with scaling was the open/close of the database connection, as this is expensive. Previously, every incoming pageview would open a database connection, bootstrap the framework, run the queries, and close the connection, which wasn't scalable. But we now have Laravel Octane, which will hold open connections in memory. That means we can open up a database connection on container boot, and subsequent requests will use that same connection. And, yes, it's so much faster, and our database CPU loves us for doing this.
Is Laravel good for big projects?

I get it. You're sitting in a meeting with management, and management is concerned about whether Laravel can scale. Or perhaps you're worried about whether your side project or new business will take off and Laravel will fall over? Well, here's the stack I would use if I was starting a new project right now and management said we could get billions of requests per month:

    Laravel Vapor (deploys SQS, Cloudfront, ALB, S3)
    AWS WAF
    SingleStore (instead of RDS, Redis and DynamoDB)
    ALB (Application Load Balancer)

And you'll be good to go. So when your management says, "Hey, we need to go international, and we need servers in Asia," you can move to use Global Accelerator. You'd deploy multiple Vapor regions and put them behind it. Then you'd talk to SingleStore about cross-region replication (we don't replicate our database across regions, but I know they have customers who do).
The end

The conclusion is that Laravel is a fantastic choice for 99.99994%+ of web applications.

But if Mark Zuckerberg emailed you right now and said:

    "Hey, just read the Facebook chat messages between you and your partner; you seem exciting and adventurous. Would you rebuild Facebook for us?"

Could you use Laravel to power Facebook? Yes.
Looking for some Laravel Tips? Check out Jack's Laravel Tips section here.
You might also enjoy reading:

    Why we ditched DynamoDB
    Building the world’s fastest website analytics
    Someone attacked our company
