# Pricing-intelligence-prototype
Backhouse Founding Engineer Case Study

## Objective
Build a working prototype that allows a Backhouse supply-side partner
(vendor or manufacturer) to monitor the pricing of three SKUs (restaurant
equipment or supplies) across major distributor marketplaces such as
Webstaurant, KaTom, and Restaurant Warehouse.

## Requirements
* Track current prices for the selected SKUs across multiple distributorsites.
* Refresh automatically on a set schedule (e.g., daily) so supply-side users see up-to-date pricing.
* Store and display basic price history so users can view trends over time.

### Approach
Before developing solutions, I like to ask myself questions about potential challenges and constraints to gain a deeper understanding of the problem by conducting hypotheses. This deeper understanding of the problem allows me to design a proof of concept of how a solution will work. This was the main question for the hypothesis I wanted to conduct:
*  "How can we reference the same SKU across multiple distributor marketplaces if the naming of the product variant and SKUs across these marketplace sites is most likely different?"


#### Original Hypotheses
**Q:** "How can we reference the same SKUs across multiple distributor marketplaces if the naming of the product variant and SKUs across these marketplace sites is most likely different?"
* **A:** Create a canonical product in your system and map each marketplace’s local listing (with that site’s SKU/ID) to it. Then you track prices per listing and roll them up to the canonical product.

### Action Steps:
Based off the original questions that I asked myself and answered, I started taking action steps to validate whether my Hypotheses were correct or not.

**Action Step for the Canonical Product Hypythesis**
```
**Q:** "How can we reference the same product variant across multiple distributor marketplaces if the naming of the product variant and SKUs across these marketplace sites is most likely different?"
* **A:** Create a canonical product in your system and map each marketplace’s local listing (with that site’s SKU/ID) to it. Then you track prices per listing and roll them up to the canonical product.
```

My first step was figuring out how I can implement an efficient method of searching for products across all sites based off of common datapoints that these sites all shared. I believed that I needed to grab generally accepted product variant data points across all the distributor sites in order to efficiently search SKUs and their prices. I started by searching the Vitamix brand's "The Quiet One" Blender, I realized cross-site data isn’t standardized. Each marketplace tracks different fields and update cycles, so SKU datapoints do not line up exactly across sites. I found that the data points across these sites were either named differently, or the values were inconsistent.

* **Figure 1**: Katom Specifications for the Vitamix brand's "The Quiet One" Blender
	* <img width="470" height="596" alt="{6851EBAC-23D7-4D48-BD73-6449EE43E6DB}" src="https://github.com/user-attachments/assets/a3d24181-4705-4dfa-b596-bcdce0f68a80" />

* **Figure 2**: Webstaurant Specifications for the Vitamix brand's "The Quiet One" Blender
  *  <img width="341" height="721" alt="{8212E05C-4682-41F9-9853-258B5BD5DE00}" src="https://github.com/user-attachments/assets/e8aeb97a-45ce-474c-943e-6ca15512e3fc" />

**Note**: The Restraunt Warehouse site did not have any results for the Vitamix brand's "The Quiet One" Blender, nor does it store specification data in a tabular format.

Based on my findings, my original hypothesis for creating a canonical product in my system proved to be **INCORRECT** as I could not build a robust canonical method for searching SKUs across multiple sites. If I were to go ahead and attempt to create a canonical method, it would involve a solution that contains a lot of **spaghetti code**-code that involes a lot of hard-coding and special casing for each site's quirks. 

#### How I Built Off of My Failed Hypothesis
I realized the only consistent data point across all sites was the **Manufacturer Part Number (MPN)**. Because MPNs are assigned by the manufacturer, they’re outside the seller’s control and can serve as a unique identifier for the same SKU across all three distributor sites. This led me to conclude that searching for a SKU across multiple distributors must be precise, since most other data points can be misaligned because they’re seller-scoped.

### Design Questions
Now that I how I can cross reference SKUs on multiple distributor sites, I started asking myself the following questions to come up with a Proof of Concept.

**Q:** "What data visualization should be displayed on the front-end to showcase the pricing history for each SKU?"
* **A:**
	*  Line chart (sparkline) per SKU: date on X, price on Y; one line per marketplace (3 lines) or small-multiples (1 mini-chart per site).
 	*  Hover tooltip: date, site, price; mark price-change points.
  	*  Window toggle: 7/30/90-day; show current price and Δ (abs/%).
  	*  Meta: “Last updated” timestamp below the chart.

**Q:** "Are there any third party api services that can be used for a more robust and programmatic approach than scraping?"
* **A:** There are none available. In order to extract the data we must implement web scraping logic across the three distributor sites.

**Q:** What data entities am I storing?
* **A:**
	* `products(id, mpn, …)`
 	* `listings(id, product_id FK → products.id, site, url, marketplace_sku, …) `
  	* `prices(id, listing_id FK → listings.id, collected_at, price, …)` <- price snapshots
  	* **MPN** lives on products. Each snapshot (prices) links → listings → products (and thus to the MPN).

**Q:** "What is the design workflow of a user's input to price tracking?"
* **A:**
	* Simple example (end-to-end)
		* User picks 3 products (by MPN).
		* System finds each product’s page on Webstaurant, KaTom, and Restaurant Warehouse, confirms the variant, grabs the current price (+ local SKU/URL), and stores a snapshot-timestamped row per market place listing saved in the DB. 
		* Dashboard shows a row per product with each site’s price + “last updated,” and a tiny price-history sparkline. 
		* A daily scheduler re-scrapes and appends new points so users always see up-to-date prices and trends—even if nobody opens the app.
  			* Scheduler queries the DB for tracked products and their mapped listings (site + URL + local SKU).
     		* It scrapes by listing URL
       			* only uses the MPN for product detail page rediscovery if the request to the listing is not a status code of 200)
          		* Handle 404s/redirects with a re-resolve-by-MPN fallback and retry/backoff.
       		* Verifies MPN on page, extract price/stock, and append a snapshot to the history table.
