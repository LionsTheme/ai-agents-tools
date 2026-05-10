# Astro: Framework Integrations

Patterns and setup for integrating UI frameworks (React, Vue, Svelte, Solid, Preact, Alpine.js) and tools (Tailwind, MDX) with Astro.

## Adding Integrations

```bash
# Auto-configure (updates astro.config.mjs)
npx astro add react
npx astro add vue
npx astro add svelte
npx astro add solid-js
npx astro add preact
npx astro add alpinejs
npx astro add tailwind
npx astro add mdx

# Add multiple at once
npx astro add react tailwind mdx
```

Manual installation:

```bash
npm install @astrojs/react react react-dom
```

## React Integration

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';

export default defineConfig({
  integrations: [react()],
});
```

```tsx
// src/components/ReactCounter.tsx
import { useState, useEffect } from 'react';

interface Props {
  initialCount?: number;
}

export default function Counter({ initialCount = 0 }: Props) {
  const [count, setCount] = useState(initialCount);

  return (
    <div className="border rounded p-4 inline-flex items-center gap-3">
      <button
        onClick={() => setCount(c => c - 1)}
        className="bg-gray-200 px-3 py-1 rounded"
      >
        -
      </button>
      <span className="text-xl font-bold">{count}</span>
      <button
        onClick={() => setCount(c => c + 1)}
        className="bg-gray-200 px-3 py-1 rounded"
      >
        +
      </button>
    </div>
  );
}
```

```astro
---
// src/pages/index.astro
import Counter from '../components/ReactCounter';
---

<Counter client:load initialCount={5} />
<Counter client:visible initialCount={0} />
```

## Vue Integration

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import vue from '@astrojs/vue';

export default defineConfig({
  integrations: [vue()],
});
```

```vue
<!-- src/components/VueGreeting.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue';

const props = defineProps<{ name: string }>();

const count = ref(0);
const message = computed(() => `Hello, ${props.name}!`);

function increment() {
  count.value++;
}
</script>

<template>
  <div class="border rounded p-4">
    <h3 class="text-lg font-semibold">{{ message }}</h3>
    <p class="text-gray-500">Clicked {{ count }} times</p>
    <button
      @click="increment"
      class="bg-blue-500 text-white px-4 py-2 rounded mt-2"
    >
      Click me
    </button>
  </div>
</template>
```

```astro
---
import VueGreeting from '../components/VueGreeting.vue';
---

<VueGreeting client:load name="Astro" />
```

## Svelte Integration

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import svelte from '@astrojs/svelte';

export default defineConfig({
  integrations: [svelte()],
});
```

```svelte
<!-- src/components/SvelteTimer.svelte -->
<script lang="ts">
  export let startTime: number = 0;

  let elapsed = 0;
  let interval: ReturnType<typeof setInterval>;

  $: display = (elapsed / 1000).toFixed(1);

  function startTimer() {
    interval = setInterval(() => {
      elapsed = Date.now() - startTime;
    }, 100);
  }

  function stopTimer() {
    clearInterval(interval);
  }
</script>

<div class="border rounded p-4">
  <p class="text-2xl font-mono">{display}s</p>
  <div class="flex gap-2 mt-2">
    <button on:click={startTimer} class="bg-green-500 text-white px-3 py-1 rounded">
      Start
    </button>
    <button on:click={stopTimer} class="bg-red-500 text-white px-3 py-1 rounded">
      Stop
    </button>
  </div>
</div>
```

```astro
---
import SvelteTimer from '../components/SvelteTimer.svelte';
---

<SvelteTimer client:visible startTime={Date.now()} />
```

## Solid.js Integration

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import solidJs from '@astrojs/solid-js';

export default defineConfig({
  integrations: [solidJs()],
});
```

```tsx
// src/components/SolidTodoList.tsx
import { createSignal, For } from 'solid-js';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

export default function TodoList() {
  const [todos, setTodos] = createSignal<Todo[]>([
    { id: 1, text: 'Learn Astro', completed: true },
    { id: 2, text: 'Build something cool', completed: false },
  ]);
  const [newText, setNewText] = createSignal('');

  function addTodo() {
    if (!newText().trim()) return;
    setTodos([
      ...todos(),
      { id: Date.now(), text: newText(), completed: false },
    ]);
    setNewText('');
  }

  function toggleTodo(id: number) {
    setTodos(
      todos().map((t) =>
        t.id === id ? { ...t, completed: !t.completed } : t,
      ),
    );
  }

  return (
    <div>
      <div class="flex gap-2 mb-4">
        <input
          type="text"
          value={newText()}
          onInput={(e) => setNewText(e.target.value)}
          class="border rounded px-2 py-1 flex-1"
          placeholder="New todo..."
        />
        <button onClick={addTodo} class="bg-blue-500 text-white px-3 py-1 rounded">
          Add
        </button>
      </div>
      <ul class="space-y-2">
        <For each={todos()}>
          {(todo) => (
            <li
              class="flex items-center gap-2 cursor-pointer"
              onClick={() => toggleTodo(todo.id)}
            >
              <span>{todo.completed ? '✅' : '⬜'}</span>
              <span class={todo.completed ? 'line-through text-gray-400' : ''}>
                {todo.text}
              </span>
            </li>
          )}
        </For>
      </ul>
    </div>
  );
}
```

