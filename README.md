# Svelte draggable draw/modal

A svelte draggable draw/modal just like what you had seen in any of the mobile app!

## Demo

![example](/example.gif)

[Svelte.dev Repl](https://svelte.dev/repl/a76f1b9d987c43f1ac457efb189d5d0d)

TODO:

- [ ] Allow adjustment for width
- [ ] Allow adjustment for custom unit instead of viewport only
- [ ] Allow adjustment for custom box shadow
- [ ] Allow adjustment for custom padding

## Setting up

```bash
npm install svelte-draggable-draw
```

## Consuming components

```html

<script>
    import DraggableDraw from 'svelte-draggable-draw';
    let visible = true;
    let maxVH = 90;
    let minVH = 85;

    function switchVisible(){
        visible = !visible;
    }
</script>

<button on:click={switchVisible}>Click me to open</button>
<DraggableDraw bind:visible {maxVH} {minVH}>
    <span slot="left" on:click={switchVisible}>Cancel</span>
    <span slot="right" on:click={switchVisible}>Submit</span>

    <div>
        <h1>Content</h1>
        <p>Can be injected here</p>
    </div>
</DraggableDraw>
```

Enjoy😎!
