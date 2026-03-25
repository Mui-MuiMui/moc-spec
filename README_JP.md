# .moc ファイル形式仕様書
[English](README.md)

- **バージョン:** 1.2.1
- **ステータス:** 正式仕様

### バージョン履歴

| バージョン | 変更内容 |
|-----------|---------|
| 1.2.1 | `@moc-memos` ブロックタグを追加 — 全付箋メモをファイルヘッダー冒頭に一覧表示し、AIエージェントによる読み飛ばしを防止。各エントリは `[nodeId] Title:... Message:...` 形式。TSX内の `@moc-memo` インラインコメントと共存（二重記載は意図的） |
| 1.2.0 | エディタデータブロックをBrotli圧縮+Base64エンコード — 大規模craftStateのファイルサイズを大幅削減し、AIエージェントが生JSONを誤って読み込むことを防止。セクションデリミタマーカー（`@moc-imports-start/end`、`@moc-tsx-start/end`）を追加し、確実なセクション分割を実現 |
| 1.1.0 | `@moc-component` タグを追加 — コンポーネントのpropsスキーマをファイルヘッダーに埋め込み、ファイル単体でAIエージェントが自己完結して読めるようにする |
| 1.0.0 | 初回正式リリース |

---

## 1. 概要

- `.moc` ファイルは、VSCode拡張「Momoc」で作成されるGUIモックアップの定義ファイルです。  
- TSX（TypeScript JSX）形式をベースとし、人間・GUI・AIエージェントが協調して開発を行うための統合データ形式です。

### 技術スタック

- **CSS**: Tailwind CSS v4（ユーティリティファーストCSS）
- **UIコンポーネント**: shadcn/ui（Radix UI + Tailwind CSSベース）
- importパス `@/components/ui/*` はshadcn/uiコンポーネントを指す
- **アイコン**: lucide-react（アイコン使用時のみ importブロックに `lucide-react` が現れる）
- 導入先にTailwind CSSやshadcn/uiがない場合は、導入先の技術スタックで同等の見た目・レイアウトを再現すること

### 設計原則

- **人間可読性**: TSX部分はそのままReactコンポーネントとして読解可能
- **AI可読性**: メタデータ・メモ・エディタデータがすべてテキストとして読解可能
- **SSOT（Single Source of Truth）**: GUIエディタの状態（craftState）が存在する場合、TSXコードよりcraftStateが正とする。TSXはcraftStateから自動生成される派生データである

---

## 2. ファイル構造

`.moc` ファイルは以下の4セクションで構成されます:

```
┌─────────────────────────────────┐
│ 1. JSDocメタデータブロック       │  必須
│    (@moc-* タグ)                │
├─────────────────────────────────┤
│ 2. importブロック               │  任意
│    /* @moc-imports-start */     │
│    (ESモジュールimport文)        │
│    /* @moc-imports-end */       │
├─────────────────────────────────┤
│ 3. TSXコンポーネント            │  必須
│    /* @moc-tsx-start */         │
│    (export default function)     │
│    /* @moc-tsx-end */           │
├─────────────────────────────────┤
│ 4. エディタデータブロック        │  任意
│    (const __mocEditorData = ``) │
└─────────────────────────────────┘
```

各セクション（importブロック、TSXコンポーネント）はデリミタマーカー（`/* @moc-*-start */` / `/* @moc-*-end */`）で囲まれ、確実なパース処理を可能にします。パーサーはこのマーカーを使用してセクションを検出すべきです。マーカーが存在しない場合（旧形式ファイル）は、ヒューリスティックベースの検出にフォールバックできます。

---

## 3. メタデータブロック（JSDocコメント）

ファイル先頭のJSDocコメント内に `@moc-*` タグでメタデータを記述します。

### 3.1 ヘッダー説明文

