# 前言

上次在看前端早早聊大会中， `Vue` 成员Anthony Fu分享了一个 `VueUse` 的一个库。 好奇了一下，点看看了看。好家伙啊， 我直接好家伙。这不就是曾经我也想自己写一个  `vue` 版的 `hooks` 库吗？（因为我觉得 `vue3` 和 `hooks` 太像了） 可是我还不太会， 你现在直接把我的梦想给破灭了，下面我们一起来看看吧！

## 什么是 `VueUse`

`VueUse` 是一个基于 `Composition API` 的实用函数集合。通俗的来说，这就是一个工具函数包，它可以帮助你快速实现一些常见的功能，免得你自己去写，解决重复的工作内容。以及进行了机遇 `Composition API` 的封装。让你在 `vue3` 中更加得心应手。

## 简单上手

安装 `VueUse`

```js
npm i @vueuse/core
```

使用 `VueUse`

```js
// 导入
import { useMouse, usePreferredDark, useLocalStorage } from '@vueuse/core'

export default {
  setup() {
    // tracks mouse position
    const { x, y } = useMouse()

    // is user prefers dark theme
    const isDark = usePreferredDark()

    // persist state in localStorage
    const store = useLocalStorage(
      'my-storage',
      {
        name: 'Apple',
        color: 'red',
      },
    )

    return { x, y, isDark, store }
  }
}
```

上面从 `VueUse` 当中导入了三个函数， `useMouse`， `usePreferredDark`， `useLocalStorage`。`useMouse` 是一个监听当前鼠标坐标的一个方法，他会实时的获取鼠标的当前的位置。`usePreferredDark` 是一个判断用户是否喜欢深色的方法，他会实时的判断用户是否喜欢深色的主题。`useLocalStorage` 是一个用来持久化数据的方法，他会把数据持久化到本地存储中。

## 还有我们熟悉的 **防抖** 和 **节流**

```js
import { throttleFilter, debounceFilter, useLocalStorage, useMouse } from '@vueuse/core'

// 以节流的方式去改变 localStorage 的值
const storage = useLocalStorage('my-key', { foo: 'bar' }, { eventFilter: throttleFilter(1000) })

// 100ms后更新鼠标的位置
const { x, y } = useMouse({ eventFilter: debounceFilter(100) })
```

还有还有在 `component` 中使用的函数

```js
<script setup>
import { ref } from 'vue'
import { onClickOutside } from '@vueuse/core'

const el = ref()

function close () {
  /* ... */
}

onClickOutside(el, close)
</script>

<template>
  <div ref="el">
    Click Outside of Me
  </div>
</template>
```

上面例子中，使用了 `onClickOutside` 函数，这个函数会在点击元素外部时触发一个回调函数。也就是这里的 `close` 函数。在 `component` 中就是这么使用

```js
<script setup>
import { OnClickOutside } from '@vueuse/components'

function close () {
  /* ... */
}
</script>

<template>
  <OnClickOutside @trigger="close">
    <div>
      Click Outside of Me
    </div>
  </OnClickOutside>
</template>
```

> 注意⚠️ 这里的 `OnClickOutside` 函数是一个组件，不是一个函数。需要`package.json` 中安装了 `@vueuse/components`。

## 还还有全局状态共享的函数

```js
// store.js
import { createGlobalState, useStorage } from '@vueuse/core'

export const useGlobalState = createGlobalState(
  () => useStorage('vue-use-local-storage'),
)
```

```js
// component.js
import { useGlobalState } from './store'

export default defineComponent({
  setup() {
    const state = useGlobalState()
    return { state }
  },
})
```

这样子就是一个简单的状态共享了。扩展一下。传一个参数，就能改变 `store` 的值了。

## 还有关于 `fetch`, 下面👇就是一个简单的请求了。

```js
import { useFetch } from '@vueuse/core'

const { isFetching, error, data } = useFetch(url)
```
它还有很多的 `option` 参数，可以自定义。

```js
// 100ms超时
const { data } = useFetch(url, { timeout: 100 })
```

```js
// 请求拦截
const { data } = useFetch(url, {
  async beforeFetch({ url, options, cancel }) {
    const myToken = await getMyToken()

    if (!myToken)
      cancel()

    options.headers = {
      ...options.headers,
      Authorization: `Bearer ${myToken}`,
    }

    return {
      options
    }
  }
})
```

```js
// 响应拦截
const { data } = useFetch(url, {
  afterFetch(ctx) {
    if (ctx.data.title === 'HxH')
      ctx.data.title = 'Hunter x Hunter' // Modifies the resposne data

    return ctx
  },
})
```

## 更多

更多还的看[VueUse 文档](https://vueuse.org/)，还有另一个 [vue-demi](https://github.com/vueuse/vue-demi)