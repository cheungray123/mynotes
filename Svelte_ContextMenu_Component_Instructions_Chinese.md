
# Svelte 右键上下文菜单组件

## 概览
这份文档解释了如何使用 Svelte 右键上下文菜单组件。这个组件允许在右键点击时显示一个自定义菜单，并在点击菜单外部或组件销毁时自动关闭菜单，以防止内存泄漏。

## 组件代码

```svelte
<script>
    import { onDestroy, tick } from 'svelte';

    export let items = [];

    let open = false;
    let top = 0, left = 0;
    let contextMenu;

    function handleClickOutside(event) {
        if (contextMenu && !contextMenu.contains(event.target)) {
            hide();
        }
    }

    export async function show(event) {
        event.preventDefault();
        open = true;
        top = event.pageY;
        left = event.pageX;

        await tick();

        adjustPosition();

        window.addEventListener('click', handleClickOutside);
    }

    export function hide() {
        open = false;
        window.removeEventListener('click', handleClickOutside);
    }

    function adjustPosition() {
        if (contextMenu) {
            const menuWidth = contextMenu.offsetWidth;
            const menuHeight = contextMenu.offsetHeight;
            const windowWidth = window.innerWidth;
            const windowHeight = window.innerHeight;

            if (left + menuWidth > windowWidth) {
                left = windowWidth - menuWidth;
            }
            if (top + menuHeight > windowHeight) {
                top = windowHeight - menuHeight;
            }
        }
    }

    onDestroy(() => {
        if (open) {
            window.removeEventListener('click', handleClickOutside);
        }
    });
</script>

{#if open}
<nav bind:this={contextMenu}
     class="contextmenu"
     style="top: {top}px; left: {left}px">
    {#each items as item}
        {#if item == '---'}
            <hr/>
        {:else}
            <button on:click|stopPropagation={item.handler}>
                {item.label}
                {#if item.shortcut}
                    <span class="shortcut">{item.shortcut}</span>
                {/if}
            </button>
        {/if}
    {/each}
</nav>
{/if}
```

## 使用方法

1. **导入组件**：在父组件中导入 `ContextMenu.svelte`。
2. **传递菜单项**：向组件传递一个菜单项数组。每个菜单项应是一个包含 `label` 和 `handler` 的对象。
3. **绑定显示函数**：将 `ContextMenu` 的 `show` 函数绑定到需要显示上下文菜单的元素的 `contextmenu` 事件上。

### 示例

```svelte
<script>
    import ContextMenu from './ContextMenu.svelte';

    let items = [
        { label: '选项一', handler: () => console.log('选项一被点击') },
        { label: '选项二', handler: () => console.log('选项二被点击') },
        '---',
        { label: '选项三', handler: () => console.log('选项三被点击') }
    ];

    let contextMenu;

    function handleContextMenu(event) {
        contextMenu.show(event);
    }
</script>

<div on:contextmenu={handleContextMenu}>
    右键点击这里显示菜单
</div>

<ContextMenu bind:this={contextMenu} {items} />
```

在此示例中，用户在指定的 `div` 元素上右键点击时，上下文菜单会出现，显示所定义的菜单项。
