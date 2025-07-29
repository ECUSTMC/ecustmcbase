---
title: 工具
date: 2024-12-10 15:09:46
---
## 地毯（Carpet）
常见命令可见https://www.mcmod.cn/class/2361.html

### 假人
允许玩家打开假人背包（右击）、末影箱（潜行右击），假人可以autofish

#### bot_sleep（永昼机）
依次执行下面两条指令
```
/player bot_sleep spawn at -3200 55 9370 facing -90 0 in minecraft:overworld
```
```
/player bot_sleep use interval 20
```

## 下载

<div id="alist-nav">当前路径: <span id="current-path">/MC/</span></div>
<div id="alist-files">加载中...</div>
<div id="alist-pagination"></div>

<script>
document.addEventListener('DOMContentLoaded', function() {
  // Alist API 配置
  const alistConfig = {
    baseUrl: 'https://ss.bestzyq.cn/', // Alist 实例地址
    basePath: '/MC/',  // 基础目录路径
    currentPath: '/MC/',  // 当前目录路径（会动态变化）
    password: '', // 如果需要密码，请在这里填写
    itemsPerPage: 10, // 每页显示的项目数
    currentPage: 1 // 当前页码
  };

  // 更新导航路径显示
  function updatePathDisplay() {
    const pathDisplay = document.getElementById('current-path');
    pathDisplay.textContent = alistConfig.currentPath;
    
    // 创建面包屑导航
    // 移除基础路径前缀，避免重复显示
    const relativePath = alistConfig.currentPath.replace(alistConfig.basePath, '');
    const parts = relativePath.split('/').filter(p => p);
    
    // 始终显示根目录
    let breadcrumb = `<span class="breadcrumb-item" data-path="${alistConfig.basePath}">根目录</span>`;
    let currentPath = alistConfig.basePath;
    
    parts.forEach((part, index) => {
      if (index < parts.length - 1) { // 不是最后一个部分
        currentPath += part + '/';
        breadcrumb += ` > <span class="breadcrumb-item" data-path="${currentPath}">${part}</span>`;
      } else if (part) { // 最后一个部分（当前目录），确保不为空
        breadcrumb += ` > <span class="breadcrumb-current">${part}</span>`;
      }
    });
    
    pathDisplay.innerHTML = breadcrumb;
    
    // 为面包屑导航项添加点击事件
    document.querySelectorAll('.breadcrumb-item').forEach(item => {
      item.addEventListener('click', function() {
        const path = this.getAttribute('data-path');
        navigateToFolder(path);
      });
    });
  }

  // 调用 Alist API 获取文件列表
  async function fetchAlistFiles(path) {
    try {
      const response = await fetch(`${alistConfig.baseUrl}api/fs/list`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          path: path,
          password: alistConfig.password
        })
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }
      
      const data = await response.json();
      return data.data.content;
    } catch (error) {
      console.error('获取文件列表失败:', error);
      return [];
    }
  }

  // 导航到指定文件夹
  async function navigateToFolder(path) {
    alistConfig.currentPath = path;
    alistConfig.currentPage = 1; // 切换文件夹时重置到第一页
    updatePathDisplay();
    await renderFileList();
  }

  // 渲染分页控件
  function renderPagination(totalPages) {
    const paginationContainer = document.getElementById('alist-pagination');
    if (!paginationContainer) return;

    if (totalPages <= 1) {
      paginationContainer.innerHTML = '';
      return;
    }

    let paginationHtml = '';
    
    // 上一页按钮
    paginationHtml += `<button class="pagination-btn" data-page="${alistConfig.currentPage - 1}" ${alistConfig.currentPage === 1 ? 'disabled' : ''}>上一页</button>`;

    // 页码信息
    paginationHtml += `<span class="pagination-info">${alistConfig.currentPage} / ${totalPages}</span>`;

    // 下一页按钮
    paginationHtml += `<button class="pagination-btn" data-page="${alistConfig.currentPage + 1}" ${alistConfig.currentPage === totalPages ? 'disabled' : ''}>下一页</button>`;

    paginationContainer.innerHTML = paginationHtml;

    // 为分页按钮添加点击事件
    document.querySelectorAll('.pagination-btn').forEach(btn => {
      btn.addEventListener('click', function() {
        if (this.disabled) return;
        const page = parseInt(this.getAttribute('data-page'));
        alistConfig.currentPage = page;
        renderFileList();
      });
    });
  }

  // 渲染文件列表
  async function renderFileList() {
    const filesContainer = document.getElementById('alist-files');
    filesContainer.innerHTML = '加载中...';
    
    try {
      const allFiles = await fetchAlistFiles(alistConfig.currentPath);
      
      if (allFiles === null || allFiles.length === 0) {
        filesContainer.innerHTML = '此文件夹为空';
        renderPagination(0); // 清空分页
        return;
      }
      
      // 先显示文件夹，再显示文件
      const folders = allFiles.filter(file => file.type === 1);
      const onlyFiles = allFiles.filter(file => file.type !== 1);
      const sortedFiles = [...folders, ...onlyFiles];

      // 分页计算
      const totalItems = sortedFiles.length;
      const totalPages = Math.ceil(totalItems / alistConfig.itemsPerPage);
      const startIndex = (alistConfig.currentPage - 1) * alistConfig.itemsPerPage;
      const endIndex = startIndex + alistConfig.itemsPerPage;
      const paginatedFiles = sortedFiles.slice(startIndex, endIndex);

      let html = '<table class="alist-table"><thead><tr><th>名称</th><th>大小</th><th>修改时间</th></tr></thead><tbody>';
      
      // 如果不是根目录，并且在第一页，才显示返回上级目录
      if (alistConfig.currentPath !== alistConfig.basePath && alistConfig.currentPage === 1) {
        const parentPath = alistConfig.currentPath.split('/').slice(0, -2).join('/') + '/';
        html += `
          <tr class="parent-dir">
            <td colspan="3">
              <span class="file-icon">↩</span>
              <a href="javascript:void(0)" class="folder-link" data-path="${parentPath}">返回上级目录</a>
            </td>
          </tr>
        `;
      }
      
      paginatedFiles.forEach(file => {
        const modTime = new Date(file.modified).toLocaleString();
        if (file.type === 1) { // 文件夹
          const folderPath = alistConfig.currentPath + file.name + '/';
          html += `
            <tr class="folder-row">
              <td data-label="名称">
                <span class="file-icon">📁</span>
                <a href="javascript:void(0)" class="folder-link" data-path="${folderPath}">${file.name}</a>
              </td>
              <td data-label="大小">-</td>
              <td data-label="修改时间">${modTime}</td>
            </tr>
          `;
        } else { // 文件
          const fileSize = formatFileSize(file.size);
          html += `
            <tr>
              <td data-label="名称">
                <span class="file-icon">📄</span>
                <a href="${alistConfig.baseUrl}d/Public${alistConfig.currentPath}${file.name}" target="_blank">${file.name}</a>
              </td>
              <td data-label="大小">${fileSize}</td>
              <td data-label="修改时间">${modTime}</td>
            </tr>
          `;
        }
      });
      
      html += '</tbody></table>';
      filesContainer.innerHTML = html;
      
      // 为文件夹链接添加点击事件
      document.querySelectorAll('.folder-link').forEach(link => {
        link.addEventListener('click', function() {
          const path = this.getAttribute('data-path');
          navigateToFolder(path);
        });
      });

      renderPagination(totalPages);

    } catch (error) {
      filesContainer.innerHTML = `加载失败: ${error.message}`;
      renderPagination(0); // 清空分页
    }
  }

  // 格式化文件大小
  function formatFileSize(bytes) {
    if (bytes === 0) return '0 B';
    
    const sizes = ['B', 'KB', 'MB', 'GB', 'TB'];
    const i = Math.floor(Math.log(bytes) / Math.log(1024));
    
    return parseFloat((bytes / Math.pow(1024, i)).toFixed(2)) + ' ' + sizes[i];
  }

  // 初始化
  updatePathDisplay();
  renderFileList();
});
</script>

