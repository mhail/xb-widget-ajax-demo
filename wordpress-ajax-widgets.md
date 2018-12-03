# WordPress Ajax Widgets - From the Web Archive

WordPress widgets are a great way to add dynamic functionality to a web site.  They offer the benefit of allowing the site administrator to place them in dynamic content areas of the theme and are well isolated when using the WP_Widget class.  As cool as they are out of the box, they can be enhanced to allow for dynamic content updates from any Ajax capable JavaScript framework.

This example will show how to create Ajax enabled widgets by simply changing your base class from WP_Widget to the custom provided Xb_Widget_Ajax class.

The Xb Ajax widget class provides three critical features that enable you to call the Ajax callback function from JavaScript:

* It provides a method for generating the Ajax callback URL. This URL is needed for submitting an Ajax request back to WordPress.
* The Ajax request is intercepted and validated using the WordPress ‘nonce’ security protocol.
* The correct widget is created, initialized, and the Ajax function callback is executed.

## Download

The latest code for this article can be downloaded here: https://github.com/oneuplogic/xb-widget-ajax-demo/
Also, the code can be added to WordPress by installing the [Xb Widget Ajax Demo plugin](https://web.archive.org/web/20121107164927/http://wordpress.org/extend/plugins/xb-widget-ajax-demo/).

Development
If you follow the [Widget Development](https://web.archive.org/web/20121107164927/http://codex.wordpress.org/Widgets_API#Developing_Widgets) documentation on the WordPress codex, the first step in creating a WordPress widget is to extend the WP_Widget class with your own class. The usage of the Xb_Widget_Ajax class is the same, in that you will create your own widget class that extends from Xb_Widget_Ajax. That in turn, extends the WP_Widget class.

```php
class Xb_Widget_Ajax_Demo extends Xb_Widget_Ajax {}
```

After creating your widget, following the example on the codex, you can add some Ajax functionality. The ajax function on the Xb_Ajax_Widget class is called when an Ajax request is intercepted by WordPress.

```php
class Xb_Widget_Ajax_Demo extends Xb_Widget_Ajax {
   public function ajax() {
            // Do cool ajax stuff here.
   }
}
```

 In addition to adding your widget to WordPress using the register_widget function, the ```Xb_Widget_Ajax::load()``` function needs to be called to set up the Ajax request hook. While setting up the Ajax widget,  en-queue the jQuery JavaScript framework.

```php
/**
 * plugins_loaded action callback
 * @return void 
 */
function xb_widget_ajax_demo_plugins_loaded() {
 
    // Load the XB_Ajax_Widget class
    require_once dirname(__FILE__) . DIRECTORY_SEPARATOR . 'inc' . DIRECTORY_SEPARATOR . 'xb-widget-ajax.php';
 
    // Load our custom widgets. 
    require_once dirname(__FILE__) . DIRECTORY_SEPARATOR . 'widgets.php';
 
    // Add action hook to add our custom widget
    add_action('widgets_init', 'xb_widget_ajax_demo_widgets_init');
 
    // Add action hook to enqueue jQuery javascript framework
    add_action('wp_enqueue_scripts', 'xb_widget_ajax_demo_wp_enqueue_scripts');
 
    // The load method needs called in the plugin load to add the init action callback
    Xb_Widget_Ajax::load();
}
 
// Add our action callback to the plugins_loaded action. 
// See: http://codex.wordpress.org/Plugin_API/Action_Reference#Actions_Run_During_a_Typical_Request
add_action( 'plugins_loaded', 'xb_widget_ajax_demo_plugins_loaded' );
 
/**
 * widgets_init action callback
 * @uses register_widget
 */
function xb_widget_ajax_demo_widgets_init() {
    register_widget( 'Xb_Widget_Ajax_Demo' );
}
 
/**
 * wp_enqueue_scripts action callback
 * @uses wp_enqueue_script
 */
function xb_widget_ajax_demo_wp_enqueue_scripts() {
    // enqueue the included no-conflicts jQuery framework
    // See: http://codex.wordpress.org/Function_Reference/wp_enqueue_script#Default_scripts_included_with_WordPress
    wp_enqueue_script( 'jquery' );
}
```

 The next item is critical and needs to be done in each derived class of ```Xb_Widget_Ajax```. The ```ajax_url``` function needs to be overridden so that the correct widget class is called.

```php
// This needs to be overridden in the derived class
//   so that the correct widget is called by the ajax callback.  
function ajax_url($widget = __CLASS__) {
    return parent::ajax_url($widget);
}
```

 At this point everything on the server side is now set up to accept Ajax requests. The Xb_Widget_Ajax works by intercepting requests made to WordPress and looks for a specific query string parameter. If the parameter is present, the request is routed to a new instance of the widget defined in the parameter. Some security savvy readers may start to get worried at this point, but the request is validated using the WordPress ```wp_verify_nonce``` function to prevent malicious requests.

## jQuery to the rescue
Now you can use jQuery to make a request to the Ajax URL. The following block in the widget function uses a div element to display the result of the Ajax callback. An anchor link button is used to trigger the Ajax request. 

```html
<a href="#" title="Call ajax">Update</a>
<div class="widget-content">
</div>
```

The Ajax URL is generated by calling the ajax_url function on the widget class. So when the [Update] link is clicked, it will update the widget-content div with the result.
```javascript
(function($) {
    $(function(e) {
        var widget = $('#<?php echo $widget_id ?>');
        var div = $('.widget-content', widget);
        var ajax_url = '<?php echo $this->ajax_url() ?>';
 
        $('a.ajax-button', widget).click(function() {
            $.ajax({
                url: ajax_url,
                cache: false,
                success: function(html){
                    div.html(html);
                }
            });
 
            // Cancel the click
            return false;
        });
    });
})(jQuery);
```

 Next, add the demo widget to a dynamic sidebar in the Appearance > Widgets screen of the WordPress admin dashboard. The easiest way to see the widget is using the Main Sidebar of the Twenty Eleven theme.  Drag the [Xb_Widget_Ajax_Demo] into the Main Sidebar box, then click on the save button inside the widget.  Then, navigate to the home page of the site. In the Sidebar there should be a box titled Xb Ajax Widget Demo. Click on the [Update] link.  The widget content area should be updated with Ajax Content @ and the current server time. 

Now you can modify the content that is served on the Ajax request.  The content is generated in the ```Xb_Widget_Ajax_Demo->ajax``` function. This function will be called whenever an Ajax request is submitted by the client browser to the Ajax URL.

# Details
To get the Ajax URL, encode a variable assignment with a call to the widgets ```ajax_url``` function within a JavaScript block.  In the demo this is supplied to the jQuery Ajax method as the url option.

```php
var ajax_url = '<?php echo $this->ajax_url() ?>';
```

The demo widget is updating the content div within the main widget. Because there can be multiple instances of a widget on any given page, you can not be sure of the widget container. To interact with the widget in jQuery, use the element id selector encoding the ```$widget_id``` variable. There isn’t a reliable way to know what the element type is, so only use the id of the element as a selector.

Assuming that the ```$args``` argument of the ```widget``` function has been extracted into the current scope, use the following line to get the current widget.

```php
var widget = $('#<?php echo $widget_id ?>');
```

A reference to the content div that will be updated with the Ajax results can then be selected from the widget container.  Note that you need to use the ```widget-content``` class selector because there can be multiple instances of the widget on the page. 
```php
var div = $('.widget-content', widget);
```
Finish with wiring up the click event to the jQuery Ajax function and updating the content div on a successful response.