```typescript
/**
 * Momoc Document (.moc)
 * VSCode拡張「Momoc」で作成されたGUIモックアップの定義ファイルです。
 * 人間・GUI・AIエージェントが協調して開発を行うための統合データ形式です。
 *
 * ファイル構造:
 *   本ファイルはTSX（TypeScript JSX）形式をベースとし、
 *   メタデータと付箋メモをJSDocコメント内に、
 *   エディタ内部状態をテンプレートリテラル変数に格納しています。
 *   TSX部分はそのままReactコンポーネントとして読解可能です。
 *
 * 技術スタック:
 *   CSS: Tailwind CSS v4（ユーティリティファーストCSS）
 *   UIコンポーネント: shadcn/ui（Radix UI + Tailwind CSSベース）
 *   importパス「@/components/ui/*」はshadcn/uiコンポーネントです。
 *   アイコン: lucide-react（importパス「lucide-react」）※アイコン使用時のみ
 *   導入先にTailwind CSSやshadcn/uiがない場合は、
 *   導入先の技術スタックで同等の見た目・レイアウトを再現してください。
 *
 * SSOT（Single Source of Truth）:
 *   GUIエディタの状態（craftState）が末尾のエディタデータに存在する場合、
 *   TSXコードよりcraftStateが正となります。
 *   TSXはcraftStateから自動生成される派生データであり、AIがページ構造を
 *   読み取るための参照用です。コンポーネントの配置変更はGUIで行ってください。
 *   (GUIで編集した場合、craftStateの内容がtsxにも上書きされます。)
 *
 * AI読み取り優先順位:
 *   1. TSXコード（構造・レイアウトの主軸 読み手理解用）
 *   2. craftState（GUIエディタの詳細プロパティ参照用）
 *
 * メタデータ:
 *   @moc-version  - ドキュメント形式バージョン（必須）
 *   @moc-intent   - このページの目的・意図（任意、人間が記述）
 *   @moc-theme    - テーマ (light | dark)（任意、デフォルト: light）
 *   @moc-layout   - レイアウトモード (flow | absolute)（任意、デフォルト: flow）
 *   @moc-viewport - 対象ビューポート (desktop | tablet | mobile | WxH)（任意、デフォルト: desktop）
 *
 * AI指示メモ:
 *   ユーザーがキャンバス上に配置した、AIエージェントへの指示付箋です。
 *   @moc-memos ブロックにメモ一覧が記載されます（Title/Message形式）。
 *   TSX内にも @moc-memo コメントとして各要素付近に記載されます。
 *   AIはこのメモを読み取り、該当要素に対する修正・提案を行ってください。
 *
 * TSX内コメント規約:
 *   @moc-node <nodeID>  - Craft.jsノードとの対応付け
 *   @moc-role <役割>     - 要素の役割説明
 *   @moc-memo <メモ>     - 付箋メモの概要
 */
```

### 3.2 メタデータタグ

| タグ | 必須/任意 | 型 | デフォルト | 説明 |
|------|-----------|------|-----------|------|
| `@moc-version` | **必須** | `string` | - | ドキュメント形式バージョン（例: `1.0.0`） |
| `@moc-intent` | 任意 | `string` | `""` | ページの目的・意図 |
| `@moc-theme` | 任意 | `"light" \| "dark"` | `"light"` | テーマ |
| `@moc-layout` | 任意 | `"flow" \| "absolute"` | `"flow"` | レイアウトモード |
| `@moc-viewport` | 任意 | `string` | `"desktop"` | `desktop`, `tablet`, `mobile`, または `WxH` 形式 |
| `@moc-memos` | 任意 | ブロック | - | メモ一覧ブロック — 全付箋メモのサマリー（v1.2.1） |
| `@moc-memo` | 任意 | - | - | AI指示メモ（複数可） |
| `@moc-component` | 任意 | `string`（JSON） | - | コンポーネントのpropsスキーマ（複数可、コンポーネント種別ごとに1つ） |

### 3.3 コンポーネントスキーマ (@moc-component) — v1.1.0

