# Gemini 生图自动化实现

使用 MCP Playwright 自动化操作 Gemini 官网进行免费生图。

## 前置条件

```bash
# 确认 MCP Playwright 已安装
claude mcp list | grep playwright

# 预期输出
# playwright@claude-plugins-official: connected ✓
```

## 核心流程

### Step 1: 打开 Gemini 官网

```javascript
// 导航到 Gemini
mcp__playwright__browser_navigate({
  url: "https://gemini.google.com/app"
});

// 检查登录状态
// 如果未登录，提示用户登录
```

### Step 2: 检测登录状态

```javascript
// 检查是否需要登录
const currentUrl = await page.url();

if (currentUrl.includes('accounts.google.com')) {
  // 需要登录
  return {
    status: 'login_required',
    message: '请在浏览器中完成 Google 登录，完成后告诉我"已登录"'
  };
}
```

### Step 3: 输入生图提示词

```javascript
// 定位输入框
mcp__playwright__browser_click({
  element: "输入框/文本区域",
  ref: "textarea_ref"
});

// 输入生图指令（需要加上生图前缀）
const prompt = `生成一张图片：${userPrompt}`;

mcp__playwright__browser_type({
  element: "输入框",
  ref: "textarea_ref",
  text: prompt
});

// 发送
mcp__playwright__browser_click({
  element: "发送按钮",
  ref: "send_button_ref"
});
```

### Step 4: 等待图片生成

```javascript
// 等待生成完成（通常 10-30 秒）
// 检测图片元素出现

mcp__playwright__browser_run_code({
  code: `async (page) => {
    // 等待图片生成完成
    await page.waitForSelector('img[src*="generated"]', {
      timeout: 60000  // 最长等待 60 秒
    });
    return 'Image generated';
  }`
});
```

### Step 5: 下载图片

```javascript
// 方法 1: 右键保存（推荐）
mcp__playwright__browser_click({
  element: "生成的图片",
  ref: "generated_image_ref",
  button: "right"  // 右键
});

// 点击"另存为"
mcp__playwright__browser_click({
  element: "保存图片",
  ref: "save_image_option"
});

// 方法 2: 提取图片 URL 后下载
mcp__playwright__browser_run_code({
  code: `async (page) => {
    const imgElement = await page.waitForSelector('img[src*="generated"]');
    const imageUrl = await imgElement.getAttribute('src');
    return imageUrl;
  }`
});

// 使用 fetch 下载到本地
```

### Step 6: 保存到当前目录

```javascript
// 生成文件名
const timestamp = new Date().toISOString().replace(/[:.]/g, '').slice(0, 15);
const sanitizedDesc = description.replace(/[^a-zA-Z0-9]/g, '_').slice(0, 30);
const filename = `${timestamp}_${sanitizedDesc}.png`;

// 保存路径
const savePath = `./images/${filename}`;

// 确保目录存在
await fs.mkdir('./images', { recursive: true });

// 下载并保存
// ... (通过 page.screenshot 或 fetch + fs.writeFile)
```

## 完整自动化脚本

```javascript
async function generateImageWithGemini(prompt, options = {}) {
  const {
    outputDir = './images',
    timeout = 60000,
    retries = 2
  } = options;

  // 1. 确保输出目录存在
  await ensureDir(outputDir);

  // 2. 打开 Gemini
  await navigateToGemini();

  // 3. 检查登录
  const loginStatus = await checkLogin();
  if (!loginStatus.loggedIn) {
    return {
      status: 'error',
      message: 'Please login to Google first'
    };
  }

  // 4. 生成图片
  const imagePrompt = `Generate an image: ${prompt}`;
  await inputPrompt(imagePrompt);
  await clickSend();

  // 5. 等待生成
  const imageUrl = await waitForImage(timeout);

  // 6. 下载保存
  const filename = generateFilename(prompt);
  const savedPath = await downloadImage(imageUrl, outputDir, filename);

  return {
    status: 'success',
    path: savedPath,
    filename: filename
  };
}
```

## 错误处理

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 登录失效 | Cookie 过期 | 重新登录 Google |
| 生成超时 | 网络慢/服务繁忙 | 增加超时时间，重试 |
| 图片未找到 | DOM 结构变化 | 更新选择器 |
| 下载失败 | 跨域限制 | 使用截图方式 |

### 重试机制

```javascript
async function generateWithRetry(prompt, maxRetries = 2) {
  for (let i = 0; i <= maxRetries; i++) {
    try {
      return await generateImageWithGemini(prompt);
    } catch (error) {
      if (i === maxRetries) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}...`);
      await sleep(2000 * (i + 1));  // 递增延迟
    }
  }
}
```

## 注意事项

1. **速率限制**: Gemini 免费版有生成次数限制，避免频繁调用
2. **登录保持**: 使用 MCP Playwright 的会话持久化功能
3. **图片质量**: Gemini 生成的图片通常为 1024x1024
4. **内容政策**: 遵守 Google 的内容生成政策

## 会话持久化

```javascript
// MCP Playwright 自动保存浏览器 Profile
// 登录一次后，后续使用无需重新登录

// Profile 存储位置
// ~/.mcp/playwright/browser-profile/

// 如需清除登录状态
// rm -rf ~/.mcp/playwright/browser-profile/
```
