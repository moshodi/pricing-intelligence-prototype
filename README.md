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
Before developing solutions, I like to ask myself questions about potential challenges and constraints to gain a deeper understanding of the problem. This deeper understanding of the problem allows me to design a proof of concept of how a solution will work. These were the following questions I originally had:
*  "How can we reference the same product variant across multiple distributor marketplaces if the naming and SKUs across these marketplace sites is most likely different?"
*  "What if a user enters a non-specific input for a product? (e.g. Keurig Coffee Maker rather than "Keurig K-Mini Plus — Black")
*  "What is the workflow of a user's input to price tracking?"
*  "What are the common attributes for a product variant that are shared across distributor sites?"
*  "What data visualization should be displayed on the front-end to showcase the pricing history for each SKU?"
*  "Are there any third party api services that can be used for a more robust and programmatic approach than scraping?"

#### Answers to These Questions
**Q:** "How can we reference the same product variant across multiple distributor marketplaces if the naming and SKUs across these marketplace sites is most likely different?"
* **A:** Create a canonical product in your system and map each marketplace’s local listing (with that site’s SKU/ID) to it. Then you track prices per listing and roll them up to the canonical product.

**Q:** "What if a user enters a non-specific input for a product? (e.g. Keurig Coffee Maker rather than "Keurig K-Mini Plus — Black")
* **A:** If a user types a non-specific input that backend will do a fuzzy search on the 'products' table. The UI will then show a short list of exact variants. A user must pick an exact one.

**Q:** "What is the workflow of a user's input to price tracking?"
* **A:** User searches by name; app shows variants -> user picks exact variant
	* Frontend:
 		* User searches by name; app shows variants -> user picks exact variant
 		* Three chosen products show a cross-site price table (Webstaurant, KaTom, Restaurant Warehouse), “last updated,” and a small price-history sparkline. (Meets “track across sites,” “auto-refresh,” “display history.”)
   	* Backend:
   		* Data model: products (canonical variant), listings (per-site: URL, local SKU/ID), prices (time-series).
   	 	* Identity resolution: when a variant is selected, link each marketplace listing to that product
   	  	* Collectors: per-listing fetch/parsers pull current price & stock and write a snapshot to prices
   	  	* Scheduler: daily job triggers collectors (and on-demand refresh) so data stays current.
   	  	* API: endpoints to read current prices, history, and manage products/listings.

**Q:** "What are the common attributes (or data points) for a product variant that are shared across distributor sites?"
* **A:** Across Webstaurant, KaTom, and Restaurant Warehouse a Manufacturer Part Number (MPN) can be used as a strong indicator of the same product variant. On a Webstaurant product variant page, the MPN is shown as `MFR: <MPN>`. On a KaTom product variant page, the MPN is shown as `MPN: <MPN>`. On the Restaurant Warehouse product variant page, the MPN declared in the actual name of each product variant.

**Q:** "What data visualization should be displayed on the front-end to showcase the pricing history for each SKU?"
* **A:**
	*  Line chart (sparkline) per SKU: date on X, price on Y; one line per marketplace (3 lines) or small-multiples (1 mini-chart per site).
 	*  Hover tooltip: date, site, price; mark price-change points.
  	*  Window toggle: 7/30/90-day; show current price and Δ (abs/%).
  	*  Meta: “Last updated” timestamp below the chart.


