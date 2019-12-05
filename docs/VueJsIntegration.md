# Vue.js Integration in Yii2

## Plugin application integration

Add 'antkaz/yii2-vue' plugin to your ``composer.json`` file :
```bash
composer require antkaz/yii2-vue  --ignore-platform-reqs
```

## View file  integration

Add this following code at the beginning of php view files  :
```php
use antkaz\vue\VueAsset;
VueAsset::register($this);
```
Now you can add Vue.js (v2) applications and components inside of your php view files.

## Running example

Add this code to your view file to test if Vue.js is working well.

Example :
```html
<div id='app'>
  <h1>Trying out Vue.js</h1>
 <p>{{ message }}</p>
  <button v-on:click="reverseMessage">Reverse Message</button>
</div>

<script>
new Vue({
  el: '#app',
  data: {
    message: "OpenSILEX is awesome!"
  },
  methods: {
    reverseMessage: function() {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
</script>
```

## Add Vue;js plugins

If you want to add Vue.js plugins, you need to register the
js and css file of this plugin using the function
```php
$this->registerJsFile("{url of plugin Vue.js js file}");
$this->registerCssFile("{url of plugin Vue.js css file}");
```