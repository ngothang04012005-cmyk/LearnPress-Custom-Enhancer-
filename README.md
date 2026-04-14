# LearnPress-Custom-Enhancer-<?php
/*
Plugin Name: LP Custom Addon
Description: Thêm thông báo, shortcode thống kê và custom CSS cho LearnPress
Version: 1.0
Author: Bạn
*/

if (!defined('ABSPATH')) {
	exit;
}

add_action('wp_head', 'lp_custom_notification_bar');

function lp_custom_notification_bar()
{
?>
<style>
.lp-notice-bar {
    background: #ff9800;
    color: white;
    text-align: center;
    padding: 40px;
    font-weight: bold;
    position: fixed;
    top: 0;
    width: 100%;
    z-index: 9999;
}

body {
    margin-top: 50px;
}
</style>

<div class="lp-notice-bar">
    <?php
		if (is_user_logged_in()) {
			$user = wp_get_current_user();
			echo "Chào " . esc_html($user->display_name) . ", bạn đã sẵn sàng bắt đầu bài học hôm nay chưa?";
		} else {
			echo "Đăng nhập để lưu tiến độ học tập!";
		}
		?>
</div>
<?php
}
add_shortcode('lp_course_info', 'lp_course_info_func');

function lp_course_info_func($atts)
{
	$atts = shortcode_atts(array(
		'id' => 0
	), $atts, 'lp_course_info');

	$course_id = intval($atts['id']);

	if (!$course_id) return "Không có ID khóa học";

	// Lấy course
	$course = learn_press_get_course($course_id);
	if (!$course) return "Khóa học không tồn tại";

	$lesson_count = 0;

	// Cách 1: Lấy từ curriculum
	if (method_exists($course, 'get_curriculum_items')) {
		$items = $course->get_curriculum_items();
		if (!empty($items) && is_array($items)) {
			foreach ($items as $item) {
				if (isset($item->post_type) && $item->post_type === 'lp_lesson') {
					$lesson_count++;
				}
			}
		}
	}

	// Duration
	$duration = get_post_meta($course_id, '_lp_duration', true);

	// Trạng thái user
	$status = "Chưa đăng nhập";

	if (is_user_logged_in()) {
		$user = learn_press_get_user(get_current_user_id());

		if ($user && method_exists($user, 'get_course_status')) {
			$course_status = $user->get_course_status($course_id);

			if ($course_status === 'completed' || $course_status === 'finished') {
				$status = "Đã hoàn thành";
			} elseif ($course_status === 'enrolled' || $course_status === 'started') {
				$status = "Đã đăng ký";
			} else {
				$status = "Chưa đăng ký";
			}
		} else {
			$status = "Chưa đăng ký";
		}
	}

	// UI hiển thị
	ob_start();
?>
<div class="lp-course-info">
    <p><b>Số bài học:</b> <?php echo esc_html($lesson_count); ?></p>
    <p><b>Thời lượng:</b> <?php echo esc_html($duration ? $duration : 'Không rõ'); ?></p>
    <p><b>Trạng thái:</b> <?php echo esc_html($status); ?></p>
</div>
<?php
	return ob_get_clean();
}
add_action('wp_head', 'lp_custom_button_style');

function lp_custom_button_style()
{
?>
<style>
/* Nút Enroll */
.lp-button,
button.learn-press-button {
    background-color: #4CAF50 !important;
    border-color: #4CAF50 !important;
    color: white !important;
}

/* Hover */
.lp-button:hover {
    background-color: #388E3C !important;
}

/* Finish Course */
.learn-press-finish-course {
    background-color: #ff5722 !important;
    border-color: #ff5722 !important;
}

.learn-press-finish-course:hover {
    background-color: #e64a19 !important;
}
</style>
<?php
}
