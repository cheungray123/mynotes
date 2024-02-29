
# Svelte Toast Component with Tailwind CSS

本文档提供了一个使用 Tailwind CSS 的 Svelte Toast 组件实现，以及相应的 store 和使用方法。

## Toast Store (toastStore.js)

这个 store 管理应用中的所有 toast 消息。

```javascript
import { writable } from 'svelte/store';

export const toasts = writable([]);

let nextBottom = 20;

export function addToast(message, type = 'info', duration = 3000) {
  const id = Date.now() + Math.floor(Math.random() * 100);
  const bottom = nextBottom;
  nextBottom += 60; // 假设每个 Toast 的高度加间距为 60px

  toasts.update(items => [...items, { id, message, type, duration, bottom }]);

  setTimeout(() => {
    removeToast(id);
  }, duration);
}

export function removeToast(id) {
  toasts.update(items => {
    let indexToRemove = items.findIndex(item => item.id === id);
    if (indexToRemove !== -1) {
      for (let i = indexToRemove + 1; i < items.length; i++) {
        items[i].bottom -= 60;
      }
      nextBottom -= 60;
    }
    return items.filter(item => item.id !== id);
  });
}
```

## Toast Component (Toast.svelte)

这个组件显示单个 toast 消息，并根据其类型使用不同的背景颜色。

```svelte
<script>
  export let message = '';
  export let type = 'info';
  export let bottom = 20;

  function getClass(type) {
    switch (type) {
      case 'info':
        return 'bg-blue-500';
      case 'success':
        return 'bg-green-500';
      case 'error':
        return 'bg-red-500';
      case 'warning':
        return 'bg-yellow-500';
      default:
        return 'bg-blue-500';
    }
  }
</script>

<div class={`fixed left-4 min-w-[200px] p-4 rounded-lg shadow-md text-white mb-4 ${getClass(type)}`} style={`bottom: ${bottom}px;`}>
  <p>{message}</p>
</div>
```

## 使用示例

在根组件中使用 `Toast` 组件并通过 `toastStore` 添加和移除消息。

### App.svelte

```svelte
<script>
  import { toasts } from './toastStore.js';
  import Toast from './Toast.svelte';
</script>

{#each $toasts as { id, message, type, duration, bottom }}
  <Toast {message} {type} {duration} {bottom} />
{/each}
```

### OtherComponent.svelte

在其他组件中触发 toast 消息。

```svelte
<script>
  import { addToast } from './toastStore.js';

  function showSuccessMessage() {
    addToast('操作成功！', 'success', 3000);
  }
</script>

<button on:click={showSuccessMessage}>显示成功消息</button>
```
