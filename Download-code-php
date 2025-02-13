<?php
/*
Plugin Name: Download with verification code
Plugin URI: 
Description: ایجاد سیستم دانلود با کد تأیید، مدیریت چند لینک، طراحی زیبا و امکانات پیشرفته
Version: 1.0.2
Author: sajjadakbari
Author URI: sajjadakbari.ir
*/

// ثبت منو در پیشخوان وردپرس
add_action('admin_menu', 'download_code_admin_menu');
function download_code_admin_menu() {
    add_menu_page(
        'مدیریت لینک‌های دانلود',
        'لینک‌های دانلود',
        'manage_options',
        'download-code-manager',
        'download_code_manager_page',
        'dashicons-download',
        6
    );
}

// ایجاد صفحه مدیریت لینک‌ها
function download_code_manager_page() {
    if (!current_user_can('manage_options')) return;

    // ذخیره لینک جدید
    if (isset($_POST['submit_new_link'])) {
        check_admin_referer('save_new_download_link');

        $download_links = array_map('esc_url_raw', $_POST['download_links']);
        $require_login = isset($_POST['require_login']) ? 1 : 0;
        $generated_code = str_pad(rand(0, 9999), 4, '0', STR_PAD_LEFT);

        $links = get_option('download_code_links', array());
        $links[] = array(
            'code' => $generated_code,
            'links' => $download_links,
            'require_login' => $require_login,
            'pages' => array(), // برای ذخیره آدرس صفحاتی که شورت‌کد در آنها استفاده شده است
        );
        update_option('download_code_links', $links);

        echo '<div class="notice notice-success"><p>لینک جدید با موفقیت ذخیره شد!</p></div>';
    }

    // حذف لینک
    if (isset($_GET['action']) && $_GET['action'] === 'delete' && isset($_GET['index'])) {
        check_admin_referer('delete_download_link');

        $index = intval($_GET['index']);
        $links = get_option('download_code_links', array());

        if (isset($links[$index])) {
            unset($links[$index]);
            update_option('download_code_links', array_values($links)); // بازسازی ایندکس‌ها
            echo '<div class="notice notice-success"><p>لینک با موفقیت حذف شد!</p></div>';
        }
    }

    $links = get_option('download_code_links', array());
    ?>
    <div class="wrap">
        <h1>مدیریت لینک‌های دانلود</h1>

        <!-- فرم افزودن لینک جدید -->
        <h2>افزودن لینک جدید</h2>
        <form method="post">
            <?php wp_nonce_field('save_new_download_link'); ?>
            <table class="form-table">
                <tr>
                    <th scope="row"><label for="download_links">لینک‌های دانلود</label></th>
                    <td>
                        <textarea name="download_links" id="download_links" rows="5" class="regular-text" required></textarea>
                        <p class="description">هر لینک را در یک خط جدید وارد کنید.</p>
                    </td>
                </tr>
                <tr>
                    <th scope="row"><label for="require_login">نیاز به ثبت‌نام</label></th>
                    <td>
                        <input type="checkbox" name="require_login" id="require_login" value="1">
                        <span class="description">اگر تیک بخورد، فقط کاربران وارد‌شده می‌توانند لینک را دریافت کنند.</span>
                    </td>
                </tr>
            </table>
            <?php submit_button('ذخیره لینک جدید', 'primary', 'submit_new_link'); ?>
        </form>

        <!-- لیست لینک‌های موجود -->
        <hr>
        <h2>لینک‌های موجود</h2>
        <table class="wp-list-table widefat fixed striped">
            <thead>
                <tr>
                    <th>کد</th>
                    <th>لینک‌ها</th>
                    <th>نیاز به ثبت‌نام</th>
                    <th>صفحات استفاده‌شده</th>
                    <th>شورت کد</th>
                    <th>عملیات</th>
                </tr>
            </thead>
            <tbody>
                <?php if (empty($links)): ?>
                    <tr>
                        <td colspan="6">هیچ لینکی ثبت نشده است.</td>
                    </tr>
                <?php else: ?>
                    <?php foreach ($links as $index => $link): ?>
                        <tr>
                            <td><?php echo esc_html($link['code']); ?></td>
                            <td>
                                <ul>
                                    <?php foreach ($link['links'] as $url): ?>
                                        <li><?php echo esc_url($url); ?></li>
                                    <?php endforeach; ?>
                                </ul>
                            </td>
                            <td><?php echo $link['require_login'] ? 'بله' : 'خیر'; ?></td>
                            <td>
                                <?php if (!empty($link['pages'])): ?>
                                    <ul>
                                        <?php foreach ($link['pages'] as $page): ?>
                                            <li><a href="<?php echo esc_url($page); ?>" target="_blank"><?php echo esc_url($page); ?></a></li>
                                        <?php endforeach; ?>
                                    </ul>
                                <?php else: ?>
                                    <span>—</span>
                                <?php endif; ?>
                            </td>
                            <td><code>[download_code code="<?php echo esc_attr($link['code']); ?>"]</code></td>
                            <td>
                                <a href="<?php echo wp_nonce_url(
                                    admin_url('admin.php?page=download-code-manager&action=delete&index=' . $index),
                                    'delete_download_link'
                                ); ?>" class="button button-danger">حذف</a>
                            </td>
                        </tr>
                    <?php endforeach; ?>
                <?php endif; ?>
            </tbody>
        </table>
    </div>
    <?php
}

