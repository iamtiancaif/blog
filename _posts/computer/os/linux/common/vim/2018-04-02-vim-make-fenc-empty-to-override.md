---
layout: post
title:  "vim make 'fenc' empty to override"
categories: computer
tags:  computer os ruby jekyll copy
author: web 
source: http://www.hachim.jp/study-by-incident/file-encode.html
---

* content
{:toc}



[VIM ：E513: WRITE ERROR, CONVERSION FAILED (MAKE ‘FENC’ EMPTY TO OVERRIDE) 解決方法と改善策(文字コードまわり)](http://www.hachim.jp/study-by-incident/file-encode.html)
-------------------------------------------------------------------------------------------------------------------------------------------------------

[](http://www.hachim.jp/study-by-incident/file-encode.html)[2015-06-21](http://www.hachim.jp/study-by-incident/file-encode.html)

[vim](http://www.hachim.jp/vim) / [事象から解像度をあげる](http://www.hachim.jp/study-by-incident)

久々に遭遇し、何をー！と一瞬おもいましたが、  
本事象に遭遇する人いるだろな～と思い、備忘録として残しときます。

**■vimを保存しようとした時**

"www.hachim.jp-access.log.20150619" E513: write error, conversion failed (make 'fenc' empty to override)
Press ENTER or type command to continue

**■本事象の根本原因**  
**file encoding の不整合**です。

ファイルエンコードがshift_jisの場合には”\\”がエラーになります。  
“\\”を使うなら、fencを削除する必要があります。  
（追記：というかutf-8とかeucなら大丈夫）

**■解決方法**  
**make ‘fenc’ empty to override**  
ご指摘どおり、fencに空(カラ)でオーバーライド（上書き）する。

エラーがでたコマンドモード画面を閉じず、

	:set fenc=

⇒ ENTER

	:wq

⇒ ENTER

以上で、解消されます。
<!--more-->
**■再発防止策**  
打開策としては  
.vimrcに書き文字コードを複数設定しておくことで本事象はまず遭遇しないでしょう。

	set fileencodings=sjis,utf-8,iso-2022-jp,euc-jp

今回のほかにも文字コードによる問題がありますよね。  
せっかくなので、文字コード周りを再度おさえて解像度をあげましょう。

たとえば、vimは標準で文字コードの自動判別や変換に対応しているので、  
vimやviewを使えば、UTF-8環境のフロントエンドサーバでもEUCのファイルを編集したり表示することができます。

WindowsのPCとのファイル転送を行う場合に、改行コードの変換が必要となりますが、  
改行コードについても文字コードと同様にvimでの自動判別や変換が可能です。

vimの文字コード自動判別を有効にするには、“.vimrc”を設定する必要があります。

**.vimrcの設定**

	set encoding=utf-8
	set fileencodings=iso-2022-jp,euc-jp,sjis,utf-8

**改行コードの自動認識**

	:set fileformats=unix,dos,mac

この設定により、日本語の文字コードを自動判別し、UTF-8環境で編集や表示が可能となります。

**.vimrcの改行コードがUNIXとなっていない(Windows PCで作成した .vimrc をサーバに転送した場合など)と、  
vimを起動した場合に、エラーとなります。**

/home/hachim/.vimrc の処理中にエラーが検出されました:
行    1:
E474: 無効な引数です: encoding=utf-8**^M**
行    3:
E474: 無効な引数です: fileformats=unix,dos,mac**^M**
続けるにはENTERを押すかコマンドを入力してください

この様な場合は、vim .vimrc としてファイルを読み込み(ENTERを押す)、以下のコマンドで、改行コードを変換して保存してください。。

	:setl ff=unix
	:wq

文字コード、改行コードを指定して保存

vimでファイルを開き、任意の文字コードでファイルを保存することができます。  
vimでファイルを開き、vimのコマンドモードで以下のコマンドで文字コードを指定すると、保存するときに、指定した文字コードに変換されます。 これにより、**nkfなどのコマンドを使用せずにファイルの文字コードを変換する**

ことができます。

**文字コードを指定**

	:setl fenc=文字コード
	  (setl=setlocal, fenc=fileencoding)

**改行コードを指定**

	:setl ff=改行コード
	  (setl=setlocal, ff=fileformat)

上記、文字コードないし改行コードを指定した後に、ファイルを保存(:wq)します。

**■文字化けしたファイルを開き直し、自動判別に失敗した場合**  
文字コードを指定してファイルを開き直すことができます。

コマンドモードで以下コマンドでファイルを開き直します。

:e ++enc=文字コード

vimでファイルを開き、^Mという文字が表示される場合、改行コードを変更してファイルを開き直します。

:e ++ff=改行コード

上記、文字コードないし改行コードを指定した後に、ファイルを保存(:wq)します。

以上、文字コード関連でありがちなものをまとめてみました。






