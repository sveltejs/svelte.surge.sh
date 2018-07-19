---
title: Template syntax
---

Rather than reinventing the wheel, Svelte templates are built on foundations that have stood the test of time: HTML, CSS and JavaScript. There's very little extra stuff to learn.

> Svelte version 1 had a slightly different template syntax. You can upgrade older components automatically using [svelte-upgrade](https://github.com/sveltejs/svelte-upgrade).


### Tags

Tags allow you to bind data to your template. Whenever your data changes (for example after `component.set(...)`), the DOM updates automatically. You can use any JavaScript expression in templates, and it will also automatically update:

```html
<!-- { title: 'Template tags' } -->
<p>{a} + {b} = {a + b}</p>
```

```json
/* { hidden: true } */
{
	"a": 1,
	"b": 2
}
```

You can also use tags in attributes:

```html
<!-- { title: 'Tags in attributes' } -->
<h1 style="color: {color};">{color}</h1>
<p hidden={hideParagraph}>You can hide this paragraph.</p>
```

```json
/* { hidden: true } */
{
	color: "steelblue",
	hideParagraph: false
}
```
[Boolean attributes](https://www.w3.org/TR/html5/infrastructure.html#sec-boolean-attributes) like `hidden` will be omitted if the tag expression evaluates to false.


### HTML

Ordinary tags render expressions as plain text. If you need your expression interpreted as HTML, wrap it in a special `@html` tag:

```html
<!-- { title: 'Triple tags' } -->
<p>This HTML: {content}</p>
<p>Renders as: {@html content}</p>
```

```json
/* { hidden: true } */
{
	content: "Some <b>bold</b> text."
}
```

As with regular tags, you can use any JavaScript expression in HTML tags, and it will automatically update the document when your data changes.

> HTML is **not** sanitized before it is rendered! If you are displaying user input, you are responsible for first sanitizing it. Not doing so potentially opens you up to XSS attacks.


### If blocks

Control whether or not part of your template is rendered by wrapping it in an if block.

```html
<!-- { repl: false } -->
{#if user.loggedIn}
	<a href="/logout">log out</a>
{/if}

{#if !user.loggedIn}
	<a href="/login">log in</a>
{/if}
```

You can combine the two blocks above with `{:else}`:

```html
<!-- { repl: false } -->
{#if user.loggedIn}
	<a href="/logout">log out</a>
{:else}
	<a href="/login">log in</a>
{/if}
```

You can also use `{:elseif ...}`:

```html
<!--{ title: 'If, else and elseif' }-->
{#if x > 10}
	<p>{x} is greater than 10</p>
{:elseif 5 > x}
	<p>{x} is less than 5</p>
{:else}
	<p>{x} is between 5 and 10</p>
{/if}
```

```json
/* { hidden: true } */
{
	x: 7
}
```

### Each blocks

Iterate over lists of data:

```html
<!--{ title: 'Each blocks' }-->
<h1>Cats of YouTube</h1>

<ul>
	{#each cats as cat}
		<li><a target="_blank" href={cat.video}>{cat.name}</a></li>
	{/each}
</ul>
```

```json
/* { hidden: true } */
{
	cats: [
		{
			name: "Keyboard Cat",
			video: "https://www.youtube.com/watch?v=J---aiyznGQ"
		},
		{
			name: "Maru",
			video: "https://www.youtube.com/watch?v=z_AbfPXTKms"
		},
		{
			name: "Henri The Existential Cat",
			video: "https://www.youtube.com/watch?v=OUtn3pvWmpg"
		}
	]
}
```

You can access the index of the current element with *expression* as *name*, *index*:

```html
<!--{ title: 'Each block indexes' }-->
<div class="grid">
	{#each rows as row, y}
		<div class="row">
			{#each columns as column, x}
				<code class="cell">
					{x + 1},{y + 1}:
					<strong>{row[column]}</strong>
				</code>
			{/each}
		</div>
	{/each}
</div>
```

```json
/* { hidden: true } */
{
	columns: ["foo", "bar", "baz"],
	rows: [
		{ foo: "a", bar: "b", baz: "c" },
		{ foo: "d", bar: "e", baz: "f" },
		{ foo: "g", bar: "h", baz: "i" }
	]
}
```

> By default, if the list `a, b, c` becomes `a, c`, Svelte will *remove* the third block and *change* the second from `b` to `c`, rather than removing `b`. If that's not what you want, use a [keyed each block](guide#keyed-each-blocks).

You can use destructuring patterns on the elements of the array:

```html
<!--{ title: 'Each block destructuring' }-->
<h1>It's the cats of YouTube again</h1>

<ul>
	{#each cats as {name, video} }
		<li><a target="_blank" href={video}>{name}</a></li>
	{/each}
</ul>
```

```json
/* { hidden: true } */
{
	cats: [
		{
			name: "Keyboard Cat",
			video: "https://www.youtube.com/watch?v=J---aiyznGQ"
		},
		{
			name: "Maru",
			video: "https://www.youtube.com/watch?v=z_AbfPXTKms"
		},
		{
			name: "Henri The Existential Cat",
			video: "https://www.youtube.com/watch?v=OUtn3pvWmpg"
		}
	]
}
```

If you want to iterate over an object you can use `Object.entries(object)` which returns the object's properties as `[key, value]` pairs:

```html
<!--{ title: 'Iterating over objects' }-->
<h1>Cats and Dogs</h1>

{#each Object.entries(animals) as [animal, names]}
	<p>{animal}: {names.join(" and ")}</p>
{/each}
```

```json
/* { hidden: true } */
{
	animals: {
		Cats: ["Buzz", "Stella"],
		Dogs: ["Hector", "Victoria"]
	}
}
```

### Await blocks

You can represent the three states of a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) — pending, fulfilled and rejected — with an `await` block:

```html
<!--{ title: 'Await blocks' }-->
{#await promise}
	<p>wait for it...</p>
{:then answer}
	<p>the answer is {answer}!</p>
{:catch error}
	<p>well that's odd</p>
{/await}

<script>
	export default {
		data() {
			return {
				promise: new Promise(fulfil => {
					setTimeout(() => fulfil(42), 3000);
				})
			};
		}
	};
</script>
```

If the expression in `{#await expression}` *isn't* a promise, Svelte skips ahead to the `then` section.


### Directives

The last place where Svelte template syntax differs from regular HTML: *directives* allow you to add special instructions for adding [event handlers](guide#event-handlers), [bindings](guide#bindings), [referencing elements](guide#refs) and so on. We'll cover each of those in later stages of this guide – for now, all you need to know is that directives can be identified by the `:` character:

```html
<!--{ title: 'Element directives' }-->
<p>Count: {count}</p>
<button on:click="set({ count: count + 1 })">+1</button>
```

```json
/* { hidden: true } */
{
	count: 0
}
```

> Technically, the `:` character is used to denote namespaced attributes in HTML. These will *not* be treated as directives, if encountered.
