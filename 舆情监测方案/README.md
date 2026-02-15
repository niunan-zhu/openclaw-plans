# 市场舆情监测方案（免费版）

> **当前选择**：免费方案

## 需求概述

- **目标**：每日自动抓取品牌、竞品、行业动态，整理为结构化报告
- **范围**：公众号、百家号、头条号、搜狐号、抖音、B站
- **推送**：邮件

---

## 一、各平台抓取方案

### 1. 微信公众号

#### 方案 A：搜狗微信搜索（推荐免费方案）

| 项目 | 说明 |
|------|------|
| 入口 | https://weixin.sogou.com/ |
| 原理 | 搜狗运营微信公众平台搜索引擎，可抓取文章标题+摘要 |
| 限制 | 需要验证码、IP 限流 |

**抓取流程：**
```
1. 访问 https://weixin.sogou.com/weixin?query=关键词
2. 解析返回的 HTML
3. 提取：标题、链接、摘要、发布时间
4. 访问文章链接获取正文（部分需要登录）
```

**数据来源标注**：
```json
{
  "source": "搜狗微信",
  "url": "https://weixin.sogou.com/...",
  "platform": "微信公众号"
}
```

**代码实现：**
```javascript
// 使用 Playwright
const response = await page.goto(`https://weixin.sogou.com/weixin?query=${encodeURIComponent(keyword)}`);
const articles = await page.evaluate(() => {
  const items = document.querySelectorAll('.news-list li');
  return Array.from(items).map(item => ({
    title: item.querySelector('.txt-box h3 a')?.textContent,
    url: item.querySelector('.txt-box h3 a')?.href,
    summary: item.querySelector('.txt-box p')?.textContent,
    time: item.querySelector('.sou')?.textContent
  }));
});
```

#### 方案 B：微信读书（更稳定）

| 项目 | 说明 |
|------|------|
| 入口 | https://weread.qq.com/ |
| 原理 | 部分公众号文章会同步到微信读书 |
| 限制 | 需要登录、覆盖率低 |

---

### 2. 百度百家号

#### 方案：百度搜索 + 站内抓取

| 项目 | 说明 |
|------|------|
| 入口 | https://www.baidu.com/s?wd=关键词+site:baijiahao.baidu.com |
| 原理 | 百度搜索结果中筛选百家号文章 |
| 限制 | 需要登录百度账号获取Cookie |

**抓取流程：**
```
1. 构造百度搜索 URL
2. 设置百度登录 Cookie（维持登录状态）
3. 解析搜索结果页
4. 提取：标题、链接、来源、发布时间
5. 访问文章页面获取正文
```

**代码实现：**
```javascript
// 百度搜索
const searchUrl = `https://www.baidu.com/s?wd=${encodeURIComponent(keyword)}+site:baijiahao.baidu.com&pn=0`;

