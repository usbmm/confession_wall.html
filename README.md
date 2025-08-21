<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>炳公学校表白墙</title>
  <style>
    /* 基础样式 - 适配微信端 */
    * { margin: 0; padding: 0; box-sizing: border-box; font-family: -apple-system, BlinkMacSystemFont, "Helvetica Neue", Helvetica, Arial, sans-serif; }
    body { background-color: #f8f9fa; color: #333; padding: 20px; }
    .container { max-width: 500px; margin: 0 auto; }
    .header { text-align: center; margin-bottom: 30px; }
    .header h1 { color: #ff4d6d; font-size: 28px; margin-bottom: 10px; }
    .header p { color: #666; font-size: 14px; }
    .post-form { background: #fff; border-radius: 12px; padding: 20px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); margin-bottom: 20px; }
    .form-group { margin-bottom: 15px; }
.表单组标签{ 显示: 块; 利润-底部: 8px; 字体大小: 14px; 颜色: #555; }
.form-group textarea{ 宽度: 100%; 填料: 12px; border: 1px solid #ddd; border-radius: 8px; font-size: 16px; height: 120px; resize: none; }
    .form-group .nickname-row { display: flex; gap: 10px; }
    .form-group .nickname-row input { flex: 1; padding: 10px; border: 1px solid #ddd; border-radius: 8px; font-size: 16px; }
    .form-group .anonymous-checkbox { display: flex; align-items: center; gap: 8px; }
    .form-btn { width: 100%; padding: 12px; background-color: #ff4d6d; color: white; border: none; border-radius: 8px; font-size: 16px; cursor: pointer; transition: background-color 0.3s; }
    .form-btn:hover { background-color: #ff2d55; }
    .posts-container { background: #fff; border-radius: 12px; padding: 20px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); }
    .posts-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
    .posts-header h2 { color: #333; font-size: 20px; }
    .posts-header span { background-color: #ff4d6d; color: white; padding: 3px 8px; border-radius: 4px; font-size: 14px; }
    .post-item { padding: 15px; border-bottom: 1px solid #f0f0f0; }
    .post-item:last-child { border-bottom: none; }
    .post-content { font-size: 16px; line-height: 1.6; margin-bottom: 10px; padding: 12px; background-color: #f9f9f9; border-radius: 8px; }
    .post-meta { display: flex; justify-content: space-between; align-items: center; font-size: 14px; color: #666; }
    .post-meta .nickname { font-weight: 500; }
    .post-meta .time { color: #999; }
    .empty-tip { text-align: center; padding: 20px; color: #999; font-size: 14px; }
    .loading { text-align: center; padding: 20px; color: #999; font-size: 14px; }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>炳公学校表白墙</h1>
      <p>勇敢说出你的心意，让青春不留遗憾～</p>
    </div>
    
    <!-- 发布表单 -->
    <div class="post-form">
      <div class="form-group">
        <label for="content">表白内容</label>
        <textarea id="content" placeholder="在这里写下你想对TA说的话... (支持emoji哦)" required></textarea>
      </div>
      <div class="form-group">
        <div class="nickname-row">
          <input type="text" id="nickname" placeholder="你的昵称" value="匿名用户">
          <div class="anonymous-checkbox">
            <input type="checkbox" id="anonymous" checked>
            <label for="anonymous">匿名</label>
          </div>
        </div>
      </div>
      <button id="postBtn" class="form-btn">发布表白</button>
    </div>
    
    <!-- 表白列表 -->
    <div class="posts-container">
      <div class="posts-header">
        <h2>最新表白</h2>
        <span id="postCount">0 条</span>
      </div>
      <div id="posts">
        <div class="empty-tip">暂无表白内容，快来留下你的心意吧～</div>
      </div>
    </div>
  </div>

  <script src="https://unpkg.com/leancloud-storage@4.5.0/dist/av-min.js"></script>
  <script>
    // 初始化LeanCloud存储（用的测试配置，你后续可自行注册替换）
    AV.init({
      appId: "vM9X58YrHcQ1T0aSd7h7kGkD-gzGzoHsz",
      appKey: "9y7C8Y0m9z7d6S5H2QrJk8Tf",
      serverURL: "https://vm9x58yr.lc-cn-n1-shared.com"
    });
    
    // 定义表白数据模型
    const Confession = AV.Object.extend('BinggongConfession');
    
    // 发布表白
    document.getElementById('postBtn').addEventListener('click', function() {
      const content = document.getElementById('content').value.trim();
      if (!content) {
        alert('请输入表白内容哦～');
        return;
      }
      
      const isAnonymous = document.getElementById('anonymous').checked;
      let nickname = document.getElementById('nickname').value.trim();
      if (isAnonymous || !nickname) {
        nickname = '匿名同学';
      }
      
      // 创建表白对象
      const confession = new Confession();
      confession.set('content', content);
      confession.set('nickname', nickname);
      confession.set('time', new Date());
      confession.set('school', '炳公学校'); // 学校标识
      
      // 保存到云端
      confession.save().then(function() {
        alert('表白发布成功！');
        document.getElementById('content').value = '';
        // 刷新列表
        loadConfessions();
      }, function(error) {
        alert('发布失败，请稍后再试：' + error.message);
      });
    });
    
    // 切换匿名状态时自动填充昵称
    document.getElementById('anonymous').addEventListener('change', function() {
      if (this.checked) {
        document.getElementById('nickname').value = '匿名同学';
        document.getElementById('nickname').disabled = true;
      } else {
        document.getElementById('nickname').value = '';
        document.getElementById('nickname').disabled = false;
      }
    });
    
    // 加载所有表白内容
    function loadConfessions() {
      const posts = document.getElementById('posts');
      posts.innerHTML = '<div class="loading">加载中...</div>';
      
      const query = new AV.Query(Confession);
      query.equalTo('school', '炳公学校'); // 只查询本校表白
      query.descending('time'); // 按时间倒序排列
      query.find().then(function(confessions) {
        posts.innerHTML = '';
        const countElement = document.getElementById('postCount');
        countElement.textContent = confessions.length + ' 条';
        
        if (confessions.length === 0) {
          posts.innerHTML = '<div class="empty-tip">暂无表白内容，快来留下你的心意吧～</div>';
          return;
        }
        
        confessions.forEach(function(confession) {
          const content = confession.get('content');
          const nickname = confession.get('nickname');
          const time = new Date(confession.get('time')).toLocaleString('zh-CN', {
            year: 'numeric',
            month: '2-digit',
            day: '2-digit',
            hour: '2-digit',
            minute: '2-digit'
          });
          
          const postDiv = document.createElement('div');
          postDiv.className = 'post-item';
          postDiv.innerHTML = `
<div class="post-content">${<div class="post-content">${content}</div>}</div>
<div class="post-meta">
<span class="昵称">${<span class=“昵称”>${昵称}</span>}</span>
<span class="time">${<span class="time">${time}</span>}</span>
</div>
          `          `;
posts.appendChild（postDiv）；appendChild(postDiv);
        });});
}，函数（错误） {}, 功能(错误) {
加载失败，请稍后再试~</div>；inner HTML='<div类="空提示">加载失败，请稍后再试~《/div>'；
错误（"加载表白失败："，错误）；错误（'加载表白失败：'，错误）；
      });});
    }}
    
    // 页面加载时自动加载表白列表，并设置每30秒刷新一次// 页面加载时自动加载表白列表，并设置每30秒刷新一次
windows.onload = function（） {onload 是函数（） {
loadConfessions（）；loadConfessions();
设置间隔（加载协议，30000）；//30秒刷新一次设置间隔（加载协议，30000）；//30秒刷新一次
    };};
    
    // 优化微信端体验：禁止长按选中文字// 优化微信端体验：禁止长按选中文字
事件监听器（'上下文菜单'，功能（e）{受控者（"上下文菜单"，功能（e）{
防止违约（）；防止违约（）；
假的；假的；假的；假的；
  </script>/脚本>
</<>
</<>
