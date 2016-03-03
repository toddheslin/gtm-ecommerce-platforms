What's this about?
==================
My central repository for the analytics tools I use, references I rely on and code snippits that save me hours of time. Feel free to use and contribute.

I will help with any issues that are submitted. Problems that are specific to your site or campaign verge on being client work. If it's custom for you, you can choose to me. If it's generic for everyone, I'll help in my space time.

Enjoy!

Terminology
-----------
TBC — There is plenty to describe here.

Platforms
---------
### Google Tag Manager
Google Tag Manager is a powerful tool for managing javascript functions that are 'injected' into the web page, instead of 'scripted' into the source code. Whilst the discussion for the good vs evil of using a tag manager can be left behind, when used properly, GTM provides a level of flexibility that is exceptional from an analytics perspective.

### Segment.com
Segment is the most transformative innovation in analytics since Google Analytics appeared. You can read more about them on the [website](http://www.segment.com) but here are the key points:
- Write a single analytics call such as `analytics.track(user)` and this will send the data to hundreds of integrations you have enabled in the Segment console. This can include Google Analytics, Intercom, Mixpanel, Kissmetrics etc.
- Use a data warehouse for hosting the **raw** data from your site analytics fee. This allows a granularity of insight that has **never** been available unless you build your own analytics platform.
- The tool is **free** if you have under 50,000 events per month and don't require more advanced integrations. Hook this up to a Postgres database and you can ask questions of your own data.

### Google Analytics
Who doesn't use Google Analytics? As it's typically the core of most web analytics, it makes sense to address how GA fits in your overal analytics strategy.

### PostgreSQL via Amazon RDS or Heroku
PostgreSQL has quickly become my favourite relational database. I think Derrek Sivers summed it up most eloquently [here](https://sivers.org/pg) and his Github repo is [here](https://github.com/sivers/pg).

Aside from a local implimentation, I've traditionally used Amazon RDS for its ease of setup. However I've more recently found Heroku to be supidly simple.

It's important to note that the great thing about Postgres is that there are many similarities with the Amazon Petabyte-scale Redshift. If you're dealing with lots of data, you will outgrow Postgres and need to fork out the $250+ per month for a Redshift instance.


### Chartio
How do you visualise data? Whilst tools range from the free Google Analytics dashboard through to paid alternatives such as Tableau, my favourite by far is [Chartio](http://www.chartio.com).

Other cool options which I don't use are:
- [Looker]
- [JackDB]
- [Mode]
- [Periscope]

What I love about Chartio is:
- you can use their visual editor or write SQL for queries (or as I sometimes do, use both)
- create different queries as layers and join them together
- connect any database as a data source and join this to Google Analytics data
- schedule updates to your queries and exports to email as CSV or PDFs

Overall it's an impressive yet simple tool. Unlike the complicated Tableau, they focus on elegant design and power under the hood.

Technology used
---------------
This is the same tech stack I use for my consulting clients. Everything here is the 'source code' of how I run my Analytics as a Service (no acronym needed) practice.


Specific guides
---------------
[Shopify Guide](https://github.com/toddheslin/gtm-ecommerce-platforms/blob/master/shopify/shopify.md)

[Squarespace Guide — TBC]()

Need help?
----------
Feel free to email me at wow@beingremarkable.me (redirects to my personal email)