# posts

Does Laravel Scale?

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
