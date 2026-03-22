<?php
/*
Plugin Name: UXNitro Fast Reviews
Plugin URI: https://uxnitro.com/nitro-plugin-scanner/
Description: High-performance, lightweight review system with star ratings, filtering, and minimal database overhead.
Version: 2.2
Author: UXNitro
Author URI: https://uxnitro.com/
License: GPL2+
License URI: https://www.gnu.org/licenses/gpl-2.0.html
Text Domain: uxnitro-fast-reviews
*/

if (!defined('ABSPATH')) exit;

// Create database table on activation
register_activation_hook(__FILE__, 'ux_reviews_create_table');
function ux_reviews_create_table() {
    global $wpdb;
    $table_name = $wpdb->prefix . 'ux_reviews_v2';
    
    $charset_collate = $wpdb->get_charset_collate();
    
    $sql = "CREATE TABLE IF NOT EXISTS $table_name (
        id mediumint(9) NOT NULL AUTO_INCREMENT,
        author varchar(100) NOT NULL,
        email varchar(100) NOT NULL,
        rating tinyint(1) NOT NULL,
        content text NOT NULL,
        status varchar(20) DEFAULT 'approved',
        created_at datetime DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY (id)
    ) $charset_collate;";
    
    require_once(ABSPATH . 'wp-admin/includes/upgrade.php');
    dbDelta($sql);
}

add_shortcode('ux_reviews_compact', 'ux_reviews_compact_display');

