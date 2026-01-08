# nowcast

気象庁の公開APIを利用した天気可視化Webアプリケーション

## 公開URL

https://michan06.github.io/nowcast/index_nowcast_rep.html

## 機能

### index_nowcast_rep.html（メイン）
- 気象庁ナウキャスト（雨雲レーダー）とひまわり衛星画像をLeaflet地図上に重畳表示
- トゥルーカラー/赤外画像の切替（昼夜自動切替対応）
- 過去データの連続再生（1x/2x/4x速度切替）
- 現在位置表示
- 半透明ダークテーマUI

### index_himawari_truecolor.html
- ひまわり衛星の全天トゥルーカラー再現画像のスライドショー表示

### index_top.html
- 各種天気サービスへのリンクダッシュボード

## 技術スタック

- Leaflet.js 2.0.0-alpha.1（ESMモジュール）
- moment.js 2.29.1
- 国土地理院タイル
- 気象庁API

## ライセンス

気象庁のデータは[気象庁ホームページについて](https://www.jma.go.jp/jma/kishou/info/coment.html)に基づき利用しています。
