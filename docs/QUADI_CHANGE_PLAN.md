<!--
SPDX-FileCopyrightText: 2026 Quadi Studio Contributors
SPDX-License-Identifier: GPL-3.0-or-later
-->

# Quadi Studio — 変更計画・開発運用基準

## 1. リポジトリ・ブランチ方針
- ベース：`Mixar-AI/mixar-app`（Blender 5.3 オーバーレイフォーク）のフォーク
- リポジトリ名：`Quadi-Studio`
- メイン開発ブランチ：`quadi-development`
- 編集環境：GitHub ブラウザ IDE（Codespaces 不使用）
- ビルド・テスト：GitHub Actions

## 2. 構造上の厳守事項（Overlay Build Model）
1. **`upstream/` は読み取り専用**：Blender 5.3 上流コードを変更しない。
2. **`source/` は生成物**：ビルド時に `upstream/` + `src/` から生成されるため直接編集・コミットしない。
3. **すべてのコード変更は `src/` 配下に行う**：
   - Python モジュール：`src/scripts/mixar/modules/` および将来の `src/scripts/` 拡張
   - C/C++ 拡張：`src/source/`、`src/intern/`
4. **ライセンス・著作権表示を保持する**：
   - `LICENSE`、`REUSE.toml`、`NOTICE.md`、`LICENSES/` および各ファイルの SPDX ヘッダーを削除しない。
   - `ucupaint`（GPL-3.0-or-later）および Blender（GPL-2.0-or-later）の帰属表示を維持する。
   - 非 GPL の Mixar ブランド資産（`LICENSES/LicenseRef-Mixar-Brand.txt`、`TRADEMARKS.md`）は Quadi Studio 資産へ完全置換する。

## 3. 最初にブラウザ IDE で実内容を確認すべき未確認ファイル一覧
コード変更に着手する前に、以下をブラウザ IDE で開いて実体を確認し、本ドキュメントへ記録する。

- [ ] `.github/workflows/` 配下の既存ワークフローファイル一覧と内容
- [ ] `Makefile`、`scripts/unix/build.sh`、`scripts/unix/settings.sh`、`scripts/unix/package_linux.sh`
- [ ] `scripts/windows/build.bat`、`scripts/windows/package.bat`
- [ ] `cmake/mixar_overrides.cmake`、`src/source/creator/CMakeLists.txt`
- [ ] `.env.example`、`VERSION`、`scripts/python_requirements.txt`
- [ ] `TRADEMARKS.md`、`LICENSES/LicenseRef-Mixar-Brand.txt`、`NOTICE.md`
- [ ] `src/scripts/startup/bootstrap/__init__.py`
- [ ] AI バックエンド接続先を定義する `src/scripts/mixar/modules/common/network/`、`auth/`、`byok/`、`space_mixie_chat/`

## 4. 段階的変更計画（Step-by-Step）

### Phase 1：CI 基盤の稼働確認（完了目標：テスト緑・ビルド計画確定）
1. `.github/workflows/quadi-lightweight-ci.yml` を `quadi-development` で実行し、標準ランナーで `pytest` 全件が通過することを確認する。
2. `.github/workflows/quadi-full-build-linux.yml` の実行先（GitHub Larger Runner または Self-hosted Runner）を用意し、実ビルド＋`linux_build_smoke.py`＋Artifacts 保存を通す。

### Phase 2：最小リブランディング（既存機能無削除・起動識別可能な状態）
※上記「未確認ファイル一覧」の実確認後に 1 箇所ずつ変更する。
1. アプリ名・バイナリ識別名・パッケージ名（`mixar` → `quadi-studio` 等）の定義箇所を特定し、互換性を壊さない範囲で段階置換。
2. Mixar ロゴ・アイコン・スプラッシュ・ウィンドウタイトルを Quadi Studio 資産へ差し替え（`LicenseRef-Mixar-Brand.txt` 準拠）。
3. `VERSION` 管理の初期化および起動確認（スモークテスト通過）。

### Phase 3：Quadi AI バックエンド抽象化
1. 既存の Mixar バックエンド通信（`https://api.mixar.app`）に依存する認証・セッション・チャット/エージェント呼び出しを抽象プロバイダインターフェースへ分離する。
2. BYOK / ローカル LLM 経路（`modules/byok/`, `modules/local_models/`）を維持しつつ、将来 Quadi AI バックエンドを差し込めるアダプタ層を新設する。
3. 既存のツール実行（`common/agent_execution/`）、シーン状態把握（`scene_graph`）、検証（`render_worker`）は維持・活用する。

### Phase 4：Quadi 独自機能群の段階的統合
1機能単位で PR / コミットを分割し、都度軽量 CI とビルドスモークテストを通しながら追加する。
1. `Quadi Model` / `Quadi Sculpt`（既存のモデリング・スカルプト UI/ワークフロー拡張）
2. `Quadi Paint`（既存 `paint` モジュール基盤の拡張）
3. `Quadi Procedural`（ジオメトリノード・プロシージャル統合）
4. `Quadi Cloth`（布・型紙・縫製ワークフロー基盤）
5. `Quadi Motion`（リギング・アニメーションワークフロー拡張）
6. `Quadi Render` / `Quadi Asset` / `Quadi AI`