<style>
.alist-table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0;
}

.alist-table th, .alist-table td {
  padding: 8px 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.alist-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.alist-table tr:hover {
  background-color: #f5f5f5;
}

.file-icon {
  margin-right: 8px;
}

#alist-nav {
  margin-bottom: 15px;
  padding: 10px;
  background-color: #f8f8f8;
  border-radius: 4px;
  font-size: 14px;
}

.breadcrumb-item {
  color: #0366d6;
  cursor: pointer;
}

.breadcrumb-item:hover {
  text-decoration: underline;
}

.breadcrumb-current {
  font-weight: bold;
  color: #333;
}

.folder-link {
  color: #0366d6;
  text-decoration: none;
  cursor: pointer;
}

.folder-link:hover {
  text-decoration: underline;
}

.parent-dir {
  background-color: #f0f0f0;
}

.folder-row td {
  font-weight: 500;
}

#alist-pagination {
  margin-top: 20px;
  text-align: center;
}
.pagination-btn {
  padding: 8px 16px;
  margin: 0 5px;
  border: 1px solid #ddd;
  background-color: #f8f8f8;
  cursor: pointer;
  border-radius: 4px;
}
.pagination-btn:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}
.pagination-info {
  margin: 0 10px;
  font-weight: bold;
  display: inline-block;
  vertical-align: middle;
}

@media screen and (max-width: 768px) {
  .alist-table thead {
    display: none;
  }
  .alist-table, .alist-table tbody, .alist-table tr, .alist-table td {
    display: block;
    width: 100%;
    box-sizing: border-box;
  }
  .alist-table tr {
    margin-bottom: 15px;
    border: 1px solid #ddd;
    border-radius: 4px;
    overflow: hidden;
  }
  .alist-table td {
    border-bottom: 1px solid #eee;
  }
  .alist-table tr td:last-child {
    border-bottom: none;
  }
  .alist-table td:not([colspan]) {
    text-align: right;
    padding-left: 50%;
    position: relative;
  }
  .alist-table td:not([colspan])::before {
    content: attr(data-label);
    position: absolute;
    left: 12px;
    width: 45%;
    padding-right: 10px;
    white-space: nowrap;
    text-align: left;
    font-weight: bold;
  }
  .folder-link, .alist-table a {
    word-break: break-all;
  }
}
</style>
