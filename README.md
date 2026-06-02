# 海龟汤 · Turtle Soup

极简的海龟汤情境推理游戏。玩家通过「是 / 否 / 无关」的问题，一步步逼近谜题背后的真相；AI 担任绝对理性的裁判，只判定、不剧透。

## 玩法

每道谜题给出一段「**汤面**」——一个看似离奇、信息残缺的场景。真正的故事（「**汤底**」）只有裁判可见。玩家不断提出可用是非回答的问题来缩小可能性，裁判逐题给出「是 / 不是 / 无关」。当玩家的提问覆盖了全部核心要素，裁判判定「破案了」。

## 技术栈

- **Next.js 15**（App Router）+ **React 19** + TypeScript
- **Tailwind CSS v4**
- **DeepSeek**（`deepseek-chat`）——通过 OpenAI SDK 指向 `api.deepseek.com`，以 LLM 充当裁判
- **Zod** 强类型校验请求与 AI 返回的 JSON

## 裁判设计

裁判逻辑全在 `src/app/api/judge/route.ts`，几个关键点：

- **清单协议（Checklist Protocol）** —— 系统提示要求裁判先把汤底拆解为 3–4 个不可再分的核心要素，并在内部 `reasoning` 中逐题追踪命中进度，避免「猜中一个就误判破案」。
- **状态机收口** —— AI 只能返回 `IN_PROGRESS` 或 `SOLVED` 两种控制状态；「是 / 不是 / 无关」放在 `message` 字段。两者分离，杜绝枚举混淆。
- **汤底不外泄** —— AI 的 `reasoning` 字段在返回客户端前会被剥离，玩家永远拿不到推理过程。
- **服务端校验** —— 请求体和 AI 返回都经 Zod 校验；JSON 解析失败时降级为友好提示而非崩溃。
- **限流** —— 单 IP 60 秒内最多 15 次请求，内存计数并定期清理。

## 项目结构

```
src/
├── app/
│   ├── api/judge/route.ts   # AI 裁判（Serverless Function）
│   ├── game/[id]/page.tsx   # 单局游戏页
│   ├── page.tsx             # 谜题列表首页
│   └── layout.tsx
├── lib/
│   ├── stories.ts           # 谜题库（汤面 + 汤底，11 个经典案例）
│   └── use-game-machine.ts  # 前端状态机 Hook
├── types/game.ts            # 领域模型 + Zod Schema
└── components/              # UI 层（纯渲染）
```

## 本地运行

```bash
# 1. 安装依赖
npm install

# 2. 配置 DeepSeek API Key
echo "DEEPSEEK_API_KEY=你的key" > .env.local

# 3. 启动开发服务器
npm run dev
```

打开 http://localhost:3000 即可开始。

> API Key 只在服务端使用，不会下发到浏览器。`.env*` 已被 `.gitignore` 忽略。

## 许可

[MIT](LICENSE)
