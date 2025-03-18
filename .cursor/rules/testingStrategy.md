# テスト方針

このドキュメントは、プロジェクトにおけるテスト戦略を定義し、品質保証の基盤となります。現在はプロジェクト初期段階であり、テスト方針は開発の進行に伴って具体化されていきます。

## テスト階層

プロジェクトでは以下のテスト階層を採用し、各レイヤーの品質を保証する予定です:

```
E2Eテスト ─────┐
              │
APIテスト ─────┤ 品質ピラミッド
              │
単体/結合テスト─┘
```

## フロントエンドテスト方針

### 単体テスト

- **対象**: 外部依存がなく、コンポーネント内で独立して状態変化により表示が変わるロジック部分
- **ツール**: Jest, React Testing Library
- **アプローチ**:
  - ユティリティ関数の入出力検証
  - UI 状態変化のテスト
  - 複雑なロジックの網羅的テスト

### 結合テスト

- **対象**: フォーム処理部分や API 通信の結果を受け取る処理
- **ツール**: React Testing Library, MSW
- **アプローチ**:
  - コンポーネント間の相互作用検証
  - API 通信のモック化と応答処理検証
  - フォーム入力と送信フローの検証

## バックエンドテスト方針

### 単体テスト

- **対象**: サービス層、コントローラー層をカバー
- **ツール**: Jest
- **アプローチ**:
  - ビジネスロジックの検証
  - 入力検証の確認
  - エラーハンドリングの確認

### 結合テスト

- **対象**: API エンドポイントとデータフロー
- **アプローチ**:
  - ステータスコード（200, 400, 401, 403, 500）のテスト
  - データ整合性のシナリオベーステスト（CRUD サイクルなど）
  - リポジトリ層の複雑性に応じて DB モック化を決定

## E2E テスト方針

- **対象**: リグレッションリスクの高い重要機能
- **ツール**: Cypress
- **アプローチ**:
  - 主要ユーザーフローのシミュレーション
  - リグレッションテストのシナリオ設計
  - 全機能実装後に着手

## テスト優先順位

1. **最優先**: 認証・アクセス制御関連
2. **高優先**: データ永続化と整合性
3. **中優先**: ユーザーインターフェース検証
4. **低優先**: 細部の UI スタイリング

## テスト自動化

- CI/CD パイプラインでの自動テスト実行
- プルリクエスト時の自動テスト実行
- テストカバレッジレポートの生成

## テスト戦略の実装ステップ

### フェーズ 1: テスト基盤の構築

- Jest, React Testing Library のセットアップ
- テストコード作成のガイドラインと標準化
- CI/CD パイプラインでのテスト自動実行の設定

### フェーズ 2: 単体テストの充実

- ユーティリティ関数の単体テスト整備
- 各コンポーネントの単体テスト作成
- カスタムフックのテスト作成

### フェーズ 3: 結合テストの実装

- コンポーネント間の結合テスト作成
- API 連携部分のテスト整備
- MSW によるモックサーバーの活用

### フェーズ 4: E2E テストの導入

- Cypress のセットアップと基本設定
- 主要ユーザーフローの E2E テスト実装
- CI/CD パイプラインでの E2E テスト実行

## テストコードのベストプラクティス

### 命名規則

- テストファイル: `<コンポーネント名>.test.tsx`
- テスト記述: `describe` / `it` ブロックを使用して階層的に記述
- テストケースの命名: `it('should <期待する動作>')`

### テストの構造

- **Arrange**: テスト前の準備、データのセットアップ
- **Act**: テスト対象のアクション実行
- **Assert**: 期待する結果の検証

### テストの独立性

- 各テストは独立して実行できること
- グローバル状態への依存を最小化
- 適切なモック化とテスト用のファクトリ関数の活用

## フロントエンドテスト例

### コンポーネントテスト

```tsx
// Button.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { Button } from "./Button";

describe("Button", () => {
  it("should render with provided label", () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText("Click me")).toBeInTheDocument();
  });

  it("should call onClick when clicked", () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    fireEvent.click(screen.getByText("Click me"));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it("should be disabled when disabled prop is true", () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByText("Click me")).toBeDisabled();
  });
});
```

### カスタムフックテスト

```tsx
// useCounter.test.ts
import { renderHook, act } from "@testing-library/react-hooks";
import { useCounter } from "./useCounter";

describe("useCounter", () => {
  it("should initialize with provided initial value", () => {
    const { result } = renderHook(() => useCounter(5));
    expect(result.current.count).toBe(5);
  });

  it("should increment counter when increment is called", () => {
    const { result } = renderHook(() => useCounter(0));
    act(() => {
      result.current.increment();
    });
    expect(result.current.count).toBe(1);
  });

  it("should decrement counter when decrement is called", () => {
    const { result } = renderHook(() => useCounter(5));
    act(() => {
      result.current.decrement();
    });
    expect(result.current.count).toBe(4);
  });
});
```
