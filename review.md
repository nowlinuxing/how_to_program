レビュー
=============

# はじめに

## 品質について

品質が低い状態でリリースしてしまうと不具合修正が定常的に発生し開発力を徐々に削いでゆく。

## 不具合について

リリース前であれば不具合臭修正はごく狭い範囲の関係者への共有だけで済むが、リリース後だとそのコストはかなり高くなる。

* メンテナンス
  + データのマイグレーション
    - 多くの場合、全削除＆再作成よりも大変
  + ダウンタイムによる機会損失、ユーザの印象悪化
* 信用↓
  + チェック、報告の頻度↑
* 修正計画の作成、周知
  + ユーザへのお知らせ
  + 隣接するシステムの担当者との共有・調整
* 作業者のストレス
  + 再び間違えることはできないというプレッシャー
  + リスケできない緊急作業（深夜休日出勤）
  + 追加の修正作業による開発リソース圧迫
    - 通常開発作業の遅延
    - 予測できない不具合。「スケジュールとは」
  + 影響範囲が広すぎる場合、あるべき姿に修正できない
    - 不具合の仕様化、固定 -> メンテナンスコスト増大

リリース前に発見・修正した方がトータルの工数は抑えられる。
修正コストは高いものから

1. ユーザデータ
2. マスターデータ
3. プロダクトコード

となるため、上の項目にはより注意する。

# レビュー観点

## 設計

* 心地よいインターフェースか
  + 引数の順序、拡張性
* DB
  + 桁あふれの恐れはないか
    - Big int, Unsigned int
  + 過剰でないか
    - TEXT -> VARCHAR
    - DATETIME -> DATE
    - VARCHAR の長さ指定
    - FLOAT -> DECIMAL
  + インデックス
    - カーディナル
    - UNIQUE 制約
    - カバリングインデックス
  + パーティショニング
  + 正規化
    - 迷ったら正規化
    - 非正規化する場合はメリットとデメリットを説明する
* REST
  + 標準的な REST に落とし込めないか
  + カスタム URL でシンプルにできないか
  + 動詞は名詞(リソース名)にできないか
* 規模を見積もる
  + 運用1年後、3年後、5年後、10年後も問題なく動作するか
    - 蓄積されるデータ
    - ユーザ数純増による同時アクセス数増加
  + スケールアウトできるか
    - ユーザ数が突然n倍に増えたら？
* ワーストケースに備える
  + うまくいかなかった時の影響範囲
  + 復旧のために何が必要になるか

## コード

* 名前にこだわる
  + 抽象的すぎないか
  + 長すぎないか
  + 英語としてもっといい表現はないか
  + 和製英語は避ける (辞書を引いて調べる)
* コメント
  + コードに出てこない情報はコメントとして残す
    - なぜそうなっているのか、なぜそう実装しなかったのか
* 異なるものを混ぜない
  + 参照メソッドと更新メソッド
    - 障害解析に参照メソッドだけ使いたいことはよくある
    - 障害対応に更新メソッドだけ使いたいことはよくある
    - それそれ別個に改修したくなることはよくある
  + データ生成時点で区別できるものはなるべく分別しておき、必要に応じて混ぜる
* コードの臭いに敏感になる
  + 読みづらくないか
  + ミスリードになっていないか
  + 解釈可能な幅が広すぎないか
* 繰り返しコード
  + ドメインに依存しないもの:
    - 標準ライブラリ、フレームワーク、外部ライブラリに支援するものがないか探してみる
    - ↑ にない場合、沿うべき定石から外れていないか再確認する
  + ドメインに依存するもの: 共通モジュールへの抽出を検討する

## コメント

* 実装との乖離がないようにする
  + 実装とコメント、どちらか一方が更新されたらもう一方を再チェックする

## テスト

* 同値分割、境界値
  + 代表値だけになっていないか
* カバレッジ
  + 場合分けしきれない場合は複数のメソッドへの分割、モジュール抽出、モデル抽出などで対応
* private メソッド
  + 直接テストしていないか
* 複数の異常値を同時にテストしていないか
* ダブルを乱用していないか
  + 落ちるべきテストが落ちず本番で初めて発覚する
  + 初実装時は手厚くテストされる。がそれ以降は手薄になる -> デグレ
* プロダクトコードを一行コメントアウトするとテストが落ちるか
  + 観点のみ。全てのプロダクトコードについて確認するのは現実的でないので
    - nil を返したり「例外が上がらないこと」だけを確認するテストは対象となるコードを通っていなくても Green になってしまうので特に注意。まず落ちるテストを書く

## コミット

VCS を使っている場合はコミットにも注意を払う。

* git rebase でマージ先を最新にする
* 1つのブランチ内に誤コミットと修正コミットが入っていないか
  + 歴史を修正し誤コミットをなくす
    - レビューしやすくする
    - コンフリクトしにくくする、あるいは解決しやすくする
    - 履歴を追う時のノイズを減らす
  + OSS の世界ではコミットが汚いだけで reject されることも
* git bisect
* コミットコメントのフォーマット
  + git の場合
    - 1 行目: title line
    - 2 行目: 空行
    - 3 行目以降: full commit message。コミットの詳細

## 改修

* ライブラリのバージョンは適切か
  + 互換性が損なわれていないか
  + 最新バージョンの確認
  
## 外部サービス

* テストの mock/stub 化

## 不具合修正

* コードの対応
  + プロダクトコードとテストコードは対になっているか (プログラマが知るべき97のこと 104「不具合にテストを書いて立ち向かう」)
    1. 最小レベルのテストコードで再現、落ちることを確認
    2. プロダクトコードを修正、通ることを確認
    3. 他のテストコードが通ることを確認

## その他

* 秘匿情報の隠蔽
  + パスワード、秘密鍵、APIキー、token などがコミットに含まれていないか
* 一次ソースで確認する
  + ググってひっかかってきた非公式ページの情報は参考にとどめる
  + がんばって英語を読む

## 参考

* [プログラミング原則一覧 - Strategic Choice](http://d.hatena.ne.jp/asakichy/20100203/1265158263)
* [デザインパターン (ソフトウェア) - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3_(%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2))
* アンチパターン
  + [アンチパターン - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%A2%E3%83%B3%E3%83%81%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3)
  + [AntiPatterns - Wikipedia, the free encyclopedia](https://en.wikipedia.org/wiki/AntiPatterns)
  + [AntiPatterns](http://antipatterns.com/)
* コードの臭い
  + [コードの臭い - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%BC%E3%83%89%E3%81%AE%E8%87%AD%E3%81%84)
  + [Code Smell](http://c2.com/cgi/wiki?CodeSmell)
* [Martin Fowler's Bliki (ja)](http://bliki-ja.github.io/)
* [あれから 10 年。まさーるさん(石井勝さん)を偲ぶ。 - t-wadaのブログ](http://t-wada.hatenablog.jp/entry/masarl-memories)
  + [まさーるのページ](http://objectclub.jp/community/memorial/homepage3.nifty.com/masarl/)

* [Railsコントリビュータへの道 · GitHub](https://gist.github.com/willnet/ca970b2033846fe3e30a8696eb39fbd5)
* [リファクタリングの勧め：柴田 芳樹 (Yoshiki Shibata)：So-netブログ](http://yshibata.blog.so-net.ne.jp/2015-09-27)
* [レビューしやすいコミット履歴でバグ削減 | Money Forward Engineers' Blog](https://moneyforward.com/engineers_blog/2015/11/30/reviewable-commit-log/)
