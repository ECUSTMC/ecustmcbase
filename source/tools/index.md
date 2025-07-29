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

<script>
document.addEventListener('DOMContentLoaded', function() {
  // Alist API 配置
  const alistConfig = {
    baseUrl: 'https://ss.bestzyq.cn/', // Alist 实例地址
    basePath: '/MC/',  // 基础目录路径
    currentPath: '/MC/',  // 当前目录路径（会动态变化）
    password: '' // 如果需要密码，请在这里填写
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
    updatePathDisplay();
    await renderFileList();
  }

  // 渲染文件列表
  async function renderFileList() {
    const filesContainer = document.getElementById('alist-files');
    filesContainer.innerHTML = '加载中...';
    
    try {
      const files = await fetchAlistFiles(alistConfig.currentPath);
      
      if (files === null) {
        filesContainer.innerHTML = '此文件夹为空';
        return;
      }
      
      let html = '<table class="alist-table"><thead><tr><th>名称</th><th>大小</th><th>修改时间</th></tr></thead><tbody>';
      
      // 如果不是根目录，添加返回上级目录选项
      if (alistConfig.currentPath !== alistConfig.basePath) {
        const parentPath = alistConfig.currentPath.split('/').slice(0, -2).join('/') + '/';
        html += `
          <tr class="parent-dir">
            <td colspan="4">
              <span class="file-icon">↩</span>
              <a href="javascript:void(0)" class="folder-link" data-path="${parentPath}">返回上级目录</a>
            </td>
          </tr>
        `;
      }
      
      // 先显示文件夹，再显示文件
      const folders = files.filter(file => file.type === 1);
      const onlyFiles = files.filter(file => file.type !== 1);
      
      // 显示文件夹
      folders.forEach(folder => {
        const modTime = new Date(folder.modified).toLocaleString();
        const folderPath = alistConfig.currentPath + folder.name + '/';
        
        html += `
          <tr class="folder-row">
            <td>
              <span class="file-icon">📁</span>
              <a href="javascript:void(0)" class="folder-link" data-path="${folderPath}">${folder.name}</a>
            </td>
            <td>-</td>
            <td>${modTime}</td>
          </tr>
        `;
      });
      
      // 显示文件
      onlyFiles.forEach(file => {
        const fileSize = formatFileSize(file.size);
        const modTime = new Date(file.modified).toLocaleString();
        
        html += `
          <tr>
            <td>
              <span class="file-icon">📄</span>
              <a href="${alistConfig.baseUrl}d/Public${alistConfig.currentPath}${file.name}" target="_blank">${file.name}</a>
            </td>
            <td>${fileSize}</td>
            <td>${modTime}</td>
          </tr>
        `;
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
    } catch (error) {
      filesContainer.innerHTML = `加载失败: ${error.message}`;
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
</style>
