# Svelte书签管理器存储

这个Svelte存储实现了一个高级书签管理功能，包括书签的加载、选择、保存，以及与Chrome书签事件的同步。

## 使用方法

首先，导入Svelte的可写存储功能。

```js
import { writable } from 'svelte/store';
```

然后，定义一个创建书签存储的函数`createBookmarkStore`。

```js
function createBookmarkStore() {
    const { subscribe, set, update } = writable({
        folders: [],
        selectedBookmarks: [],
        selectedFolder: null,
    });

    function extractFolders(bookmarkNodes, parentTitle = '') {
        // 递归提取书签中的文件夹和书签
        const folders = [];
        for (const node of bookmarkNodes) {
            if (node.children) {
                let folder = {
                    title: node.title,
                    id: node.id,
                    parentTitle: parentTitle,
                    bookmarks: node.children.filter(child => !child.children).map(bookmark => ({
                        title: bookmark.title,
                        url: bookmark.url
                    }))
                };
                folders.push(folder);
                extractFolders(node.children, node.title);
            }
        }
        return folders;
    }

    function addCustomFields(bookmarkNodes) {
        // 在保存到本地存储前为书签数据添加自定义字段
        bookmarkNodes.forEach(node => {
            if (node.children) {
                node.icon = 'folder_icon.png'; 
                node.backgroundColor = '#F0F0F0'; 
                addCustomFields(node.children);
            } else {
                node.icon = 'bookmark_icon.png'; 
                node.backgroundColor = '#FFFFFF'; 
            }
        });
    }

    return {
        subscribe,
        loadBookmarks: () => {
            // 从chrome.storage.local加载书签数据
            chrome.storage.local.get(["bookmarkData"], function(result) {
                if (result.bookmarkData && result.bookmarkData.length > 0) {
                    const folders = extractFolders(result.bookmarkData);
                    set({ folders, selectedBookmarks: [], selectedFolder: null });
                }
            });
        },
        selectFolder: (folderId) => {
            // 选择特定的书签文件夹并更新选中的书签列表
            update(store => {
                const folder = store.folders.find(f => f.id === folderId);
                if (folder) {
                    return {
                        ...store,
                        selectedBookmarks: folder.bookmarks,
                        selectedFolder: folder.title,
                    };
                }
                return store;
            });
        },
        saveBookmarks: (bookmarkTree) => {
            // 保存书签数据到chrome.storage.local
            addCustomFields(bookmarkTree);
            chrome.storage.local.set({bookmarkData: bookmarkTree}, function() {
                console.log('书签数据及自定义字段已保存到本地存储');
            });
        },
    };
}
```

接下来，定义初始化书签同步的函数`initBookmarkSync`。

```js
function initBookmarkSync(bookmarkStore) {
    function fetchAndStoreBookmarks() {
        // 获取并保存书签数据
        chrome.bookmarks.getTree(function(bookmarkTree) {
            bookmarkStore.saveBookmarks(bookmarkTree);
            bookmarkStore.loadBookmarks();
        });
    }

    // 监听书签的创建、移除、变更和移动事件
    chrome.bookmarks.onCreated.addListener(fetchAndStoreBookmarks);
    chrome.bookmarks.onRemoved.addListener(fetchAndStoreBookmarks);
    chrome.bookmarks.onChanged.addListener(fetchAndStoreBookmarks);
    chrome.bookmarks.onMoved.addListener(fetchAndStoreBookmarks);

    fetchAndStoreBookmarks();
}
```

最后，创建书签存储实例并初始化书签同步。

```js
const bookmarkStore = createBookmarkStore();
initBookmarkSync(bookmarkStore);
```

## 功能描述

- `loadBookmarks`: 从`chrome.storage.local`加载书签数据。
- `selectFolder`: 根据文件夹ID选择一个书签文件夹。
- `saveBookmarks`: 将书签数据（包括自定义字段）保存到`chrome.storage.local`。
- `initBookmarkSync`: 初始化书签同步，监听书签变化事件并更新存储。

这个存储简化了书签数据的管理过程，使得在Svelte应用中集成和利用书签功能变得简单直接。