```
@moc-component <コンポーネント名> <スキーマJSON>
```

- `<コンポーネント名>`: Momoc内部のコンポーネント名（例: `CraftButton`, `CraftDataTable`）
- `<スキーマJSON>`: コンポーネントのpropsスキーマを記述したJSONオブジェクト

**JSONの構造:**

```typescript
{
  "displayName": string,           // 人間向けの表示名
  "props": {
    "<propName>": {
      "type": string,              // プリミティブ ("string" | "boolean" | "number")、
                                   // Union型 ("optA|optB|optC")、
                                   // または特殊型 ("JSON string" | "CSV string")
      "default": unknown           // コンポーネント定義のデフォルト値
    }
  }
}
```

**出力例:**

```
@moc-component CraftButton {"displayName":"Button","props":{"text":{"type":"string","default":"Button"},"variant":{"type":"default|destructive|outline|secondary|ghost|link","default":"default"},"disabled":{"type":"boolean","default":false}}}
```

**出力ルール:**
- `craftState` で実際に使用されているコンポーネントのみ出力する
- 内部スロットコンポーネント（`TableCellSlot`、`ResizablePanelSlot` など）は除外する
- コンポーネント名のアルファベット順でソートして出力する
- `editorData`（craftState）が存在しない場合はタグを出力しない

AIエージェントはこのスキーマを参照して `craftState` の各ノードのpropsを外部ドキュメントなしに解釈できます。

### 3.4 AI指示メモ (@moc-memo)

```
@moc-memo #<対象要素ID> "指示テキスト"
```

- `#<対象要素ID>`: TSX内の要素を特定するID（`id` 属性値）
- `"指示テキスト"`: AIエージェントへの指示（ダブルクォートで囲む）
- 複数指定可能

### 3.5 メモ一覧ブロック (@moc-memos) — v1.2.1

ファイルヘッダー冒頭に全付箋メモを一覧表示するブロックタグです。AIエージェントによる読み飛ばしを防止します。

```
@moc-memos
  [nodeId1] Title:ダイアログ仕様 Message:担当者フィールドは検索ダイアログを開く仕様
  [nodeId2] Title:承認後のボタン動作 Message:承認後はボタンを非活性にする
```

**各行のフォーマット:**

```
[<nodeID>] Title:<タイトル> Message:<メッセージ>
```

- `<nodeID>`: メモが紐づいているCraft.jsノードID。紐づくノードがない場合は `_` を使用
- `Title:<タイトル>`: メモのタイトル。タイトルが空の場合は省略
- `Message:<メッセージ>`: メモの本文。本文が空の場合は省略
- 1つのメモが複数ノードに紐づく場合、ノードごとに1行出力
- メモが0件の場合はブロック自体を出力しない

**二重記載:** 同じメモ内容がTSX内の `{/* @moc-memo "..." */}` インラインコメントにも記載されます。これは意図的な二重記載です — ヘッダーブロックは一覧性を提供し、インラインコメントは位置的なコンテキストを提供します。

---

## 4. importブロック

標準のESモジュールimport文を記述します。v1.2.0以降、ブロックはデリミタマーカーで囲まれます。

```typescript
/* @moc-imports-start */
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
/* @moc-imports-end */
```

---

## 5. TSXコンポーネント

`export default function` 形式のReactコンポーネントを記述します。v1.2.0以降、ブロックはデリミタマーカーで囲まれます。

```typescript
/* @moc-tsx-start */
export default function LoginForm() {
  return (
    <>
      {/* @moc-node abc123 */}
      {/* @moc-role "ログインフォーム" */}
      <div className="flex min-h-screen items-center justify-center">
        {/* @moc-node def456 */}
        {/* @moc-memo "メールバリデーション必須" */}
        <Input id="emailInput" type="email" placeholder="Email" />
      </div>
    </>
  );
}
/* @moc-tsx-end */
```

