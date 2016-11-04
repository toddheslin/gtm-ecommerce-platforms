Shopify Custom Analytics
========================

Analytics Setup
---------------
Shopify has three primary ways of collecting analytics data.
1. The Shopify built-in analytics tracking (limited to higher plans)
2. Adding the Google Analytics universal tag and Facebook Pixel into `Online Store/Preferences` setting
3. Injecting tracking code or a tag manager into the `theme.liquid` file (e.g GTM)

You can also add 'Additional content and scripts' in the `Settings/Checkout` page, however this will _only_ fire on the final confirmation page after a payment has been processed. It's also worth noting that all analytics enabled in (1) and (2) above will *also* fire on the confirmation page but you will **also** need to add GTM from option (3) if you want it to fire on this page.

**As you could fire tracking events in GTM AND in the in-built Analytics tool, be careful that you don't double up your conversion counting.**

[You can read the Shopify GA Setup blog post here](https://docs.shopify.com/manual/reports-and-analytics/google-analytics)

[There is an additional GA Goals and funnels post here](https://docs.shopify.com/manual/reports-and-analytics/google-analytics/google-analytics-goals-and-funnels)


## 1. Built-in Analytics
You haven't made it down to option 3 yet, however I've recently discovered that Shopify use Segment.com to handle their analytics tracking. It exists within the DOM as the javascript object: `window.ShopifyAnalytics.lib`.

As these are sent to the Shopify-owned segment account, you won't be able to intercept the values and send them to your own analytics account. However you can query the annonymous ID of all users simply by calling: `ShopifyAnalytics.lib.user().anonymousId()`.

I tested whether a logged-in user would populate the `ShopifyAnalytics.lib.user().id()` variable. It turns out that it doesn't. Therefore the only reliable use would be using the **anonymous ID** before and after checkout to identify the same user.

**This is how you could stitch together multiple sessions for the same user without any additional tracking.**

It's also interesting to note that the `ShopifyAnalytics.lib.user().anonymousId()` assigned to a user/browser before checkout is one value (e.g 'X'), and then when they click 'checkout' and are redirected to the `checkout.shopify.com` domain, they have a different value (e.g 'Y'). If you close your window but don't clear your cookies, these values remain in tact for the two different domains. On the final confirmation page you can fire your own code (Google Analytics, or other custom code), and 'Y' anonymousId will be active, and when you head back to your main site from the checkout page 'X' will be active.

This opens up possibilities to track users from the beginning to the end of the checkout process with the obvious gray area in between.

I haven't seen if Shopify is tracking traits for a user (eg location), but if they did, it would show up in: `ShopifyAnalytics.lib.user().traits()`. At the moment this only returns:
```
{
	visitToken: "5574EA57-9E78-4F56-3973",
	uniqToken: "B951F1EB-2D8E-4DC8-8C23"
}
```

Not much to do here, but we might be able to use these values within GTM later on to recognise consistent users. Just an idea to explore later.

## 2. Google Analytics
It's been noted that some shopify themes will have Google Analytics installed. To check this, use a Plugin like Ghostery or [Google's Tag Assistant Plugin](https://chrome.google.com/webstore/detail/tag-assistant-by-google/kejbdjndbnbjgmefkgdddjlbokphdefk?hl=en).

You should only have GA loading once, and I recommend it to be setup within the Shopify admin panel in `Online Store/Preferences/Google Analytics`. The primary reason is that you will have automatic sending of enhanced eCommerce data into Google Analytics from the Shopify integration. It's just easier.

If you choose _not_ to use the inbuilt integration, you could otherwise:
- Use Google Tag Manager to inject it
- Use Segment.com to inject it

These are both advanced methods and can be explored if you're trying to achieve a particular objective with your Google Analytics setup. My preference is to use the inbuilt integration for general tracking and then GTM to trigger events.

In terms of best practice, I would recommend you have **two** GA views. One which is raw, unfiltered and has no eCommerce enabled, and one which has been modified. This is likely to help you later when you need to debug your analytics. Simply do this:

- On a new GA account setup, rename the default view 'All Web Site Data' to '(RAW) All Web Site Data' to remind you that this is raw data. (No filters, goals, eCommerce...nothing)

- Copy that view (there's a button in 'view settings') and then make the required changes for your site. I call this 'Production' as it's your `production` site. You could make a `test` view too, but don't bother now.

_In case you're wondering, yes, if you have already set up your site with all modifications in your 'All Web Site Data' view, you could copy that and then **make** a 'raw' view by deleting all of the options. Note that it won't retroactively make these changes so you will need to wait for some time to pass for a true raw view._

[This is also a brilliant reference for ideal Google Analytics setup on shopify](https://www.digitaldarts.com.au/google-analytics-and-shopify-for-splendid-data)

An important step with Google Analytics is to add your checkout domains in the GA Referral Exclusion list. You might be thinking 'why would I want to exclude traffic from these domains?' the simple answer is that you don't want to double-count sessions from the beginning to the end of the checkout process.

Consider that the customer is on your site, clicking around and adding products to their cart. This is all one session. The moment they click 'Checkout', they head over to the `checkout.shopify.com` domain. They return into the visibility of your Google Analytics after checkout with the url that looks like this:
`https://checkout.shopify.com/11438914/checkouts/b31d8c844bc19f28f86a18f76ea941a0/thank_you`
And then they will click 'continue' and head to your primary site.

If you didn't **exclude** the `checkout.shopify.com` domain, you would have three sessions for this single user. It should be a single session.

In the GA Admin panel, choose the **Property** and then 'js Tracking Info' and then 'Referral Exclusion List'. Add the following:

- `checkout.shopify.com` (if you aren't using Shopify Plus)

- `paypal.com` (if you're using the Paypal gateway)
...or any other payment gateway that shows up in your Google Analytics reports as a 'referrer'


## 3. Injecting code in the theme file (e.g GTM or Segment)
If you plan on injecting any other script tags in your theme file (`theme.liquid`), then I **strongly** recommend using Google Tag Manager as your **only** hardcoded script. Alternatively you could use Segment and it will inject GTM (see below). There may be a small number of exceptions where you need scripts loaded very fast on the page (e.g. A/B testing software such as Optimizely), however using GTM will be appropriate for injecting almost everything relating to analytics.

The one exception I have found is when you would like to populate liquid variables into the GTM dataLayer. This is best done in the `.liquid` template files because GTM won't recognise the liquid variables.

I recommend installing your **own** implementation of [Segment](http://www.segment.com) *(remember: shopify uses Segment to provide their 'insights' to users)*. You could install Segment *through* a GTM tag, or you could 'turn on' the GTM integration within Segment. The result should be close to the same.

I choose to add Segment scripts in all `.liquid` files and let it populate the GTM dataLayer for me. [See Segment/GTM API here](https://segment.com/docs/integrations/google-tag-manager/).

I believe Segment is the most transformative innovation in analytics since Google Analytics appeared. You can read more about them on their website but here are the key points:

- Write a single analytics call, e.g `analytics.track({eyes: 'blue'})` and this will send the data to hundreds of integrations [including the GTM dataLayer](https://segment.com/docs/integrations/google-tag-manager/). One line of code goes everywhere.

- Use your own data warehouse for hosting the **raw** data from your site. This allows a granularity of insight that has **never** been available unless you build your own analytics platform and data pipeline. Considering you can set up a [PostgreSQL database on Heroku for free](https://www.heroku.com/postgres) AND this is part of the free version of Segment...Why wouldn't you?

- The tool is **free** if you have under 50,000 events per month and don't require more advanced integrations. Best. Free. Tool. Ever!

However there is one limitation specifically with Shopify:

- We want to use the **Shopify** Google Analytics integration **not** the Segment Google Analytics Integration. This is because the tracking code for enhanced eCommerce won't be enabled in Shopify unless you enable it **in Shopify**. If you were to turn Google Analytics 'on' in Segment, it would be double-loading data into GA, or would have to mimick all of the calls to GA that Shopify has built-in. Don't do that.

Shopify uses the `window.ShopifyAnalytics` global variable for their Segment tracking. Where your GTM-injected segment tracking uses simply `window.analytics`. However your `anonymousId` variable and `ajs_anonymous_id` cookie will be **the same**. That is:
`ShopifyAnalytics.lib.user().anonymousId() === analytics.user().anonymousId()` is
`true`

This is interesting indeed.

Data warehouses are an outstanding feature of Segment. With raw access to the analytics tables, you can send data into Segment for further analysis. Here are some examples:

- Pulling out the user's checkout information (but no credit card) and putting it into your Postgres `users` table
- Tracking views to product pages, including all metadata about the products
- Tracking cart views, including cart data

[Although it's now unsupported, you can read about the Segment Shopify integration here](https://segment.com/docs/platforms/shopify/). I've tested it out and generally works quite well. All limitations have been documented however if you have a custom theme built there could be additional tweaks to be made.

---

## Capturing details on checkout page

This is one area where you need to use GTM instead of Segment. Explanation at the bottom of this section.

You can capture the checkout objects (`window.Shopify.checkout` and `window.Shopify.Checkout`) on the final checkout page using GTM. The difference is that the `Checkout` object (capital **C**) is present throughout each stage of the checkout, whereas the `checkout` object is only available on the final 'order complete' or 'thank you' page.

`window.Shopify.checkout` includes details about the person and their order. As you can't call liquid templates within GTM code (they generally fail), you have two options to get these values into GTM:

- get the values directly from the `window.Shopify.checkout` object with some Javascript in a GTM tag that only fires on this page.

- add some liquid templates into the `Settings/Checkout/Additional Scripts` setting to push the data into GTM.

It's also worth noting that this is called when a customer chooses to 'View Order' from one of your emails and ends up at `https://checkout.shopify.com/...`.

Although I'd prefer to use Segment instead of GTM here it turns out that this `Additional scripts` script is loaded BEFORE the hidden checkout page hack I've outlined below. This means that the `analytics` variable isn't available to call `analytics.track()`
[See this URL in section '2f: Thank You Page'](https://segment.com/docs/platforms/shopify/)

The solution? Call the segment code from within GTM that only triggers on the GTM event `transactionComplete`. How does this work?

1. dataLayer is populated with these data (see below)

2. Segment library is loaded on the checkout page using the same method as it does on the *hidden* pages (see next section)

3. Because Segment loads GTM, the GTM container will load, check the dataLayer for past events and process them according to any triggers set. Read [Simon Ahava's blog](http://www.simoahava.com/gtm-tips/add-the-event-key-to-datalayer-pushes/) about this one.

All will be sequential. Here is the tag to add into GTM:

Thanks to the awesome work of [Alec in his article here](http://www.whencanistop.com/2015/11/analytics-on-shopify.html). I've modified his script to make it work for exposing the values into GTM without making a bunch of individual variables. All you need is one called 'checkout'.

Copy this into the Shopify `Checkout/Additional Scripts` box.

```
// calculate totalDiscount for Completed Order Event
var discounts = "{{ order.discounts | json }}"
var totalDiscount = 0;

for (var i = 0; i< discounts.length; i++ ) {
totalDiscount += discounts[i].savings;
}

window.dataLayer = window.dataLayer || [];
dataLayer.push({
    'event' : 'transactionComplete',
    'checkout' : function(){
      return {
        'transactionId': '{{order.order_number}}',
        'transactionTotal': {{total_price | times: 0.01}},
        'transactionRevenue': {{subtotal_price | times: 0.01}},
        'transactionShipping': {{shipping_price | times: 0.01}},
        'transactionTax': {{tax_price | times: 0.01}},
        'transactionDiscount': totalDiscount,
        'transactionProducts': [
        {% for line_item in line_items %}
          {
          'id': '{{line_item.product_id}}',
          'sku': '{{line_item.sku}}',
          'name': '{{line_item.title}}',
          'category': '{{line_item.type}}',
          'price': {{line_item.line_price | times: 0.01}},
          'quantity': {{line_item.quantity}}
          },
        {% endfor %}]
      }
}})
</script>
```

In GTM you need to create a single variable of type `Data Layer Variable`. I call mine `DL-Checkout` and the 'Data Layer Variable Name' field is 'checkout'.

Then create a new tag. I call mine 'Checkout Push to Segment' and make it only trigger on the Custom Trigger *(I call mine 'Shopify Checkout')* that is of type `Custom Event` and Event Name of `transactionComplete`.

In the tag, use the following Segment call:

```
<script>
window.analytics.track('Completed Order', {
   orderId: {{DL-Checkout}}().transactionId,
   total: {{DL-Checkout}}().transactionTotal,
   revenue: {{DL-Checkout}}().transactionRevenue,
   shipping: {{DL-Checkout}}().transactionShipping,
   tax: {{DL-Checkout}}().transactionTax,
   discount: {{DL-Checkout}}().transactionDiscount,
   products: {{DL-Checkout}}().transactionProducts
});  
</script>
```

## Accessing the hidden checkout pages in Google Analytics
As you know by know, the pages after the user clicks 'Checkout' go off the radar of your own GTM and custom analytics implementation until you reach the final `/checkout/thank_you/` page. However, if you are using the built-in Google Analytics implementation by Shopify, they will track the cross-domain events and send this back into your Google Analytics account. Very cool.

Thanks to [Alec for pointing this out in his Shopify notes here](http://www.whencanistop.com/2015/11/analytics-on-shopify.html)

In fact, the progressive pages of the checkout look like this:
`https://checkout.shopify.com/11438914/checkouts/092d0570a9d704fad8422e9fbfdb8c12?step=contact_information`

`https://checkout.shopify.com/11438914/checkouts/092d0570a9d704fad8422e9fbfdb8c12?previous_step=contact_information&step=shipping_method`

`https://checkout.shopify.com/11438914/checkouts/092d0570a9d704fad8422e9fbfdb8c12?previous_step=shipping_method&step=payment_method`

What you will notice is that in Google Analytics, you can go to the report in: `Behaviour/Site Content/All Pages` and you will find some URLs that begin with `/checkout/` and are followed by the 'step' that you see at the end of each of those links above.

This lets us create a funnel to track the beginning to the end of the checkout process in Google Analytics.

## Adding Custom GTM code in checkout
So it turns out it can be done. I've done it myself. Here is how.

When you install Google Analytics onto Shopify it will ask you if you want to add any 'Additional Google Analytics Javascript'. Why yes I do! I'd like to add Segment, which in turn will add GTM:
```
if(location.hostname == "checkout.shopify.com"){

    [ADD YOUR SEGMENT JS SNIPIT HERE]
		window.analytics.load("YOUR WRITE KEY HERE");
  	window.analytics.page();
		window.analytics.track('Order Progressed', {step : window.Shopify.Checkout.step})
};
```
Note that I added the first line to check it the website was on the checkout.shopify.com hostname. Why? Because you don't want this firing on every page in your site (remember this is a global analytics implementation). For those other pages, it will be ignored and only on the checkout domain it will fire.

With this in place you will now be tracking an event in Segment called 'Order Progressed' with the `step` for each stage of the process [contact_information, shipping_method, payment_method, thank_you].

One problem with this implementation for the FINAL checkout page is that it will load **after** the Shopify `Additional scripts` option in `Settings/Checkout`. What to do? Instead of adding the code there, I would instead add it as a conditional like we did with the if() block above. That is:

```
if(location.pathname.split('/').pop() == 'thank_you'){

}
```

**Warning: this is dangerous if left in the hands of the wrong people. Why? Because you could write a GTM tag that captures the credit card details from the user.** Firstly, don't do it. Secondly, protect your Segment and GTM logins.


---

# TODO: Specific Problems

- Cart abandonment
- Funnel analysis
- Attribution
- Capturing the details on final page
