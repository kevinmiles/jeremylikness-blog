---
title: "Build a Serverless Link Shortener with Analytics Faster than Finishing your Latte"
author: "Jeremy Likness"
date: 2017-09-04T16:18:59.417Z
years: "2017"
lastmod: 2019-06-13T10:43:45-07:00

description: "How to leverage Azure Functions, Azure Table Storage, and Application Insights to build a serverless custom URL shortening tool."

subtitle: "How to leverage Azure Functions, Azure Table Storage, and Application Insights to build a serverless custom URL shortening tool."
tags:
 - Azure 
 - Dotnet 
 - Serverless 
 - Cloud Computing 
 - Azure Functions 

image: "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/3.png" 
images:
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/1.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/2.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/3.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/4.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/5.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/6.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/7.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/8.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/9.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/10.png" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/11.jpeg" 
 - "/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/12.gif" 


aliases:
    - "/build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte-8c094bb1df2c"
---

#### How to leverage Azure Functions, Azure Table Storage, and Application Insights to build a custom URL shortening tool.

Our team thrives on real world data. I continuously analyze my online presence to better understand what topics people are interested in so that I can focus on curating content that will drive value. A lot of what I share goes through social media feeds like Facebook, Google+ (yes, it’s still around), LinkedIn and of course Twitter. I can’t afford to take a “fire and forget” approach or I could end up sharing content and topics no one really cares about.




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/1.png)

Application Insights Dashboard for Redirects



Although there are plenty of freely available online URL shortening and tracking utilities, I was frustrated with my options. Today, Twitter doesn’t count the full link size you tweet against your 140 character budget, so it’s not really about making the links short. Instead, it’s more about tagging and tracking. I tag links so that know which medium is the most effective. Most freely available tools require me to painstakingly paste each variation of the link in order to get a short URL, and then I don’t have full control over the analytics. What’s more, the scheduling tool that promises to shorten URLs automatically ends up taking over the tracking tags so my data is corrupted.

I decided to build a tool of my own, but I didn’t want to spin up VMs and configure expensive infrastructure to handle the load of a ton of redirects going through my servers. So, I decided to go serverless: the perfect task for [Azure Functions](https://jlik.me/5x) to take on!




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/2.png)

Getting Started Templates for Azure Functions



This post walks through how the entire site was set up, but you don’t have to manually repeat the same steps. Instead, you can navigate directly to my GitHub repository:

