---
layout: post
title: wordpress 小工具
categories: wordpress
description: 每天记录一点点，快乐工作一辈子
keywords: wordpress
---

> 有时候我们想在首页右边栏放入一个导航模块，可以通过小工具实现这样的功能。


### 安装插件

首先你要做的是安装你的插件。在你的 `wp-content/plugins `目录下创建一个新文件夹（可以命名为 integral），在该文件夹中新建一个名为“integral.php”的php文件。

```php
<?php
/*
Plugin Name: 积分测试
Plugin URI: 
Description: 积分应用
Version: 1.0
Author: Luming
Author URI: http://luming.men
License: GPLv2
*/
?>
```

### 创建小工具

创建一个新类来拓展 WP_Widget 类,将以下代码复制到你的插件文件中：

```php
<?php
class WP_Widget_Integral extends WP_Widget { 
    function __construct() {

    } 
    function form( $instance ) {

    } 
    function update( $new_instance, $old_instance ) { 

    } 
    function widget( $args, $instance ) { 

    } 
}
?>

```

让我们来看一看这个新类包含些什么：

`__construct` 函数会完成你所期望的——它会构造一个函数。在这个函数里面你可以做出一些定义，比如WordPress小工具的ID、标题和说明。
`form`函数会在WordPress小工具界面创建表单，让用户来定制或者激活它。
`update`函数确保WordPress能及时更新用户在WordPress小工具界面输入的任何设置。
`widget`函数则定义了在网站前端通过WordPress小工具输出的内容。
最后这三个函数的参数我会在相关课程中做进一步详细的解释。

### 注册小工具

只有通过WordPress进行了注册，你的WordPress小工具才能工作。在你的新类下面加入以下函数和钩子

```php
function widgets_view_Integral_init() {
    register_widget( 'WP_Widget_Integral' );
}
add_action( 'widgets_init', 'widgets_view_Integral_init' );
```

register_widget()函数是一个WordPress函数，它唯一的参数就是你刚刚创建的类的命名。
随后将你创建的函数挂入 widgets_init 钩子，确保它可以被WordPress拾起。

### 完善'构造函数'

需要在 WP_Widget_Integral 类里面填充你自己建立的__construct() 函数。

打开你的插件文件，并找到“构造函数”。编辑如下：

```php
function __construct() {
    $widget_ops = array('description' => __('积分排行', 'integral-ranking'));
    parent::__construct('integral', __('积分排行','integral-ranking'),$widget_ops);
}
```

以上代码定义了创建你的WordPress小工具需要的相关参数。它们分别是：

*. WordPress小工具的唯一ID
*. WordPress小工具在其界面上的名称
*. 一系列在WordPress小工具界面显示的选项，包括选项说明

### 创建表单

要为你的小工具创建表单，你需要填充已经添加WP_Widget_Integral 类中的form函数。

打开你的插件，找到form函数，编写如下

```php
    function form( $instance ) {
        $instance = wp_parse_args((array) $instance, array('title' => __('省积分排行','integral-ranking'), 'limit' => 5));
        $title = esc_attr($instance['title']);
        $limit = intval($instance['limit']);
        ?>
        <p>
            <label for="<?php echo $this->get_field_id('title'); ?>"><?php _e('标题:', 'integral-ranking'); ?> <input class="widefat" id="<?php echo $this->get_field_id('title'); ?>" name="<?php echo $this->get_field_name('title'); ?>" type="text" value="<?php echo $title; ?>" /></label>
        </p>
        <p>
            <label for="<?php echo $this->get_field_id('limit'); ?>"><?php _e('选择排行数量:', 'integral-ranking'); ?> <input class="widefat" id="<?php echo $this->get_field_id('limit'); ?>" name="<?php echo $this->get_field_name('limit'); ?>" type="text" value="<?php echo $limit; ?>" /></label>
        </p>
        <input type="hidden" id="<?php echo $this->get_field_id('submit'); ?>" name="<?php echo $this->get_field_name('submit'); ?>" value="1" />
        <?php
    }
```

### 表单更新

要做到这一点你需要处理之前创建的update函数。编码如下：

```php
function update( $new_instance, $old_instance ) {
    if (!isset($new_instance['submit'])) {
        return false;
    }
    $instance = $old_instance;
    $instance['title'] = strip_tags($new_instance['title']);
    $instance['limit'] = intval($new_instance['limit']);
    return $instance;
}
```

### 编辑widget函数

下一步你需要做的便是在你的插件文件中编辑你之前就建立的但还未填充的widget 函数。从定义基于表单输入的变量开始：

```php
function widget( $args, $instance ) {
    $title = apply_filters('widget_title', esc_attr($instance['title']));
    $limit = intval($instance['limit']);
    echo $args['before_widget'] . $args['before_title'] . $title . $args['after_title'];
    echo '<ul>'."\n";
    echo get_integral_viewed($limit);
    echo '</ul>'."\n";
    echo  $args['after_widget'];
}
```

### 查询处理函数

```php
if(!function_exists('get_integral_viewed')) {
    function get_integral_viewed( $limit = 5) {
        global $wpdb;
        $integralData = $wpdb->get_results("SELECT province,value FROM wp_integral ORDER BY `value` desc  LIMIT $limit");
        
        if($integralData) {
            $temp = '';
            $num  = 1;
            foreach ($integralData as $prv) {
                $color = $num > 3 ? "#777":"#d9534f";
                $temp .='<li style="line-height: 25px;">
                            <span style="display:inline-block;text-align: center;width: 25px;height: 25px;background-color: '.$color.';border-radius:50%;font-size: 13px;line-height: 25px;margin-right: 10px;color:#fff;">'.$num.'</span>
                            <span>'.__($prv->province, 'integral-ranking').'</span>
                            <span style="float:right;padding-right:20px;">'.__($prv->value, 'integral-ranking').'</span>
                        </li>';
                $num = $num + 1;
            }
            $temp.='';
            return $temp;
        }else{
            return '<li>'.__('暂无数据', 'integral-ranking').'</li>'."\n";
        }
        return $temp;
    }
}
```

> ok,小工具开发流程介绍完了。