function ux_reviews_compact_display($atts) {
    $atts = shortcode_atts(array('limit' => 10), $atts);
    
    global $wpdb;
    $table_name = $wpdb->prefix . 'ux_reviews_v2';
    
    // Handle review submission
    $success_message = '';
    $errors = array();
    
    if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['submit_review'])) {
        $name = sanitize_text_field($_POST['review_name']);
        $email = sanitize_email($_POST['review_email']);
        $rating = intval($_POST['review_rating']);
        $comment = sanitize_textarea_field($_POST['review_comment']);
        
        if (empty($name)) $errors[] = 'Name is required.';
        if (empty($email) || !is_email($email)) $errors[] = 'Valid email is required.';
        if ($rating < 1 || $rating > 5) $errors[] = 'Please select a rating (1-5 stars).';
        if (strlen($comment) < 10) $errors[] = 'Review must be at least 10 characters.';
        
        if (empty($errors)) {
            // Save to database
            $result = $wpdb->insert(
                $table_name,
                array(
                    'author' => $name,
                    'email' => $email,
                    'rating' => $rating,
                    'content' => $comment,
                    'status' => 'approved',
                    'created_at' => current_time('mysql')
                ),
                array('%s', '%s', '%d', '%s', '%s', '%s')
            );
            
            if ($result) {
                $success_message = 'Thank you for your review! It has been added.';
                
                // Optional: Send email notification
                $admin_email = get_option('admin_email');
                $subject = 'New Review Submitted';
                $message = "New review from: $name\nRating: $rating/5\n\n$comment";
                wp_mail($admin_email, $subject, $message);
                
                // Redirect to avoid form resubmission
                $redirect_url = add_query_arg('review_success', '1', remove_query_arg('rating'));
                wp_redirect($redirect_url);
                exit;
            } else {
                $errors[] = 'Error saving your review. Please try again.';
            }
        }
    }
    
    // Check for success message from redirect
    if (isset($_GET['review_success'])) {
        $success_message = 'Thank you for your review! It has been added.';
    }
    
    // Get filter from URL
    $min_rating = isset($_GET['rating']) ? intval($_GET['rating']) : 0;
    
    // Get reviews from database
    $reviews = $wpdb->get_results("
        SELECT author, rating, content, created_at 
        FROM $table_name 
        WHERE status = 'approved' 
        ORDER BY created_at DESC 
        LIMIT 100
    ", ARRAY_A);
    
    // Sample reviews if no database reviews exist
    if (empty($reviews)) {
        $reviews = array(
            array('author' => 'James Patterson', 'rating' => 5, 'content' => 'Ran this on 5 client sites. Identified plugin conflicts on 3 of them that I never noticed. The child theme warning saved one client from losing custom code during an update. Well worth installing.'),
            array('author' => 'Mike Thompson', 'rating' => 4, 'content' => 'Great tool overall. The Cloudflare detection worked perfectly and the recommendations were solid. Only giving 4 stars because of external font detection.'),
            array('author' => 'Lisa Wong', 'rating' => 4, 'content' => 'Good insights but had minor glitches. The scan got stuck at 90% twice.'),
            array('author' => 'Rachel Martinez', 'rating' => 5, 'content' => 'Went from a 35% grade to 82% in one week! The vault functions cleaned up so much WordPress bloat. Best free scanner I\'ve used.'),
            array('author' => 'Tom Harrison', 'rating' => 5, 'content' => 'The 3-step code guide is brilliant! Even as a non-technical store owner, I could copy-paste the performance code and see immediate results.')
        );
    }
    
    // Filter reviews by rating
    $filtered = array();
    foreach ($reviews as $review) {
        if ($min_rating == 0) {
            $filtered[] = $review;
        } elseif ($min_rating == 5 && $review['rating'] == 5) {
            $filtered[] = $review;
        } elseif ($min_rating == 4 && $review['rating'] >= 4) {
            $filtered[] = $review;
        } elseif ($min_rating == 3 && $review['rating'] >= 3) {
            $filtered[] = $review;
        } elseif ($min_rating == 2 && $review['rating'] >= 2) {
            $filtered[] = $review;
        } elseif ($min_rating == 1 && $review['rating'] >= 1) {
            $filtered[] = $review;
        }
    }
    
    // Apply limit
    $filtered = array_slice($filtered, 0, intval($atts['limit']));
    
    // Calculate average rating
    $avg = 0;
    $total = 0;
    $rating_counts = array(1 => 0, 2 => 0, 3 => 0, 4 => 0, 5 => 0);
    
    foreach ($reviews as $review) {
        $total += $review['rating'];
        $rating_counts[$review['rating']]++;
    }
    if (count($reviews) > 0) {
        $avg = round($total / count($reviews), 1);
    }
    
    // Start output
    ob_start();
    ?>
    
    <div class="ux-reviews-simple">
        
        <!-- Summary with all ratings -->
        <div class="ux-summary">
            <div class="ux-avg-rating">
                ⭐ <?php echo $avg; ?>/5 from <?php echo count($reviews); ?> reviews
            </div>
            <div class="ux-rating-breakdown">
                <?php for ($i = 5; $i >= 1; $i--): ?>
                    <div class="ux-rating-line">
                        <span class="ux-star-label"><?php echo str_repeat('★', $i); ?></span>
                        <div class="ux-rating-bar">
                            <div class="ux-rating-fill" style="width: <?php echo count($reviews) > 0 ? ($rating_counts[$i] / count($reviews) * 100) : 0; ?>%"></div>
                        </div>
                        <span class="ux-rating-count">(<?php echo $rating_counts[$i]; ?>)</span>
                    </div>
                <?php endfor; ?>
            </div>
        </div>
        
        <!-- Complete Filter Links - 1-5 Stars -->
        <div class="ux-filters">
            <a href="?rating=0" class="<?php echo $min_rating == 0 ? 'active' : ''; ?>">⭐ All Reviews</a>
            <a href="?rating=5" class="<?php echo $min_rating == 5 ? 'active' : ''; ?>">★★★★★ 5 Stars</a>
            <a href="?rating=4" class="<?php echo $min_rating == 4 ? 'active' : ''; ?>">★★★★ 4 Stars+</a>
            <a href="?rating=3" class="<?php echo $min_rating == 3 ? 'active' : ''; ?>">★★★ 3 Stars+</a>
            <a href="?rating=2" class="<?php echo $min_rating == 2 ? 'active' : ''; ?>">★★ 2 Stars+</a>
            <a href="?rating=1" class="<?php echo $min_rating == 1 ? 'active' : ''; ?>">★ 1 Star+</a>
        </div>
        
        <!-- Reviews List -->
        <div class="ux-reviews-list">
            <?php if (empty($filtered)): ?>
                <div class="ux-no-reviews">
                    <p>✨ No reviews found with this rating. Be the first to leave a review! ✨</p>
                </div>
            <?php else: ?>
                <?php foreach ($filtered as $review): ?>
                <div class="ux-review-item">
                    <div class="stars">
                        <?php echo str_repeat('★', $review['rating']) . str_repeat('☆', 5 - $review['rating']); ?>
                    </div>
                    <p><?php echo esc_html($review['content']); ?></p>
                    <small>— <?php echo esc_html($review['author']); ?></small>
                </div>
                <?php endforeach; ?>
            <?php endif; ?>
        </div>
        
        <!-- Leave a Review Form -->
        <div class="ux-review-form-container">
            <h3>⭐ Leave a Review</h3>
            <p class="ux-form-note">Your email address will not be published. Required fields are marked *</p>
            
            <?php if ($success_message): ?>
                <div class="ux-success-message">✅ <?php echo $success_message; ?></div>
            <?php endif; ?>
            
            <?php if (!empty($errors)): ?>
                <div class="ux-error-message">
                    <?php foreach ($errors as $error): ?>
                        <div>❌ <?php echo $error; ?></div>
                    <?php endforeach; ?>
                </div>
            <?php endif; ?>
            
            <form method="POST" class="ux-review-form">
                <div class="ux-form-group">
                    <label for="review_name">Name *</label>
                    <input type="text" id="review_name" name="review_name" required value="<?php echo isset($_POST['review_name']) ? esc_attr($_POST['review_name']) : ''; ?>">
                </div>
                
                <div class="ux-form-group">
                    <label for="review_email">Email *</label>
                    <input type="email" id="review_email" name="review_email" required value="<?php echo isset($_POST['review_email']) ? esc_attr($_POST['review_email']) : ''; ?>">
                </div>
                
                <div class="ux-form-group">
                    <label>Rating *</label>
                    <div class="ux-star-rating-input">
                        <input type="radio" id="star5" name="review_rating" value="5" <?php echo (isset($_POST['review_rating']) && $_POST['review_rating'] == 5) ? 'checked' : ''; ?> required>
                        <label for="star5" title="5 stars">★</label>
                        
                        <input type="radio" id="star4" name="review_rating" value="4" <?php echo (isset($_POST['review_rating']) && $_POST['review_rating'] == 4) ? 'checked' : ''; ?>>
                        <label for="star4" title="4 stars">★</label>
                        
                        <input type="radio" id="star3" name="review_rating" value="3" <?php echo (isset($_POST['review_rating']) && $_POST['review_rating'] == 3) ? 'checked' : ''; ?>>
                        <label for="star3" title="3 stars">★</label>
                        
                        <input type="radio" id="star2" name="review_rating" value="2" <?php echo (isset($_POST['review_rating']) && $_POST['review_rating'] == 2) ? 'checked' : ''; ?>>
                        <label for="star2" title="2 stars">★</label>
                        
                        <input type="radio" id="star1" name="review_rating" value="1" <?php echo (isset($_POST['review_rating']) && $_POST['review_rating'] == 1) ? 'checked' : ''; ?>>
                        <label for="star1" title="1 star">★</label>
                    </div>
                </div>
                
                <div class="ux-form-group">
                    <label for="review_comment">Your Review *</label>
                    <textarea id="review_comment" name="review_comment" rows="4" required minlength="10"><?php echo isset($_POST['review_comment']) ? esc_textarea($_POST['review_comment']) : ''; ?></textarea>
                </div>
                
                <button type="submit" name="submit_review" class="ux-submit-btn">Submit Review</button>
            </form>
        </div>
        
        <!-- CTA Button -->
        <div class="ux-cta">
            <a href="https://uxnitro.com/nitro-plugin-scanner/" class="ux-button">Run Your Scan Now</a>
        </div>
    </div>
    
    <style>
    .ux-reviews-simple {
        max-width: 820px;
        margin: 20px auto;
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    }
    
    /* Summary */
    .ux-summary {
        background: #f9fafb;
        border-radius: 12px;
        padding: 20px;
        margin-bottom: 20px;
        border: 1px solid #e5e7eb;
    }
    .ux-avg-rating {
        font-size: 24px;
        font-weight: 700;
        margin-bottom: 15px;
        color: #1f2937;
    }
    .ux-rating-breakdown {
        margin-top: 10px;
    }
    .ux-rating-line {
        display: flex;
        align-items: center;
        gap: 10px;
        margin-bottom: 8px;
        font-size: 13px;
    }
    .ux-star-label {
        min-width: 50px;
        color: #f59e0b;
    }
    .ux-rating-bar {
        flex: 1;
        height: 8px;
        background: #e5e7eb;
        border-radius: 4px;
        overflow: hidden;
    }
    .ux-rating-fill {
        height: 100%;
        background: #f59e0b;
        border-radius: 4px;
        transition: width 0.3s ease;
    }
    .ux-rating-count {
        min-width: 35px;
        color: #6b7280;
    }
    
    /* Filters */
    .ux-filters {
        margin-bottom: 20px;
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
    }
    .ux-filters a {
        display: inline-block;
        padding: 8px 14px;
        text-decoration: none;
        color: #3b82f6;
        background: #f3f4f6;
        border-radius: 8px;
        font-size: 14px;
        font-weight: 500;
        transition: all 0.2s;
    }
    .ux-filters a:hover {
        background: #e5e7eb;
        transform: translateY(-1px);
    }
    .ux-filters a.active {
        background: #3b82f6;
        color: white;
    }
    
    /* Reviews List */
    .ux-reviews-list {
        margin-bottom: 30px;
    }
    .ux-review-item {
        padding: 20px 0;
        border-bottom: 1px solid #e5e7eb;
        transition: all 0.2s;
    }
    .ux-review-item:hover {
        background: #fafafa;
        padding-left: 10px;
    }
    .stars {
        color: #f59e0b;
        font-size: 16px;
        margin-bottom: 10px;
        letter-spacing: 2px;
    }
    .ux-review-item p {
        margin: 10px 0;
        line-height: 1.6;
        color: #374151;
        font-size: 15px;
    }
    .ux-review-item small {
        color: #6b7280;
        font-size: 12px;
        display: inline-block;
        padding: 4px 8px;
        background: #f3f4f6;
        border-radius: 6px;
    }
    .ux-no-reviews {
        text-align: center;
        padding: 40px;
        color: #6b7280;
        background: #f9fafb;
        border-radius: 12px;
    }
    
    /* Review Form */
    .ux-review-form-container {
        background: #f9fafb;
        padding: 28px;
        border-radius: 12px;
        margin: 30px 0;
        border: 1px solid #e5e7eb;
    }
    .ux-review-form-container h3 {
        margin-top: 0;
        margin-bottom: 8px;
        font-size: 22px;
        font-weight: 700;
        color: #1f2937;
    }
    .ux-form-note {
        font-size: 12px;
        color: #6b7280;
        margin-bottom: 20px;
    }
    .ux-form-group {
        margin-bottom: 20px;
    }
    .ux-form-group label {
        display: block;
        margin-bottom: 8px;
        font-weight: 600;
        font-size: 14px;
        color: #374151;
    }
    .ux-form-group input[type="text"],
    .ux-form-group input[type="email"],
    .ux-form-group textarea {
        width: 100%;
        padding: 10px 12px;
        border: 1px solid #d1d5db;
        border-radius: 8px;
        font-size: 14px;
        font-family: inherit;
        transition: all 0.2s;
    }
    .ux-form-group input:focus,
    .ux-form-group textarea:focus {
        outline: none;
        border-color: #3b82f6;
        box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
    }
    
    /* Star Rating Input */
    .ux-star-rating-input {
        display: flex;
        flex-direction: row-reverse;
        justify-content: flex-end;
        gap: 10px;
    }
    .ux-star-rating-input input {
        display: none;
    }
    .ux-star-rating-input label {
        font-size: 32px;
        color: #d1d5db;
        cursor: pointer;
        transition: all 0.2s;
        margin: 0;
        padding: 0;
    }
    .ux-star-rating-input label:hover,
    .ux-star-rating-input label:hover ~ label,
    .ux-star-rating-input input:checked ~ label {
        color: #f59e0b;
        transform: scale(1.05);
    }
    
    /* Messages */
    .ux-success-message {
        background: #d1fae5;
        color: #065f46;
        padding: 14px;
        border-radius: 8px;
        margin-bottom: 20px;
        border: 1px solid #a7f3d0;
        font-size: 14px;
        font-weight: 500;
    }
    .ux-error-message {
        background: #fee2e2;
        color: #991b1b;
        padding: 14px;
        border-radius: 8px;
        margin-bottom: 20px;
        border: 1px solid #fecaca;
        font-size: 14px;
    }
    .ux-error-message div {
        margin: 4px 0;
    }
    
    /* Submit Button */
    .ux-submit-btn {
        background: #3b82f6;
        color: white;
        border: none;
        padding: 12px 28px;
        border-radius: 8px;
        font-weight: 600;
        cursor: pointer;
        font-size: 15px;
        transition: all 0.2s;
        width: 100%;
    }
    .ux-submit-btn:hover {
        background: #2563eb;
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
    }
    
    /* CTA Button */
    .ux-cta {
        margin-top: 30px;
        text-align: center;
    }
    .ux-button {
        display: inline-block;
        background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
        color: white;
        padding: 12px 28px;
        border-radius: 8px;
        text-decoration: none;
        font-weight: 600;
        font-size: 15px;
        transition: all 0.2s;
    }
    .ux-button:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
    }
    
    /* Responsive */
    @media (max-width: 640px) {
        .ux-reviews-simple {
            margin: 10px;
        }
        .ux-filters {
            gap: 6px;
        }
        .ux-filters a {
            padding: 6px 10px;
            font-size: 12px;
        }
        .ux-review-form-container {
            padding: 20px;
        }
        .ux-star-rating-input label {
            font-size: 28px;
        }
        .ux-avg-rating {
            font-size: 20px;
        }
    }
    </style>
    
    <script>
    // Enhance star rating interaction
    document.addEventListener('DOMContentLoaded', function() {
        // Make stars clickable
        var starLabels = document.querySelectorAll('.ux-star-rating-input label');
        starLabels.forEach(function(label) {
            label.addEventListener('click', function(e) {
                var radioId = this.getAttribute('for');
                var radio = document.getElementById(radioId);
                if (radio) {
                    radio.checked = true;
                }
            });
        });
        
        // Add smooth scroll to form after filter click
        var filterLinks = document.querySelectorAll('.ux-filters a');
        filterLinks.forEach(function(link) {
            link.addEventListener('click', function(e) {
                // Allow the link to work normally but add smooth scroll
                setTimeout(function() {
                    var formContainer = document.querySelector('.ux-review-form-container');
                    if (formContainer && window.location.hash === '') {
                        formContainer.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
                    }
                }, 100);
            });
        });
    });
    </script>
    
    <?php
    return ob_get_clean();
}