// ثبت شورت کد
add_shortcode('download_code', 'download_code_shortcode');
function download_code_shortcode($atts) {
    $atts = shortcode_atts(array(
        'code' => '',
    ), $atts, 'download_code');

    if (empty($atts['code'])) return 'کد معتبر نیست!';

    // ذخیره آدرس صفحه در لیست لینک‌ها
    $links = get_option('download_code_links', array());
    foreach ($links as &$link) {
        if ($link['code'] === $atts['code']) {
            $current_page = get_permalink();
            if (!in_array($current_page, $link['pages'])) {
                $link['pages'][] = $current_page;
            }
            break;
        }
    }
    update_option('download_code_links', $links);

    wp_enqueue_style('download-code-style');
    wp_enqueue_script('download-code-script');

    ob_start(); ?>
    <div class="download-code-container">
        <input type="text" class="download-code-input" placeholder="کد تأیید را وارد کنید">
        <button class="download-code-button" data-code="<?php echo esc_attr($atts['code']); ?>">تأیید کد</button>
        <div class="download-code-loader" style="display:none;">
            <div class="loader"></div>
            <p class="loading-text">فایل در حال پردازش برای دانلود است...</p>
        </div>
        <div class="download-link-container" style="display:none;">
            <p class="success-message">تعداد <span class="link-count"></span> لینک در حال پردازش است.</p>
            <ul class="download-link-list"></ul>
        </div>
    </div>
    <?php
    return ob_get_clean();
}

// ثبت اسکریپت و استایل
add_action('wp_enqueue_scripts', 'download_code_assets');
function download_code_assets() {
    wp_register_style('download-code-style', false);
    wp_enqueue_style('download-code-style');
    wp_add_inline_style('download-code-style', '
        .download-code-container {
            max-width: 400px;
            margin: 20px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            text-align: center;
            background: #f9f9f9;
        }
        .download-code-input {
            width: 70%;
            padding: 10px;
            margin-bottom: 10px;
            border: 2px solid #0073aa;
            border-radius: 4px;
            font-size: 16px;
        }
        .download-code-button {
            background: #0073aa;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            transition: background 0.3s;
            font-size: 16px;
        }
        .download-code-button:hover {
            background: #005177;
        }
        .download-code-loader {
            margin-top: 20px;
        }
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #0073aa;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .loading-text {
            color: #0073aa;
            font-weight: bold;
        }
        .download-link-container {
            margin-top: 20px;
        }
        .success-message {
            color: #28a745;
            font-weight: bold;
        }
        .download-link-list {
            list-style: none;
            padding: 0;
        }
        .download-link-list li {
            margin: 10px 0;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            background: #fff;
            display: flex;
            align-items: center;
            justify-content: space-between;
        }
        .download-link-list li .tick {
            color: #28a745;
            font-weight: bold;
        }
    ');

    wp_register_script('download-code-script', false, array('jquery'), null, true);
    wp_enqueue_script('download-code-script');
    wp_add_inline_script('download-code-script', '
        jQuery(document).ready(function($) {
            $(".download-code-button").click(function() {
                var code = $(".download-code-input").val();
                var expectedCode = $(this).data("code");
                var container = $(this).closest(".download-code-container");
                var loader = container.find(".download-code-loader");
                var downloadLinkContainer = container.find(".download-link-container");
                var linkList = container.find(".download-link-list");
                var linkCount = container.find(".link-count");

                if (code === expectedCode) {
                    $.ajax({
                        url: "' . admin_url('admin-ajax.php') . '",
                        type: "POST",
                        data: {
                            action: "get_download_links",
                            code: code,
                            security: "' . wp_create_nonce('download_code_nonce') . '"
                        },
                        beforeSend: function() {
                            loader.show();
                        },
                        success: function(response) {
                            if (response.success) {
                                loader.hide();
                                downloadLinkContainer.show();
                                linkCount.text(response.data.length);

                                var delay = 10000; // 10 ثانیه
                                response.data.forEach(function(link, index) {
                                    setTimeout(function() {
                                        linkList.append(
                                            `<li>
                                                <a href="${link}" target="_blank">${link}</a>
                                                <span class="tick">✓</span>
                                            </li>`
                                        );
                                    }, delay * index);
                                });
                            } else {
                                loader.hide();
                                alert("خطا در دریافت لینک‌ها!");
                            }
                        }
                    });
                } else {
                    alert("کد وارد شده نامعتبر است!");
                }
            });
        });
    ');
}

// دریافت لینک‌ها بر اساس کد
add_action('wp_ajax_get_download_links', 'get_download_links');
add_action('wp_ajax_nopriv_get_download_links', 'get_download_links');
function get_download_links() {
    check_ajax_referer('download_code_nonce', 'security');

    $entered_code = sanitize_text_field($_POST['code']);
    $links = get_option('download_code_links', array());

    foreach ($links as $link) {
        if ($link['code'] === $entered_code) {
            if ($link['require_login'] && !is_user_logged_in()) {
                wp_send_json_error(array('message' => 'برای دریافت لینک‌ها باید وارد شوید.'));
            }
            wp_send_json_success($link['links']);
        }
    }

    wp_send_json_error(array('message' => 'کد وارد شده نامعتبر است!'));
}
?>