[JeremyLikness/serverless-url-shortener](https://github.com/JeremyLikness/serverless-url-shortener/)


Click on the [Deploy to Azure button](https://jlik.me/51) and you will be prompted to fill out a few values before the template engine creates and configures a fully functioning serverless app for you! Here’s a quick video that demonstrates the full process.




Walk-through of Azure Deployment and Testing

To get started, I simply added a new function app from the portal and checked the box for [Application Insights](https://jlik.me/5y). This gives me all of the data and metrics I’ll ever need or hope for with minimal effort. For storage, I chose to go with [Azure Table Storage](https://jlik.me/5z). It’s a key/value store and is perfect for matching the key (short URL) to the value (long URL). It does its job fast!




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/3.png)

Azure Table Storage Speeds



As you can see from the chart, even the “slow” operations finish in a few hundred milliseconds and the vast majority are faster — closer to six milliseconds.

### Shortening the URL

The first function to build is the URL shortening code. It’s a good idea to keep this function secure so not just anyone can make new links. I created an HTTP binding to make it straightforward to hook up a small web app utility to generate the links, with an input binding for table storage.




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/4.png)

Input Binding



The parameters to the input binding automatically bring in the latest key to generate a new id for the shortening utility. I also created an output binding to write the new key out. This is the full bindings file for the “ingest” function that takes a long URL and makes a short one:




Notice that I’m reusing the existing storage credentials created for the function — no need to create a new storage instance. The nice thing about table storage is that you also don’t have to worry about creating the table, because it will be automatically built the first time you insert a value. You can map tables to strongly typed classes, so I created a common “models.csx” file to use in the project.




The “NextId” is a special entity used to keep track of the short URL. I simply increment the id and use an algorithm to turn it into a smaller alphanumeric value. I didn’t see the need for any exotic hash algorithms or unique generators when I can just use a simple identity field that is incremented each time. The “ShortUrl” holds the URL and the “Medium” property (i.e. did this link come from Twitter, LinkedIn, etc.). Both are based on “TableEntity” that provides some basic fields including “PartitionKey” and “RowKey.”

For the next-up identifier (“NextId”) I hard coded a partition of “1” and a row of “KEY” to always grab a single value. For the short URLs, I partition the data by the first character of the short URL. For example, a value of “ab” will go to partition “a” and row “ab” and a value of “1z” will go to partition “1” and row “1z”. This ensures an even distribution across partitions with values for performance.

The algorithm to generate a short URL simply takes the integer identifier and converts it into an alphanumeric value that is based on using all digits and alphabet values.




The code for the function first casts the request to a strongly typed class and ensures all values are present. Note in the parameters that because the input binding for the table is set to retrieve a single value, we can pass in a strongly typed model for it (“keyTable”). The output table requires some additional operations and is cast to “CloudTable.” This type of automatic binding makes functions extremely flexible and easy to use.




The first time the function is called, the key doesn’t exist so it is passed in as null. On subsequent operations the current value will be automatically bound and passed in. A little logic checks for a null key and seeds the first value.




The new URL is generated based on the “next up” key, and stored in the table. The result is collected in a list that is returned to the client in order to display the shortened URLs. If “tag mediums” is set to “true” the code will iterate through an array of mediums (Twitter, LinkedIn, etc.) and generate URLs for each variation.




The final step is to save the new key value to the database and return the results.
> Because is this is a “personal use” URL shortening tool, I’m not worried about concurrency (i.e. what if two clients request short URLs at the exact same time) or I might code the key logic differently.



That’s it! You can now pass in a value and receive the short URL. This is an example of what a request/response looks like:




### A Simple Client

To make it easier to shorten URLs, I created a simple single page web application using the [Vue.js](https://vuejs.org/) framework. Because it is just for personal use, I didn’t bother with a full web application or complex authentication. Instead, the app has a hard-coded endpoint to the function with the function secret embedded in the URL. The app simply accepts input, calls the function app and displays the results.




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/5.png)

The Shortening Utility



Use [this link](https://github.com/JeremyLikness/serverless-url-shortener/tree/master/webApp) to browse the source code for the web app. It includes a Dockerfile that builds a tiny docker image that I run locally when I’m using it to post URLs. I keep the Docker image local and run it when needed. Because the function to generate short URLs is secured, I added a Cross-Origin Resource Sharing entry to the function app so the browser will allow requests from “localhost” and a custom port. Use this link to: [Learn how to configure CORS](https://jlik.me/59).### Redirection and Custom Telemetry

The next step is to provide an endpoint for the redirection. This is done with the “UrlRedirect” function. The route is set up so that the short URL is passed as part of the path. The function itself accepts an input binding to the table and the short URL itself.

The logic is simple. A fallback URL is used to redirect to a common website if the URL is invalid. Table storage is queried for the matching entry, and if it is found the redirect URL is updated from the default to the URL from the table. Finally, the method returns a “302” redirect code with the target URL.




Now all of the pieces are in place: a function to encode short URLs and a function to decode and redirect. Earlier I mentioned checking a box for “Application Insights” when creating the function. Application Insights provides a ton of metrics “out of the box” but really shines in its ability to track custom telemetry.### Application Insights

Application insights provides rich functionality out of the box without changing a single line of code. I can see the overall health of my functions including average response time.




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/6.png)

Application Insights Request Data



You can see there was a slow spike at one point that I can investigate by clicking on the spike and drilling into individual transactions. The majority of redirects happen quickly. “Smart detection” uses machine learning to scan data and notify you when behaviors falls outside of norms, such as failed responses or extremely slow response times. In this snapshot, the host allocated a second server to accommodate requests based on request rates and response times. I didn’t have to do a thing — the functions scale themselves automatically.

Of course, seeing how long it takes to return a request is important, but I need more data to troubleshoot why slower responses happen. For example, it would be great to see how much time in a request is spent looking up the redirect URL in table storage.

Adding custom telemetry is very straightforward. I added a “project.json” file to reference the Application Insights SDK for use by the function.




Upon saving that, the function app loads the NuGet package and installs it to be available to the function. At the top of the function app I use a special keyword for package references “#r” to include the NuGet package followed by a standard “using” statement for the code.




With that, I am able to instantiate a client in the function for use. Notice that I pass it a key that is already configured in the application settings to connect to the right instance of Application Insights.




Next, I capture the current date and start a timer before calling the table operation. After it completes, I use the telemetry client to log the results.




This allows me to view the dependency data in Application Insights and see that the table queries happen very fast — at the 50th percentile in less than seven milliseconds for an overall average of about 15ms!




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/7.png)