## Preact Integration

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import preact from '@astrojs/preact';

export default defineConfig({
  integrations: [preact({ compat: true })], // compat enables React compatibility
});
```

```tsx
// src/components/PreactSearch.tsx
import { useState } from 'preact/hooks';

interface Props {
  items: string[];
}

export default function Search({ items }: Props) {
  const [query, setQuery] = useState('');

  const filtered = items.filter((item) =>
    item.toLowerCase().includes(query.toLowerCase()),
  );

  return (
    <div>
      <input
        type="text"
        value={query}
        onInput={(e) => setQuery(e.currentTarget.value)}
        placeholder="Search..."
        class="border rounded px-3 py-2 w-full"
      />
      {query && (
        <ul class="mt-2 border rounded divide-y">
          {filtered.map((item) => (
            <li class="px-3 py-2">{item}</li>
          ))}
          {filtered.length === 0 && (
            <li class="px-3 py-2 text-gray-400">No results</li>
          )}
        </ul>
      )}
    </div>
  );
}
```

## Alpine.js Integration

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import alpinejs from '@astrojs/alpinejs';

export default defineConfig({
  integrations: [alpinejs()],
});
```

```astro
---
// Alpine.js works directly in .astro files
---

<div x-data="{ open: false, count: 0 }">
  <button
    @click="open = !open"
    class="bg-gray-200 px-4 py-2 rounded"
  >
    Toggle
  </button>

  <div x-show="open" class="mt-2 p-4 border rounded">
    <p>Count: <span x-text="count"></span></p>
    <button
      @click="count++"
      class="bg-blue-500 text-white px-3 py-1 rounded mt-2"
    >
      Increment
    </button>
  </div>
</div>
```

## Tailwind CSS Integration

```bash
npx astro add tailwind
```

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [tailwind()],
});
```

```css
/* src/styles/global.css */
@import "tailwindcss";

@theme {
  --color-primary: oklch(45% 0.2 260);
  --color-secondary: oklch(65% 0.15 200);
}

@custom-variant dark (&:where(.dark, .dark *));
```

## MDX Integration

```bash
npx astro add mdx
```

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import mdx from '@astrojs/mdx';

export default defineConfig({
  integrations: [mdx()],
});
```

```astro
---
// src/pages/mdx-demo.astro
import BaseLayout from '../layouts/BaseLayout.astro';
import HelloWorld from '../content/hello.mdx';
---

<BaseLayout title="MDX Demo">
  <!-- Render MDX component directly -->
  <HelloWorld />
</BaseLayout>
```

### MDX with Custom Components

```astro
---
// src/pages/blog/[slug].astro
import { getEntry, render } from 'astro:content';
import CustomHeading from '../../components/CustomHeading.astro';
import CodeBlock from '../../components/CodeBlock.astro';

const entry = await getEntry('blog', Astro.params.slug);
const { Content } = await render(entry);
---

<!-- Pass custom components to MDX -->
<Content
  components={{
    h1: CustomHeading,
    pre: CodeBlock,
  }}
/>
```

## Component Comparison by Framework

| Feature          | React          | Vue              | Svelte           | Solid            | Preact          | Alpine.js       |
| ---------------- | -------------- | ---------------- | ---------------- | ---------------- | --------------- | --------------- |
| **Bundle size**  | ~40KB          | ~33KB            | ~2KB             | ~7KB             | ~3KB            | ~15KB           |
| **Reactivity**   | Virtual DOM    | Proxies          | Compiler         | Signals          | Virtual DOM     | DOM attributes  |
| **TypeScript**   | Full           | Full (.vue)      | Partial          | Full             | Partial         | N/A             |
| **Best for**     | Complex apps   | Interactive UI   | Animations       | Performance      | Lightweight     | Simple interactivity |
| **Client directive** | `client:load` | `client:load`   | `client:load`   | `client:load`   | `client:load`  | `client:load` (or none, auto-hydrates) |

## Progressive Enhancement Pattern

Use framework components only where interactivity is needed:

```astro
---
// src/pages/index.astro
// Static content: .astro (zero JS)
import HeroBanner from '../components/Hero.astro';
import FeaturesGrid from '../components/FeaturesGrid.astro';

// Interactive islands: framework components
import SearchBar from '../components/SearchBar.jsx';
import NewsletterForm from '../components/NewsletterForm.vue';
import TestimonialsCarousel from '../components/Testimonials.svelte';
---

<BaseLayout>
  <!-- Zero JS -->
  <HeroBanner />
  <FeaturesGrid />

  <!-- Hydrate when visible -->
  <SearchBar client:visible />

  <!-- Hydrate on load -->
  <NewsletterForm client:load />

  <!-- Hydrate when idle (low priority) -->
  <TestimonialsCarousel client:idle />
</BaseLayout>
```
