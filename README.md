Manage RTL CSS with Sass and Grunt.
-----------------------------------

As a native Arabic speaker I have worked on many multilingual websites before, some websites was already exist with only LTR CSS support then I have to support RTL and some projects started it from scratch. Before I start using Sass and Grunt, supporting both directions was a nightmare and a time waste process in addition to code repetition.

The very important thing when working on multimilngual projects with two directions is to write CSS that support both RTL and LTR in an effective, automated and dynamic way that we don't have to repeat or overite CSS.

What's the difference between the two directions is generl and most cases is working float, direction, padding and margin properties. Theses are mostly what we will work with and mostly all our CSS code consist of all of them.

## The problem

Let's how this can be solved by old methods.

Let's see how supporting TRL is not a straitforward in practice and how we can solve this. In most cases i saw before adding a new direction support was to add a new CSS file in the header and start to overite and repeat code over and over to change `floats`, `padding-left` or `text-align` for different section.

Let's say this is the code for the RTL language template, we added the `lang="ar"` attribute for the labguage support, this attribute value should be dynamically change based on the language detection from the serverside for example.

``` html
<!DOCTYPE html>
<html lang="ar">
  <head>
  	<link rel="stylesheet" href="css/app.css">
  </head>
  <body>
  	<main>Main contant</main>
  	<aside>Sidebar</aside>
  </body>
</html>
```

The layout requires that in the normal direction the `main` section should be floated to the left, while `aside` should be to the right.

``` css
main  { float: left; }
aside { float: right; }
```

Now for the RTL direction we should do the same thing above but in the oppisite direction so the `main` section should be floated to the right, while `aside` should be to the left or to mirror the layout. so this will be the code to overide the original style in the same file

``` css
[lang='ar'] main  { float: right; }
[lang='ar'] aside { float: left; }
```

or in a new CSS file appended to the original CSS file in the header we can do it like so

``` css
main  { float: right; }
aside { float: left; }
```

The problem here is that we will write more code that just do overight to original code and the old one and loading two files in the second option.This is a very bad practice and time consuming.

Now how can we improve this workflow, the solution I have used is to use a CSS preprocessor like [Sass][sass] and a JavaScript task runner like [Grunt][grunt] to automate and enhance the workflow.

## Setup Grunt

By using a Grunt task runner and Sass we can automate and solve all the problems we have, the theory here is to write CSS in one core file and then generate two other files each one for each direction then include each file in the header based on the language beign used.

The Grunt task used for this that will compile Sass to CSS is [grunt-sass][grunt-sass].

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

The Sass task takes two files one for `ltr-app.scss` and `rtl-app.scss` and generate the CSS files from them. We will see what's inside theses files later.

We talked about the core file, let's call it `style.scss` and it's job is to include all our CSS or include other project files. This file will be imported in both `ltr-app.scss` and `rtl-app.scss`.

We can now setup and see what's inside `ltr-app.scss` file

```css
// LTR languages directions.

$def-float:       left  !default;
$opp-float:      right  !default;

$def-direction:   ltr   !default;
$opp-direction:  rtl    !default;

// Import the main style

@import 'style';
```

And `rtl-app.scss` wil be.

```css
// LTR languages directions.

$def-float:      right  !default;
$opp-float:       left  !default;

$def-direction:    rtl  !default;
$opp-direction:    ltr  !default;

// Import the main style

@import 'style';
```

We can include these variables in external files, but this is just for keeping things more clear.

And this should be the content inside our main `style.scss`

```
// Imports
@import 'mixins';

// Author Style

body {
	direction: $default-direction
}

.media {
	float: $default-float;
  	padding-#{$opposite-float}: 10px;
}
```

We used Sass variable interpolation fo `padding-#{$opposite-float}`, so we can then generate the CSS property which will be evaluated as

``` css
// ltr-app.css
.media {
	float: left;
  	padding-right: 10px;
}
```

``` css
// rtl-app.css
.media {
	float: right;
  	padding-left: 10px;
}
```

As you can see this file is the core Sass file which will include all other files and may be contain some general styles.

## Working with Helper Classes and Templates

When working with Helper Classes mixed with template files (HTML - erb) or any other template engine, we will face a problem here. Let's see an example.

``` html
<div class="text-left">
  <!-- content -->
</div>
```

``` css
.text-left   { text-align: left; }
.text-right  { text-align: right; }
```

The html above will not change as we write HTML for all languages, so the above `div` will be always be aligned to the left. But this is not what we want to do, we want to align it to the left in LTR and to the right in RTL.

We can solve this issue by using some variables in HTMl like

``` html
<div class="text-#{default-float}">
  <!-- content -->
</div>
```

The `default-float` variable above will be generated by the template engine and the server side or client side solution. Ask your genius coworker to generate theses variables for you or you can google it. What will be generated after the template above is parsed in

LTR

``` html
<div class="text-float">
  <!-- content -->
</div>
```

RTL

``` html
<div class="text-right">
  <!-- content -->
</div>
```

## Working with background images

A very good trick I experienced before is how to add some image that have a specefic direction as a CSS background, let's see an example.

```sass
.button {
	background-image: url("images/arror-#{default-float}.png");
}
```

The result will be as

```css
// app.ltr.css

.button {
	background-image: url(images/arror-left.png);
}
```

```css
// app.ltr.css

.button {
	background-image: url(images/arror-right.png);
}
```

## Sass Mixins

So we have tried to use Sass variables and interpolations in our Sas code, is ther any thing that could be better and easier in writing. yes. We can create Sass mixins to handle theses things for us.

```css
@mixin float($dir) {
  @if $dir == left {
    float: $default-float;
    } @else {
      float: $opposite-float;
    }
}

@mixin text-align($dir) {
  @if $dir == left {
    text-align: $default-float;
    } @else if $dir == right {
      text-align: $opposite-float;
    } @else {
      text-align: $dir;
    }
}

@mixin padding-left($unit) {
  padding-#{$def-float}: $unit;
}
```

For more Sass mixins for different usage you can read the source code of [bi-app-sass][bi-app-sass]

## Conclusion

As we saw this solution fixed the RTL and LTR was very well, Grunt Sass are doing things we no longer want to do manually.

http://stackoverflow.com/questions/25749044/add-a-rtl-scss-file-to-zurb-foundation-5

[bi-app-sass]: https://github.com/anasnakawa/bi-app-sass/blob/master/bi-app/_mixins.scss

[sass]: http://sass-lang.com/
[grunt]: http://gruntjs.com/
[grunt-sass]: https://github.com/sindresorhus/grunt-sass

