# react-lazy
[![Version](http://img.shields.io/npm/v/react-lazy.svg)](https://www.npmjs.org/package/react-lazy)
[![Build Status](https://travis-ci.org/Merri/react-lazy.svg)](https://travis-ci.org/Merri/react-lazy)

> Lazy load your content without breaking the internet!

Supports universal rendering including disabled JavaScript by using `noscript` elements that are also friendly to all
search engines. Uses modern IntersectionObserver API and exposes `Observer` component which works exactly like the
excellent [@researchgate/react-intersection-observer](https://github.com/researchgate/react-intersection-observer), but
with a few minor changes.

Also optionally supports displaying the content on IE8 and earlier by adding conditional comments to skip `noscript`
elements.

[View lazy loading demo](https://merri.github.io/react-lazy/)

```js
npm install react-lazy

import { Lazy } from 'react-lazy'
// or
import { LazyGroup } from 'react-lazy'
// or
import { Observer } from 'react-lazy'
```

You also need to polyfill `intersection-observer`! Use polyfill.io or `require('intersection-observer')`. Check
[Can I use](https://caniuse.com/#feat=intersectionobserver) for browser support status. Additionally features like `Map`
and `Set` are also required.


----

## Why `react-lazy`?

1. Minimalistic and performant implementation with less dependencies than other solutions
2. You can choose between ease-of-use (LazyGroup) and do-it-yourself (Lazy, Observer)
3. The hard part of handling `noscript` is done for you (Lazy, LazyGroup)


----

### Why lazy load content such as images?

You want to save your bandwidth and/or server load. As a side effect you may also gain some performance benefits on
client side, especially on mobile devices. However the main benefit (and main purpose) for you should always be the
reduction of bandwidth/server load.

Likely side effect of lazy loading is that user may see content flashing as it comes into view; sometimes with a lot of
delay as it depends on connectivity. You can make the experience less flickery by adding a transition when image is
loaded (a bit harder to develop) or by giving `Lazy` a large cushion (500 pixels or more) to load image before it is
actually in the viewport. Using both strategies together is recommended. You can test the experience on your own site by
dropping mobile connection to 2G / edge.

Chrome developer tools also has network throttling so you don't need to get yourself into a train to nowhere to test how
well or poorly your site works in high latency conditions. However it is also recommended you do get yourself into a
train to nowhere as it does good for your mind and soul to abandon the hectic although convenient city lifestyle every
once in a while.


----

## Usage: `<Lazy />`

```jsx
// curly brackets are required
import { Lazy } from 'react-lazy'

...

    <Lazy component="a" href="/" className="image-link image-link--100px" ltIE9>
        <img alt="My Lazy Loaded Image" className="image-link__image" src="my-lazy-loaded-image.png" />
    </Lazy>
```

```html
<!-- server render and render before component is in viewport -->
<a href="/" class="image-link image-link--100px">
    <!--[if IE 9]><!--><noscript><!--<![endif]-->
        <img alt="My Lazy Loaded Image" class="image-link__image" src="my-lazy-loaded-image.png" />
    <!--[if IE 9]><!--></noscript><!--<![endif]-->
</a>

<!-- client DOM after component is in viewport -->
<a href="/" class="image-link image-link--100px">
    <img alt="My Lazy Loaded Image" class="image-link__image" src="my-lazy-loaded-image.png" />
</a>
```


----

## Component introduction

First of all, `Observer` component is exposed. This is a powerful component for using the new IntersectionObserver API.
The notable differences to the implementation of
[@researchgate/react-intersection-observer](https://github.com/researchgate/react-intersection-observer) are:

- `viewport` prop is added and preferred over `root` prop
- `cushion` prop is added and preferred over `rootMargin` prop
- `onlyOnce` prop deprecation has been enforced and is unsupported
- Other changes are internal code optimizations for further minor speed improvements and reduced code size

Essentially `react-lazy`'s implementation of `<Observer />` is fully compatible with all the examples and demos you find
at [@researchgate/react-intersection-observer](https://github.com/researchgate/react-intersection-observer). (This part
of the text holds true in 2018-05-30 at version 0.7.1 - throw issue if this state changes).


As for lazy loading there are two components: `<Lazy />` and `<LazyGroup />`.

**Lazy** provides basic functionality for lazy loading: it keeps render in `noscript` element until it has come into
viewport and then simply swaps render. **Everything** inside the component is wrapped into `noscript`. As the component
is quite simple and generic it doesn't support many other things that provide convenience; for example, with images you
have to write your own logic for handling `onError` and `onLoad` so that you can do things like trigger transitions as
images are loaded, or change what to render instead of the image if loading the image fails.

**LazyGroup** extends `Lazy` functionality by wrapping only specified component types inside `noscript`. So only the
specified components like `img` or `iframe` elements are wrapped to `noscript`. Other elements are simply rendered
as-is.

The wrappable components (`img`s and `iframe`s by default) are also always wrapped inside another component. This custom
component will receive information on whether loading the `img` or `iframe` has succeeded or failed, thus allowing a
single place to control lifecycles as images or other content is loaded.


## Shared features

These features are supported by both `<Lazy />` and `<LazyGroup />`.


### IntersectionObserver props

- `viewport` (= `root` option)
- `cushion` (= `rootMargin` option)
- `threshold`

These props work like you would expect them to work with IntersectionObserver.


### `clientOnly` prop

Disables `noscript` element rendering, instead rendering no HTML for the contents in server side. This gives behavior
similar to most other lazy loaders, which is why it is not enabled by default in `react-lazy`.


### `ltIE9` prop

Renders Internet Explorer 8 friendly syntax by adding conditional comments around `noscript`, effectively hiding
existance of the tag from IE8 and earlier. This allows for minimal legacy browser support, since it is highly unlikely
anyone writes their JS to execute on IE8 anymore.

Essentially this feature allows to render a visually non-broken page to users of legacy browsers, making it possible to
give minimally acceptable user experience to users of browsers that should be dead.

This means there is no lazy rendering on legacy browsers, images load immediately.

This prop has no effect if `clientOnly` is enabled.


### `onLoad`

- On `Lazy` triggers after removing `noscript` element.
- On `LazyGroup` triggers after **all** wrapped child components `onLoad` or `onError` events have triggered.

```jsx
<Lazy onLoad={yourCustomFunction}>...</Lazy>
```


### `onViewport`

Triggers before removing `noscript` elements. Given function receives IntersectionObserver event object.


### `visible`

Allows you to manually tell if the element is actually visible to the user or not.


----

## `<LazyGroup />`

`Lazy` works fine with single images, but sometimes you may want to have slightly more control or better performance
when you know multiple images load at the same time (for example, a row of thumbnails). In this case it makes no sense
to check each individual image's position in viewport when checking for just the container component will be good enough
&mdash; and also less for a browser to execute.

You can also use `Lazy` for multiple images, but there are some practical limitations such as the fact that everything
inside `Lazy` is within `noscript` element, thus there is nothing rendered inside. `LazyGroup` solves this issue by
rendering `noscript` only around specific wrapped elements (`img` and `iframe` by default). Also, further control is
given with `childWrapper` component that will receive a set of props to make life easier.

Use cases:

1. You want all contained images/iframes to be transitioned at the exact same time after everything is loaded.
2. You want to use the abstraction provided by `childWrapper` instead of writing custom logic.
3. You want to have slightly better performance by only checking the container element's location relative to the view.


### Usage

```jsx
// curly brackets are required
import { LazyGroup } from 'react-lazy'

function ImageContainer({ childProps, children, isFailed, isLoaded, ...props }) {
    return (
        // usually the other props include `dangerouslySetInnerHtml` when rendering `noscript` element
        <div {...props}>
            {isFailed ? 'The image did not load :( ' + childProps.src : children}
        </div>
    )
}

...

    <LazyGroup component="ul" className="image-list" childWrapper={ImageContainer}>
        {this.props.images.map((image, index) =>
            <li key={index} className="image-list__item">
                <img {...image} />
            </li>
        )}
    </LazyGroup>
```

### `childWrapper` lifecycle

1. On server side render and before the `LazyGroup` container is in viewport in client `childWrapper` will receive
`dangerouslySetInnerHtml` prop (thus rendering `noscript` element that contains the lazily loaded content).
2. After coming into viewport `isFailed` and `isLoaded` are false. `childProps` also become available.
3. `isFailed` is set to true when `img`'s or `iframe`'s `onError` event triggers. You can use `childProps` to decide
what to render.
4. `isLoaded` is set to true when `img`'s or `iframe`'s `onLoad` event triggers.


### `childrenToWrap`

Use this array to decide which components are wrapped by `childWrapper`. Default value: `['iframe', 'img']`

**Note!** The components **must** support `onError` and `onLoad` events as these are used to detect loading.


----

## Other components

You can see these components via React developer tools, but as of 1.0.2 they have not been exposed.


### `DefaultWrapper`

This is the `childWrapper` used to render `LazyGroup`'s wrapped childs if no custom wrapper is given. The wrapper is a
simple div with a `className` of `react-lazy-wrapper`. BEM convention is used to tell about the lifecycle:

1. `react-lazy-wrapper--placeholder` is set on server render and client render before `LazyGroup` is in viewport.
2. `react-lazy-wrapper--loading` is set once `LazyGroup` is in viewport.
3. `react-lazy-wrapper--failed` is set if lazy loaded component's `onError` event has triggered.
4. `react-lazy-wrapper--loaded` is set if lazy loaded component's `onLoad` event has triggered.


### `LazyChild`

This is the component used by `LazyGroup` to handle rendering of the wrapped child components. It manages the `onLoad` /
`onError` handling. It takes two props: `callback` and `wrapper`. `callback` is called by `LazyChild` once loading
result has been resolved. `wrapper` is the component rendered around wrapped child element.


----

## Developing

```
npm install
npm run build
npm test
```

**Note!** This component uses jsdom in it's tests. This means you may need to install stuff, especially on
Windows. The following is copied from [node-jsdom's readme](https://github.com/darrylwest/node-jsdom):

### Contextify

[Contextify](https://npmjs.org/package/contextify) is a dependency of jsdom, used for running `<script>` tags within the
page. In other words, it allows jsdom, which is run in Node.js, to run strings of JavaScript in an isolated environment
that pretends to be a browser environment instead of a server. You can see how this is an important feature.

Unfortunately, doing this kind of magic requires C++. And in Node.js, using C++ from JavaScript means using "native
modules." Native modules are compiled at installation time so that they work precisely for your machine; that is, you
don't download a contextify binary from npm, but instead build one locally after downloading the source from npm.

Getting C++ compiled within npm's installation system can be tricky, especially for Windows users. Thus, one of the most
common problems with jsdom is trying to use it without the proper compilation tools installed. Here's what you need to
compile Contextify, and thus to install jsdom:

#### Windows

- The latest version of [Node.js for Windows](http://nodejs.org/download/)
- A copy of [Visual Studio Express 2013 for Windows Desktop](http://www.visualstudio.com/downloads/download-visual-studio-vs#d-express-windows-desktop)
- A copy of [Python 2.7](http://www.python.org/download/), installed in the default location of `C:\Python27`
- Set your system environment variable GYP_MSVS_VERSION like so (assuming you have Visual Studio 2013 installed):
  ```shell
  setx GYP_MSVS_VERSION 2013
  ```

- Restart your command prompt window to ensure required path variables are present.

There are some slight modifications to this that can work; for example other Visual Studio versions often work too. But
it's tricky, so start with the basics!

#### Mac

- XCode needs to be installed
- "Command line tools for XCode" need to be installed
- Launch XCode once to accept the license, etc. and ensure it's properly installed

#### Linux

You'll need various build tools installed, like `make`, Python 2.7, and a compiler toolchain. How to install these will
be specific to your distro, if you don't already have them.
