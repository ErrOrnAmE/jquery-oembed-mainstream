# jquery-oembed-mainstream

Use oEmbed data with jQuery - Rework of https://github.com/nfl/jquery-oembed-all

DEMO: https://errorna.me/tools/jquery-oembed

Instead of providing an exhaustive list of all oEmbed providers, this jQuery plugin lets you configure the providers that you need.

If you wish, you can find a list of mainstream providers in the file `providers.oembed.js`

Quick Start
-----------
Include `jquery.oembed.js` in your web application.
```html
<script src="//code.jquery.com/jquery-3.1.0.min.js"></script>
<script src="jquery.oembed.js"></script>
```

You can also add the mainstream providers.
```html
<script src="providers.oembed.js"></script>
```

To include an oEmbed resource, you can:
```html
<a href="https://www.flickr.com/photos/sohailkarmani/24048210319" class="oembed"></a>

<script>
	$('.oembed').oembed();
</script>
```
Note: Read the documentation for more explanation on how to call `.oembed()`

---

Documentation
-------------

1. [Calling the oEmbed function](#1-calling-the-oembed-function)
2. [OEmbed settings](#2-oembed-settings)
3. [Adding a provider](#3-adding-a-provider)
4. [Provider settings](#4-provider-settings)
5. [Proxy](#5-proxy)

### 1) Calling the oEmbed function

```html
<a class="oembed" href="https://www.flickr.com/photos/sohailkarmani/24048210319"></a>

<script>
	$('a.oembed').oembed() // Use default settings. Use the url contained in the "href" attribute.
	$('a.oembed').oembed({settings...}) // Use given settings. Use the url contained in the "href" attribute.
	$('a.oembed').oembed(url) // Use default settings. Use given url.
	$('a.oembed').oembed(url,{settings...}) // Use given settings. Use given url.
</script>
```

===

### 2) OEmbed settings

```js
$('a.oembed').oembed({
	providers: '*', // List authorized providers
	classContainer: 'oembed-container', // Classname of the container
	removeOrigin: true, // Origin (DOM Element from where you call oembed) will be removed after embedding
	codeBuilder: null, // Function to build DOM Element from oEmbed data
	beforeEmbed: function(data, elements, provider) { // Function to interact with the soon to be embedded DOM Elements
		return elements;
	},
	afterEmbed: function(data, container, provider) { // Callback after successful embedding
		console.log('OEmbed succeeded!');
	},
	onError: function(error) { // Callback after reaching an error
		console.log('OEmbed error:');
		console.log(error);
	}
});
```

##### providers : Array(String)

You can list all authorized array for this call. If you want to allow all the providers present in $.fn.oembed.providers, you can use the `'*'` string.

```js
$('a.oembed').oembed({
	providers: ['youtube','dailymotion','soundcloud'] // Will only allow these three providers for this call
});
```

##### classContainer : String

This class will be associated to the <div> container of the embedded element.

```js
$('a.oembed').oembed({
	classContainer: 'custom-container'
});
```

##### removeOrigin : Boolean

This boolean triggers or not the removal of the origin DOM Element from where you called the jQuery oembed function.

```js
$('a.oembed').oembed({
	removeOrigin: true // This will remove 'a.oembed' from the DOM
});
```

##### codeBuilder : Function(Object)

This function is used to construct a custom DOM Element based on the oEmbed data.

This function must return a DOM Element.

Note: If a `codeBuilder` option is defined with the associated provider, this function will be used over the one from the provider.

```js
$('a.oembed').oembed({
	codeBuilder: function(oembedData) {
		return $('<img>').attr('src',data.url).attr('alt',data.title);
	}
});
```

##### beforeEmbed : Function(Object, Array[DOMElement], Object)

Can access and operate changes on the DOM Elements soon to be embedded.

This function must return a DOMElement or an Array of DOMElement to embed.

```js
$('a.oembed').oembed({
	beforeEmbed: function(embedData, elements, provider) {
		elements.push($('<p>').html('OEmbed item'));
		return elements;
	}
});
```

##### afterEmbed : Function(Object, DOMElement, Object)

Can access and operate changes on the DOM Element container that have been embedded.

```js
$('a.oembed').oembed({
	afterEmbed: function(embedData, container, provider) {
		$(container).prepend('<h1>OEmbed item</h1>');
	}
});
```

##### onError : Function(Object)

Define the expected behaviour when encountering an error.

```js
$('a.oembed').oembed({
	onError: function(error) {
		console.log('An error have been found: '+error.type);
	}
});
```

===

### 3) Adding a provider

To add a provider, you can do the following:
```js
$.fn.oembed.providers['soundcloud'] = new $.fn.oembed.OEmbedProvider({
	name: 'soundcloud', // name of the provider
	type: 'rich', // type of the resource (still in beta!)
	urls: [ // regex of the resources
		"soundcloud.com/.+",
		"snd.sc/.+"
	],
	api: "//soundcloud.com/oembed", // oEmbed api endpoint
	dataType: null, // data type for the ajax request
	callbackParameter: null, // callback name for jsonp ajax requests
	codeBuilder: null, // function to build a custom DOM Element from the oEmbed Data
	yql: null, // function to retrieve the object from a YQL Response Object
	proxy: false // boolean to retrieve the object from a local proxy script
});
```

===

### 4) Provider settings

##### Name : String

The name of the provider.

##### Type : String

The oEmbed type of the resource (Photo, Video, Link or Rich). This feature is still in beta. Use "rich" if possible.

##### URLs : Array(String)

The URL regex schemes to link an URL to a provider.

##### Api : String

The oEmbed API endpoint of the provider.

##### Data Type : String

Sometime, the ajax request return a CORS error. You can add a `jsonp` data type to perform the request.

##### Callback Parameter : String

Some providers need a specific callback function name for jsonp.

##### Code Builder : Function(Object)

This function is used to construct a custom DOM Element based on the oEmbed data.

This function must return an Object.

Note: If a `codeBuilder` option is present at the call of `.oembed()`, the function from options will be used instead of this one.

```js
codeBuilder: function(oembedData) {
	return $('<img>').attr('src',data.url).attr('alt',data.title);
}
```

##### YQL : Function(Object)

Sometime, a provider will not allow a jsonp request. Therefore, you need to use YQL to perform the request.

This function is use to extract the oEmbed data from the YQL response data.

This function must return an Object.

```js
yql: function(data) {
	return data.query.results.json
}
```

##### Proxy : Boolean

YQL is a good way to proxy a request to retrieve data from a provider not allowing a jsonp request, or not having the CORS header. However, you are creating a dependency to the Yahoo service.

Instead, you may opt to use a local proxy script (proxy.php).

If you set the proxy attribute to true, this plugin will use the local proxy script instead of directly querying the provider or YQL.

### 5) Proxy

Some providers (e.g: Youtube) don't allow jsonp request or don't have CORS header. It forces us to use a proxy in these cases.

One possibility is to use YQL, a Yahoo service to query the web. But it creates a dependency to a third-party service.

Another possibility is to use a local proxy script using whatever language or framework you wish.

The proxy must complete a really simple task. Using the GET attribute named "csurl", it must return the response when querying this url.

You can find a PHP proxy script in this repository.

To define the path to the proxy (default is "proxy.php"):

```js
$.fn.oembed.proxyPath = '/my/path/to/proxy';
```