### 5.1 TSX内コメント規約

GUIエディタがTSXを生成する際、各要素に以下のコメントを付与します:

| コメント | 説明 |
|----------|------|
| `{/* @moc-node <nodeID> */}` | Craft.jsノードIDとの対応付け。すべてのノードに付与 |
| `{/* @moc-role "<役割>" */}` | 要素のroleプロパティ値。roleが設定されている場合に付与 |
| `{/* @moc-memo "<メモ概要>" */}` | 付箋メモの概要テキスト。メモが紐づいている場合に付与 |

これらのコメントはAIエージェントがTSX内の要素を理解するための補助情報です。

---

## 6. エディタデータブロック

GUIエディタの内部状態を保存するためのブロックです。
テンプレートリテラル変数としてファイル末尾に格納されます。

### 6.1 形式（v1.2.0 — Brotli圧縮）

v1.2.0以降、エディタデータはBrotli圧縮+Base64エンコードされます:

```typescript
const __mocEditorData = `brotli:<Base64エンコードデータ>`;
```

`brotli:` プレフィックスで圧縮形式を識別します。プレフィックス以降は、エディタ状態のJSONをBrotli圧縮しBase64エンコードした文字列です。

**エンコード手順:** `JSON.stringify(data)` → `brotliCompress` → `Base64エンコード` → `brotli:` を付与

**デコード手順:** `brotli:` を除去 → `Base64デコード` → `brotliDecompress` → `JSON.parse`

AIエージェントはこのデータを直接読み取る**必要はありません**。ページ構造の理解にはTSXコードと `@moc-component` スキーマを使用してください。

### 6.1.1 旧形式（v1.0.0–1.1.0）

以前のバージョンでは、エディタデータはテンプレートリテラル内に生JSONとして格納されていました:

```typescript
const __mocEditorData = `
{
  "craftState": { ... },
  "memos": [ ... ],
  "viewport": { ... }
}
`;
```

パーサーは両方の形式をサポートすべきです: 内容が `brotli:` で始まる場合はBrotli解凍、それ以外は生JSONとしてエスケープ処理後にパースします。

### 6.2 フィールド

| フィールド | 必須/任意 | 型 | 説明 |
|-----------|-----------|------|------|
| `craftState` | **必須** | `Record<string, unknown>` | Craft.jsノードツリー |
| `memos` | **必須** | `MocEditorMemo[]` | 付箋メモの完全データ |
| `viewport` | 任意 | `{ mode, width, height }` | ビューポート設定 |

### 6.3 エスケープ規則（旧形式のみ）

旧形式（v1.0.0–1.1.0）の生JSONでは、テンプレートリテラル内で以下のエスケープが必要です:

| 文字 | エスケープ後 |
|------|-------------|
| `` ` `` (バッククォート) | `\`` |
| `${` (テンプレート式) | `\${` |

パーサーはこれらを復元（アンエスケープ）してからJSON.parseします。

Brotli圧縮形式（v1.2.0+）ではBase64出力に特殊文字が含まれないため、エスケープは不要です。

---

## 7. 正規化ルール

`.moc` ファイルを保存する際は以下の正規化を行います:

- **改行コード**: LF (`\n`)
- **末尾改行**: あり（ファイル末尾に1つの改行）
- **エディタデータJSON**: 2スペースインデント
- **セクション間**: 空行1行で区切り

---

## 8. 互換性ポリシー

### 8.1 廃止タグ

以下のタグは廃止されました。パーサーはこれらを検出しても無視します:

| 廃止タグ | 理由 |
|----------|------|
| `@moc-id` | ファイルコピー・テンプレート化時のID整合性管理が困難なため |
| `@moc-editor-data` (base64コメントブロック) | 人間・AI可読性が低いため、テンプレートリテラル形式に移行 |

### 8.2 未知フィールドの扱い

