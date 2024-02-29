
# Svelte书签管理器使用说明

## 代码说明

此代码示例展示了如何使用Svelte框架和Chrome扩展API来管理浏览器书签。它定义了一个Svelte存储，该存储提供了一种方式来存取和更新用户的书签数据。

### 引入Svelte的可写存储功能

```javascript
import { writable } from 'svelte/store';
```

### 定义一个创建书签存储的函数

```javascript
function createBookmarkStore() {
  // 使用Svelte的writable创建一个新的存储，初始化包含文件夹列表、选中的书签、选中的文件夹
  const { subscribe, set, update } = writable({
    folders: [],
    selectedBookmarks: [],
    selectedFolder: null,
  });
```

### 返回一个对象，包含subscribe方法以及定义的两个方法：loadBookmarks和selectFolder

```javascript
  return {
    subscribe,
```

#### 定义loadBookmarks方法，用于加载书签

```javascript
    loadBookmarks: () => {
      // 使用chrome.bookmarks.getTree方法获取书签树
      chrome.bookmarks.getTree(function(bookmarkTree) {
        const folders = [];
```

##### 定义extractFolders函数，递归提取书签中的文件夹和书签

```javascript
        function extractFolders(bookmarkNodes, parentTitle = '') {
          for (const node of bookmarkNodes) {
            if (node.children) {
              let folder = {
                title: node.title, // 文件夹标题
                id: node.id, // 文件夹ID
                parentTitle: parentTitle, // 父文件夹标题
                bookmarks: node.children.filter(child => !child.children).map(bookmark => ({
                  title: bookmark.title, // 书签标题
                  url: bookmark.url // 书签URL
                }))
              };
              folders.push(folder); // 将文件夹添加到folders数组中
              extractFolders(node.children, node.title); // 递归调用以处理所有子节点
            }
          }
        }
```

##### 更新存储后，自动选中第一个文件夹的书签

```javascript
        extractFolders(bookmarkTree);
        set({ folders, selectedBookmarks: [], selectedFolder: null });
        // 在set之后立即调用selectFolder，选中第一个文件夹
        if (folders.length > 0) {
            this.selectFolder(folders[0].id);
        }
```

#### 定义selectFolder方法，用于选择一个文件夹并更新选中的书签

```javascript
    selectFolder: (folderId) => {
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
  };
}
```

### 使用createBookmarkStore函数创建书签存储，并导出


export const bookmarkStore = createBookmarkStore();
```

## 使用方法

1. 将上述代码保存为JavaScript文件，例如`bookmarkStore.js`。
2. 在Svelte项目中引入并使用`bookmarkStore`来管理书签。
3. 使用`loadBookmarks`方法来加载当前浏览器中的书签。
4. 使用`selectFolder`方法来选中特定的书签文件夹并显示其内容。
5. 通过订阅`bookmarkStore`的`subscribe`方法来响应书签数据的变化，并在UI中相应地展示书签数据。


```javascript
function createBookmarkStore() {
  const { subscribe, set, update } = writable({
    folders: [],
    selectedBookmarks: [],
    selectedFolder: null,
  });

  return {
    subscribe,

    loadBookmarks: () => {
      chrome.bookmarks.getTree(function(bookmarkTree) {
        const folders = [];
        function extractFolders(bookmarkNodes, parentTitle = '') {
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
        }

        extractFolders(bookmarkTree);

        // 更新存储后，选中第一个文件夹的书签
        // 注意：这里假设folders数组至少有一个元素
        if (folders.length > 0) {
          const firstFolderId = folders[0].id; // 获取第一个文件夹的ID
          // 由于此时set操作是异步的，我们不能直接调用selectFolder方法
          // 需要在set操作内部调用selectFolder，确保folders已更新
          set({ folders, selectedBookmarks: [], selectedFolder: null }, () => {
            // 选中第一个文件夹，这里需要稍作修改以适应这种用法
            this.selectFolder(firstFolderId);
          });
        } else {
          // 如果没有文件夹，只设置folders为空
          set({ folders, selectedBookmarks: [], selectedFolder: null });
        }
      });
    },

    selectFolder: (folderId) => {
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
  };
}
```
