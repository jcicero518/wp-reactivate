![alt-text](https://cloud.githubusercontent.com/assets/1805604/26199709/55c91bda-3bcb-11e7-871e-94b7a022cfa9.jpg "WP Reactivate - WordPress React Boilerplate")
# WP Reactivate
WP Reactivate is a React boilerplate built specifically for WordPress, allowing you to quickly and easily integrate React into your WordPress plugins.

## Setup and installation
* **Install [Node 4.0.0 or greater](https://nodejs.org)**
* **Install [Yarn](https://yarnpkg.com/en/docs/install)** (Or use npm if you prefer)

## Usage
* Install required modules: `yarn` (or `npm install`)
* Build development version of app and watch for changes: `yarn build` (or `npm run build`)
* Build production version of app:`yarn prod` (or `npm run prod`)

## Quick Start
### Introduction
This boilerplate plugin provides three different WordPress views in which an independant React app can be rendered:

- Shortcode
- Widget
- Settings page in the backend (wp-admin)

Each JavaScript root file will correspond to the independant React app to be bundled by Webpack.

*webpack.config.js*
```javascript =6
entry: {
  'js/admin': path.resolve(__dirname, 'app/admin.js'),
  'js/shortcode': path.resolve(__dirname, 'app/shortcode.js'),
  'js/widget': path.resolve(__dirname, 'app/widget.js'),
},
```
  
### Using the Shortcode
In order to get the shortcode attributes into our Javascript we need to pass them to an object which will be made available to the *shortcode.js* app via ```wp_localize_script```. Be careful with the security of data you pass here as this will be output in a ```<script>``` tag in the rendered html.

*includes/Shortcode.php*
```php =79
public function shortcode( $atts ) {
  wp_enqueue_script( $this->plugin_slug . '-shortcode-script' );
  wp_enqueue_style( $this->plugin_slug . '-shortcode-style' );

  $object_name = 'wpr_object_' . uniqid();

  $object = shortcode_atts( array(
    'title'       => 'Hello world',
    'api_nonce'   => wp_create_nonce( 'wp_rest' ),
    'api_url'	  => site_url( '/wp-json/wp-reactivate/v1/' ),
  ), $atts, 'wp-reactivate' );

  wp_localize_script( $this->plugin_slug . '-shortcode-script', $object_name, $object );

  $shortcode = '<div class="wp-reactivate-shortcode" data-object-id="' . $object_name . '"></div>';
  return $shortcode;
}
```

You can access the shortcode attributes via the ```wpObject``` prop which is passed into your React container component.

*app/containers/Shortcode.jsx* 
```javascript =1
import React, { Component } from 'react';

export default class Shortcode extends Component {
  render() {
    return (
      <div className="wrap">
        <h1>WP Reactivate Frontend</h1>
        <p>Title: {this.props.wpObject.title}</p>
      </div>
    );
  }
}
```

### Using the Widget
In order to get the widget options into our Javascript we need to pass them to an object which will be made available to the *widget.js* app via ```wp_localize_script```. Be careful with the security of data you pass here as this will be output in a ```<script>``` tag in the rendered html.


*includes/Widget.php*
```php =41
public function widget( $args, $instance ) {
  wp_enqueue_script( $this->plugin_slug . '-widget-script', plugins_url( 'assets/js/widget.js', dirname( __FILE__ ) ), array( 'jquery' ), $this->version );
  wp_enqueue_style( $this->plugin_slug . '-widget-style', plugins_url( 'assets/css/widget.css', dirname( __FILE__ ) ), $this->version );

  $object_name = 'wpr_object_' . uniqid();

  $object = array(
    'title'       => $instance['title'],
    'api_nonce'   => wp_create_nonce( 'wp_rest' ),
    'api_url'	  => site_url( '/wp-json/wp-reactivate/v1/' ),
  );

  wp_localize_script( $this->plugin_slug . '-widget-script', $object_name, $object );

  echo $args['before_widget'];

  ?><div class="wp-reactivate-widget" data-object-id="<?php echo $object_name ?>"></div><?php

  echo $args['after_widget'];
}
```
You can access the widget options via the ```wpObject``` prop which is passed into your React container component.

*app/containers/Widget.jsx* 
```javascript =1
import React, { Component } from 'react';

export default class Widget extends Component {
  render() {
    return (
      <div className="wrap">
        <h1>WP Reactivate Widget</h1>
        <p>Title: {this.props.wpObject.title}</p>
      </div>
    );
  }
}

```

### Using REST Controllers

We have included a single base REST controller class in the plugin. You will need to use this controller to create endpoints to interact with your React components. Depending on the complexity of your plugin you may need to have multiple controllers or may want to extend default WordPress REST API endpoints. 

We have chosen the custom controller approach for its control and flexibility. Please see the WordPress developer documentation on adding [custom endpoints](https://developer.wordpress.org/rest-api/extending-the-rest-api/adding-custom-endpoints/) and specifically the [controller pattern](https://developer.wordpress.org/rest-api/extending-the-rest-api/adding-custom-endpoints/#the-controller-pattern) to familiarise your with our choice of implementation.

It is important to become well versed in using the [WordPress REST API](https://developer.wordpress.org/rest-api/) as this is how you will be passing data to and from your React applications.
### Using the Settings Page
In our admin class we add a sub menu page to the Settings menu using ```add_options_page``` and render the React Admin container onto the root DOM node.

*includes/Admin.php*
``` php =166
public function display_plugin_admin_page() {
 ?><div id="wp-reactivate-admin"></div><?php
}
```


In the React container component we show how to retrieve and update this setting via this example REST controller.

We polyfill the browser [Fetch API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) to make requests to the WordPress REST API. It is a powerful API, which can be seen as an evolution of XMLHttpRequest or alternative to jQuery.ajax().

*app/containers/Admin.jsx*
```javascript
getSetting = () => {
    fetch(`${this.props.wpObject.api_url}settings`, {
        credentials: 'same-origin',
        method: 'GET',
        headers: {
            'Content-Type': 'application/json',
            'X-WP-Nonce': this.props.wpObject.api_nonce,
        },
    })
    .then(response => response.json())
    .then(
        (json) => this.setState({ settings: json.wpreactivate }),
        (err) => console.log('error', err)
    );
};
```
## Technologies
| **Tech** | **Description** |
|----------|-------|
|  [React](https://facebook.github.io/react/)  |   A JavaScript library for building user interfaces. |
|  [Babel](http://babeljs.io) |  Compiles next generation JS features to ES5. Enjoy the new version of JavaScript, today. |
| [Webpack](http://webpack.js.org) | For bundling our JavaScript assets. |
| [ESLint](http://eslint.org/)| Pluggable linting utility for JavaScript and JSX  |

The boilerplate has been updated to use PHP **namespaces and autoloading**. Please see Tom McFarlin's [article](https://tommcfarlin.com/namespaces-and-autoloading-2017/) on the subject if you are not familiar.
## Credits
*Made by [Pangolin](https://gopangolin.com)*
