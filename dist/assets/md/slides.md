# Pimping the Feedback Loop

## The never-ending Quest for faster and more Interactive Dev experience

## Before we start

### Clear outcomes

### Speed of implementation

## Some Context

### Tech Stack

### App Flow
![app-arch](./assets/img/app-architecture.png)

### App Flow (In Code)
```js
const actions = require('actions');
const ui = require('ui');

// reducing the stream of actions to the app state
const state$ = actions.stream
	.scan(
		(state, reducer) => reducer(state),
		actions.initial
	);

// mapping the state to the ui
const ui$ = state$.map(
	state => ui({state, actions})
);

vdom.patchStream(ui$, '#ui');

```

### Counter Example
```js
// actions
const actions$ = new Rx.Subject();
const actions = {
	incr: () => actions$.onNext(
		state => ({num: state.num + 1})),
	initial: {num: 0}
};

// ui
const {div, span, button} = vdom;
const ui = ({state, actions}) => div('#ui', [
	button({on:{click: () => actions.incr()}}, 'Incr'),
	span(`Num: ${state.num}`)
]);

// reducing the stream of actions to the app state
const state$ = actions$
	.startWith(() => actions.initial)
	.scan((state, reducer) => reducer(state), {})
	.share();

// mapping the state to the ui
const ui$ = state$.map(state => ui({state, actions}));

vdom.patchStream(ui$, document.querySelector('#ui'));
```

## Task Automation

### Purely NPM Based
```sh
npm i
npm build
npm build:electron
npm start
```

### CLI instead of Gulp modules
- **node-sass** src/sass/style.sass dist/css/style.css
- **browserify** src/js/index.js -o dist/js/app.js

### Simple Scripts when needed
```js
'use strict';

const path = require('path');

const paths = [].concat(
	require('bourbon').includePaths,
	require('bourbon-neat').includePaths,
	require('code-prettify').includePaths,
	path.resolve(__dirname, '..', 'node_modules/font-awesome/scss')
);

process.stdout.write(paths.join(':'));
```

## Hot Module Replacemet
- Updating the functionality without Refreshing or Changing the State

### Usage

- Install: `npm i -D browserify-hmr`
- watchify **-p browserify-hmr src/js/index.js** -o dist/js/app.js

### In Code
```js
// hot reloading
if (module.hot) {
	// actions
	actions$ = $.fromEventPattern(
		h => module.hot.accept("./actions", h)
	).flatMap(() => {
		actions = app.adapt(require('./actions'));
		return actions.stream.startWith(state => state);
	}).merge(actions.stream);
	// ui
	module.hot.accept("./ui", function() {
		ui = require('./ui');
		actions.stream.onNext(state => state);
	});
} else {
	actions$ = actions.stream;
}
```

## CSS Vars

### Basic Usage
```css
// declaring the var
element {
  --main-bg-color: brown;
}

// using the var
element {
  background-color: var(--main-bg-color);
}
```

### My Approach
```sass
#ui
	--stsh-heading-font: 'Rajdhani'
	--stsh-body-font: 'Rajdhani'
	--stsh-bg-gradient: linear-gradient(to bottom, #f2f5f6 0%, #e3eaed 29%, #c8d7dc 100%)
	position: absolute
	width: 100%
	height: 100%
	background: var(--stsh-bg-gradient)
	font-family: var(--stsh-body-font)
	overflow-y: scroll
	input, textarea, button
		font-family: var(--stsh-body-font)
```

### Parsing in JS
```js
const parseGradient = g => [g.match(/([a-z-]+)\(([a-z 0-9]+),? ?(([a-z0-9#% ]+,? ?)*)?\)/)].map(matches => ({
	type: matches[1] || 'linear-gradient',
	direction: matches[2] || 'to right',
	steps: matches[3].trim().split(', ').map(p => p.split(' ')) || []
})).pop();

document.addEventListener("DOMContentLoaded", ev =>
	[window.getComputedStyle(document.querySelector('#ui'))].map(cstyle =>
		actions.set('cssVars', {
			stshHeadingFont: cstyle.getPropertyValue('--stsh-heading-font').trim().replace(/"/g, ''),
			stshBodyFont: cstyle.getPropertyValue('--stsh-body-font').trim().replace(/"/g, ''),
			stshBgGradient: parseGradient(cstyle.getPropertyValue('--stsh-bg-gradient'))
		})
	));
```

### In UI
```js
section('#ui', {
	style: {
		'--stsh-heading-font': state.cssVars.stshHeadingFont || '',
		'--stsh-body-font': state.cssVars.stshBodyFont || '',
		'--stsh-bg-gradient': state.cssVars.stshBgGradient && buildGradient(state.cssVars.stshBgGradient) || ''
	}
}, [
	header({state, actions}),
	list({state, actions})
]);
```

## Markdown

## Questions
