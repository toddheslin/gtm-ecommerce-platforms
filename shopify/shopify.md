Shopify Custom Analytics
========================

Setup
-----
Shopify has three primary ways of collecting analytics data.
1. The Shopify built-in analytics tracking
2. Adding the Google Analytics universal tag into Settings/Online Store
3. Injecting tracking code into the theme file (e.g GTM)

You can also add 'Additional content and scripts' in the Settings/Checkout page (`/admin/settings/checkout`), however this will only fire on the final confirmation page after a payment has been processed. It's also worth noting that all analytics enabled in (1), (2), and (3) above will *also* fire on the confirmation page.

**Be careful that you don't double up your conversion counting.**

[You can read the Shopify GA Setup blog post here](https://docs.shopify.com/manual/reports-and-analytics/google-analytics)
[There is an additional GA Goals and funnels post here](https://docs.shopify.com/manual/reports-and-analytics/google-analytics/google-analytics-goals-and-funnels)


## 1. Built-in Analytics
You haven't made it down to option 3 yet, however I've recently discovered that Shopify use Segment.com to handle their analytics tracking. It exists within the DOM as the javascript object `window.ShopifyAnalytics.lib`.

As these are sent to the Shopify-owned segment account, you won't be able to intercept the values and send them to your own analytics account. However you can query the annonymous ID and user ID of all users simply by calling: `ShopifyAnalytics.lib.user().anonymousId()` or `ShopifyAnalytics.lib.user().id()`.

I tested whether a logged-in user would populate the `ShopifyAnalytics.lib.user().id()` variable. It turns out that it doesn't. Therefore the only reliable use would be using the anonymous ID before and after checkout to identify the same user.

It's also interesting to note that the `ShopifyAnalytics.lib.user().anonymousId()` assigned to a user/browser before checkout is one value (e.g 'X'), and then when they click checkout and go to the `checkout.shopify.com' domain, they have a different value (e.g 'Y'). If you close your window but don't clear your cookies, these values remain in tact for the two different domains. On the final confirmation page you can fire your own code (Google Analytics, or other custom code), and 'Y' anonymousId will be active, and when you head back to your main site from the checkout page 'X' will be active.

I haven't seen if Shopify is tracking traits for a user (eg location), but if they did, it would show up in: `ShopifyAnalytics.lib.user().traits()`. At the moment this only returns:
```
{
	visitToken: "5574EA57-9E78-4F56-3973",
	uniqToken: "B951F1EB-2D8E-4DC8-8C23"
}
```

Not much to do here, but we might be able to use these values within GTM later on to recognise consistent users.

## 2. Google Analytics
It's been noted that some shopify themes will have Google Analytics installed. To check this, use a Plugin like Ghostery or [Google's Tag Assistant Plugin](https://chrome.google.com/webstore/detail/tag-assistant-by-google/kejbdjndbnbjgmefkgdddjlbokphdefk?hl=en)

You should only have GA loading once, and I recommend it to be setup within the Shopify admin panel in `Settings/Online Store/Google Analytics`. The primary reason is that you will have automatic sending of user and eCommerce data into Google Analytics from the Shopify integration. It's just easier.

If you chose _not_ to use the inbuilt integration, you could otherwise:
- Use Google Tag Manager to inject it
- Use Segment.com to inject it

These are both advanced methods and can be explored if you're trying to achieve a particular objective with your Google Analytics setup.

In terms of best practice, I would recommend you have two GA views. One which is raw, unfiltered and has no ecommerce enabled, and one which has been modified. This is likely to help you later when you need to debug your analytics. Simply do this:
- On a new GA account setup, rename the default view 'All Web Site Data' to '(RAW) All Web Site Data' to remind you that this is raw data. (No filters, goals, ecommerce...nothing)
- Copy that view (there's a button in 'view settings') and then make the required changes for your site. I call this 'Production' as it's your `production` site. You could make a `tes` view too, but don't bother now.

_In case you're wondering, yes if you have already set up your site with all modifications in your 'All Web Site Data' view, you could copy that and then **make** a 'raw' view by deleting all of the options. Note that it won't retroactively make these changes so you will need to wait for some time to pass for a true raw view._

[This is also a great reference for Google Analytics setup](https://www.digitaldarts.com.au/google-analytics-and-shopify-for-splendid-data)

An imporant step with Google Analytics is to add your checkout domains in the GA Referral Exclusion list. You might be thinking 'why would I want to exclude traffic from these domains?' the simple answer is that you don't want to double-count sessions from the beginning to the end of the checkout process.

Consider that the customer is on your site, clicking around and adding products to their cart. This is all one session. The moment they click 'Checkout', they head over to the `checkout.shopify.com` domain. They return into the visibility of your Google Analytics after checkout with the url that looks like this:
`https://checkout.shopify.com/11438914/checkouts/b31d8c844bc19f28f86a18f76ea941a0/thank_you`
And then they will click 'continue' and head to your primary site.

If you didn't exclude the `checkout.shopify.com` domain, you would have three sessions for this single user.

In the GA Admin panel, choose the **Property** and then 'js Tracking Info' and then 'Referral Exclusion List'. Add the following:
- `checkout.shopify.com` (if you aren't using Shopify Plus)
- `paypal.com` (if you're using the Paypal gateway)
...or any other payment gateway that shows up in your Google Analytics reports as a 'referrer'


## 3. Injecting code in the theme file (e.g GTM)
If you plan on injecting any other script tags in your theme file (`theme.liquid`), then I **strongly** recommend using Google Tag Manager as your only hardcoded script. There may be exceptions where you need scripts loaded very fast on the page (e.g. A/B testing software), however using GTM will be appropriate for almost everything relating to analytics.

When you have a GTM container loaded, I'd recommend installing your **own** implementation of [Segment](http://www.segment.com). I believe Segment is the most transformative innovation in analytics since Google Analytics appeared. You can read more about them on the website but here are the key points:
- Write a single analytics call, e.g `analytics.track(user)` and this will send the data to hundreds of integrations
- Use a data warehouse for hosting the **raw** data from your site. This allows a granularity of insight that has **never** been available unless you build your own analytics platform.
- The tool is **free** if you have under 50,000 events per month and don't require more advanced integrations

However there are some limitations specifically with shopify:
- We want to use the Shopify Google Analytics integration **not** the Segment Google Analytics Integration. This is because the tracking code for eCommerce won't be enabled in Shopify unless you enable it in Shopify. If you were to turn Google Analytics 'on' in Segment, it would be double-loading data into GA.
- Segment will only be loaded on your domain and on the last page of the `https://checkout.shopify.com` domain.
- Because of some inconsistencies with loading Segment during testing, I found it much more reliable to load through GTM. However this means some of the liquid coding not to work on the Segment/Shopify integration site.

As Shopify uses the `window.ShopifyAnalytics` global varible for their Segment tracking and your GTM-injected segment tracking uses simply `window.analytics`. However your `anonymousId` variable and `ajs_anonymous_id` cookie will be **the same**. That is:
`ShopifyAnalytics.lib.user().anonymousId() === analytics.user().anonymousId()`
`true`

This is interesting indeed.

Data warehouses is an oustanding feature of Segment. With raw access to the analytics tables, you can send data into Segment for further analysis. Here are some examples:
- Pulling out the user's checkout information (but no credit card) and putting it into your Postgres `users` table
- Tracking views to product pages, including all metadata about the products
- Tracking cart views, including cart data
- Using an array function

[Read about all the things you can do here](https://segment.com/docs/platforms/shopify/)
Although Segment recommends loading the code into your `theme.liquid` file, I have found that there were significant caching issues on Shopify. That is, the `<head>` would not reload for each page, and therefore pageviews or data triggers wouldn't fire. It was much more reliable to load GTM first and then **load segment inside GTM**.

Using GTM you can capture the checkout object (`window.Shopify.checkout`). This includes details about the person and their order. As you can't call liquid templates within GTM code (they generally fail), you are best off using this checkout object and creating GTM varibles to feed into your conversion tags.

TODO: Specific Problems
--------
- Cart abandonment
- Funnel analysis
- Attribution
- Capturing the details on final page