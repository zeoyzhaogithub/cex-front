# C2E-CEX 前端项目搭建流程

从零搭建 CEX 前端，技术栈：**React 19 + Next.js 16 App Router + TypeScript + Tailwind 4 + shadcn/ui**。

> 架构说明见 [architecture.md](./architecture.md)。后端与本地链需单独启动，见 `../cex-backend/README.md`、`../cex-chain/README.md`。

---

## 1. 环境准备

| 工具 | 版本 | 用途 |
|------|------|------|
| Node.js | ≥ 20 | 运行时 |
| pnpm | 10.x | 依赖管理 |

```bash
corepack enable
corepack prepare pnpm@10.17.1 --activate
```

也可用 `npm install -g pnpm@10.17.1` 替代上述两条命令。

---

## 2. 创建项目（package.json 的由来）

### 2.1 脚手架自动生成 package.json

`pnpm create next-app` 会创建项目目录，并**自动生成**初始 `package.json`，包含 `name`、`scripts`、`dependencies`（next、react、react-dom）等字段。

```bash
mkdir cex-front && cd cex-front

pnpm create next-app@latest . \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --turbopack
```

执行后得到的初始 `package.json` 大致如下：

```json
{
  "name": "cex-front",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint"
  },
  "dependencies": {
    "next": "16.x",
    "react": "19.x",
    "react-dom": "19.x"
  },
  "devDependencies": {
    "@tailwindcss/postcss": "^4",
    "tailwindcss": "^4",
    "typescript": "^5",
    "eslint": "^9",
    "eslint-config-next": "16.x"
  }
}
```

> 版本号以脚手架实际输出为准；后续 `pnpm add` 会在此基础上追加依赖并更新 lock 文件。

### 2.2 对齐核心版本

```bash
pnpm add next@16 react@19 react-dom@19
pnpm add -D tailwindcss@4 @tailwindcss/postcss typescript@5
```

### 2.3 安装业务依赖

以下每条 `pnpm add` 都会**自动写入** `package.json` 的 `dependencies` 或 `devDependencies`，无需手动编辑。

```bash
# UI
pnpm dlx shadcn@latest init
pnpm dlx shadcn@latest add button input table tabs dialog select \
  dropdown-menu tooltip sonner form label card badge separator

# 状态与数据
pnpm add valtio valtio-define @tanstack/react-query nuqs
pnpm add @hairy/react-lib @hairy/utils

# 国际化
pnpm add next-intl

# HTTP 与 API 生成
pnpm add axios axios-extras
pnpm add -D @genapi/core @genapi/presets dotenv tsx

# 实时行情
pnpm add sockjs-client stompjs
pnpm add -D @types/sockjs-client @types/stompjs

# 图表与工具
pnpm add echarts mathjs dayjs qrcode.react copy-to-clipboard swiper motion

# 全局 Providers
pnpm add next-themes @overlastic/react sonner
pnpm add react-hook-form @hookform/resolvers zod

# 开发工具
pnpm add -D unimport-loader eslint-config-prettier eslint-plugin-prettier \
  @typescript-eslint/eslint-plugin @typescript-eslint/parser \
  prettier husky lint-staged
```

### 2.4 补充 scripts

在 `package.json` 的 `scripts` 中手动追加（或覆盖默认值）：

```json
{
  "scripts": {
    "dev": "next dev --port 8080",
    "build": "next build",
    "start": "next start --port 8080",
    "type-check": "tsc --noEmit",
    "lint": "eslint .",
    "lint:check": "eslint . --max-warnings 0"
  }
}
```

### 2.5 完整 package.json 参考

所有依赖安装完毕后，`package.json` 应类似：

