
# Svelte 书签管理存储代码

此文档提供了一个Svelte存储的完整代码，该存储用于管理书签，包括添加、删除和更新书签的功能。

## 存储定义

```javascript
import { writable } from 'svelte/store';

// 创建一个书签存储的函数
function createBookmarkStore() {
    // 使用Svelte的writable创建一个可写存储，初始化包含文件夹列表、选中的书签、选中的文件夹
    const { subscribe, set, update } = writable({
        folders: [], // 文件夹列表
        selectedBookmarks: [], // 选中的书签
        selectedFolder: null, // 选中的文件夹
    });

    return {
        subscribe,

        // 添加新书签或文件夹
        // bookmark: 新书签或文件夹对象，folderId: 目标文件夹的ID
        addBookmark: (bookmark, folderId) => {
            update(store => {
                // 遍历文件夹列表，找到目标文件夹并将新书签添加到该文件夹的书签列表中
                const updatedFolders = store.folders.map(folder => {
                    if (folder.id === folderId) {
                        return { ...folder, bookmarks: [...folder.bookmarks, bookmark] };
                    }
                    return folder;
                });
                return { ...store, folders: updatedFolders };
            });
        },

        // 删除书签
        // bookmarkId: 要删除的书签的ID，folderId: 书签所在文件夹的ID
        removeBookmark: (bookmarkId, folderId) => {
            update(store => {
                // 遍历文件夹列表，找到目标文件夹并从该文件夹的书签列表中移除指定的书签
                const updatedFolders = store.folders.map(folder => {
                    if (folder.id === folderId) {
                        const updatedBookmarks = folder.bookmarks.filter(bookmark => bookmark.id !== bookmarkId);
                        return { ...folder, bookmarks: updatedBookmarks };
                    }
                    return folder;
                });
                return { ...store, folders: updatedFolders };
            });
        },

        // 更新书签信息
        // bookmarkId: 要更新的书签的ID，folderId: 书签所在文件夹的ID，updatedInfo: 包含更新信息的对象
        updateBookmark: (bookmarkId, folderId, updatedInfo) => {
            update(store => {
                // 遍历文件夹列表，找到目标文件夹并更新该文件夹的书签列表中指定书签的信息
                const updatedFolders = store.folders.map(folder => {
                    if (folder.id === folderId) {
                        const updatedBookmarks = folder.bookmarks.map(bookmark => {
                            if (bookmark.id === bookmarkId) {
                                return { ...bookmark, ...updatedInfo };
                            }
                            return bookmark;
                        });
                        return { ...folder, bookmarks: updatedBookmarks };
                    }
                    return folder;
                });
                return { ...store, folders: updatedFolders };
            });
        },
    };
}

export default createBookmarkStore;
```

## 使用方法

在您的Svelte组件中使用这个存储：

1. 导入存储：`import createBookmarkStore from './path/to/store';`
2. 创建实例：`const bookmarkStore = createBookmarkStore();`
3. 使用存储的方法来管理书签。

请根据您项目的实际路径替换导入语句中的路径。
