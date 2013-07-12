# Using Sass to workaround the mobile first IE8 view issue

## Introduction
When following the responsive, mobile first methodology, there's always an issue of how to support those browsers which don't have media query support, such as IE8. Some advocate serving up a modified version of the mobile view to such browsers, or a completely separate set of styles, which adds another HTTP request and even more to the page weight. While others advocate the use of JavaScript polyfills, such as [respond.js](https://github.com/scottjehl/Respond) to emulate media queries. The latter goes some way to solving this issue, but also raises a few of its own, notably performance and the reliance on JavaScript.

### An alternative approach

My approach utilises Sass mixins, named, modular media queries and conditional comments to output pure CSS/HTML, so you can continue building mobile first as you normally would, while allowing you to choose a specific view to fallback to for &lt;=IE8 and other non-supporting browsers.

#### Benefits

* Allows you to specify a proper default view for IE8 and below.
* Produces a single HTTP request
* Improves organisation of media queries by using predefined media queries
* Pure CSS/HTML, no JS reliance
* Flexible

## Usage

Carry on building mobile first, but when you reach for the media queries, instead of using the CSS3 `@media only screen and (...)` expressions, you `@include` the `media-query` mixin below:

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

*Note: Incidentally, this technique is also helpful for specifying font stacks by name.*

The media-query mixin is as follows:

``` scss
// @media-query
// $breakpoint (string) Breakpoint ID
// $include (boolean) Flag to include @content
@mixin media-query( $breakpoint, $include: true ){
	$media-query : get-item-by-name( $breakpoint, $media-query-ids, $media-queries );
	@if $supports-mq {
		@media #{ $media-query } { @content; };
	}
	@else {
		@if $breakpoint == $default-breakpoint {
			@if $include {
				@content;
			}
		}
	}
}
```

It takes two parameters. 

* `breakpoint` (string).

	*Note: this param name is going to change as it self-limiting and misleading in that it only describes breakpoints, and isn't inclusive of device-pixel queries and other such expressions)*. 

* `include` (boolean) *Default: true*

	If set to `false`, excludes the specified `@content` from the non-media query supporting stylesheet for the chosen default view.

#### Stylesheets
We have two main stylesheets (for the sake of brevity I'm keeping things simple): 

**ie.scss** (for IE)
```
$supports-mq: false;
$default-breakpoint: wide;

@import "_styles";
```

**main.scss** (for those that support media queries). 

```
$supports-mq: true;

@import "_styles";
```

In these we set two variables: `$supports-mq` and `$default-breakpoint`. It's these which the `media-query` uses to control the output of the mixin.

`_styles.css` is just my way of keeping things DRY but you could include all your `@imports` directly within each file, if you wish.


We then include these using IE conditional comments to serve the right CSS for the browser. IE8 ignores main.css and just pulls ie.css, saving us an HTTP request.

#### In the HTML

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
	/* My Mobile first styles*/
	background: #000;
	color: #fff;
	font-weight: bold;
	padding: 10px;
	
	@include media-query( wide ){
		float: left;
		padding: 10px;
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
* Improve variable naming

#### Credits
Justin Perry, 2013
