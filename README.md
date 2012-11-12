# README

## Intro

Shopify provides APIs for both fulfillment and carrier service creation and management.

This app provides an example of using both.

Read the section below and browse the code to learn how to set these APIs up for your own purposes.

## Setup  

### Common steps:

1. Sign up as a [Shopify Partner](http://partners.shopify.com).
2. Request beta access by emailing [apps@shopify.com](mailto:apps@shopify.com). Tell us what you're planning on building as well as the email associated with your Partners account.
3. Create an app through the Partners Dashboard.

### Steps to set up a fulfillment service:

1. Add the `write_fulfillments` permission to your [requested scopes](http://api.shopify.com/authentication.html)
2. On install, create a new fulfillment service. It needs the following params:

  * `name` - The name of your service
  * `handle` - The URL-compatible version of your app's name
  * `remote_address` (If doing stock checks) - The endpoint that Shopfify should hit to get stock and tracking data
  * `inventory_management` - Does your service support inventory management?
  * `tracking_support` - Does your service provide tracking numbers?
  * `requires_shipping_method` - Does products fulfilled by your service require a shipping method?

  Example cURL request: `curl -X POST -d @fulfillment_service.json -H "Content-Type:application/json" http://myshop.myshopify.com/admin/fulfillment_services`

  fulfillment_service.json:
  <pre><code>{
      'name': 'My Fulfillment Service',
      'handle': 'my_fulfillment_service',
      'remote_address': 'http://myapp.com',
      'inventory_management': true,
      'tracking_support': true,
      'requires_shipping_method': true
  }</code></pre>
3. Subscribe to `fulfillments/create` and `fulfillments/update` webhooks
4. Expose the following GET endpoints:
  * `[remote_address]/fetch_tracking_numbers`: expects a list of Shopify order IDs, returns the tracking numbers for those orders.
      * Example request: `http://myapp.com/fetch_tracking_numbers?order_ids[]=1&order_ids[]=2&order_ids[]=3`
      * Example response:

          <pre><code>{
              'tracking_numbers': {
                '1': 'qwerty',
                '2': 'asdfg',
                '3': 'zxcvb'
              },
              'message': 'Successfully received the tracking numbers',
              'success': true
          }</code></pre>
  * `[remote_address]/fetch_stock`: Expects a sku and a shop name, returns the stock level for that sku
      * Example request: `https://myapp.com/fetch_stock?sku=123&shop=testshop.myshopify.com`
      * Example response: `{'123': 1000}`

### Steps to set up a carrier/shipping service:

1. Add the `write_shipping` permission to your requested scopes
2. On install, create a new carrier service through the API. It needs the following params:
  * `name` - The name of your service
  * `callback_url` - The endpoint Shopify should hit for shipping rates
  * `format` - The format you want to talk to Shopify in (`xml` or `json`)
  * `service_discovery` - Should merchants be able to send test requests to your service?

  Example cURL request: `curl -X POST -d @carrier_service.json -H "Content-Type:application/json" http://myshop.myshopify.com/admin/carrier_services`

  carrier_service.json:
  <pre><code>{
      'name': 'My Carrier Service',
      'callback_url': 'http://myapp.com',
      'format': 'json',
      'service_discovery': true
    }
  </code></pre>
3. Expose the following POST endpoint:
  * `[callback_url]`: Expects a request for shipping rates. Should return an array of applicable rates.
      * Example request:
      <pre><code>{
              "rate": {
                "origin": {
                  "country": "CA", 
                  "postal_code": "K1S4J3", 
                  "province": "ON", 
                  "city": "Ottawa", 
                  "name": "", 
                  "address1": "520 Cambridge Street South", 
                  "address2": "", 
                  "address3": "", 
                  "phone": "", 
                  "fax": "", 
                  "address_type": "", 
                  "company_name": ""
                }, 
                "destination": {
                  "country": "CA", 
                  "postal_code": "K1S 3T7", 
                  "province": "ON", 
                  "city": "Ottawa", 
                  "name": "Jason Normore", 
                  "address1": "520 Cambridge Street South Apt. 5", 
                  "address2": "", 
                  "address3": "", 
                  "phone": "7097433959", 
                  "fax": "", 
                  "address_type": "", 
                  "company_name": ""
                }, 
                "items": [
                  {
                    "name": "My Product 3", 
                    "sku": "", "quantity": 1, 
                    "grams": 1000, 
                    "price": 2000, 
                    "vendor": "TestVendor", 
                    "requires_shipping": true, 
                    "taxable": true, 
                    "fulfillment_service": "manual"
                  }
                ], 
                "currency": "CAD"
              }
            }
      </code></pre>
      * Example response:
      <pre><code>{
              "rates": [ {
                  'service_name': 'canadapost - overnight',
                  'service_code': 'ON',
                  'total_price': '12.95',
                  'currency': 'CAD'
                },
                {
                  'service_name': 'fedex - 2 day ground',
                  'service_code': '1D',
                  'total_price': '29.34',
                  'currency': 'USD'
                }
              ]
            }
      </code></pre>