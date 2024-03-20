---
# try also 'default' to start simple
# theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: /background.jpeg
# some information about your slides, markdown enabled
title: Vuejs Amsterdam
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
hideInToc: true
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# VUEJS AMSTERDAM 2024
<br><br><br><br><br><br><br><br><br><br>

---
hideInToc: true
---

# Table of contents


<Toc minDepth="1" maxDepth="1"></Toc>

---

# Nuxt server components

Non-interactive component without any client JS
- smaller bundle size
- privileged code runs securely
- support all the features of normal components

<br>

```html {all|5|all}
<template>
  <div>
    <HighlightedMarkdown markdown="# Headline" />
    <!-- Counter will be loaded and hydrated client-side -->
    <Counter nuxt-client :count="5" />
  </div>
</template>
```

<arrow v-click="[1,2]" x1="350" y1="290" x2="235" y2="335" color="#953" width="2" arrowSize="1" />

[Learn more](https://nuxt.com/docs/guide/directory-structure/components#server-components)

<!--
Nuxt server components are non interactive components without any client javascript

When rendering an island component, the content of the island component is static, thus no JS is downloaded client-side.

It can have the props
When their props update, this will result in a network request that will update the rendered HTML in-place.

-these components no longer need to be hydrated or tracked by Vue, 

-Server components allow you to extract logic out of your client-side bundle.
When your logic requires access to a database or needs a private key or secret, server components can be a useful solution

[click] You can partially hydrate a component by setting a nuxt-client attribute on the component you wish to be loaded client-side.
-->

---

# Nuxt hub (alpha)

Build full-stack Nuxt apps, on the <span v-mark.red>edge</span>
<ul >
<li>integrated with Cloudflare</li>
<li>local development support</li>
<li>SQL database, key-value storage, blob storage</li>
</ul>


[Learn more](https://hub.nuxt.com/)

<!--
NuxtHub aims to provide a complete backend experience for Nuxt apps, allowing developers to build full-stack applications on the Edge

What does it mean to build applications on the Edge?

[click]In September 2017, Cloudflare introduced Cloudflare Workers, giving the ability to run JavaScript on their edge network. This means your code will deploy on the entire edge network in over a hundred locations worldwide in about 30 seconds. This technology allows you to focus on writing your application close to your users, wherever they are in the world. It unlocks the ability to server-render pages in ~50ms from all over the world when using a platform like CloudFlare Workers, without having to deal with servers, load balancers and caching, for about $0.5 per million requests. 

Nuxt hub offers a seamless integration with Cloudflare's infrastructure, providing access to SQL databases to store your applications data, key-value storage to store JSON data accessible globally with low latency and blob storage to store static assets such as images videos and more

This setup is ideal for applications that require low-latency data access and high-speed caching.
-->

---
title: Server state
---

# Server state

Data fetched from an API or a backend server

<v-clicks>

- Challenges with Global Stores
- [TanStack Query](https://tanstack.com/query/latest) and soon [Pinia Colada (WIP)](https://github.com/posva/pinia-colada) as a solution
- Caching and Automatic Updates
- Still a Place for Global Stores

</v-clicks>

<!--
Server state refers to the data fetched from an API or a backend server. 
This data is not yet processed or manipulated by the client-side code. When working with server state, it is essential to handle asynchronous data fetching and caching to ensure a smooth user experience.

Dividing state into two categories, server state and client state, can make the task of managing state more manageable, reducing the amount of code required and minimizing the risk of bugs and other issues.

[click] Using Pinia (or similar global stores) for server state can lead to unnecessary complexity and boilerplate code, as it wasn't specifically designed for server-state management. It's more suited for client-side state that doesn't frequently update from server responses. 

 For server-state, especially data that benefits from caching and invalidation patterns
[click] tools like TanStack Query are more efficient, reducing requests and simplifying data synchronization across components

[click] It offers a powerful caching mechanism that reduces the number of requests to the server, improving the performance of applications. Automatically updates the UI with the latest data without manual intervention, ensuring the user interface is always up-to-date. It simplifies data fetching, reduces boilerplate, and enables easy sharing of server state across components.

Actually I have never used tanstack query myself and I know many of you have, so If I am wrong, please feel free to correct me

[click] Global stores like Pinia still play a crucial role in managing non-server state, such as UI state, user preferences, or complex inter-component interactions that do not directly correspond to server data. These scenarios benefit from a centralized state management system, where the focus is on client-side. Global stores provide a structured way to manage these types of state, ensuring consistency and reactivity across the application.
-->

---
title: Server state example
level: 2
---

# Server state
````md magic-move
```ts
import { defineStore } from 'pinia'

type MetadataDto = {
  name: string,
  imgUrl: string
}

export const useMetadataStore = defineStore('metadata', {
  const state = { 
    name: undefined,
    imgUrl: undefined
  }

  const isLoading = computed(() => !state.name || !state.imgUrl);

  const updateMetadata = async () => {
    const response = await fetch("https://639df98e1ec9c6657bb6f91b.mockapi.io/myself");
    const newData = await response.json() as MetadataDto;
    state.name = newData.name;
    state.imgUrl = newData.imgUrl;
  },
})

return { state, isLoading, updateMetadata };
```

```vue
<script setup lang="ts">
import { useQuery } from '@tanstack/vue-query'
 
type MetadataDto = {
  name: string,
  imgUrl: string
}

const useMyselfQuery = () => {
  return useQuery({
    queryKey: ['myself'],
    queryFn: async () => {
      const response = await fetch(`https://639df98e1ec9c6657bb6f91b.mockapi.io/myself`);
      const data = await response.json() as MetadataDto;
      return data;
    },
    staleTime: Infinity,
  })
}

