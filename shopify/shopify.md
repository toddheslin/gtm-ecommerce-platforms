Shopify Custom Analytics
========================

Analytics Setup
---------------
Shopify has three primary ways of collecting analytics data.
1. The Shopify built-in analytics tracking (limited to higher plans)
2. Adding the Google Analytics universal tag and Facebook Pixel into `Online Store/Preferences` setting
3. Injecting tracking code or a tag manager into the `theme.liquid` file (e.g Google Tag Manager)

You can also add 'Additional content and scripts' in the `Settings/Checkout` page, however this will _only_ fire on the final confirmation page after a payment has been processed. It's also worth noting that all analytics enabled in (1) and (2) above will *also* fire on the confirmation page but you will **also** need to add GTM from option (3) if you want it to also fire on this page.

**As you could fire tracking events in GTM AND in the in-built Analytics tool, be careful that you don't double up your conversion counting.**

Always use the [Google Tag Assistant](https://chrome.google.com/webstore/detail/tag-assistant-by-google/kejbdjndbnbjgmefkgdddjlbokphdefk?hl=en) to check your implementation.

[You can read the Shopify GA Setup blog post here](https://docs.shopify.com/manual/reports-and-analytics/google-analytics)

[There is an additional GA Goals and funnels post here](https://docs.shopify.com/manual/reports-and-analytics/google-analytics/google-analytics-goals-and-funnels)


## 1. Built-in Analytics
~~You haven't made it down to option 3 yet, however I've recently discovered that Shopify use Segment.com to handle their analytics tracking. It exists within the DOM as the javascript object: `window.ShopifyAnalytics.lib`.~~

_Update: It turns out that Shopify might be using a custom implementation of the Segment analytics.js library. They seem to make lots of references to 'trekkie' in their tracking code._

There's not much to do here, but we might be able to use these values within GTM later on to recognise consistent users. Just an idea to explore later.

## 2. Google Analytics
It's been noted that some shopify themes will have Google Analytics pre-installed. To check this, use a Plugin like Ghostery or [Google's Tag Assistant Plugin](https://chrome.google.com/webstore/detail/tag-assistant-by-google/kejbdjndbnbjgmefkgdddjlbokphdefk?hl=en).

You should only have GA loading once, and I recommend it to be setup within the Shopify admin panel in `Online Store/Preferences/Google Analytics`. The primary reason is that you will have automatic sending of enhanced eCommerce data into Google Analytics from the Shopify integration. It's simply easier.

If you choose _not_ to use the inbuilt integration, you could otherwise:
- Use Google Tag Manager to inject it
- Use Segment.com to inject it

These are both advanced methods and can be explored if you're trying to achieve a particular objective with your Google Analytics setup. My preference is to use the inbuilt integration for general tracking and then GTM to trigger specific events (but not page views).

In terms of best practice, I would recommend you have **two** GA views. One which is raw:

- unfiltered
- no eCommerce enabled
- no goals

And one which has been modified with all of these things. This is likely to help you later when you need to debug your analytics. Simply do this:

- On a new GA account setup, rename the default view 'All Web Site Data' to '(RAW) All Web Site Data' to remind you that this is raw data. (No filters, goals, eCommerce...nothing)

- Copy that view (there's a button in 'view settings') and then make the required changes for your site. I call this 'Production' as it's your `production` site. You could make a `test` view too, but don't bother now.

_In case you're wondering, yes, if you have already set up your site with all modifications in your 'All Web Site Data' view, you could copy that and then **make** a 'raw' view by deleting all of the options. Note that it won't retroactively make these changes so you will need to wait for some time to pass for a true raw view._

[This is also a brilliant reference for ideal Google Analytics setup on Shopify](https://www.digitaldarts.com.au/google-analytics-and-shopify-for-splendid-data). It's a long read.

An important step with Google Analytics is to add your checkout domains in the GA Referral Exclusion list. You might be thinking 'why would I want to exclude traffic from these domains?' The simple answer is that you don't want to double-count sessions from the beginning to the end of the checkout process.

Consider that the customer is on your site, clicking around and adding products to their cart. This is all one session. The moment they click 'Checkout', they head over to the `checkout.shopify.com` domain. They return into the visibility of your Google Analytics after checkout with the url that looks like this:
`https://checkout.shopify.com/11438914/checkouts/b31d8c844bc19f28f86a18f76ea941a0/thank_you`
And then they will click 'continue' and head to your primary site.

If you didn't **exclude** the `checkout.shopify.com` domain, you would have three sessions for this single user. It should be a single session.

In the GA Admin panel, choose the **Property** and then 'js Tracking Info' and then 'Referral Exclusion List'. Add the following:

- `checkout.shopify.com` (if you aren't using Shopify Plus)

- `paypal.com` (if you're using the Paypal gateway)
...or any other payment gateway that shows up in your Google Analytics reports as a 'referrer'

### Using permalinks
There's a cool feature to Shopify where you can add a permalink to your shopify cart, preloading the specific product, quantity and discount. It would look something like this `yoursite.com/cart/0000000:1?discount=amazing`. [Here is the Shopify article about this](https://help.shopify.com/themes/customization/cart/use-permalinks-to-preload-cart).

You can send people from a blog post or other site direct to your cart. Whilst this is a cool feature, the significant limitation I found was that your Google attribution tracking (along with adwords and everything else) goes *bang*. That's right, if you have a big 'buy now' button on your home page that skipped the entire cart process and pushed people straight to the checkout, you'll never know how they converted. :(

The problem was hard to diagnose but easy to fix. Within `Online Store/Preferences/Google Analytics` you can add additional scripts. The first step is to identify your buttons that use the permalink and set them as variables. Then you run the GA _linker_ plugin to automatically append the Google client ID to the URL. E.g:
```
var heroCta = document.getElementsByClassName('hero__cta')[0];
var footerCta = document.getElementsByClassName('footer__cta')[0];
var buyTwoCta = document.getElementById('buy-two');
ga('require', 'linker');
ga('linker:decorate', heroCta);
ga('linker:decorate', footerCta);
ga('linker:decorate', buyTwoCta);
```

## 3. Injecting code in the theme file (e.g GTM or Segment)
If you plan on injecting any other script tags in your theme file (`theme.liquid`), then I **strongly** recommend using Google Tag Manager as your **only** hardcoded script. Alternatively you could use a service called Segment and it will inject GTM for you (see below). There may be a small number of exceptions where you need scripts loaded very fast on the page (e.g. A/B testing software such as Optimizely), however using GTM will be appropriate for injecting almost everything relating to analytics.

The one exception I have found is when you would like to populate liquid variables into the GTM dataLayer. This is best done in the `.liquid` template files or the 'Additional content and scripts' in the `Settings/Checkout`. As GTM won't recognise the liquid variables, you'll need to push a dataLayer event to GTM with liquid template variables (e.g. order ID and order total) and then use the GTM dataLayer variable within your tracking tags.

I recommend installing an implementation of [Segment](http://www.segment.com) if you plan to also push your data to a data warehouse. I won't get into how awesome this functionality is, but if you really want to understand what is happening on your site (and you can write SQL queries), Segment is your best friend. You could install Segment *through* a GTM tag, or you could 'turn on' the GTM integration within Segment. The result should be close to the same but I prefer to do the latter.

I choose to add Segment scripts in the `theme.liquid` file and let it populate the GTM dataLayer for me. [See Segment/GTM API here](https://segment.com/docs/integrations/google-tag-manager/).

I believe Segment is the most transformative innovation in analytics since Google Analytics appeared. You can read more about them on their website but here are the key points:

- Write a single analytics call, e.g `analytics.track({eyes: 'blue'})` and this will send the data to hundreds of integrations [including the GTM dataLayer](https://segment.com/docs/integrations/google-tag-manager/). One line of code goes everywhere.

- Use your own data warehouse for hosting the **raw** data from your site. This allows a granularity of insight that has **never** been available unless you build your own analytics platform and data pipeline. Considering you can set up a [PostgreSQL database on Heroku for free](https://www.heroku.com/postgres) AND this is part of the free version of Segment... If you're a nerd, why wouldn't you?

- The tool is **free** if you have under 50,000 events per month and don't require more advanced integrations. Best. Free. Tool. Ever!

However there is one limitation specifically with Shopify:

- We want to use the **Shopify** Google Analytics integration **not** the Segment Google Analytics Integration. This is because the tracking code for enhanced eCommerce won't be enabled in Shopify unless you enable it **in Shopify**. If you were to turn Google Analytics 'on' in Segment, it would be double-loading data into GA, or would have to mimick all of the calls to GA that Shopify has built-in. Don't do that.

Data warehouses are an outstanding feature of Segment. With raw access to the analytics tables, you can send data into Segment for further analysis. Here are some examples:

- Pulling out the user's checkout information (but no credit card) and putting it into your Postgres `users` table
- Tracking views to product pages, including all metadata about the products
- Tracking cart views, including cart data

[Although it's now unsupported, you can read about the Segment Shopify integration here](https://segment.com/docs/platforms/shopify/). I've tested it out and generally works quite well. All limitations have been documented however if you have a custom theme built there could be additional tweaks to be made.

---

## Capturing details on final checkout page

This is one area where you need to use GTM instead of Segment. Explanation at the bottom of this section.

You can capture the checkout objects (`window.Shopify.checkout` and `window.Shopify.Checkout`) on the final checkout page using GTM. The difference is that the `Checkout` object (capital **C**) is present throughout each stage of the checkout, whereas the `checkout` object is only available on the **final** 'order complete' / 'thank you' page.

`window.Shopify.checkout` includes details about the person and their order. As you can't call liquid templates within GTM code but you can in the 'additional scripts' code, you have two options to get these values into GTM:

- get the values directly from the `window.Shopify.checkout` object with some Javascript in a GTM tag that only fires on this page.

- add some liquid templates into the `Settings/Checkout/Additional Scripts` setting to push the data into the GTM dataLayer.

I'm choosing the second option for safety. Shopify can easily change their checkout javascript variables but would be more cautious about changing their liquid API.

It's also worth noting that this is called when a customer chooses to 'View Order' from one of your emails and ends up at `https://checkout.shopify.com/...`. Therefore you need to use the `{% if first_time_accessed %}` liquid control tag to ensure your conversion events aren't fired twice.

While we are on this page, it's worth identifying the user so that you can send the order metadata to any of your other integrations.

I'd prefer to call the Segment API (which in turn loads and passes data into GTM) instead of directly calling GTM here however it turns out that this `Additional scripts` script is loaded *BEFORE* the hidden checkout page hack I've outlined in the next section below. This means that the `analytics` variable isn't available to call `analytics.track()`
[See this URL in section '2f: Thank You Page'](https://segment.com/docs/platforms/shopify/)

The solution? Call the Segment code from *within* GTM and only trigger on the GTM `transactionComplete` event. Let's break out the steps:

1. dataLayer is populated with these data (see below)

2. Segment library is loaded on the checkout page using the same method as it does on the *hidden* pages (see next section).

3. Because Segment loads GTM, the GTM container will load, check the dataLayer for past events and process them according to any triggers set. Read [Simon Ahava's blog](http://www.simoahava.com/gtm-tips/add-the-event-key-to-datalayer-pushes/) about this one.

All will be sequential. Here is the tag to add into GTM:

Thanks to the awesome work of [Alec in his article here](http://www.whencanistop.com/2015/11/analytics-on-shopify.html). I've modified his script to make it work for exposing the values into GTM without making a bunch of individual variables.

Copy this into the Shopify `Checkout/Additional Scripts` box.

```
<script>
window.dataLayer = window.dataLayer || [];

function transactionComplete() {
	var totalDiscount = 0;

	{% for discount in order.discounts %}
		totalDiscount += {{ discount.savings | times: 0.01 }};
	{% endfor %}
	return dataLayer.push({
			event : 'transactionComplete',
			checkout : function(){
				return {
					'transactionId': '{{order.order_number}}',
					'transactionTotal': {{total_price | times: 0.01}},
					'transactionRevenue': {{subtotal_price | times: 0.01}},
					'transactionCurrency': window.ShopifyAnalytics.meta.currency || "USD",
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
	}});
};

function orderStatus() {
	return dataLayer.push({
			event : 'orderStatus',
			customer : function(){
				return {
					id: '{{customer.id}}',
					name: "{{ customer.name }}",
			    firstName: "{{ customer.first_name }}",
			    lastName: "{{ customer.last_name }}",
			    email: "{{ customer.email }}",
			    phone: "{{ customer.default_address.phone }}",
			    address: {  // uses the default address
			        street: "{{ customer.default_address.street }}",
			        city: "{{ customer.default_address.city }}",
			        state: "{{ customer.default_address.province }}",
			        stateCode: "{{ billing_address.province_code }}",
			        postalCode: "{{ customer.default_address.zip }}",
			        country: "{{ customer.default_address.country }}",
			        countryCode: "{{ customer.default_address.country_code }}"
			    },
			    totalSpent: "{{ customer.total_spent }}",
			    allOrdersCount: "{{ customer.orders_count }}",
			    allOrderIds: [{% for order in customer.orders %}"{{ order.id }}",{% endfor %}],
			    tags: [{% for tag in customer.tags %} "{{ tag }}", {% endfor %}]
				}
	}});
}

{% if first_time_accessed %}
	transactionComplete();
	orderStatus();
{% else %}
	orderStatus();
{% endif %}
</script>
```

For the instructions below I will refer to two sets of variables, tags and triggers. I always refer to them in the same order.

In GTM you need to create two variables of type `Data Layer Variable`. I call mine `DL-Checkout` and `DL-Status` and the 'Data Layer Variable Name' field is `checkout` and `customer` respectively.

Then create two new tags. I call mine `Checkout Push to Segment` and `Customer Push to Segment`. Ensure each only triggers on their **own** Custom Trigger for each tag *(I call my triggers 'Shopify Checkout' and 'Viewed Checkout Status Page - non-order')*. These triggers are both of type `Custom Event` and Event Name is `transactionComplete` and `orderStatus` respectively.

In the first tag, use the following Segment call:

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

and the second:
```
<script>
  window.analytics.identify({{DL-Status}}().id, {
     name: {{DL-Status}}().name,
     firstName: {{DL-Status}}().firstName,
     lastName: {{DL-Status}}().lastName,
     email: {{DL-Status}}().email,
     phone: {{DL-Status}}().phone,
     address: {
         street: {{DL-Status}}().address.street,
         city: {{DL-Status}}().address.city,
         state: {{DL-Status}}().address.state,
         stateCode: {{DL-Status}}().address.stateCode,
         postalCode: {{DL-Status}}().address.postalCode,
         country: {{DL-Status}}().address.country,
         countryCode: {{DL-Status}}().address.countryCode
     },
     totalSpent: {{DL-Status}}().totalSpent,
     allOrdersCount: {{DL-Status}}().allOrdersCount,
     allOrderIds: {{DL-Status}}().allOrderIds,
     tags: {{DL-Status}}().tags
  });

  window.analytics.track('Viewed Status Page');
</script>
```

## Accessing data in the hidden checkout pages
As you know by now, the pages after the user clicks 'Checkout' go off the radar of your own GTM and custom analytics implementation until you reach the final `/checkout/thank_you/` page. However, if you are using the built-in Google Analytics implementation by Shopify, they will track the cross-domain events and send this back into your Google Analytics account. Very cool.

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

	  if (!window.Shopify.checkout) {
	    window.analytics.track('Order Progressed', {
	      step : window.Shopify.Checkout.step
	    })
	  }
};
```

If you had any GA linker scripts as mentioned at the bottom of section 2 above, these are at the **very top** of this script box.

Note that I added the first line to check it the website was on the `checkout.shopify.com` hostname. Why? Because you don't want this firing on every page in your site. This script box is global to every page on your site. For those other pages, the extra segment code will be ignored and it will only on the checkout domain where your `theme.liquid` implementation won't fire.

With this in place you will now be tracking an event in Segment called 'Order Progressed' with the `step` for each stage of the process [contact_information, shipping_method, payment_method]. We purposely don't track the final thank you page because we're sending a custom 'Order Complete' event. 'Order Progressed' and 'Order Complete' shouldn't fire on the same page.

**Warning: this is dangerous if you provide full Shopify access to the wrong people. Why? Because you could write a GTM tag that captures the credit card details from the user.** Firstly, don't do it. Secondly, protect your Segment and GTM logins.

## Cross Domain Cookie tracking
Thanks to [this](https://github.com/contently/xdomain-cookies), I've also implemented cross domain tracking to take the user ID from the checkout page and make it available when the user returns to the site. I'll update these details in due course.

---

# TODO: Specific Problems

- Cart abandonment
- Funnel analysis
- Attribution
- Capturing the details on final page