- メタデータブロック内の未知の `@moc-*` タグ → **無視する**（エラーにしない）
- エディタデータJSON内の未知フィールド → **無視する**（エラーにしない）
- 将来のバージョンで新しいフィールドが追加される可能性があるため、未知フィールドを見ても処理を中断しない

### 8.3 下位互換

本仕様（v1.0.0正式版）は暫定版との下位互換を**保証しません**。
旧形式ファイルは新バージョンのパーサーで正しく読み込めない場合があります。

### 8.4 v1.2.0の互換性

**エディタデータ形式:**
- パーサーは生JSON（v1.0.0–1.1.0）とBrotli圧縮（v1.2.0）の両形式を**サポートしなければならない**
- `brotli:` プレフィックスで2つの形式を区別する
- シリアライザーは常にBrotli圧縮形式（v1.2.0+）で出力**すべきである**
- v1.2.0+で保存されたファイルは、Brotli解凍をサポートするよう更新された古いパーサーでのみ開くことができる

**セクションデリミタマーカー:**
- v1.2.0以降、importブロックとTSXコンポーネントセクションはデリミタマーカーで囲まれる
- マーカー: `/* @moc-imports-start */` / `/* @moc-imports-end */` および `/* @moc-tsx-start */` / `/* @moc-tsx-end */`
- パーサーはこのマーカーを使用して確実にセクションを分割**すべきである**
- マーカーが存在しない場合（v1.0.0–1.1.0のファイル）は、ヒューリスティックベースの検出にフォールバックできる
- シリアライザーは常にデリミタマーカーを出力**すべきである**（v1.2.0+）

### 8.5 v1.2.1の互換性

**メモ一覧ブロック（`@moc-memos`）:**
- v1.2.1以降、JSDocヘッダーに `@moc-memos` ブロックで全付箋メモの一覧を出力する
- `@moc-memos` を認識しない旧パーサーは未知タグ処理ルール（§8.2）により安全に無視できる
- 同じメモ内容はTSX内の `{/* @moc-memo "..." */}` インラインコメントにも記載される — 旧パーサーでもこれらのコメントからメモを読み取れる
- シリアライザーはメモが存在する場合 `@moc-memos` ブロックを出力**すべきである**（v1.2.1+）

---

## 9. 完全なファイル例

```typescript
/**
 * Momoc Document (.moc)
 * VSCode拡張「Momoc」で作成されたGUIモックアップの定義ファイルです。
 * 人間・GUI・AIエージェントが協調して開発を行うための統合データ形式です。
 *
 * ...（ヘッダー説明文省略）...
 *
 * @moc-version 1.2.1
 * @moc-intent Login form mockup
 * @moc-theme light
 * @moc-layout flow
 * @moc-viewport desktop
 *
 * @moc-memos
 *   [node1] Title:メールバリデーション Message:メールバリデーション必須
 *   [node2] Title:ログインボタン Message:ログインアクション用の送信ボタン
 *
 */
/* @moc-imports-start */
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
/* @moc-imports-end */

/* @moc-tsx-start */
export default function LoginForm() {
  return (
    <>
      {/* @moc-node ROOT */}
      <div className="flex min-h-screen items-center justify-center">
        {/* @moc-node node1 */}
        {/* @moc-memo "Email validation required" */}
        <Input id="emailInput" type="email" placeholder="Email" />
        {/* @moc-node node2 */}
        {/* @moc-memo "Submit button for login action" */}
        <Button id="loginButton">Login</Button>
      </div>
    </>
  );
}
/* @moc-tsx-end */

const __mocEditorData = `
{
  "craftState": {
    "ROOT": {
      "type": { "resolvedName": "CraftContainer" },
      "props": { "className": "flex min-h-screen items-center justify-center" },
      "nodes": ["node1", "node2"],
      "linkedNodes": {},
      "parent": null
    }
  },
  "memos": [],
  "viewport": {
    "mode": "desktop",
    "width": 1280,
    "height": 800
  }
}
`;
```