Viewing Telemetry for Azure Table Storage Queries



For my own analytics, I capture a custom event to track the medium used and a “page view” for the target URL.




This enables me to see which mediums are the most popular.




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/8.png)

Redirects by Medium



I can also see which page views have been the most popular and learn what my audience is interested in and likely wants to see more of.




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/9.png)

Top Page Views in 24 Hours



The analytics tool is actually very comprehensive and allows you to build your own queries and aggregations and create your own charts and graphs for insights. All of this is part of the free “out of the box” insights offering. The only thing limited in the free version is the amount of data that is stored. You can opt-in to paid models that retain more data for a longer period of time to get monthly, yearly, and other trends as needed.### The Finishing Touches: Proxies and Hosts

There are a few steps I took to make my URL shortening tool production-ready. First, a URL like this:
> “http://hostname.azurewebsites.net/api/UrlRedirect/aaa”

is hardly short! Therefore, I used a proxy to map the root path or route to the API. Proxy configuration is a powerful feature for functions because it enables precise control over the endpoints of your APIs. Even if you implement five different function apps with different URLs, you can use a proxy to aggregate them under the same path. Here is the proxy configuration that is setup for this project:




The first rule flattens the root redirect, so that:
> “http://hostname.azurewebsites.net/abc”

maps to:
> “http://hostname.azurewebsites.net/api/UrlRedirect/abc”

The second rule ensures the original “api” path is preserved, so that the full path is still valid and essentially “passes through.”

I acquired a domain name to use for the short URL. One of the settings for the function app is “Custom Domains.”




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/10.png)

Custom Domains Setting



The subsequent dialog will walk you through the steps needed to verify you own the domain and point it to your function app by pointing your domain to the public IP address Azure configured. After the step is completed, the function app recognizes the custom domain and allows the redirect to work with a custom URL:




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/11.jpeg)

Using Proxies



There are several options to secure your endpoint with SSL. If you own your own SSL certificate, you can upload it to the site and leverage it for secure calls. I personally prefer to use [CloudFlare](https://www.cloudflare.com/) as a free and easy to configure option. CloudFlare allows me to configure my domain for secured connections and generates their own SSL certificate. CloudFlare connects to my Azure function app “behind the scenes” but presents a valid certificate to the client that secures the connection. It also offers various security features and caches requests to improve the responsive of the site. It will even serve cached content when your website goes down!

Now the URL shortening tool is fully up and running, allowing me to generate short URLs and happily routing users to their final destination. Now is the perfect time to mention one of the biggest benefits I received by going serverless: cost. As of this writing, the [Azure Functions pricing model](https://jlik.me/57) grants the first million function calls and 400,000 gigabyte seconds (GB-s) of consumption for free. My site has low risk of hitting those levels, so my main cost is storage. How much is storage?

Update: I recently spent thirty minutes in an interview discussing and demoing the link shortener that I use. Check that out here:

[Azure Functions: Less-Server and More Code](https://jlik.me/b1x)
> **Although actual results will differ for everyone, in my experience, running the site for a week while generating around 1,000 requests per day resulted in a massive seven cent U.S.D. charge to my bill. I don’t think I’ll have any problem affording this!**

What are you waiting for? Head over to the [repository](https://github.com/JeremyLikness/serverless-url-shortener/) and click the button to get started on your own to see just how powerful functions are!




![image](/blog/2017-09-04_build-a-serverless-link-shortener-with-analytics-faster-than-finishing-your-latte/images/12.gif)



Read the next article in this series:

[Expanding Azure Functions to the Cosmos](https://blog.jeremylikness.com/expanding-azure-functions-to-the-cosmos-423d0cb920a)