
[AlpineJS](https://alpinejs.dev/) is a lightweight JavaScript library that makes it easy to add client-side interactivity to your web pages. It was originally built to compliment tools like Livewire where a more JavaScript-centric utility is helpful for sprinkling interactivity around your app.

Livewire ships with Alpine out of the box so there is no need to install it into your project separately.

The best place to learn about using AlpineJS is [the Alpine documentation](https://alpinejs.dev).

## A Basic Alpine component

To lay a foundation for the rest of this documentation, here is one of the most simple and informative examples of an Alpine component. A small "counter" that shows a number on the page and allows the user to increment that number by clicking a button:

```html
<!-- Declare a JavaScript object of data... -->
<div x-data="{ count: 0 }">
    <!-- Render the current "count" value inside an element... -->
    <h2 x-text="count"></h2>

    <!-- Increment the "count" value by "1" when a click event is dispatched... -->
    <button x-on:click="count++">+</button>
</div>
```

The Alpine component above can be used inside any Livewire component in your application without a hitch. Livewire takes care of maintaining Alpine's state across Livewire component updates. In essance, you should feel free to use Alpine components inside Livewire as if you were using Alpine in any other non-Livewire context.

## Using Alpine inside Livewire

Let's explore a more real-life example of using a Alpine inside a Livewire component.

Below is a simple Livewire component showing the details of a post model from the database. By default, only the title of the post is shown:

```html
<div>
    <h1>{{ $post->title }}</h1>

    <div x-data="{ expanded: false }">
        <button type="button" x-on:click="expanded = ! expanded">
            <span x-show="! expanded">Show post content...</span>
            <span x-show="expanded">Hide post content...</span>
        </button>

        <div x-show="expanded">
            {{ $post->content }}
        </div>
    </div>
</div>
```

By using Alpine, we can hide the content of the post until the user presses the "Show post content..." button. At that point, Alpine's `expanded` property will be set to `true` and the content will be shown on the page because `x-show="content"` is used to give Alpine control over the visibility of the post's content.

This is an example of where Alpine shines: adding interactivity into your application without triggering Livewire server-roundtrips.

## Controlling Livewire from Alpine using `$wire`

One of the most powerful features available to you as a Livewire developer is `$wire`. The `$wire` object is a magic object available to all your Alpine components that are used inside of Livewire.

You can think of `$wire` as a gateway from JavaScript into PHP. It allows you to access and modify Livewire component properties, call Livewire component methods, and do much more; all from inside AlpineJS.

### Accessing Livewire properties

Here is an example of a simple "character count" utility in a form for creating a post. This will instantly show a user how many characters are contained inside their post's content as they type:

```html
<form wire:submit="save">
    <!-- ... -->

    <input wire:model="content" type="text">

    <small>
        Character count: <span x-text="$wire.content.length"></span> <!-- [tl! highlight] -->
    </small>

    <button type="submit">Save</button>
</form>
```

As you can see `x-text` in the above example is being used to allow Alpine to control the text content of the `<span>` element. `x-text` accepts any JavaScript expression inside of it and automatically reacts when any dependancies are updated. Because we are using `$wire.content` to access the value of `$content`, Alpine will automatically update the text content every time `$wire.content` is updated from Livewire; in this case by `wire:model="content"`.

### Mutating Livewire properties

Here is an example of using `$wire` inside Alpine to clear the "title" field of a form for creating a post.

```html
<form wire:submit="save">
    <input wire:model="title" type="text">

    <button type="button" x-on:click="$wire.title = ''">Clear</button> <!-- [tl! highlight] -->

    <!-- ... -->

    <button type="submit">Save</button>
</form>
```

As a user is filling out the above Livewire form, they can press "Clear" and the title field will be cleared without sending a network request from Livewire. The interaction will be "instant".

Here's a brief explanation of what's going on to make that happen:

* `x-on:click` tells Alpine to listen for a click on the button element
* When clicked, Alpine runs the provided JS expression: `$wire.title = ''`
* Because `$wire` is a magic object representing the Livewire component, all properties from your component can be accessed or mutated straight from JavaScript
* `$wire.title = ''` sets the value of `$title` in your Livewire component to an empty string
* Any Livewire utilities like `wire:model` will instantly react to this change, all without sending a server-roundtrip
* On the next Livewire network request, the `$title` property will be updated to an empty string on the backend

### Calling Livewire methods

Alpine can also easily call any Livewire methods/actions by simply calling them directly on `$wire`.

Here is an example of using Alpine to listen for a "blur" event on an input and triggering a form save. The "blur" event is dispatched by the browser when a user presses "tab" to remove focus from the current elemement and focus on the next one on the page:

```html
<form wire:submit="save">
    <input wire:model="title" type="text" x-on:blur="$wire.save()">  <!-- [tl! highlight] -->

    <!-- ... -->

    <button type="submit">Save</button>
</form>
```

Typically you would just use `wire:model.blur="title"` in this situation, however, it's helpful for demonstration purposes how you can achieve this using Alpine.

#### Passing parameters

You can also pass parameters to Livewire methods by simply passing the to the `$wire` method call.

Consider a component with a `deletePost()` method like so:

```php
public function deletePost($postId)
{
    $post = Post::find($postId);

    // Authorize user can delete...
    auth()->user()->can('update', $post);

    $post->delete();
}
```

Now, you can pass a `$postId` parameter to the `deletePost()` method from Alpine like so:

```html
<button type="button" x-on:click="$wire.deletePost(1)">
```

In general, something like a `$postId` would be generated in Blade. Here's an example of using Blade to determine which `$postId` Alpine passes into `deletePost()`:

```html
@foreach ($posts as $post)
    <button type="button" x-on:click="$wire.deletePost({{ $post->id }})">
        Delete "{{ $post->title }}"
    </button>
@endforeach
```

If there are three posts on the page, the above Blade template will render to something like the following in the browser:

```html
<button type="button" x-on:click="$wire.deletePost(1)">
    Delete "The power of walking"
</button>

<button type="button" x-on:click="$wire.deletePost(2)">
    Delete "How to record a song"
</button>

<button type="button" x-on:click="$wire.deletePost(3)">
    Delete "Teach what you learn"
</button>
```

As you can see, we've used Blade to render different post IDs into the Alpine `x-on:click` expressions.

#### Blade parameter "gotchas"

This is an extremely powerful technique, but can be confusing when reading your Blade templates. It can be hard to know which parts are Blade and which parts are Alpine at first glance. Therefore, it's helpful to inspect the HTML rendered on the page to make sure what you are expecting to be rendered is accurate.

Here's an example that commonly confuses people:

Let's say, instead of an ID, your Post model uses UUIDs for indexes (IDs are integers, and UUIDs are long strings of characters).

If we render the following just like we did with an ID there will be an issue:

```html
<!-- Warning: this is an example of problematic code... -->
<button
    type="button"
    x-on:click="$wire.deletePost({{ $post->uuid }})"
>
```

The above Blade template will render the following in your HTML:

```html
<!-- Warning: this is an example of problematic code... -->
<button
    type="button"
    x-on:click="$wire.deletePost(93c7b04c-c9a4-4524-aa7d-39196011b81a)"
>
```

Notice the lack of quotes around the UUID string? When Alpine goes to evaluate this expression, JavaScript will throw an error: "Uncaught SyntaxError: Invalid or unexpected token".

To fix this, we need to add quotations around the Blade expression like so:

```html
<button
    type="button"
    x-on:click="$wire.deletePost('{{ $post->uuid }}')"
>
```

Now the above template will render properly and everything will work as expected:

```html
<button
    type="button"
    x-on:click="$wire.deletePost('93c7b04c-c9a4-4524-aa7d-39196011b81a')"
>
```

### Refreshing a component

You can easily refresh a Livewire component (trigger network roundtrip to re-render a component's Blade view) using `$wire.$refresh()`:

```html
<button type="button" x-on:click="$wire.$refresh()">
```

## Manually bundling Alpine in your JavaScript build

By default, Livewire and Alpine's JavaScript is injected onto each Livewire page automatically.

This is ideal for simpler setups, however, you may want to include your own Alpine components, stores, and plugins into your project.

To include Livewire and Alpine via your own JavaScript bundle on a page is straightforward.

First, you must include the `@livewireScriptConfig` directive in your layout file like so:

```blade
<html>
<head>
    <!-- ... -->
    @livewireStyles
    @vite(['resources/js/app.js'])
</head>
<body>
    {{ $slot }}

    @livewireScriptConfig <!-- [tl! highlight] -->
</body>
</html>
```

This allows Livewire to provide your bundle with certain configuration it needs for your app to run properly.

Now you can import Livewire and Alpine in your `resources/js/app.js` file like so:

```js
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';

// Register any Alpine directives, components, or plugins here...

Livewire.start()
```

Here is an example of registering a custom Alpine directive called "x-clipboard" in your application:

```js
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';

Alpine.directive('clipboard', (el) => {
    let text = el.textContent

    el.addEventListener('click', () => {
        navigator.clipboard.writeText(text)
    })
})

Livewire.start()
```

Now the `x-clibboard` directive will be available to all your Alpine components in your Livewire application.
