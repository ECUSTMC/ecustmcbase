---
title: 服务器
date: 2024-04-05 01:19:33
---

## 🏰 服务器
> ⚠️ 注意：进入需使用[皮肤站](https://mcskin.ecustvr.top/)认证，具体流程见[萌新指南](/tutorial/)。
- 🎮 版本：1.20.2-1.21.10均可进入
- ⏰ 开放时间：24h
- 🌐 服务器群组地址：`mc.ecustvr.top`
    - 🧩 mod服地址：`mcmod.ecustvr.top`
- 📋 介绍：设有原版服和模组服以及不定期开放的活动服务器
- 🔄 切换服务器：/server <服务器名>，例：`/server lobby`
- 🏫 切换其他高校服务器：/hub <学校代码[（点此查询）](https://docs.mualliance.cn/zh/dev/union/lobby)>，例如：`/server SJMC`

## 🗺️ 实时地图
- 🏃‍♂️ Lobby地图：[点击进入](http://mcmap.ecustvr.top/)
<!-- - 🔨 插件服（即将撤销）地图：[点击进入](http://out.ecust.cloud:25501/)
- 🏗️ 创造服（即将撤销）地图：[点击进入](http://out.ecust.cloud:25502/) -->

## 📊 在线状态
> 📊 由 [上海交通大学Minecraft社](https://mc.sjtu.cn/) 提供状态监测服务（离线情况不一定准确，可以多刷新几次）

<div id="serverStatus" class="server-status">
</div>

## 💻 技术介绍
详见[Minecraft高校联盟资料站](https://docs.mualliance.cn/)：[联合大厅](https://docs.mualliance.cn/zh/dev/union/lobby)、[联合认证](https://docs.mualliance.cn/zh/dev/union/auth)

### 🌐 接入点
> ⚠️ 非实时更新，详见[联合大厅](https://docs.mualliance.cn/zh/dev/union/lobby)

- 上交接入点：lobby.mualliance.cn / mua.sjmc.club （支持IPv6）
- 浙江接入点：hb.mualliance.cn
- 湖北接入点：imu.mualliance.cn / unions.imucraft.cn / unions6.imucraft.cn （IPv6）
- 四川接入点：taru.mualliance.cn / mcs.taru.xj.cn / union.mc.taru.xj.cn
- 北京接入点：bj.mualliance.cn / union-bgp.imucraft.cn
- 四川接入点2：union-sc.imucraft.cn

可输入接入点地址进入MUA大厅后，输入`/hub ECUST`进入ECUST群组服务器。

<!-- ### 🔄 备用地址
- 🏢 电信：`out.ecust.cloud`
- 📱 移动：`mch.ecustvr.top` -->

## 🙏 致谢
⛏️ 服务器现由 [百鬼Polaris](https://github.com/Polaris-Leo) 和 [文野](https://wenye.ecustvr.top/) 无偿提供！
✨ 特此亦向历任服主、维护者、建设和参与者表示衷心感谢！


<script>
document.addEventListener('DOMContentLoaded', function() {
  // 服务器配置
  const servers = [
    { address: 'mcs.ecustvr.top' },
    { address: 'gtnh.ecustvr.top' }
  ];

  // 生成服务器状态卡片
  const serverStatusContainer = document.getElementById('serverStatus');
  servers.forEach(server => {
    const card = document.createElement('article');
    card.className = 'post post-list-thumb post-list-show';
    card.dataset.server = server.address;
    
    card.innerHTML = `
      <div class="post-content">
        <div class="img">
          <img class="server-favicon" src="" alt="${server.address}">
        </div>
        <div>
          <div class="title-container">
            <h2 class="entry-title">加载中...</h2>
            <button class="refresh-button" title="刷新服务器状态">
              <i class="fa fa-sync"></i>
            </button>
          </div>
          <h2 class="entry-address">${server.address}</h2>
          <div class="post-meta">
            <div class="mcs-status">
              <span class="players"><i class="fa fa-user"></i>加载中...</span>
              <span class="ping"><i class="fa fa-stopwatch"></i>加载中...</span>
              <span class="version"><i class="fa fa-tag"></i>加载中...</span>
              <span class="time"><i class="fa fa-clock"></i>加载中...</span>
            </div>
          </div>
          <div class="online-players">
            <ul></ul>
          </div>
        </div>
      </div>
    `;
    
    serverStatusContainer.appendChild(card);
  });

  // 更新服务器状态
  const updateServerStatus = async (targetCard = null) => {
    const cards = targetCard ? [targetCard] : document.querySelectorAll('[data-server]');
    
    // 为所有卡片添加加载状态
    cards.forEach(card => {
      const refreshButton = card.querySelector('.refresh-button');
      if (refreshButton) {
        refreshButton.classList.add('loading');
        refreshButton.disabled = true;
      }
    });

    // 并行处理所有请求
    const updatePromises = Array.from(cards).map(async (card) => {
      const address = card.dataset.server;
      try {
        const response = await fetch('https://mcapi.ecustvr.top/custom/serverlist/?query=' + address);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        
        // 获取服务器描述
        let description = '无描述';
        if (data.description_raw) {
          if (typeof data.description_raw === 'string') {
            description = data.description_raw;
          } else {
            description = data.description_raw.text || data.description_raw.translate || data.description?.text || '无描述';
          }
        }
        if (description.includes('服务器已离线...')) {
          description = description.replace('...', '或查询失败');
        }

        // 更新时间戳
        const timestamp = new Date().toLocaleString('zh-CN', {
          year: 'numeric',
          month: '2-digit',
          day: '2-digit',
          hour: '2-digit',
          minute: '2-digit',
          second: '2-digit',
          hour12: false
        });

        if (data.online) {
          card.classList.remove('offline');
          // Update favicon
          if (data.favicon) {
            const favicon = card.querySelector('.server-favicon');
            favicon.src = data.favicon;
          }
          // Update title
          const title = card.querySelector('.entry-title');
          title.textContent = description;
          
          // Update player count
          const players = card.querySelector('.players');
          const playersOnline = data.players.online;
          const playersMax = data.players?.max || '未知';
          players.innerHTML = `<i class="fa fa-user"></i>${playersOnline}/${playersMax}`;
          
          // Update ping
          const ping = card.querySelector('.ping');
          ping.innerHTML = `<i class="fa fa-stopwatch"></i>${data.ping}ms`;

          // Update version
          const version = card.querySelector('.version');
          version.innerHTML = `<i class="fa fa-tag"></i>${data.version || '未知'}`;

          // Update timestamp
          const time = card.querySelector('.time');
          time.innerHTML = `<i class="fa fa-clock"></i>${timestamp}`;

          // Update online players
          const playersList = card.querySelector('.online-players ul');
          playersList.innerHTML = '';
          if (playersOnline > 0 && data.players?.sample) {
            data.players.sample.forEach(player => {
              const li = document.createElement('li');
              li.textContent = player.name;
              playersList.appendChild(li);
            });
          }
        } else {
          card.classList.add('offline');
          // Update title
          const title = card.querySelector('.entry-title');
          title.textContent = description;
          
          // Update status
          const players = card.querySelector('.players');
          players.innerHTML = '<i class="fa fa-user"></i>离线';
          const ping = card.querySelector('.ping');
          ping.innerHTML = '<i class="fa fa-stopwatch"></i>--';
          const version = card.querySelector('.version');
          version.innerHTML = '<i class="fa fa-tag"></i>--';
          const time = card.querySelector('.time');
          time.innerHTML = `<i class="fa fa-clock"></i>${timestamp}`;
          
          // Clear online players
          const playersList = card.querySelector('.online-players ul');
          playersList.innerHTML = '';
        }
      } catch (error) {
        console.error('Error fetching server status:', error);
        card.classList.add('offline');
        // Update error state
        const title = card.querySelector('.entry-title');
        title.textContent = '查询失败';
        const players = card.querySelector('.players');
        players.innerHTML = '<i class="fa fa-user"></i>未知';
        const ping = card.querySelector('.ping');
        ping.innerHTML = '<i class="fa fa-stopwatch"></i>--';
        const version = card.querySelector('.version');
        version.innerHTML = '<i class="fa fa-tag"></i>--';
        const time = card.querySelector('.time');
        time.innerHTML = `<i class="fa fa-clock"></i>${new Date().toLocaleString('zh-CN')}`;
        const playersList = card.querySelector('.online-players ul');
        playersList.innerHTML = '';
      }
    });

    // 等待所有请求完成
    await Promise.all(updatePromises).finally(() => {
      // 重置所有刷新按钮状态
      cards.forEach(card => {
        const refreshButton = card.querySelector('.refresh-button');
        if (refreshButton) {
          refreshButton.classList.remove('loading');
          refreshButton.disabled = false;
        }
      });
    });
  };

  // Add click event listeners to refresh buttons
  document.querySelectorAll('.refresh-button').forEach(button => {
    button.addEventListener('click', async (e) => {
      e.preventDefault();
      if (!button.disabled) {
        const card = button.closest('[data-server]');
        if (card) {
          await updateServerStatus(card);
        }
      }
    });
  });

  // Initial update
  updateServerStatus();
  
  // Update every 60 seconds
  setInterval(() => updateServerStatus(), 60000);
});
</script>
<style>
/* Server Status Cards Styles */
.server-status {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  margin: 20px 0;
}

.server-status .post-list-thumb {
  margin: 0;
  transition: all 0.3s ease;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.server-status .post-list-thumb:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0,0,0,0.15);
}

.server-status .post-content {
  display: flex;
  padding: 15px;
  gap: 15px;
}

.server-status .img {
  width: 64px;
  height: 64px;
  flex-shrink: 0;
}

.server-status .img img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 8px;
}

.server-status .entry-title {
  margin: 0 0 0px;
  font-size: 1.2em;
  color: #333;
}

.server-status .entry-address {
  margin: 0 0 5px;
  font-size: 1.2em;
  color: #9e9e9e;  /* 更柔和的灰色 */
  font-weight: 300;  /* 更细的字体 */
  letter-spacing: 0.5px;  /* 轻微字距调整 */
}

.server-status .mcs-status {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  font-size: 0.9em;
  color: #666;
}

.server-status .mcs-status span {
  display: inline-flex;
  align-items: center;
  gap: 5px;
}

.server-status .mcs-status i {
  font-size: 1em;
  width: 16px;
  text-align: center;
}

/* Online players list */
.server-status .online-players {
  margin-top: 10px;
  font-size: 0.85em;
  color: #666;
}

.server-status .online-players ul {
  list-style: none;
  padding: 0;
  margin: 5px 0 0 0;
}

.server-status .online-players li {
  display: inline-block;
  margin-right: 10px;
  background: #f5f5f5;
  padding: 2px 8px;
  border-radius: 12px;
}

/* Offline state */
.server-status .offline {
  opacity: 0.7;
  filter: grayscale(1);
}

.server-status .offline .entry-title {
  color: #999;
}

/* Loading state */
.server-status [data-server] {
  position: relative;
}

.server-status [data-server]::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255,255,255,0.8);
  display: none;
}

.server-status [data-server].loading::before {
  display: block;
}

/* Dark mode support */
@media (prefers-color-scheme: dark) {
  .server-status .post-list-thumb {
    background: #2d2d2d;
  }
  
  .server-status .entry-title {
    color: #e1e1e1;
  }
  
  .server-status .mcs-status {
    color: #999;
  }
  
  .server-status .offline .entry-title {
    color: #666;
  }

  .server-status .online-players li {
    background: #3d3d3d;
    color: #e1e1e1;
  }
}

/* Refresh Button Styles */
.server-status .title-container {
  display: flex;
  align-items: center;
  gap: 10px;
}

.server-status .refresh-button {
  background: none;
  border: none;
  color: #666;
  cursor: pointer;
  padding: 5px;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.server-status .refresh-button:hover {
  background-color: rgba(0, 0, 0, 0.05);
  color: #333;
  transform: scale(1.1);
}

.server-status .refresh-button:active {
  transform: scale(0.95);
}

.server-status .refresh-button.loading i {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Dark mode support for refresh button */
@media (prefers-color-scheme: dark) {
  .server-status .refresh-button {
    color: #999;
  }
  
  .server-status .refresh-button:hover {
    background-color: rgba(255, 255, 255, 0.1);
    color: #e1e1e1;
  }
}

/* Mobile responsive */
@media screen and (max-width: 768px) {
  .server-status {
    grid-template-columns: 1fr;
  }
  
  .server-status .post-content {
    padding: 10px;
  }
  
  .server-status .img {
    width: 48px;
    height: 48px;
  }
  
  .server-status .entry-title {
    font-size: 1.1em;
  }
  
  .server-status .mcs-status {
    font-size: 0.85em;
  }
}
</style>