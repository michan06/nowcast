# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

気象庁の公開APIを利用した天気可視化Webアプリケーション。GitHub Pagesでホストされる静的HTMLファイル群で構成される。

## 技術スタック

- **フロントエンド**: 純粋なHTML/JavaScript（ビルドツール不要）
- **地図ライブラリ**: Leaflet.js 2.0.0-alpha.1（ESMモジュール）
- **日時処理**: moment.js 2.29.1
- **UIフレームワーク**: Bootstrap 5.0.2（index_top.htmlのみ）
- **地図タイル**: 国土地理院タイル

### Leaflet 2.0移行メモ（index_nowcast_rep.html）
- ESM importmapを使用してLeafletをインポート
- ファクトリメソッド廃止: `L.map()` → `new LeafletMap()`, `L.tileLayer()` → `new TileLayer()` 等
- `L.Control.extend({})` → `class extends Control {}` (ES6クラス構文)
- グローバル変数`L`は使用せず、必要なクラスを個別にインポート
- **重要**: Controlのposition設定は`super({ position: 'bottomleft' })`でコンストラクタに渡す（クラスフィールド`options = {}`では効かない）

## ファイル構成

| ファイル | 用途 |
|---------|------|
| `index_top.html` | ダッシュボード。各種天気サービスへのリンクボタンとiframeを配置 |
| `index_nowcast.html` | 気象庁ナウキャスト（雨雲レーダー）＋ひまわり赤外画像をLeaflet地図上に重畳表示 |
| `index_nowcast_rep.html` | index_nowcastの改良版。トゥルーカラー画像対応、昼夜自動切替、再生速度切替（1x/2x/4x）機能付き |
| `index_himawari_truecolor.html` | ひまわり衛星の全天トゥルーカラー再現画像のスライドショー表示 |

## 使用している気象庁API

- **ナウキャスト時刻リスト**: `https://www.jma.go.jp/bosai/jmatile/data/nowc/targetTimes_N1.json`
- **ナウキャストタイル**: `https://www.jma.go.jp/bosai/jmatile/data/nowc/{basetime}/none/{validtime}/surf/hrpns/{z}/{x}/{y}.png`
- **ひまわり時刻リスト**: `https://www.jma.go.jp/bosai/himawari/data/satimg/targetTimes_jp.json`
- **ひまわりタイル**: `https://www.jma.go.jp/bosai/himawari/data/satimg/{basetime}/jp/{validtime}/{type}/{z}/{x}/{y}.jpg`
  - type: `B13/TBB`（赤外）、`REP/ETC`（トゥルーカラー）
- **ひまわり全天画像**: `https://www.jma.go.jp/bosai/himawari/data/Fd_disk/sat_rep_{timestamp}.png`

## アーキテクチャの特徴

### ダブルバッファリング
`nowCastLayerArr`と`himawariLayerArr`で2枚のレイヤーを交互に使用し、タイル更新時のちらつきを防止。

### 自動更新
- ナウキャスト: 5分間隔
- ひまわり: 2.5分間隔
- 現在位置: 5分間隔（index_nowcast_rep.html）
- 🔄ボタン: 全データ＋現在位置を手動更新（初回位置取得成功時のみ）

### 昼夜自動切替（index_nowcast_rep.html）
5時〜18時はトゥルーカラー画像、それ以外は赤外画像に自動切替。
手動でモード切替した場合は自動切替が無効化され、昼夜が切り替わるか🔄ボタンを押すと再有効化。

### 再生機能（index_nowcast_rep.html）
- 過去データの連続再生機能（フレームスキップ方式）
- 再生速度: 1x/2x/4x（デフォルト2x）
- 再生中は⏸表示、停止中は▶️表示

### 背景地図切替（index_nowcast_rep.html）
- ひまわり表示時: 白地図（blankLayer）
- ひまわり非表示時: 衛星写真（seamlessPhotoLayer）でナウキャストデータを見やすく

### UIデザイン（index_nowcast_rep.html）
- 半透明ダークテーマ（背景: rgba(30, 30, 30, 0.85)）
- Leafletズームボタンも同じテーマに統一
- トグルボタン: アイコン+テキスト（🌧️ Nowcast, 🛰️ Himawari, 🌈 TrueColor/🌙 IR）
- OFF状態は半透明で状態を表現

## 開発・デプロイ

ビルド不要。HTMLファイルを直接編集し、GitHubにpushするとGitHub Pagesで公開される。

公開URL: `https://michan06.github.io/nowcast/`

### 奇数ズームレベル対応（index_nowcast_rep.html）
気象庁のナウキャストタイルは偶数ズームレベル（4, 6, 8, 10）のみ提供。
奇数ズームレベル（5, 7, 9）では`maxNativeZoom`を動的に変更し、1つ下の偶数レベルのタイルを拡大表示。
- `zoomstart`: ズーム開始レベルを記録
- `zoom`: ズームイン時は即座に`maxNativeZoom`を更新（高解像度タイル取得のため）
- `zoomend`: 最終的な`maxNativeZoom`を確定

## 注意事項

- 気象庁APIは公式ドキュメントが限定的。URLパターンはブラウザの開発者ツールで確認したもの
- APIの仕様変更により動作しなくなる可能性あり
- ナウキャストタイルは偶数ズームレベルのみ提供される
