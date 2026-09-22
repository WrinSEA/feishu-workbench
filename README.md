# 飞书「工作台」应用（占位演示）

一个单文件 H5 应用（`index.html`），模拟飞书移动端工作台：

- **首页「工作台」**：2×2 应用卡片入口 + 搜索框 + 公告位
- **营运系统**：今日概览统计、近 7 日营收柱状图、今日待办
- **培训系统**：学习统计、课程列表（含进度条）、培训公告
- **报修系统**：发起报修（底部弹窗表单，可交互提交）、快捷报修分类、报修记录（状态标签）
- **巡查系统**：今日巡查进度条、可勾选的巡查任务清单（勾选实时更新完成率）

所有数据均为写死在前端 `state` 里的模拟数据，无后端依赖。

## 本地预览

直接双击打开 `index.html` 即可；或起一个本地服务：

```bash
cd feishu-workbench
python -m http.server 8000
# 浏览器访问 http://localhost:8000
```

## 已发布到飞书妙搭（2026-09-22）

页面已通过 `lark-cli` 发布到妙搭托管平台，共两个企业环境：

| 项目 | 企业 A（原身份） | 企业 B（当前身份） |
|---|---|---|
| app_id | `app_17em2zrx1er` | `app_17em6ww0zgz` |
| 线上地址 | https://o08ce4sft8c.feishu.cn/page/OzYDmQVUkdTJLdaSuR4cFukdnVe | https://gcnidq3jsbiv.feishu.cn/page/Wto5mLGfDdYpz7aUreOcTwiQnCc |
| 版本 | 首版（四入口全占位） | 含巡查系统 → 光印巡检跳转 |
| CLI profile | `cli_aac894b44dbcdbb5`（抖音舆论监测） | `ent2-workbench`（cli_aa38a67154385cb5） |

当前本地仓库 `../workbench-app` 绑定的是**企业 B** 的应用（`app_17em6ww0zgz`）。

### 双企业身份切换

```bash
# 切回原企业（抖音舆论监测应用配置仍保留，切换后需重新 auth login 一次）
lark-cli profile use cli_aac894b44dbcdbb5

# 切到企业 B 的工作台发布身份
lark-cli profile use ent2-workbench

# 切回企业 A 后如需恢复旧应用的本地代码目录：
lark-cli apps +init --as user --app-id app_17em2zrx1er --dir ./workbench-app-a
```

### 修改后如何重新发布（企业 B）

页面源码以 `../workbench-app/index.html` 为准（改完同步一份到本目录即可）：

```bash
cd ../workbench-app
# 编辑 index.html 后：
git add index.html
git commit -m "feat: xxx"
git push origin sprint/default
lark-cli apps +release-create --as user --app-id app_17em6ww0zgz --branch sprint/default
# 用返回的 release_id 轮询，status=finished 即发布完成：
lark-cli apps +release-get --as user --app-id app_17em6ww0zgz --release-id <release_id>
```

### 巡查系统外部跳转配置

`index.html` 顶部的 `LINKS` 对象控制外部真实应用的跳转：

```js
var LINKS = {
  xuncha: 'https://app.feishu.cn/app/cli_a31aa82f07be1013'  // 光印巡检
};
```

带 `link` 的入口点击后直接跳转（飞书客户端内拉起目标应用），不带 `link` 的入口进入内置占位页。要接入更多真实系统，往 `LINKS` 里加对应入口的链接并在 `APPS` 数组对应项上引用即可。

注意事项：

- `+release-create` 发布的是远端 `sprint/default` 上**已 push** 的代码，本地未提交的改动不生效。
- 可见范围：创意模式应用不能用 CLI 设置，需在[妙搭后台](https://app.feishu.cn/miaoda)（对应企业的身份）中把应用的可见范围从「仅自己」放开到企业成员/全员。**两个企业各自都要设置一次。**
- Git Bash 环境下 `+init` 若报 `tar (child): Cannot connect to C` ，先 `export PATH="/c/Windows/System32:$PATH"` 再执行（让 Windows 自带 tar 优先）。
- 后续要求数据库 / 后端能力时，不在 html 应用上原地升级，需在妙搭平台走类型升级。

### 备选方案：企业自建应用（工作台独立图标）

若希望它在飞书「工作台」Tab 里显示为独立应用图标（而不仅是链接），可在
[飞书开放平台](https://open.feishu.cn) 创建企业自建应用 → 添加「网页应用」能力 →
主页 URL 填上面的线上地址 → 创建版本申请发布，管理员审核通过即可；纯静态页无需申请任何 API 权限。

后续真实化建议：

- 用[飞书 JSSDK](https://open.feishu.cn/document/client-docs/web-jssdk/overview) 做免登，获取当前用户身份；
- 把四个入口卡片 `data-go` 换成真实系统的跳转链接，或继续扩展对应子页面；
- 报修 / 巡查数据接入真实接口（当前仅存在前端内存中，刷新即重置）。

## 目录结构

```
feishu-workbench/
├── index.html   # 全部页面与逻辑（单文件，无依赖）
└── README.md
```

## 如何修改

- 应用入口配置：`index.html` 中的 `APPS` 数组（名称 / 描述 / 颜色）。
- 各子页面内容：`pageYunying` / `pagePeixun` / `pageBaoxiu` / `pageXuncha` 函数。
- 模拟数据：`state` 对象（报修记录、巡查清单）。