// 设置 Cookie 维持登录
await page.setCookie({
  name: 'BDUSS',
  value: '你的百度登录Cookie',
  domain: '.baidu.com'
});
```

---

### 3. 今日头条（头条号）

#### 方案：头条搜索 + m.toutiao.com

| 项目 | 说明 |
|------|------|
| 入口 | https://www.toutiao.com/search/?keyword=关键词 |
| 原理 | 头条号文章通过头条搜索获取 |
| 限制 | 部分需要登录 |

**抓取流程：**
```
1. 访问 https://www.toutiao.com/search/?keyword=关键词
2. 解析 JSON 数据（头条返回结构化数据）
3. 提取：标题、链接、来源、发布时间
4. 访问文章页面获取正文
```

---

### 4. 搜狐号

#### 方案：搜狐搜索

| 项目 | 说明 |
|------|------|
| 入口 | https://www.sohu.com/search?keyword=关键词 |
| 原理 | 搜狐站内搜索 |
| 限制 | 较稳定 |

**抓取流程：**
```
1. 访问 https://www.sohu.com/search?keyword=关键词
2. 解析 HTML
3. 提取：标题、链接、来源、发布时间
```

---

### 5. 抖音

#### 方案：抖音网页版搜索

| 项目 | 说明 |
|------|------|
| 入口 | https://www.douyin.com/search/关键词 |
| 原理 | 抖音网页版可查看视频信息 |
| 限制 | 视频内容难以获取文字资讯 |

**抓取流程：**
```
1. 访问 https://www.douyin.com/search/关键词
2. 解析返回的 JSON 数据
3. 提取：视频标题、作者、发布时间
4. 仅获取文字信息，视频内容跳过
```

**代码实现：**
```javascript
// 抖音搜索结果在 JSON 中
const data = await page.evaluate(() => {
  const script = document.querySelector('#__NEXT_DATA__');
  return script ? JSON.parse(script.textContent) : null;
});
```

---

### 6. B站（哔哩哔哩）

#### 方案：B站开放 API（推荐）

| 项目 | 说明 |
|------|------|
| API | https://api.bilibili.com/web/x3-interface/search?keyword=关键词 |
| 原理 | B站开放平台接口 |
| 限制 | 需要 Cookie（可选） |

**抓取流程：**
```
1. 调用 B站搜索 API
2. 解析返回 JSON
3. 提取：标题、链接、作者、发布时间、简介
```

**代码实现：**
```javascript
// B站搜索 API
const response = await fetch(`https://api.bilibili.com/x/web-interface/search?search_type=video&keyword=${encodeURIComponent(keyword)}`);
const data = await response.json();
const results = data.data.result.map(item => ({
  title: item.title,
  url: item.arcurl,
  author: item.author,
  pubdate: item.pubdate
}));
```

---

## 二、技术架构

### 目录结构

```
scripts/
└── monitor/
    ├── config/
    │   ├── keywords.json      # 关键词配置
    │   └── platforms.json    # 平台配置
    ├── src/
    │   ├── platforms/
    │   │   ├── sogou.js       # 搜狗微信
    │   │   ├── baidu.js       # 百度百家号
    │   │   ├── toutiao.js     # 今日头条
    │   │   ├── sohu.js        # 搜狐
    │   │   ├── douyin.js      # 抖音
    │   │   └── bilibili.js    # B站
    │   ├── storage.js         # SQLite 存储
    │   ├── deduplication.js   # 去重
    │   ├── reporter.js        # 报告生成
    │   └── index.js           # 入口
    └── output/
        └── reports/           # 报告输出
```

### 核心流程

```
每日定时触发
    │
    ▼
读取关键词配置
    │
    ▼
遍历 6 个平台
    │
    ├── 微信 ──► 搜狗搜索
    ├── 百家号 ──► 百度搜索
    ├── 头条号 ──► 头条搜索
    ├── 搜狐号 ──► 搜狐搜索
    ├── 抖音 ──► 抖音网页
    └── B站 ──► B站 API
    │
    ▼
数据存储 (SQLite)
    │
    ▼
去重 + 分类
    │
    ▼
生成 Markdown 报告
    │
    ▼
发送邮件
```

### 依赖

```json
{
  "dependencies": {
    "playwright": "^1.40.0",
    "better-sqlite3": "^9.2.0",
    "nodemailer": "^6.9.0",
    "axios": "^1.6.0",
    "dotenv": "^16.3.0"
  }
}
```

---

## 三、关键词配置

```json
{
  "brand": [
    "水下机器人",
    "水下无人机",
    "鳍源科技",
    "FIFISH"
  ],
  "competitor": [
    "深之蓝",
    "潜行创新",
    "博雅工道",
    "世航智能"
  ],
  "benchmark": [
    "大疆DJI",
    "影石insta360",
    "拓竹科技",
    "道通科技",
    "优必选",
    "宇树科技"
  ],
  "industry": [
    "海洋科技",
    "水下科技",
    "深海科技",
    "海上油气",
    "海上风电",
    "应急救援",
    "水下检测",
    "水下巡检",
    "海洋科研",
    "水下摄影"
  ]
}
```

---

## 四、报告模板

```markdown
# 市场舆情监测日报

**日期**：2026-02-16

---

## 📊 数据概览

| 平台 | 数量 |
|------|------|
| 微信公众号 | 12 篇 |
| 百家号 | 8 篇 |
| 头条号 | 5 篇 |
| 搜狐号 | 3 篇 |
| 抖音 | 6 条 |
| B站 | 4 条 |

---

## 🔴 品牌动态

### 水下机器人
- **标题**：FIFISH 发布新一代水下机器人
- **来源**：微信公众号 - 鳍源科技 **[1]**
- **链接**：[查看原文](url)
- **摘要**：...

