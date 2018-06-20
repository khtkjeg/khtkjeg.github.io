---
layout: post
title: wordpress ajax接口
categories: wordpress
description: 每天记录一点点，快乐工作一辈子
keywords: wordpress
---

> 有时候我们想获取前台或者后台表单的信息，然后作为参数传给某个PHP程序中处理，这种情况，可以在wordpress上实现ajax，可以通过前端代码后/wp-admin/admin-ajax.php进行交互。


### 注册javascript

首先我们是操作后台页面，所以使用`admin_enqueue_scripts`这个hook,`publish_post_enqueues`,为回调函数，使用`wp_register_script`将我们的js文件注册进去,钩子名称为`admin_integral`,`wp_localize_script`的作用是传递参数，将`daArg`对象传给js中，js调用访问为`daArg.ajaxProvince`

```php
add_action('admin_enqueue_scripts', '`publish_post_enqueues`');
function publish_post_enqueues()
{
    wp_register_script( 'admin_integral', plugins_url( 'js/integral.js', __FILE__ ),array( 'jquery' ));
    wp_enqueue_script( 'admin_integral' );
    wp_localize_script( 'admin_integral', 'daArg', array(
        'ajaxProvince'            => 'beijing'
    ));
}
```


### 实现ajax请求处理hook

`wp_ajax_`是默认必须要加的，后面是自定义的名字。

```php
add_action('wp_ajax_admin_integral_setting', 'publish_post_integral');

function publish_post_integral()
{
    global $wpdb;
    global $current_user;
    global $provinceAll;

    $roles = $current_user->roles[0];
    $myprv = $provinceAll[$roles];
    $post_ID = $_POST['postID'];
    $points = $_POST['hasImg'];
    $state = get_post_meta($post_ID, 'integral', true );

    if(!$state){
        add_post_meta( $post_ID, 'integral', true);

        update_usermeta( $current_user->ID, 'integral', (int) get_the_author_meta( 'integral', $current_user->ID ) + $points );

        $wpdb->get_results("update wp_integral set VALUE = VALUE+$points WHERE  province = '$myprv' ");
    }

    echo "success";
    
    die();
}

```

### JS请求

javascript文件内容,`action`中必须传人我们注册的钩子名称

```js
( function( $ ) {
    $(document).on('click','#publishing-action',function(event) {
        var psotid = $('#post_ID').val()
        var catName = $("input[name='radio_tax_input[category][]']:checked").parent().text()

        if($.trim(catName) == '应用反馈'){
            var hasImg =$("#content_ifr").contents().find('img').length > 0 ? 2 : 1
            var pre ={
                action:'admin_integral_setting',
                postID:psotid,
                hasImg:hasImg
            }
            $.post(ajaxurl,pre,function(data) {
                console.log(data)
            })
        }

    });

})(jQuery)
```


> 亲测可行