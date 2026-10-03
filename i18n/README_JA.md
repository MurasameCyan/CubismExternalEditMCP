# Cubism External Edit MCP

[![Cubism Editor](https://img.shields.io/badge/Cubism%20Editor-5.4%20Alpha-ff69b4)](https://www.live2d.com/cubism/download/editor/)
[![Python](https://img.shields.io/badge/python-%3E%3D3.10-blue)](https://www.python.org/)
[![MCP](https://img.shields.io/badge/MCP-1.0-8A2BE2)](https://modelcontextprotocol.io/)
[![PyPI](https://img.shields.io/pypi/v/cubism-mcp)](https://pypi.org/project/cubism-mcp/)

[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/nana7chi/CubismExternalEditMCP?style=flat)](https://github.com/nana7chi/CubismExternalEditMCP/stargazers)
[![GitHub last commit](https://img.shields.io/github/last-commit/nana7chi/CubismExternalEditMCP)](https://github.com/nana7chi/CubismExternalEditMCP/commits)

[中文](../README.md) | [English](README_EN.md) | 日本語 | [한국어](README_KO.md)

Live2D Cubism Editor の外部連携 API を **MCP (Model Context Protocol)** ツールとしてラップし、AI Agent が自然言語で Cubism Editor を操作できるようにします。

> 公式リファレンス：https://creatorsforum.live2d.com/t/topic/3938

## アーキテクチャ

```mermaid
graph TD
    AI["AI Agent"]
    MCP["cubism_mcp.py<br/>MCP サーバー, 45 ツール"]
    Editor["Cubism Editor 5.4 Alpha<br/>外部連携 API"]

    AI -->|"stdio (MCP プロトコル)"| MCP
    MCP -->|"WebSocket (ws://localhost:22033)"| Editor
```

## 機能

- **読み書き操作** — モデルのパラメータ値読み書き、ドキュメント一覧表示、編集モード取得（Editor 4.x 以降対応）
- **モデルの完全な検査** — パラメータ構造、パーツ構造、デフォーマ構造、個別オブジェクトの詳細（5.4 Alpha が必要）
- **編集操作** — パラメータ/パーツ/デフォーマ/アートメッシュ/グルーの追加・編集・削除、自動トランザクション処理（5.4 Alpha が必要）
- **バッチ編集** — 単一トランザクションで複数操作を実行、失敗時は自動ロールバック
- **権限の段階分け** — 参照には「許可」、編集には「編集」の承認が必要
- **自動再接続** — Editor 再起動後、3 秒間隔で自動再接続
- **トークンの永続化** — 認証トークンを `~/.cubism-mcp/token.txt` にキャッシュし、再認証を回避

## 要件

| コンポーネント | バージョン |
|--------------|-----------|
| Python | ≥ 3.10 |
| Cubism Editor | 5.4 Alpha（有効期限：2026-09-14） |
| OS | Windows / macOS |

## 使用方法

### クイックスタート

**以下のプロンプトを AI Agent にコピーして送信してください**：

> https://github.com/nana7chi/CubismExternalEditMCP/blob/master/README.md に従って、cubism-mcp をインストールして設定してください。このコンピュータに `uv` がインストールされていない場合は、先にインストールしてください。準備ができたら教えてください。


### ステップ 1：uv のインストール（初回のみ）

`uv` は軽量な Python パッケージマネージャで、本 MCP の自動インストールと実行に使用します。インストール後は Python 環境の管理が不要になります。

**macOS**（ターミナルに貼り付け）：
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows**（PowerShell に貼り付け）：
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

インストール完了後、**ターミナルを再起動**し、`uv --version` でバージョンが表示されれば成功です。

> uv をインストールしたくない場合：Python（≥3.10）をローカルに用意し、`pip install -r requirements.txt` で依存パッケージをインストール後、`python cubism_mcp.py` で実行することもできます。ただし、依存関係はご自身で管理してください。

### ステップ 2：AI Agent に MCP を設定

> `ClaudeCode`、`Codex`、`Workbuddy` など各種 MCP 対応クライアントで動作します。

> 初回起動時は依存パッケージの自動ダウンロードに約 1〜2 分かかります。以降は即時起動します。

#### 方法 1：PyPI インストール（推奨）

```json
{
  "mcpServers": {
    "cubism-mcp": {
      "type": "stdio",
      "command": "uvx",
      "args": ["cubism-mcp"],
      "description": "Cubism Editor MCP",
      "env": { "NO_PROXY": "localhost,127.0.0.1" }
    }
  }
}
```

#### 中国ミラー設定

PyPI 公式が遅い場合は、中国ミラー（清華大学/Alibaba/Tencent Cloud など）を使用できます：

```json
{
  "mcpServers": {
    "cubism-mcp": {
      "type": "stdio",
      "command": "uvx",
      "args": ["--index-url", "https://pypi.tuna.tsinghua.edu.cn/simple", "cubism-mcp"],
      "description": "Cubism Editor MCP",
      "env": { "NO_PROXY": "localhost,127.0.0.1" }
    }
  }
}
```

> Alibaba `https://mirrors.aliyun.com/pypi/simple` または Tencent Cloud `https://mirrors.cloud.tencent.com/pypi/simple` に置き換えも可能です。

#### 方法 2：uvx オンライン実行（GitHub ソース）

以下のMCP設定を追加：

```json
{
  "mcpServers": {
    "cubism-mcp": {
      "type": "stdio",
      "command": "uvx",
      "args": ["--from", "git+https://github.com/nana7chi/CubismExternalEditMCP.git", "cubism-mcp"],
      "description": "Cubism Editor MCP",
      "env": { "NO_PROXY": "localhost,127.0.0.1" }
    }
  }
}
```

#### 方法 3：ローカルクローン実行

1. ソースコードをクローン（またはZIPをダウンロードして解凍）

```bash
git clone https://github.com/nana7chi/CubismExternalEditMCP.git
```

2. 以下のMCP設定を追加（`cwd` を実際のパスに変更）：

```json
{
  "mcpServers": {
    "cubism-mcp": {
      "type": "stdio",
      "command": "python",
      "args": ["cubism_mcp.py"],
      "cwd": "J:/実際のパスに変更/CubismExternalEditMCP",
      "description": "Cubism Editor MCP",
      "env": { "NO_PROXY": "localhost,127.0.0.1" }
    }
  }
}
```

### ステップ 3：Cubism Editor で外部連携を有効化

1. Cubism Editor 5.4 Alpha を起動し、モデルを開く
2. メニュー「**ファイル**」→「**外部アプリケーション連携の設定**」
3. ポートが `22033` であることを確認し、「**使用**」トグルをオンにする
4. 認証ダイアログが表示されたら `cubism-mcp` を見つけ、**「許可」と「編集」にチェックを入れて** OK をクリック

> ダイアログが表示されない場合は、Editor 右下の点滅する外部アプリアイコンを確認してください。クリックするとダイアログが開きます。

![外部アプリケーション連携の設定](../外部应用程序集成的设置.png)

### ステップ 4：使用開始

AI Agent で自然言語を使って Editor を操作します。例：

```
「現在のモデルのパラメータ構造をリスト表示」
「パーツ階層を確認」
「眉毛パーツのラベル色を青に変更」
「パラメータ ParamsTest、ID ParamTest、範囲 0-1、デフォルト 0.5 を作成」
「ParamAngleX にキーフォームを 3 つ一括追加」
```

## 利用可能なツール

### 診断

| ツール | APIバージョン | 説明 |
|------|------------|------|
| `cubism_status` | 接続状態、登録状態、許可/編集の認証状態を確認 |

### 読み書き

| ツール | APIバージョン | パラメータ | 説明 |
|------|------------|-----------|------|
| `cubism_get_model_uid` | — | 現在開いているモデルの UID を取得 |
| `cubism_get_documents` | — | 開いているすべてのドキュメントを一覧表示（モデリング/物理演算/アニメーション） |
| `cubism_get_document` | `document_uid` | UID で単一ドキュメントの詳細を取得 |
| `cubism_get_current_edit_mode` | — | 現在の編集モードを取得（Physics/Modeling/Animation/…） |
| `cubism_get_parameter_values` | `model_uid`, `ids?` | モデルのパラメータ現在値を読み取り |
| `cubism_set_parameter_values` | `model_uid`, `parameters[]` | パラメータ値を書き込み（編集トランザクション不要） |
| `cubism_clear_parameter_values` | `model_uid` | SetParameterValues の一時バッファをクリア |
| `cubism_get_parameters` | `model_uid` | パラメータのメタ情報（名前/範囲/キーフォーム/タイプ） |
| `cubism_get_parameter_groups` | `model_uid` | パラメータグループ一覧 |

### 構造の検査（5.4 Alpha 新規）

| ツール | APIバージョン | パラメータ | 説明 |
|------|------------|-----------|------|
| `cubism_get_parameter_structure` | `model_uid` | 完全なパラメータ構造ツリー（グループ+パラメータ階層） |
| `cubism_get_part_structure` | `model_uid` | パーツ構造ツリー（アートメッシュ/デフォーマ/パーツ/グルー） |
| `cubism_get_deformer_structure` | `model_uid` | デフォーマ構造ツリー |
| `cubism_get_object` | `model_uid`, `id` | 指定オブジェクトの詳細を取得 |
| `cubism_get_selected` | `model_uid` | Editor で現在選択中のオブジェクト一覧を取得 |
| `cubism_get_parameter_keys` | `model_uid`, `object_id` | オブジェクトのキーフォーム関連を取得 |
| `cubism_get_objects_by_parameter_keys` | `model_uid`, `parameter_id`, `key_value` | パラメータのキーフォームから関連オブジェクトを逆引き |

### 編集（5.4 Alpha 新規）

| ツール | APIバージョン | パラメータ | 説明 |
|------|------------|-----------|------|
| `cubism_edit` | `action`, `params` | 単一の編集操作を実行（EditBegin/EditEnd を自動処理） |
| `cubism_edit_batch` | `actions[]` | バッチ編集（同一トランザクション、失敗時は自動ロールバック） |
| `cubism_add_selected_objects` | `model_uid`, `ids[]` | プログラムでオブジェクトを選択（既存の選択を保持） |
| `cubism_clear_selected_objects` | `model_uid` | すべての選択状態を解除 |

#### サポートされる編集 Action

| Action | 主要パラメータ | 説明 | プロンプト例 |
|--------|--------------|------|------------|
| `AddParameter` | `GroupId`, `ParameterName`, `ParameterId`, `Default`, `Minimum`, `Maximum` | 指定グループにパラメータを追加 | 「パラメータ'テスト'、ID ParamTest、範囲0〜1、デフォルト0.5を'表情切替'グループに作成」 |
| `EditParameter` | `Id`, `ParameterName`, `Default`, `Minimum`, `Maximum` | パラメータプロパティを編集 | 「ParamTest の最大値を 2 に変更」 |
| `DeleteParameter` | `Id` | パラメータを削除 | 「ParamTest パラメータを削除」 |
| `AddParameterGroup` | `GroupName`, `GroupId`, `ParentGroupId` | パラメータグループを追加 | 「パラメータグループ'テストグループ'を作成」 |
| `EditParameterGroup` | `Id`, `GroupName`, `LabelColorType`, `LabelCustomColor` | グループプロパティを編集 | 「'XYZ'グループのラベル色を青に変更」 |
| `DeleteParameterGroup` | `Id` | パラメータグループを削除 | 「'テストグループ'パラメータグループを削除」 |
| `MoveParameter` | `Id`, `NewGroupId`, `InsertPosition` | パラメータを新しい位置/グループに移動 | 「ParamTest を'XYZ'グループの先頭に移動」 |
| `MoveParameterGroup` | `Id`, `InsertPosition` | パラメータグループの順序を変更 | 「'眉毛'グループを先頭に移動」 |
| `AddParameterKey` | `ParameterId`, `KeyValue` | パラメータにキーフォームを追加 | 「ParamAngleX の 0.5 にキーフォームを追加」 |
| `DeleteParameterKey` | `ParameterId`, `KeyValue` | パラメータのキーフォームを削除 | 「ParamAngleX の -30 にあるキーフォームを削除」 |
| `MoveParameterKey` | `ParameterId`, `OldKeyValue`, `NewKeyValue` | キーフォームの位置を移動 | 「ParamAngleX の 0.5 のキーフォームを 0.8 に移動」 |
| `AddPart` | `Name`, `Id`, `ParentId` | パーツを追加 | 「'左目'の下にパーツ'瞳孔'を作成」 |
| `EditPart` | `Id`, `Name`, `LabelColorType`, `LabelCustomColor`, `Opacity` | パーツプロパティを編集<br>⚠️ ラベル色は `LabelColorType`+`LabelCustomColor` を使用。`LabelColor` ではありません | 「'眉毛'パーツのラベル色を青に変更」 |
| `AddWarpDeformer` | `Name`, `Id`, `ParentId` | ワープデフォーマを追加 | 「'前髪'の下にワープデフォーマを作成」 |
| `AddRotationDeformer` | `Name`, `Id`, `ParentId` | 回転デフォーマを追加 | 「'頭'の下に回転デフォーマを作成」 |
| `EditWarpDeformer` | `Id`, `Name`, ... | ワープデフォーマのプロパティを編集 | 「ワープデフォーマ'曲面2'の名前を'顔'に変更」 |
| `EditRotationDeformer` | `Id`, `Name`, `Angle`, `Scale`, ... | 回転デフォーマのプロパティを編集 | 「顔の回転角度を15度に変更」 |
| `EditArtMesh` | `Id`, `Opacity`, ... | アートメッシュのプロパティを編集 | 「アートメッシュ'左目ハイライト'の不透明度を50%に変更」 |
| `EditGlue` | `Id`, ... | グルーのプロパティを編集 | 「グルーオブジェクトのウェイトを調整」 |
| `DeleteObject` | `Id` | パーツパレットからオブジェクトを削除 | 「ID Warp999 のオブジェクトを削除」 |
| `MoveObjectOnPartsPalette` | `Id`, `NewParentId`, `InsertPosition` | パーツパレット内のオブジェクト位置を移動 | 「ワープデフォーマ'曲面2'を位置 0 に移動」 |

## ネイティブブリッジ（オプション）

公式の外部連携 API は限られた操作のみを提供します（例：回転デフォーマの軸位置は読み取りのみで書き込み不可）。
ネイティブ編集機能を拡張するには、オプションのブリッジコンポーネント `cubism_bridge.dll` を導入できます。

導入して Editor を再起動すると、以下のツールが追加されます：

| ツール | 説明 |
|------|------|
| `cubism_bridge_status` | ブリッジの導入・接続状態を確認 |
| `cubism_bridge_ops` | 登録済みのネイティブ操作一覧 |
| `cubism_bridge_invoke` | 指定したネイティブ操作を実行（ホワイトリストはブリッジ側で管理） |

> 未導入の場合、これら 3 つのツールは `BridgeNotInstalled` と導入手順を返します。他のツールには影響ありません。

**CubismBridge 0.4.0 は 61 個の操作を登録**します。形状編集、物理演算、画像書き出し、ドキュメント保存に加え、ワークスペース / パレット / ツール / キャンバス状態、レイヤー付き PSD、CMOX、SDK2/SDK3 モデルデータ、テクスチャアトラス作成、アニメーショントラック / パラメータキーフレーム / フォームアニメーション、動画書き出しに対応します。

- 物理演算の読み込みとグローバル設定の変更は、ネイティブの元に戻す / やり直しに対応します。CMO3 は設定グループと FPS を保存しますが、重力と風の保存には physics3 JSON を使用してください。
- 4 種類のネイティブターゲットと CAN3 の保存 / 再オープンに対応します。`editor.document.save` の `path` は未保存ドキュメントでは必須です。モデル / アニメーションの書き込み失敗時は再試行ダイアログを開かずエラーを返します。
- レイヤー付き PSD は元の素材と現在のポーズに対応し、CMOX と SDK2/SDK3 は実際のモデルアセットを書き出します。テクスチャアトラス作成は元に戻す / やり直しと保存 / 再オープンに対応しますが、配置はネイティブの初期配置であり、最適化配置ではありません。
- モデルトラック、明示したトラック名の保存、数値パラメータのキーフレームと補間、フレーム / シーン設定、フォームアニメーションへの移行に対応します。MOV/MP4/WebM のエンコードとデコードを検証済みですが、音声トラックは未対応です。既存のシーン配置を維持し、モデルを自動的に中央へ移動しません。
- ワークスペース切り替え、パレット表示、ツール切り替え、キャンバス / タイムライン設定に固定のネイティブ操作を提供します。失敗時の分離や編集モードの保護は、すべてのパレット項目が書き込み可能という意味ではありません。
- `editor.command.invoke` は型付き引数と `IDocument` の自動注入に対応します。ダイアログを開く可能性のあるコマンドには明示的な `allowDialog=true`、削除 / 終了などの破壊的操作にはさらに `confirm=true` が必要です。手動のメニュー操作は変更しません。
- `cubism_bridge_invoke` はネイティブ操作の上限 180 秒を含め、結果を最大 190 秒待ちます。起動確認と状態照会には従来の短い待機時間を使用します。Python アダプター更新後は MCP サービスを再起動してください。
- 送信後に接続が切断された場合や応答がタイムアウトした場合は `BridgeOutcomeUnknown` を返し、別ポートへ再送しません。操作が完了済み、または実行中の可能性があるため、再実行の前に Editor の状態を確認してください。
- 表示状態、階層選択、Solo の限定シナリオを検証済みです。`command_loadVisibleMap` は 2 つの表示状態を交換し、元に戻す履歴を追加しません。`command_solo` は開始時の対象を固定し、終了時に一時ロックを解除します。空の表示記録や空の選択では警告が出る場合があり、静的なコマンド分類は全入力でのダイアログ不表示を保証しません。

操作一覧は `cubism_bridge_ops` で確認できます。**登録済みであることは全ネイティブ操作の検証完了を意味しません**。42 個のコマンドに限定シナリオの動作証拠がありますが、残りのモデル操作、同一ドキュメントの複数ビュー、追加の境界条件は拡張中です。Solo の色 / 不透明度の 4 組み合わせは実際のキャンバスで検証済みで、通常のモデル書き出しにはこの表示効果は含まれません。モデルを閉じて再度開くと Solo とそのオプションはリセットされます。Editor 全体の完全対応は主張しません。

### ダウンロードとインストール（本 fork の Releases）

ブリッジは**単体 DLL** として配布され、`Live2D_Cubism.jar` を変更しません：
<https://github.com/MurasameCyan/CubismExternalEditMCP/releases>

| アセット | 説明 |
|---------|------|
| `cubism_bridge.dll` | ブリッジ本体（agent 内蔵。`%LocalAppData%\CubismPatch\` または Editor の `app\jre\bin\` に配置） |
| `jli_generic.dll` | 汎用 jli プロキシ：Editor の `app\jre\bin\jli.dll` として導入し、ブリッジを読み込みます |
| `install.bat` / `uninstall.bat` | 導入 / 削除。使い方：`install.bat "<Live2D Cubism のインストール先>"` |
| `SHA256SUMS.txt` | SHA-256 チェックサム |

導入：

**汎用チャネル**：`install.bat "<Live2D Cubism のインストール先>"` を実行し、Editor を再起動；

> ブリッジのバイナリは Cubism Editor 5.4.00 alpha2 向けです。大型アップデートでネイティブ API が変わった場合は再ビルドが必要になることがあります。

## トラブルシューティング

| 症状 | 原因 | 解決策 |
|------|------|--------|
| MCP ステータスが赤 | Python パス/依存関係/`cwd` の誤り | Python ≥ 3.10 の確認、依存パッケージのインストール、`cwd` パスの確認 |
| Editor に未接続 | Editor が未起動または外部連携が無効 | Editor を起動 → モデルを開く → ファイルメニューから外部連携を有効化 |
| 未認証 | ダイアログで「許可」が未チェック | 外部連携ダイアログで「許可」をチェック |
| 編集エラー | ダイアログで「編集」が未チェック | 外部連携ダイアログで「編集」をチェック |
| 再起動後に動作しない | Editor 再起動時に再認証が必要 | 外部連携を再度有効化し、権限を再チェック |
| 操作エラー | パラメータ/ID が誤り | 先に `cubism_get_*_structure` で構造を確認してから操作 |

## 開発

```bash
# テスト用に直接実行
python cubism_mcp.py

# 依存パッケージのインストール
pip install -r requirements.txt
```

### 依存パッケージ

| パッケージ | 用途 |
|---------|------|
| `mcp` | MCP サーバーフレームワーク（FastMCP + stdio 通信） |
| `websockets` | WebSocket クライアント、Editor API 接続用 |

## 注意事項

- **Alpha 版の制限**：Cubism Editor 5.4 Alpha の有効期限は 2026-09-14 です。期限後はアップグレードが必要です
- **再起動時の再認証**：Editor を再起動するたびに、外部連携の再有効化と権限の再チェックが必要です
- **単一モデル**：MCP サーバーは同時に 1 つのモデルのみ操作できます
- **トランザクションの安全性**：編集操作は自動的に `EditBegin/EditEnd` でラップされ、バッチ操作は失敗時に自動 `Cancel` でロールバックします

## ライセンス

MIT