**[1]** https://weixin.sogou.com/...

### 鳍源科技
- **标题**：...
- **来源**：百家号 - 科技前沿 **[2]**
- **链接**：[查看原文](url)

**[2]** https://baijiahao.baidu.com/...

---

## 🟡 竞品动向

### 深之蓝
- **标题**：...
- **来源**：今日头条 - 海洋之心 **[3]**
- **链接**：[查看原文](url)

**[3]** https://www.toutiao.com/...

---

## 🟢 行业热点

### 海洋科技
- **标题**：...
- **来源**：B站 - 科技探索 **[4]**
- **链接**：[查看原文](url)

### 视频内容总结

**[抖音] 水下机器人行业分析**
- **来源**：抖音 - 科技评测 **[5]**
- **AI 摘要**：视频介绍了水下机器人在海洋科研领域的应用...

**[5]** https://www.douyin.com/...

---

## 📈 趋势分析

1. 本周"水下机器人"热度上升 15%
2. "海上风电"相关内容增多
3. 大疆发布新品引发关注

---

## 📎 数据来源

| 编号 | 平台 | 链接 |
|------|------|------|
| 1 | 微信公众号 | https://weixin.sogou.com/... |
| 2 | 百家号 | https://baijiahao.baidu.com/... |
| 3 | 今日头条 | https://www.toutiao.com/... |
| 4 | B站 | https://www.bilibili.com/... |
| 5 | 抖音 | https://www.douyin.com/... |

---

## 📎 原始数据

[附件：data_2026-02-16.csv]
```

---

## 五、实施计划

### 第一阶段：基础抓取（Week 1）

| 天数 | 任务 |
|------|------|
| Day 1-2 | 搭建项目框架 + 搜狗微信抓取 |
| Day 3-4 | 百度百家号 + 今日头条 |
| Day 5 | 搜狐号 + B站 API |
| Day 6-7 | 数据存储 + 去重逻辑 |

### 第二阶段：自动化（Week 2）

| 天数 | 任务 |
|------|------|
| Day 8-9 | 报告生成模板 |
| Day 10 | 邮件发送集成 |
| Day 11 | OpenClaw Cron 定时任务 |
| Day 12-14 | 测试 + 优化 |

---

## 六、AI 视频内容总结

### 方案

对于抖音/B站视频，使用 AI 提取内容摘要：

```
视频链接 ──► 下载字幕/音频 ──► LLM 总结 ──► 摘要文本
```

### 实现方式

| 步骤 | 工具 |
|------|------|
| 1. 获取视频 | 抖音/B站网页版 |
| 2. 提取字幕 | 视频网站自带字幕 API 或 ASR |
| 3. AI 总结 | MiniMax API |

### 示例

```javascript
// 视频内容总结
const summary = await llm.summarize({
  videoTitle: "FIFISH V6 水下机器人测评",
  subtitles: "字幕文本内容...",
  prompt: "提取视频核心内容，50字总结"
});
```

### 成本

- B站字幕 API：免费
- 抖音 ASR：需要第三方服务（约 ¥0.01/条）
- LLM 总结：MiniMax ¥0.1/千tokens

---

## 七、风险与限制

| 平台 | 风险 | 应对措施 |
|------|------|----------|
| 微信搜狗 | 验证码、IP封禁 | 降低请求频率、使用代理 |
| 百度 | Cookie 失效 | 定期手动更新 |
| 抖音 | 反爬严格 | 仅抓取文字信息 |

---

## 七、实施计划

**预计周期**：2 周

### Week 1：基础抓取

| 天数 | 任务 |
|------|------|
| Day 1-2 | 项目 搜狗微信抓取 |
| Day 3-4 | 百度百家号 + 今日头条 |
| Day 5 | 搜狐号 + B站 API |
| Day 6-7 | 数据存储框架 + + 去重 |

### Week 2：自动化

| 天数 | 任务 |
|------|------|
| Day 8-9 | 报告生成模板 |
| Day 10 | 邮件发送 |
| Day 11 | Cron 定时任务 |
| Day 12-14 | 测试 + 优化 |

---

## 八、下一步

1. 确认是否立即开始开发
2. 提供百度登录 Cookie（可选，用于百家号）
3. 确认报告收件人