```json
{
  "name": "cex-front",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev --port 8080",
    "build": "next build",
    "start": "next start --port 8080",
    "type-check": "tsc --noEmit",
    "lint": "eslint .",
    "lint:check": "eslint . --max-warnings 0"
  },
  "dependencies": {
    "@hairy/react-lib": "^1.x",
    "@hairy/utils": "^1.x",
    "@hookform/resolvers": "^3.x",
    "@overlastic/react": "^0.x",
    "@tanstack/react-query": "^5.x",
    "axios": "^1.x",
    "axios-extras": "^1.x",
    "copy-to-clipboard": "^3.x",
    "dayjs": "^1.x",
    "echarts": "^5.x",
    "mathjs": "^13.x",
    "motion": "^11.x",
    "next": "16.x",
    "next-intl": "^4.x",
    "next-themes": "^0.x",
    "nuqs": "^2.x",
    "qrcode.react": "^4.x",
    "react": "19.x",
    "react-dom": "19.x",
    "react-hook-form": "^7.x",
    "sockjs-client": "^1.x",
    "sonner": "^2.x",
    "stompjs": "^2.x",
    "swiper": "^11.x",
    "valtio": "^2.x",
    "valtio-define": "^0.x",
    "zod": "^3.x"
  },
  "devDependencies": {
    "@genapi/core": "^1.x",
    "@genapi/presets": "^1.x",
    "@tailwindcss/postcss": "^4",
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "@types/sockjs-client": "^1.x",
    "@types/stompjs": "^2.x",
    "@typescript-eslint/eslint-plugin": "^8.x",
    "@typescript-eslint/parser": "^8.x",
    "dotenv": "^16.x",
    "eslint": "^9",
    "eslint-config-next": "16.x",
    "eslint-config-prettier": "^10.x",
    "eslint-plugin-prettier": "^5.x",
    "husky": "^9.x",
    "lint-staged": "^15.x",
    "prettier": "^3.x",
    "tailwindcss": "^4",
    "tsx": "^4.x",
    "typescript": "^5",
    "unimport-loader": "^1.x"
  }
}
```

> shadcn/ui 组件以文件形式落在 `src/components/ui/`，其运行时依赖（如 `class-variance-authority`、`lucide-react`）会由 `shadcn init` 自动写入 `package.json`。

---

## 3. 基础配置文件

### 3.1 PostCSS（Tailwind 4）

`postcss.config.mjs`：

```javascript
const config = {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
export default config;
```

### 3.2 TypeScript 路径别名

`tsconfig.json` 中确认：

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### 3.3 环境变量

创建 `.env.local`：

```env
NEXT_PUBLIC_UC_API_URL=http://127.0.0.1:6001
NEXT_PUBLIC_EXCHANGE_API_URL=http://127.0.0.1:6003
NEXT_PUBLIC_MARKET_API_URL=http://127.0.0.1:6004
NEXT_PUBLIC_ETH_WALLET_API_URL=http://127.0.0.1:7003
NEXT_PUBLIC_USDT_WALLET_API_URL=http://127.0.0.1:7004
NEXT_PUBLIC_MARKET_WS_URL=ws://127.0.0.1:6004
```

### 3.4 Next.js 配置

`next.config.ts` 需包含 next-intl 插件与开发代理：

```typescript
import createNextIntlPlugin from 'next-intl/plugin';
import type { NextConfig } from 'next';

const withNextIntl = createNextIntlPlugin();

const config: NextConfig = {
  async rewrites() {
    return [
      { source: '/api/uc/:slug*',       destination: `${process.env.NEXT_PUBLIC_UC_API_URL}/:slug*` },
      { source: '/api/exchange/:slug*', destination: `${process.env.NEXT_PUBLIC_EXCHANGE_API_URL}/:slug*` },
      { source: '/api/market/:slug*',   destination: `${process.env.NEXT_PUBLIC_MARKET_API_URL}/:slug*` },
      { source: '/api/wallet-eth/:slug*',  destination: `${process.env.NEXT_PUBLIC_ETH_WALLET_API_URL}/:slug*` },
      { source: '/api/wallet-usdt/:slug*', destination: `${process.env.NEXT_PUBLIC_USDT_WALLET_API_URL}/:slug*` },
    ];
  },
};

export default withNextIntl(config);
```

---

## 4. 目录结构

按以下结构组织源码：

