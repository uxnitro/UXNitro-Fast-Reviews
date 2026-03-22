UXNitro Compact Reviews – Installation & Optimization Guide

Plugin Version: 2.6
Author: UXNitro
Description: Display compact, star-rated reviews with admin moderation and WordPress comment integration.

1. Installation
Download the plugin .zip file.
Go to your WordPress Dashboard → Plugins → Add New → Upload Plugin.
Upload the .zip file and click Install Now.
Activate the plugin.
Add the reviews anywhere using the shortcode:
[ux_reviews_compact limit="10"]
limit sets the number of reviews shown initially. Default is 10.
Optionally, use the UXNitro Soul Plugin for automatic page-level optimization. If using Soul, you do not need to manually disable the plugin on unused pages, as Soul handles this for you.
2. Using the Plugin
Leave a review form appears automatically under the review list.
Users submit reviews which are pending moderation by default.
Admin can approve reviews under Dashboard → Comments.
Rating stats appear at the top of the review block with a filter to view 1–5 star reviews.
Filter works dynamically with no page reload (pure JS).
3. Optimizing for Performance

To keep your site fast:

Load reviews only on pages that need them
If not using Soul, add the shortcode only on pages where reviews are required.
Avoid adding [ux_reviews_compact] in global templates unless needed.
Remove unused comment data
Disable WordPress comments on pages that won’t receive reviews:
Edit the page in your block builder (e.g., Gutenberg, Elementor).
In Discussion Settings, uncheck Allow Comments.
This prevents unnecessary review/comment code from loading.
Lazy load review lists if needed
Reviews are compact by default and loaded inline, but if you have many reviews, consider using plugins or code to lazy-load.
Minimize CSS/JS
The plugin loads minimal CSS/JS inline.
You can move CSS/JS to your theme for caching optimization if needed.
4. Admin Tips
View pending reviews under Comments → All Comments.
Rating is shown as a ★ visual in the admin column.
Filter reviews by star rating using the drop-down in the comments admin.
Approved reviews appear on the front-end automatically.
5. Troubleshooting
Form not submitting? Check that wp_mail() works on your server for admin notifications.
Review block not showing? Ensure the shortcode is placed inside the main content area or a compatible block.
Performance issues? Only place the shortcode on necessary pages, or use the Soul plugin for automatic page-level optimization.
6. Best Practices
Keep the review plugin active only where needed.
Disable WordPress comments on pages where reviews are not used.
Use caching and optimization plugins for large sites.
Always moderate reviews to maintain quality.
Example: Disable Comments via Gutenberg Block Builder
Open the page → Click Settings (⚙️).
Scroll to Discussion → Uncheck Allow Comments.
Update the page.

This removes unnecessary review/comment code from pages and improves speed.
