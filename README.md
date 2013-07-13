# Mobile First: Using Sass to provide a default view for old IE (8 and below)

## Introduction
When following the responsive, mobile first methodology, there's always an issue of how to support those browsers which don't have media query support, such as old IE. By default, browsers without MQ support will see the mobile view. To workaround this some advocate creating a set of duplicate IE-specific styles as a workaround. This approach leads to duplicate code, increased page weight and possibly a degraded experience. Another approach makes use of JavaScript polyfills, such as [respond.js](https://github.com/scottjehl/Respond), to emulate media queries. The latter goes some way to solving this issue, but also raises a few of its own, notably performance and the reliance on JavaScript.

### An alternative approach

My approach utilises Sass mixins, named, modular media queries and conditional comments to output pure CSS/HTML, so you can continue building mobile first as you normally would, while allowing you to choose a specific view to fallback for old IE and other non-supporting browsers.

#### Benefits

* Allows you to specify a proper default view for old IE.
* Produces a single HTTP request
* Improves CSS organisation by using predefined media queries
* Pure CSS/HTML, no JS reliance
* Flexible enough to fit within any CSS methodology

## Usage

Carry on building mobile first, but when you reach for the media queries, instead of using a CSS3 `@media only screen and (...)` expression, `@include` the `media-query` mixin below:

```scss
@include media-query( wide ){
	width: 25%;
};
```

The key factor of what makes this work is its reliance on *predefined* media queries. Normally you would use media queries with explicit properties, such as `@media only screen and (min-width 320px)...`. Instead, we are using named media queries, stored in a "linked list" within a Sass variable to include predefined media queries:

``` scss
$media-query-ids: narrow, medium, wide, superwide;
$media-queries: "only screen and (max-width: 320px)",
				"only screen and (min-width: 600px)",
				"only screen and (min-width: 960px)",
				"only screen and (min-width: 1280px)";
```

A handy helper function then picks the right media query for us. 

``` scss
@function get-item-by-name( $name, $names-list, $items ){
	@return #{ nth( $items, index( $names-list, $name ) ) };
}
```

*Note: Incidentally, this technique is also helpful for [including font stacks by name](https://gist.github.com/ourmaninamsterdam/5388493).*

The media-query mixin is as follows:

``` scss
@mixin media-query( $name, $include: true ){
	$media-query : get-item-by-name( $name, $media-query-ids, $media-queries );
	@if $supports-mq {
		@media #{ $media-query } { @content; };
	}
	@else {
		@if $name == $default-view {
			@if $include {
				@content;
			}
		}
	}
}
```

* `name` (string).

* `include` (boolean) *Default: true*

	If set to `false`, excludes the specified `@content` from the non-media query supporting stylesheet for the chosen default view.

#### Stylesheets
We have two main stylesheets (for the sake of brevity I'm keeping things simple): 

**ie.scss** - for old IE
```
$supports-mq: false;
$default-view: wide;

@import "_styles";
```

Outputs:

```css
* {
  box-sizing: border-box;
  -moz-box-sizing: border-box;
  -o-box-sizing: border-box;
}

body {
  background: #666;
}

.col {
  background: #000;
  color: #fff;
  outline: 1px solid #ccc;
  padding: 5px;
  float: left;
  width: 25%;
}
```

**main.scss** - for those browsers that support media queries. 

```
$supports-mq: true;

@import "_styles";
```

Outputs:

```css
* {
  box-sizing: border-box;
  -moz-box-sizing: border-box;
  -o-box-sizing: border-box;
}

body {
  background: #666;
}

.col {
  background: #000;
  color: #fff;
  outline: 1px solid #ccc;
  padding: 5px;
}
@media only screen and (min-width: 600px) {
  .col {
    float: left;
    width: 50%;
  }
}
@media only screen and (min-width: 960px) {
  .col {
    float: left;
    width: 25%;
  }
}
@media only screen and (min-width: 960px) {
  .col {
    background: #333;
  }
}
@media only screen and (min-width: 1280px) {
  .col {
    float: left;
    width: 12.5%;
  }
}
```


In these we set two variables: `$supports-mq` and `$default-view`. It's these which the `media-query` mixing uses to control its output.

`_styles.scss` is just my way of keeping things DRY but you could include all your `@imports` directly within each file, if you wish.

#### In the HTML

We then include these using IE conditional comments to serve the right CSS for the browser. old IE ignores main.css and just pulls ie.css, saving us an HTTP request.

``` html
...
<!--[if gt IE 8]> <!-- -->
<link rel="stylesheet" href="css/main.css">
<!-- <![endif]-->
<!--[if lt IE 9]>
<link rel="stylesheet" href="css/ie.css">
<![endif]-->
...

```

## Extended example

``` scss
.col{
	/* Mobile first styles*/
	background: #000;
	color: #fff;
	font-weight: bold;
	padding: 10px;
	
	@include media-query( medium ){
		padding: 10px;
	};
	/* This is our chosen fallback view for old IE */
	@include media-query( wide ){ 
		float: left;
		padding: 5px;
		width: 25%;
	};
	@include media-query( superwide ){
		float: left;
		width: 12.5%;
	};
}
```

### Future development

* Add query chaining
* Improve examples
* DONE Improve variable naming

#### Credits
Justin Perry, 2013