```
cex-front/
├── public/
│   ├── charting_library/       # TradingView 静态库
│   └── locales/                # i18n 语言包（en.json、zh-CN.json 等）
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   └── [locale]/
│   │       ├── layout.tsx
│   │       ├── page.tsx              # 首页
│   │       ├── login/page.tsx
│   │       ├── register/page.tsx
│   │       ├── exchange/[id]/page.tsx
│   │       └── spot-account/page.tsx
│   ├── apis/                       # Axios 实例 + 各微服务 API
│   ├── services/                     # React Query Hooks
│   ├── components/
│   │   ├── common/                   # Header、Providers、AuthGuard
│   │   ├── exchange/
│   │   ├── chart/
│   │   └── ui/                       # shadcn 组件
│   ├── config/                       # coins.ts、QueryClient、stomp
│   ├── hooks/
│   ├── store/modules/
│   ├── lib/charting/datafeed/        # TradingView datafeed
│   ├── layouts/
│   ├── i18n/
│   └── middleware.ts
├── components.json                   # shadcn 配置
├── genapi.config.ts
├── next.config.ts
├── postcss.config.mjs
├── .env.local
└── package.json
```

### 4.1 复制 TradingView 资源

从现有 Vue 版仓库复制：

```bash
cp -r ../cex-front/public/charting_library ./public/
cp -r ../cex-front/src/assets/js/charting_library/datafeed ./src/lib/charting/datafeed/
```

### 4.2 国际化语言包

将 Vue 版 `assets/local/*.js` 转为 JSON，放入 `public/locales/`。支持 9 种语言：`en`、`zh-CN`、`zh-HK`、`ja`、`ko`、`de`、`fr`、`it`、`es`。

---

## 5. 核心模块搭建顺序

按依赖关系依次实现：

| 步骤 | 模块 | 要点 |
|------|------|------|
| 1 | 全局 Providers | QueryClient、ThemeProvider、next-intl、AuthBootloader |
| 2 | HTTP 层 | `src/apis/index.http.ts` 创建 4 个 Axios 实例（uc / exchange / market / wallet），注入 token 与 lang |
| 3 | 认证 | 登录/注册页、AuthGuard、`POST /uc/check/login` |
| 4 | 布局 | default / exchange / auth 三套布局 + Header |
| 5 | 首页 | 轮播、行情概览、ECharts 折线 |
| 6 | 交易页 | 盘口、最新成交、下单面板、委托列表 |
| 7 | K 线 | TradingView Client Component + STOMP datafeed |
| 8 | 现货账户 | 资产、充值地址、提币 |
| 9 | 代码质量 | husky + lint-staged |

### 5.1 Git 钩子

```bash
pnpm exec husky init
```

`.husky/pre-commit`：

```bash
pnpm lint-staged
pnpm type-check
```

---

## 6. 启动与验证

### 6.1 安装依赖并启动

```bash
cd cex-front
pnpm install    # 根据 package.json + pnpm-lock.yaml 安装全部依赖
pnpm dev        # http://localhost:8080
```

> 需确保后端微服务（6001/6003/6004/7003/7004）已运行，否则 API 调用会失败。

### 6.2 验证清单

- [ ] `pnpm type-check` 无错误
- [ ] `pnpm lint:check` 无错误
- [ ] `pnpm build` 成功
- [ ] 访问 `http://localhost:8080` 首页正常
- [ ] 登录/注册可调用 UC 接口
- [ ] `/exchange/eth_usdt` 可加载盘口与 K 线
- [ ] i18n 切换正常
- [ ] `public/charting_library/` 静态资源完整

### 6.3 生产构建

`next.config.ts` 添加 `output: 'standalone'`，然后：

```bash
pnpm build
pnpm start
```

---

## 7. 一键命令速查

```bash
# 创建项目（生成 package.json）
pnpm create next-app@latest cex-front --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd cex-front

# 安装全部依赖（自动更新 package.json）
pnpm add next@16 react@19 react-dom@19
pnpm add next-intl valtio valtio-define @tanstack/react-query axios axios-extras \
  sockjs-client stompjs echarts mathjs dayjs next-themes nuqs @overlastic/react \
  @hairy/react-lib @hairy/utils react-hook-form @hookform/resolvers zod sonner \
  qrcode.react copy-to-clipboard swiper motion
pnpm add -D @tailwindcss/postcss tailwindcss unimport-loader @genapi/core @genapi/presets \
  @types/sockjs-client @types/stompjs eslint-config-prettier husky lint-staged prettier tsx dotenv

pnpm dlx shadcn@latest init
pnpm dlx shadcn@latest add button input table tabs dialog select dropdown-menu form sonner

cp .env.example .env.local   # 或手动创建 .env.local
pnpm install
pnpm dev
```
