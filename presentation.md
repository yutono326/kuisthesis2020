
これから，「スマートコントラクトのガス消費量のResource Aware MLを用いた静的解析」という題目で発表させて頂きます．
# 背景

## ブロックチェーンとスマートコントラクト
ブロックチェーンで用いられる技術としてスマートコントラクトがあります．
スマートコントラクトは，ブロックチェーン上で動作するプログラムとして実装され，仮想通貨の取引における計算などを自動化する仕組みです．
スマートコントラクトは，それぞれのブロックチェーンごとに決められたプログラミング言語で記述されています．
例えば，Ethereumの言語としてSolidity，Tezosの言語としてMichelsonやSCamlがあります．


## ガス
スマートコントラクトにはガスの概念が存在します．
ガスはコントラクトが実行のために利用する計算資源にかかる手数料を表しています．
コントラクトの実行において，各命令の実行ごとに，その計算コストに応じた量のガスが消費されます．
消費量の合計が通貨に換算され，手数料として支払われます．

## 研究の動機
このガスの消費量が上限値を超えると，プログラムの実行が停止されます．
このとき，実行前に一定量の通貨をあらかじめ支払う必要がありますが，これが返金されずに無駄なコストとなってしまいます．
これらの観点から，ガスの消費量を静的に解析する手法が求められています．

# 本研究の概要
本研究では，仮想通貨Tezosのスマートコントラクトのガス消費量を静的に解析する手法を提案します．
具体的には，Michelsonで記述されたプログラムをもとにガス消費量の見積もりを行います．
解析には，Resource Aware ML(RAML)という，プログラムのリソース消費量の解析を行うことができる関数型プログラミング言語を用います．
方針としては，まず，MichelsonプログラムをRAMLプログラムに手動でエンコードします．
続いて，エンコードしたプログラムをRAMLで解析することでガス消費量の見積もりを行います．


# Resource Aware ML
手法の説明の前に，RAMLの説明をします．
RAMLは，OCaml文法を備えた関数型プログラミング言語で，
入力として与えられたプログラムのリソース消費量の解析を行う機能を備えています．
解析においてメトリックを指定することで，様々なリソース消費量の解析を行うことができます．
本研究では，そのうちの一つであるtickメトリックを用います．
これは，ユーザーの定義したリソース消費量を解析するメトリックで，プログラム中にtick関数というRAMLに備えられた関数を呼び出すことで，関数のリソース消費を表現することができます．

## RAMLのプログラム例
これは，intのリストの和を計算する簡単なRAMLプログラム例です．
基本的にはOCamlプログラムと同じですが，これはtick関数で，先ほどのtickメトリックを用いる際に関数のリソース消費量を定める関数です．
また，let _ ...から始まる式はmain式で，解析の対象となる式です．

## 例の解析結果
これは，このプログラムを解析を行った出力の一部です．
解析の際にメトリックや解析の次数の範囲といったオプションをコマンドラインで指定することができます．
解析の結果として，main式に用いられた関数のそれぞれの解析の結果が示されます．
これは関数のリソース消費量を表す多項式です．
そして一番下に，main式のリソース消費量の上界が示されます．

# 手法の説明
このRAMLを用いてMichelsonプログラムの解析を行う手法を説明します．

## Michelsonプログラム
Michelsonはスタックベースのプログラミング言語で，
Michelsonのコントラクトは，初期スタックに積まれるparameterとstorageという値の型宣言と，命令列によって構成されています．
そして各命令は受け取ったスタックを書き換えます．
例としてADD命令に注目すると，命令が適用される前のスタックはこのようにintが2つ積まれています．これにADD命令が適用されると2つのint取り出され，それらの足された値がスタックに積まれます．


## RAMLライブラリ
このようなMichelsonプログラムをエンコードするために，Michelsonの各命令の挙動を模倣する関数ライブラリをRAMLで実装します．
スタックの要素をヴァリアント型tとして定義することで，スタックを型tのリスト，
命令を(t list → t list)型をもつ関数として定義することができます．
例えばADD命令を模倣する関数はこのようになります．
関数の定義中に，その命令のガス消費量に相当するtick関数を呼び出します．
このように設計したライブラリを用いて，Michelsonプログラムを，初期スタックを表すリストに対して関数を連続適用するプログラムとしてエンコードできます．
エンコードしたプログラムをtickメトリックで解析することで，ガス消費量の見積もりを行います．

# 解析例

いくつかのMichelsonプログラムについて，それぞれをエンコードしたRAMLプログラムを作成に対して解析を行いガス消費量の見積もりを行いました．
それぞれのプログラムの内容と解析結果はこの通りです．
example3とexample5ではガス消費量を正しく見積もることができました．
一方で，example1とexample2では見積もりに多少の誤差が見られましたが，
これは整数値演算の命令のガス消費量がスタック中の値に依存しているという点を正しく見積もることができなかったと考えられます．
example4のようなリストに対する再帰を行う命令を含むプログラムは解析自体が行えませんでした．
以上のような正しく見積もりが行えない命令を見直すことは，今後の課題といえます．


# まとめ
本研究のまとめと今後の課題はご覧の通りです．発表を終わります．ありがとうございました．

本研究のまとめです．
本研究ではMichelsonの各命令を模倣するライブラリをRAMLで作成し，そのライブラリを用いてMichelsonプログラムをRAMLでエンコードしました．
エンコードしたRAMLプログラムをtickメトリックを用いて解析することで，コントラクトのガス消費量を見積もりました．
今後の課題として，実装したライブラリの拡張などが挙げられます．