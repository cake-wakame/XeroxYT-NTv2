XeroxYT-NTv2 Render デプロイガイド
変更点まとめ
1. 新規ファイル作成
server.js (新規)
Express サーバーを作成し、API エンドポイントと静的ファイル配信を統合
Vercel のサーバーレス関数形式から、通常の Express サーバー形式に変換
すべての API エンドポイント (/api/video, /api/search, /api/comments, /api/channel, /api/channel-shorts, /api/channel-playlists, /api/playlist, /api/fvideo) を統合
SPAルーティング対応（React Routerのクライアントサイドルーティング）
Express 5.x 対応のルーティング構文を使用
render.yaml (新規)
Render のサービス設定ファイル
ビルドコマンドとスタートコマンドを定義
2. 既存ファイルの変更
package.json
変更前:

{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "@vercel/node": "^3.2.0",
    ...
  },
  "engines": {
    "node": "22.x"
  }
}

変更後:

{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "start": "node server.js",
    "render-build": "npm install && npm run build"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.4",
    ...
  },
  "engines": {
    "node": ">=18.0.0"
  }
}

主な変更点:

start スクリプト追加（本番サーバー起動用）
render-build スクリプト追加（Render用ビルドコマンド）
@vercel/node を削除、@vitejs/plugin-react を追加
Node.js バージョン要件を緩和 (22.x → >=18.0.0)
vite.config.ts
変更前:

import path from 'path';
import { defineConfig, loadEnv } from 'vite';
export default defineConfig(({ mode }) => {
    const env = loadEnv(mode, '.', '');
    return {
      define: {...},
      resolve: {...}
    };
});

変更後:

import path from 'path';
import { defineConfig, loadEnv } from 'vite';
import react from '@vitejs/plugin-react';
export default defineConfig(({ mode }) => {
    const env = loadEnv(mode, '.', '');
    return {
      plugins: [react()],
      define: {...},
      resolve: {...},
      build: {
        outDir: 'dist',
        emptyOutDir: true
      },
      server: {
        host: '0.0.0.0',
        port: 5000,
        proxy: {
          '/api': {
            target: 'http://localhost:3000',
            changeOrigin: true
          }
        }
      }
    };
});

主な変更点:

React プラグインを追加
ビルド出力設定を追加
開発サーバー設定を追加
index.html
charset="UTF--8" のタイポを charset="UTF-8" に修正
CDN importmap を削除（Viteでバンドルするため不要）
Render デプロイ方法
ビルドコマンド
npm run render-build

または

npm install && npm run build

スタートコマンド
npm start

または

node server.js

Render ダッシュボードでの設定
New Web Service をクリック
GitHub リポジトリを接続
以下を設定:
Runtime: Node
Build Command: npm run render-build
Start Command: npm start
Environment: NODE_ENV=production
環境変数（オプション）
必要に応じて以下を設定:

GEMINI_API_KEY: Gemini API キー（使用する場合）
ファイル構造
/
├── server.js          # Express サーバー（APIと静的ファイル配信）
├── package.json       # 依存関係とスクリプト
├── vite.config.ts     # Vite ビルド設定
├── render.yaml        # Render 設定
├── index.html         # エントリーポイント
├── index.tsx          # React アプリケーションのエントリー
├── App.tsx            # メインアプリコンポーネント
├── api/               # (Vercel用 - Renderでは不使用)
├── components/        # React コンポーネント
├── contexts/          # React コンテキスト
├── pages/             # ページコンポーネント
├── hooks/             # カスタムフック
├── utils/             # ユーティリティ関数
└── dist/              # ビルド出力（自動生成）

注意事項
api/ ディレクトリのファイルはVercel用のサーバーレス関数形式です。Renderでは server.js のAPIエンドポイントが使用されます。

Render の無料プランでは、15分間アクセスがないとスリープします。

youtubei.js ライブラリはYouTubeの内部APIを使用しているため、警告ログが出力されることがありますが、機能には影響しません。
