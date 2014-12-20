Manage RTL CSS with Sass and Grunt.
-----------------------------------

As a native Arabic speaker I have worked on multilingual websites before, some was already exist with LTR (Left to Right) CSS support then I have to support RTL (Right to Left) and some projects started from scratch. Before I start using Sass and Grunt, supporting both directions was a nightmare and a time waste process plus to code repetition.

The important thing when working on multilingual projects with two directions is to write CSS that support both RTL and LTR in an effective, automated and dynamic way that we don't have to repeat or override CSS.

The differences between the two directions in general and most cases is floating direction, text align, padding and margin sides values.

## The Problem

Let's see how supporting RTL and LTR in the same project is not a straightforward in practice and how we can solve this. In some cases I saw before adding a new direction support was to add a new CSS file in the header and start to override and repeat code over and over to change some CSS properties like `floats`, `padding-left` or `text-align` for different components.

Let's say this is the code for the RTL language template, we added the `lang="ar"` attribute for the language support, this attribute value should be dynamically changed based on the language detection from the server side.

``` html
<!DOCTYPE html>
<html lang="ar">
  <head>
  	<link rel="stylesheet" href="css/app.css">
  	<link rel="stylesheet" href="css/rtl-app.css">
  </head>
  <body>
  	<main>Main contant</main>
  	<aside>Sidebar</aside>
  </body>
</html>
```

The layout requires that in the normal direction (LTR) the `main` section should be floated to the left, while `aside` should be to the right.


``` css
// app.css
main  { float: left; }
aside { float: right; }
```

Now for the RTL direction we should do the same thing above but in the opposite direction or to mirror the layout, this will be the code to override the original style in the same file.

``` css
// app.css
[lang='ar'] main  { float: right; }
[lang='ar'] aside { float: left; }
```

or in a new CSS file appended in the header we can do it like so.

``` css
// rtl-app.css
main  { float: right; }
aside { float: left; }
```

The problem here is that we will write more code to override the original code, loading more than one CSS file. This is not a good practice and a time consuming.

Now how we can improve this workflow, the solution I have used is to use a CSS preprocessor like [Sass][sass] and a JavaScript task runner like [Grunt][grunt] to automate and enhance the workflow.

## Setup Grunt

By using Grunt and Sass we can automate and solve all the problems we have, the theory here is to write CSS in one core file and then generate two other files each one for each direction then include each file in the header based on the language used.

The Grunt task used for compiling	 Sass to CSS is [grunt-sass][grunt-sass].

``` javascript
{
  "name": "rtl-sass-grunt",
  "version": "0.1.0",
  "devDependencies": {
    "grunt": "^0.4.5",
    "grunt-sass": "^0.16.1"
  }
}
```

``` javascript
module.exports = function(grunt) {
  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),

    sass: {
      dist: {
        files: {
          'css/ltr-app.css': 'scss/ltr-app.scss',
          'css/rtl-app.css': 'scss/rtl-app.scss'
        }
      }
    }

  });

  grunt.loadNpmTasks('grunt-sass');

  grunt.registerTask('default', ['sass']);
};
```

The Sass task takes two files one for `ltr-app.scss` and `rtl-app.scss` then generate the CSS files from them. In both files we will define some variables for floats and directions and then import the core Sass file at the end.

`ltr-app.scss` file

```css
// LTR languages directions.

$default-float:          left  !default;
$opposite-float:        right  !default;

$default-direction:       ltr  !default;
$opposite-direction:      rtl  !default;

// Import the main style

@import 'style';
```

And `rtl-app.scss`

```css
// RTL languages directions.

$default-float:         right  !default;
$opposite-float:         left  !default;

$default-direction:       rtl  !default;
$opposite-direction:      ltr  !default;

// Import the main style

@import 'style';
```

We can include these variables in external files, but this is just for keeping things more clear and to give you the main idea.

The `style.scss` will include all our CSS or include other project files. This file will imported in both `ltr-app.scss` and `rtl-app.scss`.

And this is an example of what `style.scss` looks like.

``` css
body {
	direction: $default-direction;
}

.media {
	float: $default-float;
  	padding-#{$opposite-float}: 10px;
}

.button { background-image: url("images/arror-#{default-float}.png"); }
```

