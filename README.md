<h1 align="center">UXNitro Fast Reviews</h1>

<h2 align="center">⚡ Lightweight Review System Built for Performance</h2>

<p align="center">
  <img src="uxnitro-fast-reviews-ui-preview.png" alt="UXNitro Fast Reviews UI" />
</p>


Plugin Version: 2.6
Author: UXNitro
Description: High-performance, lightweight review system with star ratings, filtering, and minimal database overhead.

Installation
Download the plugin .zip file.
Go to your WordPress Dashboard → Plugins → Add New → Upload Plugin.
Upload the .zip file and click Install Now.
Click Activate Plugin.
Add Reviews anywhere using the shortcode:
[ux_reviews_compact limit="10"] 
limit sets the number of reviews shown initially (default: 10).
Optional: Use the UXNitro Soul Plugin for automatic page-level optimization. If you use Soul, you don’t need to manually disable the plugin on unused pages — Soul handles it for you.
Using the Plugin
A leave a review form appears automatically below the review list.
Submitted reviews are pending moderation by default.
Admin can approve reviews via Dashboard → Comments.
Rating stats appear at the top of the review block.
Users can filter reviews by 1–5 stars dynamically (no page reload, pure JS).
Optimizing for Performance

To keep your site fast:

Load reviews only on necessary pages
If not using Soul, add the shortcode only where reviews are required.
Avoid placing [ux_reviews_compact] in global templates unless needed.
Remove unused comment data
Disable WordPress comments on pages that won’t have reviews:
Edit the page in Gutenberg, Elementor, or your block builder.
In Discussion Settings, uncheck Allow Comments.
Lazy-load reviews if needed
Reviews are compact by default and loaded inline.
For large numbers of reviews, consider lazy-loading via a plugin or custom code.
Minimize CSS/JS
The plugin loads minimal CSS/JS inline.
For extra optimization, move CSS/JS to your theme for caching.
Admin Tips
View pending reviews under Comments → All Comments.
Ratings appear as stars (★) in the admin column.
Filter reviews by star rating using the drop-down in the admin.
Approved reviews appear automatically on the front-end.
Troubleshooting
Form not submitting?
Check that wp_mail() works on your server (used for admin notifications).
Review block not showing?
Make sure the shortcode is placed inside the main content area or a compatible block.
Performance issues?
Only place the shortcode on pages that need reviews, or use the Soul plugin for automatic page-level optimization.
Best Practices
Keep the plugin active only where needed.
Disable WordPress comments on pages without reviews.
Use caching and optimization plugins for large sites.
Always moderate reviews to maintain quality.

Example: Disable Comments via Gutenberg

Open the page → Click Settings (⚙️).
Scroll to Discussion → Uncheck Allow Comments.
Update the page.

This removes unnecessary review/comment code and improves page speed.