const { isLoading, isError, data, error } = useMyselfQuery();
</script>
```
````

<!--
Talk about example using pinia

[click] Example using TanStack query
-->

---
class: px-20
---

# Supabase Realtime

Send and receive messages to connected clients

<v-clicks>

   - broadcast
   - presence
   - postgres changes

</v-clicks>

```ts {hide|1-3|10-14|4-14|all}
// Initialize the JS client
import { createClient } from '@supabase/supabase-js'
const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY)

// Create a function to handle inserts
const handleInserts = (payload) => {
  console.log('Change received!', payload)
}

// Listen to inserts
supabase
  .channel('todos')
  .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'todos' }, handleInserts)
  .subscribe()
```

[Learn more](https://supabase.com/docs/guides/realtime)

<!--
What is supabase? 
Supabase is an open-source platform that provides developers with tools to quickly build apps. It offers a PostgreSQL database, authentication, real-time data subscriptions, and file storage, all accessible through an API. Designed as an alternative to Firebase.

Supabase provides a globally distributed cluster of Realtime servers that enable the following functionality:

[click] Broadcast: sends rapid, temporary messages to other connected clients with low latency. You can use it to track mouse movements, for example.

[click] Presence: sends user state between connected clients. You can use it to show an "online" status, which disappears when a user is disconnected.

[click]  Postgres Changes: Listen to Postgres database changes and send them to authorized clients in real-time.

[click:4] chat applications, live content updates, or collaborative tools, showcasing
-->

---
layout: two-cols
---

# Formkit

Efficient way for developers to create forms

<v-clicks>

- Single-component System
- Built-in Validation
- Accessibility-focused
- Flexible Styling
- Decentralized Node System
- Extensible Schemas
- UI Framework Compatibility

</v-clicks>

[Learn More](https://formkit.com/)

::right::
<br><br>

````md magic-move
```vue
<template>
  <FormKit type="text" />
</template>
```

```vue
<template>
<FormKit 
    type="text" 
    name="name" 
    id="name" 
  />
</template>
```

```vue
<template>
  <FormKit
    type="text"
    name="name"
    id="name"
    label="Name"
    help="Your full name"
    placeholder="“Jon Doe”"
  />
</template>
```

```vue
<template>
  <FormKit
    type="range"
    name="strength"
    id="strength"
    label="Strength"
    value="5"
    min="1"
    max="10"
    step="1"
    help="How many strength points should this character have?"
  />
</template>
```

```vue
<template>
  <FormKit
    type="range"
    name="strength"
    id="strength"
    label="Strength"
    value="5"
    validation="min:2|max:9"
    validation-visibility="live"
    min="1"
    max="10"
    step="1"
    help="How many strength points should this character have?"
  />
</template>
```

```vue
<template>
  <FormKit type="form">
    <FormKit
      type="range"
      name="strength"
      id="strength"
      label="Strength"
      value="5"
      validation="min:2|max:9"
      validation-visibility="live"
      min="1"
      max="10"
      step="1"
      help="How many strength points should this character have?"
    />
  </FormKit>
</template>
```
````


 <!--
 FormKit is a comprehensive form-building framework designed specifically for Vue developers. It provides a robust platform for creating high-quality, production-ready forms with less code, offering features like inputs, validation, error handling, and more. What sets FormKit apart is its focus on a single-component system, accessibility by default, built-in validation, and extensible schemas for form generation. 

[click] Single-component System: This simplifies form creation by using a unified component approach, reducing complexity and making form management more straightforward.

[click] Built-in Validation: Offers out-of-the-box validation rules, making it easier to ensure data integrity without extensive custom coding.

[click] Accessibility-focused: Prioritizes accessibility, ensuring forms are usable by everyone, including those with disabilities.

[click] Flexible Styling: FormKit allows for easy customization of form appearance, enabling developers to align forms with their design systems or branding guidelines.

[click] Decentralized Node System: Utilizes a unique, decentralized node system for managing form inputs, enhancing form performance and scalability.

[click] Extensible Schemas: Supports dynamic form generation through extensible schemas, allowing for complex form structures to be defined and reused.

[click] UI Framework Compatibility: Designed to work alongside any UI framework, FormKit provides versatility in development environments.

CODE EXAMPLE 

[click:5] In FormKit's architecture, nodes form the backbone of its system, representing inputs within a form. These nodes are interconnected in a tree-like structure, allowing for efficient and organized data and event management. This structure ensures that each node can operate independently while still communicating within the form's ecosystem, enhancing performance and customization.
 -->
 

---
title: PrimeVue & TresJs
---

# PrimeVue

UI suite for Vue.js consisting of a rich set of UI components

[Learn more](https://primevue.org/)

<br>
<v-click>

# TresJS

library to create WebGL 3D websites

[Learn more](https://tresjs.org/)

</v-click>

<!--
component library

created by PrimeTek a world-renowned vendor of popular UI Component suites, including PrimeFaces, PrimeNG, and PrimeReact

PrimeFaces has been maintained actively since 2008

version 4 of prime vue will be introduced this year

[click]

Build 3D scene as they were Vue components

It brings all the updated features of ThreeJS right away.

-->

---
layout: center
class: text-center
hideInToc: true
---

# Questions?