We used our Sass variables like `$default-float` and [Sass interpolation][interpolation] in `padding-#{$opposite-float}`, then Grunt can take care of this and generate two files as:

``` css
// ltr-app.css

body { direction: ltr; }

.media {
	float: left;
  padding-right: 10px;
}

.button { background-image: url(images/arror-left.png); }
```

``` css
// rtl-app.css

body { direction: rtl; }

.media {
	float: right;
  padding-left: 10px;
}

.button { background-image: url(images/arror-left.png); }
```

A good trick I experienced before is how to add an image with a specefic direction as a CSS background, from the code above we will create two images `arror-left.png` and `arror-right.png` and then in the Sass code the variable will change between `left` and `right`.

## Server Side Setup

For more flexibility working on multilingual projects, configuring the server side to provide some variables to use in template files will be important.

What we need from the server side is to define a similar variables like we did with Sass but this time we will use them inside templates or views. Every back-end solution should provide a way to create theses variables and pass them in views. Variables like `def-float` and `def-direction`.

Setting server side configuration will enable us to switch between the generated CSS files for the detected direction.

``` html
<!DOCTYPE html>
<html lang="..">
  <head>
    <%= if def-direction is ltr %>
  	<link rel="stylesheet" href="css/ltr-app.css">
  	<%= else %>
  	<link rel="stylesheet" href="css/rtl-app.css">
  	<%= end %>
  </head>
  <body>
  	<main>Main contant</main>
  	<aside>Sidebar</aside>
  </body>
</html>
```

Or we can use the variable inside the CSS file name itself.

``` html
<link rel="stylesheet" href="css/#{def-direction}-app.css">
```

The if statement above and the file name variable is just a [Pseudocode][pseudocode] and it will depend on your template engine and your server side configuration. Ask your smart fellow developer to help you on this or search for Google based on your technology to do it.

### Working with Helper Classes and Templates

When working with [Helper Classes][helper-classes] mixed with template files like HTML, erb or any other template engine, we need to dynamically change the class names based on the direction, let's see an example.

``` html
<div class="text-left">
  <!-- content -->
</div>
```

``` css
.text-left   { text-align: left; }
.text-right  { text-align: right; }
```

The `div` content will always align the content to the left as it takes the `text-left` class, but we need a way to dynamically change the `-left` part to `-right` for RTL.

We can solve this issue by using one of our template variables.

``` html
<div class="text-#{default-float}">
  <!-- content -->
</div>
```

## Sass Mixins

We have used Sass variables and interpolations, another way for making this more simpler is to build a set of Sass mixins.

```css
@mixin float($dir) {
  @if $dir == left {
    float: $def-float;
    } @else if $dir == right {
      float: $opp-float;
    } @else {
      float: $dir;
  }
}

@mixin text-align($dir) {
  @if $dir == left {
    text-align: $def-float;
    } @else if $dir == right {
      text-align: $opp-float;
    } @else {
      text-align: $dir;
  }
}

@mixin padding-left($unit) {
  padding-#{$def-float}: $unit;
}

@mixin padding-right($unit) {
  padding-#{$opp-float}: $unit;
}
```

Which later we can use it like

``` css
.media {
  @include float(left);
  @include padding-right(10px);
  @include text-align(left);
}
```

For more Sass mixins for different usage you can read the source code of [bi-app-sass][bi-app-sass] which inspired me for the above mixins.

## Conclusion

You can use whatever tool other than Grunt and Sass for doing the same thing, getting the idea of using Sass variables for manipulating directions and changing variables in template files is the core idea.

You can check out the code used in this post in [rtl-grunt-sass][rtl-grunt-sass].

[sass]: http://sass-lang.com/
[grunt]: http://gruntjs.com/
[grunt-sass]: https://github.com/sindresorhus/grunt-sass
[interpolation]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#interpolation_
[pseudocode]: http://en.wikipedia.org/wiki/Pseudocode
[helper-classes]: http://www.sitepoint.com/using-helper-classes-dry-scale-css/
[bi-app-sass]: https://github.com/anasnakawa/bi-app-sass/blob/master/bi-app/_mixins.scss
[rtl-grunt-sass]: https://github.com/ahmadajmi/rtl-grunt-sass
