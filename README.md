# 企業向け社員教育プラットフォーム

導入するだけで自動的に社員のスキルアップを促進する教育プラットフォーム

## 主要機能

- 📝 **毎日の5択クイズ**: 10時に自動配信される問題で知識をチェック
- 🎯 **スコア&ゲーミフィケーション**: ポイント制度と実績システムでモチベーション向上
- 📚 **学習コンテンツ管理**: 動画・記事・スライドなど多様な教材
- 📊 **分析ダッシュボード**: 部署別・個人別の学習進捗を可視化
- 🔔 **マルチチャネル通知**: メール・Slack・ブラウザ通知対応

## 技術スタック

- **フロントエンド**: Next.js 14, TypeScript, Tailwind CSS
- **バックエンド**: Node.js, Express, Prisma
- **データベース**: PostgreSQL (Supabase)
- **認証**: NextAuth.js
- **ホスティング**: Vercel (Frontend), Railway/Render (Backend)

## セットアップ

```bash
# 依存関係のインストール
npm install

# 環境変数の設定
cp .env.example .env

# データベースのマイグレーション
npm run db:migrate

# 開発サーバーの起動
npm run dev
```

## プロジェクト構造

詳細は [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md) を参照

## システム設計

詳細は [SYSTEM_DESIGN.md](./SYSTEM_DESIGN.md) を参照

## ライセンス

Proprietary