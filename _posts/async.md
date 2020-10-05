---
title: 处理异步请求的实践
date: 2020-09-05 16:20:37
tags: javascript
---

在小米实习的一点小小收货总结下来！关于异步请求的方案总结

在一般情况下，我们调用异步方法后会有某些标识用于提示用户。比如表单点击确定后按钮会变成`loading`或`disabled`的状态，请求成功或失败总会有些提示给到用户。

首先我们声明一个示例异步函数

<!-- more -->

```js
// 一个一秒内返回的异步函数
// 二分之一的几率成功返回随机字符串
const asyncFunction = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const r = Math.random();
      if (r < 0.5) {
        resolve(Math.random().toString(16).slice(-8));
      } else {
        reject("Something went wrong");
      }
    }, 1000);
  });
}; 
```

## 传统解决方案

使用当下流行的声明式UI框架，我们需要为每个异步请求提供标志状态。

```js
class App extends React.Component {
  constructor() {
    super();
    this.state = {
      title: {
        loading: false,
        value: null,
        error: null,
      },
      author: {
        loading: false,
        value: null,
        error: null,
      },
    };
  }

  async fetchTitle() {
    const { title } = this.state;
    try {
      title.loading = true;
      title.value = null;
      title.error = null;
      this.setState({ title });
      title.value = await asyncFunction();
      this.setState({ title });
    } catch (e) {
      title.error = e;
      this.setState({ title });
    } finally {
      title.loading = false;
      this.setState({ title });
    }
  }

  async fetchAuthor() {
    const { author } = this.state;
    
    try {
      author.loading = true;
    	author.value = null;
    	author.error = null;
      this.setState({ author });
      author.value = await asyncFunction();
      this.setState({ author });
    } catch (e) {
      author.error = e;
      this.setState({ author });
    } finally {
      author.loading = false;
      this.setState({ author });
    }
  }

  componentWillMount() {
    this.fetchTitle();
  }

  render() {
    const { title, author } = this.state;
    return (
      <div>
        {/* Title */}
        {title.error ? (
          <p style={{ color: "red" }}>{title.error}</p>
        ) : (
          <h1>Title: {title.loading ? "Loading..." : title.value}</h1>
        )}

        {/* Author */}
        {author.error ? (
          <p style={{ color: "red" }}>{author.error}</p>
        ) : (
          <p>Author: {author.loading ? "Loading..." : author.value}</p>
        )}

        <div>
          <button
            onClick={this.fetchAuthor.bind(this)}
            disabled={author.loading}
          >
            Load Author
          </button>
        </div>
      </div>
    );
  }
}
```

代码很长不想看？是的，确实很长，但却是最简单的逻辑，仅仅是**调用异步方法并显示数据**而已。长久以来，我们不得不为异步数据添加额外的标志状态用于提示用户，这导致我们的代码又长又臭。而且，每个异步方法的调用方式都是相同的，如代码中的`fetchTitle`和`fetchAuthor`，我们不得已一遍又一遍地复制粘贴相同的代码。

为什么不封装起来复用这些逻辑？究其原因，我们调用异步方法以后需要修改的不是普通变量而是组件的状态，纯逻辑复用难，是所有框架的硬伤。封装为组件是可行的，但会失去灵活性，React 还好，在 Vue template 中写逻辑你可能会抓狂。

#### React Hooks

Hooks的出现带来了一种编程思想的转变，只要使用得当，它可以帮助我们解决几乎任何逻辑复用的问题。

```js
import { useState, useEffect, useCallback } from 'React'

const useAsync = (asyncFunc, immediate = true) => {
  const [pending, setPending] = useState(0);
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  const execute = useCallback(() => {
    setPending(val => val + 1);
    setValue(null);
    setError(null);
    return asyncFunc()
      .then(response => setValue(response))
      .catch(error => setError(error))
      .finally(() => setPending(val => val - 1));
  }, [asyncFunc]);

  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return { execute, value, error, pending: pending > 0 };
};
```

这实在是一种很美妙的复用方式，现在我们可以这样来声明状态

```js
export default function App() {
  const title = useAsync(asyncFunction);
  const author = useAsync(asyncFunction, false);
  return (
    <div>
      {/* Title */}
      {title.error ? (
        <p style={{ color: "red" }}>{title.error}</p>
      ) : (
        <h1>Title: {title.pending ? "Loading..." : title.value}</h1>
      )}

      {/* Author */}
      {author.error ? (
        <p style={{ color: "red" }}>{author.error}</p>
      ) : (
        <p>Author: {author.pending ? "Loading..." : author.value}</p>
      )}
      <button onClick={author.execute}>Load Author</button>
    </div>
  );
}
```

再也不需要声明烦人的标志位了，也不需要手动`try catch`了，都交给`useAsync`解决。`useAsync`允许立即调用一次异步函数，也可以手动调用`execute`函数，这保证了灵活性。

## Composition API

截至2020年4月7日，Vue 3.x依然未能发布正式版本，不过我们可以通过插件的方式，在Vue 2.x中尝鲜Composition API。

```js
import { ref, computed, reactive } from "@vue/composition-api";

function useAsync(asyncFunc, immediate = true) {
  const pendingCount = ref(0);
  const pending = computed(() => pendingCount.value > 0);
  const value = ref(null);
  const error = ref(null);

  function execute() {
    pendingCount.value++;
    value.value = null;
    error.value = null;

    return asyncFunc()
      .then(res => (value.value = res))
      .catch(err => (error.value = err))
      .finally(() => pendingCount.value--);
  }

  if (immediate) {
    execute();
  }

  return reactive({
    pending,
    value,
    error,
    execute
  });
}
```

在组件中使用它，同样很简单

```vue
<template>
  <div>
    <p style="color: red" v-if="title.error">{{title.error}}</p>
    <h1 v-else>Title: {{title.pending ? 'Loading...':title.value}}</h1>
    <p style="color: red" v-if="author.error">{{author.error}}</p>
    <p v-else>Author: {{author.pending ? 'Loading...':author.value}}</p>
    <div>
      <button @click="author.execute">Load Author</button>
    </div>
  </div>
</template>

<script>
import { defineComponent } from "@vue/composition-api";
import { useAsync, asyncFunction } from "./utils";

export default defineComponent({
  setup() {
    const title = useAsync(asyncFunction);
    const author = useAsync(asyncFunction, false);

    return {
      title,
      author
    };
  }
});
</script>

```

当然，我们也可以自定义或扩展能力，比如把`value`设置为`response.data`，把error设置为`response.message`，截获错误的时候提示错误对话框等等，在不同的项目中扩展不同的能力，是比较方便的。