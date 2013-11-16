# さぁ、唄ってgolang...

筆者は普段はRubyistとして、Rubyでアプリケーションを書いたりfluentdのプラグインを作ったりしています。
新しい言語を使えるようになって芸事の幅を広げようと思い、最近話題を聞くことが増えてきたgolangに挑戦してみました。

この記事では、golangに興味はあるけど、どこから始めたらいいだろうか、即物的に役に立てるものを作るにはどうしたらいいだろうかという悩みを持つプログラマーを対象として、golangをHello Worldからはじめて、テスト実行、普段から使える実用コマンドを作るに至る道筋と、その中で困った事の試行錯誤について解説します。

## 目次

+ はじめる前に
+ 早速goをインストールして実行してみる
+ テストを書きながら敗北を知る
+ golangの書き方を覚える
+ 実務と同じくパッケージ分けしてみる
+ 普段から使っているWebサービスを叩く実用的なコマンドを作る
+ 作ったコマンドをゴルーチンを使ってマルチスレッドっぽい動きに変える
+ golangの地平線を目指して

## はじめる前に

今回の方針としては、事前に使いこなし記事やリファレンスを精読せず、必要になったら参照しつつ、少しずつつまづきながら進めていこうと考えました。
攻略方法に頼ってゲームを進めるのはつまらないものです。このgolangの習得についてもたのしくゲームを進めるのとおなじように、こんな進め方をとったのは、エンタティメントとしてあえてつまづきながら少しずつ謎を解いていくように進めていきたいと考えたのです。

なお、環境は下記を利用してます

//TODO mac、goのバージョン、homebrew、IDE http://www.winsoft.sk/go.htm

## 早速goをインストールして実行してみる

では早速やってみましょう。homebrew経由でお手持ちのmacに簡単に入ります。いい時代になりましたね。

````sh
brew install go
````

インストール完了。
これでgoコマンドが入るので、ヘルプを見てみましょう。多分--helpで見れるでしょうね。

````sh
$ go --help
````

案の定ですね。

````
Go is a tool for managing Go source code.

Usage:

	go command [arguments]

The commands are:

    build       compile packages and dependencies
    clean       remove object files
    doc         run godoc on package sources
    env         print Go environment information
    fix         run go tool fix on packages
    fmt         run gofmt on package sources
    get         download and install packages and dependencies
    install     compile and install packages and dependencies
    list        list packages
    run         compile and run Go program
    test        test packages
    tool        run specified go tool
    version     print Go version
    vet         run go tool vet on packages

Use "go help [command]" for more information about a command.

Additional help topics:

    gopath      GOPATH environment variable
    packages    description of package lists
    remote      remote import path syntax
    testflag    description of testing flags
    testfunc    description of testing functions

Use "go help [topic]" for more information about that topic.
````

どうやらgit、gem、bundlerなどのコマンドと同様にサブコマンドで制御していくやり方のようですね。

では、インストールしたgoコマンドでこれから開発を進めていけるのか、実際に確認してみましょう。
http://golang.org/doc/install#testing を参考に、Hello Worldにてgoの世界を動かしてみます。

下記のコードをgo_world.goとして保存し、実行してみます。

````go
package main

import "fmt"

func main() {
    fmt.Printf("Go World!\n")
}
````

実行はgo runです。コンパイルのステップを意識せず動かすことができます。

````sh
$  go run go_world.go 
````

````
Go World!
````

無事動作しました。Wellcome to Go World...

では喜び勇んで次に進みましょう。


## テストを書きながら敗北を知る

go --helpを実行したとき、testという項目が目に留まりました。

````
    test        test packages
````

golangにははじめからテストの仕組みが備わっているのですね。素晴らしいですね。
テストコードは実行結果の成功・失敗がわかりやすく、開発をドライブさせることができます。
なのでHello worldの次はテストを動かしていきましょう。

http://golang.org/doc/code.html#Testing にテストの書き方が載っているので、あるのでやってみます。

テストコードは＜テスト対象ファイル名＞_test.goって名前をつけて、下記を書いていくとのこと。

- テスト対象と同じpackageを指定する
- testingをimportする
-  *testing.T型を引数に取る名前がTestから始まる関数を作る
-  *testing.TのError関数が呼び出されるとテストを失敗にさせられる

実行は、テストコードのあるディレクトリでgo testとだけ打てばいいとのこと。

では、先ほどのgo_world.goに対応するテストコードとして、go_world_test.goを作ってみます。

とはいえ、標準出力のテストは作りにくいだろうので、別のテストを書きやすい仕様で進めていきます。

テストを書きやすいというとやはり数の計算。名前はDouble関数として、int型の引数をとり、２倍にして返すという仕様とします。

では、まずテスト対象が無いために失敗するところから始めましょう。

````go
package main

import (
	"testing"
)

func TestDouble(t *testing.T) {
	param := 1
	result := Double(param)
}
````

上記をgo_world_test.goという名前で保存し、テスト実行します

````
 go test
````

````
# /path/to/go_world
./go_world_test.go:9: undefined: Double
FAIL	/path/to/go_world [build failed]
````

undefined: Doubleとあるので、想定通りテスト対象が無いためにテストが失敗しました。
では、Double関数をgo_world.goに書いて再実行しましょう。

````go
package main

import "fmt"

func main() {
	fmt.Printf("Go World!\n")
}

func double(i int) int {
}
````

テスト実行。

````
# /path/to/go_world
./go_world.go:9: function ends without a return statement
FAIL	/path/to/go_world [build failed]
````

テスト対象ができたので、Doubleが無いというエラーではなくなりました。でも何かreturnせよと怒られました。
このエラーを解きましょう。go_world.goで固定で0でも返せば通るようになるでしょう。

````go
package main

import "fmt"

func main() {
	fmt.Printf("Go World!\n")
}

func Double(i int) int {
	return 0
}
````

テスト実行。

````
# /path/to/go_world
./go_world_test.go:8: param declared and not used
./go_world_test.go:9: result declared and not used
FAIL	/path/to/go_world [build failed]
````

何かreturnせよというエラーはでなくなりました。では、仮実装に進んでいきましょう。
仕様上では、1を渡すと倍の2が返るはずなので、そのとおりにgo_world_test.goに書きましょう。

````go
package main

import (
	"testing"
)

func TestDouble(t *testing.T) {
	param := 1
	result := Double(param)
	if result != 2 {
		t.Errorf("Double(%d) = %d, want %d.", param, result, 2)
	}
}
````

テスト実行。

````
# /path/to/go_world
--- FAIL: TestDouble (0.00 seconds)
go_world_test.go:11: 	Double(1) = 0, want 2.
FAIL
exit status 1
FAIL	/path/to/go_world [build failed]
````

固定で0を返しているので当然失敗します。go_world.goでは2を返すようにしましょう。

````go
package main

import "fmt"

func main() {
	fmt.Printf("Go World!\n")
}

func Double(i int) int {
	return 2
}
````

テスト実行

````
PASS
ok  	/path/to/go_world [build failed]
````

あっさり通りました。では1以外の数字を渡すというのを試しましょう。go_world_test.goに2を渡したら4が返ってくるケースを追記しましょう。

````go
package main

import (
	"testing"
)

func TestDouble(t *testing.T) {
	param := 1
	result := Double(param)
	if result != 2 {
		t.Errorf("Double(%d) = %d, want %d.", param, result, 2)
	}
}

func TestDouble2(t *testing.T) {
	param := 2
	result := Double(param)
	if result != 2 {
		t.Errorf("Double(%d) = %d, want %d.", param, result, 2)
	}
}
````
テスト実行

````
# /path/to/go_world
--- FAIL: TestDouble2 (0.00 seconds)
go_world_test.go:19: 	Double(2) = 2, want 4.
FAIL
exit status 1
FAIL	/path/to/go_world [build failed]
````

無事失敗しました。何を渡しても倍が返るようにgo_world.goを書き換えましょう。

````go
package main

import "fmt"

func main() {
	fmt.Printf("Go World!\n")
}

func Double(i int) int {
	return i * 2
}
````
テスト実行

````
PASS
ok  	/path/to/go_world [build failed]
````

パーフェクト…！

標準でこういう仕組みがあるのは便利なんですが、テストコードの中で都度都度ifで想定通りかチェックして、想定外ならt.Errorを呼び出さなくてはなりません。
http://golang.jp/go_faq#testing_framework によるとassertは良くないとの仰せですが、都度分岐いれていくのは書いていく歯車がズレてしまいます。
せっかくなので、xUnitでお馴染みのAssertほげほげを作ってみましょう。

go_world_test.goに、expectとactualとErrorを呼び出すための*testing.Tを受取って、expectとactualが同じでなければエラーとする関数を作ってみます。

````
func AseertEquals(expect, actual, t *testing.T) {
	if expect != actual {
		t.Errorf("%s =! %s", expect, actual)
	}
}
````

という関数を追記して、テストケースからはAseertEqualsの呼び出しに変更してみました。

````
package main

import (
	"testing"
)

func TestDouble(t *testing.T) {
	param := 1
	result := Double(param)
	AseertEquals(2, result, t)
}

func TestDouble2(t *testing.T) {
	param := 2
	result := Double(param)
	AseertEquals(4, result, t)
}

func AseertEquals(expect, actual, t *testing.T) {
	if expect != actual {
		t.Errorf("%s =! %s", expect, actual)
	}
}
````

おお、これは簡潔。やはり分岐は汚らしい。

じゃあテスト実行。

````
# /path/to/go_world
./go_world_test.go:10: cannot use 2 (type int) as type *testing.T in function argument
./go_world_test.go:10: cannot use result (type int) as type *testing.T in function argument
./go_world_test.go:16: cannot use 4 (type int) as type *testing.T in function argument
./go_world_test.go:16: cannot use result (type int) as type *testing.T in function argument
FAIL	/path/to/go_world [build failed]
````

もしかして、expect, actual, t は*testing.T型であると解釈された…？
今はintを指定すればいいのだろうけど、実際には文字列か整数かはたまた別物か、何がくるかわからないわけで、どうしたらいいのだろう。
静的に型が決まる言語とは聞いていたけど、Errorfみたいに数字でも文字でもなんでも取るもののがあるわけで、どうにかできるはず。

よし、コードを読もう。http://golang.org/src/pkg/testing/testing.go だ。

````go
// Errorf is equivalent to Logf followed by Fail.
func (c *common) Errorf(format string, args …interface{}) {
	c.log(fmt.Sprintf(format, args…))
	c.Fail()
}
````

ワカラヌ。

これまでサクサク進んできたのに一気に難易度が上がってしまった感じ。
これは挫折のピンチ。焦る。辛い。

## golangの書き方を覚える

慌てても焦っても理解できるようになるわけでなし、一つ一つの要素を解きほぐしていきましょう。

````go
// Errorf is equivalent to Logf followed by Fail.
func (c *common) Errorf(format string, args …interface{}) {
	c.log(fmt.Sprintf(format, args…))
	c.Fail()
}
````

golangでは、関数名の後ろの小かっこに引数を、小かっこの後ろに戻り値の型を書くというのはなんとなくわかっていました。ただ、Errorfの場合は戻り値は無いので、小かっこの後ろは即ブランケットになっている、と。

わからないのは、関数名の前の小かっこと、第二引数の型の...interface{}とは何者かというものです。

まず関数名の前の小かっこについて http://golang.jp/effective_go#methods を引いてみました。

````
これにはまず、メソッドと結びつけるために新たに名前付きの型を宣言し、メソッドにこの型のレシーバを定義します。

type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // 本体部分は前と全く同じ
}
````

つまり、上記Errorfの場合は、(c *common) Errorfとかいているので、*commonにErrorfを生やしているという事なんですね。
そして、*commonにはcという名前でアクセスすることができるということか。なるほど。

次に、...interface{}について調べていたら、 http://golang.jp/effective_go#printing にたどり着きました。

````
Printfのシグネチャの最後の引数には...interface{}型が使われているため、フォーマット指定以降には、いくつでも、どんな型のパラメータでも指定できます。

func Printf(format string, v …interface{}) (n int, errno os.Error) {
Printf関数の中で、vは、[]interface{}型の変数のように振る舞いますが、これを他の可変変数関数に渡すときは、通常の引数リストのように振る舞います。下は、以前使用した関数log.Printlnの実装です。ここでは、実際にフォーマット処理を行うために、fmt.Sprintlnに引数をそのまま渡しています。

// Printlnは、fmt.Printlnに沿って標準ロガーに出力します。
func Println(v …interface{}) {
    std.Output(2, fmt.Sprintln(v…))  // Outputはパラメータ (int, string)を取る
}
入れ子にしたSprintlnの呼び出しで、vの後ろに記述した…は、vを引数のリストとして扱うようコンパイラに指示するためです。こうしないと、vは、単にひとつのスライス引数として渡されます。
````

かいつまむと、...は可変長な引数で、interface{}はどんな型でも受けるということと理解しました。
interface{}で受けた場合、元の型として扱うにはどうすればいいかとか、interface{}とは何者かとか深ぼるべきところは多々ありますが、実際にgolangと戯れながら把握していくことにしましょう。

### だいたいわかったからリベンジ

go_world_teset.goに書いたAseertEquals関数では引数のexpectとactualにはinterface{}を指定します。

````go
package main

import (
	"testing"
)

func TestDouble(t *testing.T) {
	param := 1
	result := Double(param)
	AseertEquals(2, result, t)
}

func TestDouble2(t *testing.T) {
	param := 2
	result := Double(param)
	AseertEquals(4, result, t)
}

func AseertEquals(expect interface{}, actual interface{}, t *testing.T) {
	if expect != actual {
		t.Errorf("%s =! %s", expect, actual)
	}
}
````

テスト実行。

````
PASS
ok  	/path/to/go_world [build failed]
````

通った！
せっかくなので、別パッケージにAseertEqualsを置いて、今後使いまわせるようにしてみましょう。
実務で投入する場合、パッケージ分けは避けては通れぬので、今のうちに把握するメリットも高そうです。

## 実務と同じくパッケージ分けしてみる

具体的にどのようにパッケージを分けたら、どのようにパスや依存が解決されるかまったくわかりません。
http://golang.org/doc/code.html にパッケージ分けに至るステップが英弱にも読みやすいかたちで書かれているので、これをサンプルとして読みながら進めます。
ここまででテストを書いて、関数を作って、呼び出すという事はできるようにっているので、相当サクサク進めるはずです。

ちなみに、これまgolang.jpの記事にも大いに助けられていましたが、今となっては古いバージョンの記述も多々あります。できるだけ本家ドキュメントを中心に読むようにしましょう。


まず環境まわりの設定をします。
筆者の環境では、~/dev/goをGOPATHにして、binディレクトリを作り、binにパスを切っておきます。

````
export GOPATH=~/dev/go
mkdir -p $GOPATH/bin
export PATH=$PATH:$GOPATH/bin
`````

次に、さんざん動かしてきたgo_worldを適切なパスに置きましょう。
いずれgithubで管理するので、$go_world/src/github.com/＜github_id＞/go_world/に置くこととします。

では、AssertEqualsを適切なパッケージに置くことにしましょう。
assertパッケージを作って、そこにEquals関数を生やすことにします。

 $go_world/src/github.com/＜github_id＞の下にassertディレクトリを作り、assert.goを書いていきます。

````go
package assert

import "testing"

func Equals(expect interface{}, actual interface{}, t *testing.T) {
	if expect != actual {
		t.Errorf("%s =! %s", expect, actual)
	}
}
````

//todo: go build
//todo: go install

これでassert.Equalsという名前でやれるようになった。
できればt.AssertEqualsという呼び出し方にしたいところですが、どうにもやり方がわからなかったため、いずれリベンジしようと思います。

## 普段から使っているWebサービスを叩く実用的なコマンドを作る

ここからは更にgolangの場数を踏んでいこうと思います。
言語の勉強というと、ハノイの塔とかフィボナッチ数列とか数学の問題を解いていくよというパターンをよく見ますが、筆者にとってはまったくワクワクしてきません。普段つかいのコマンドを作って、日々の生活をよりよくしつつ、golangを習得したいです。

そこで、普段お世話になっているidobataというWebサービスを叩くコマンドをgolangで作っいきます。

### idobataとは

https://idobata.io/

idobataは、Rubyやアジャイルで有名な老舗のシステム開発会社の株式会社永和システムマネジメントが提供しているグループチャットサービスです。

普段使いの道具として動作がとてもサクサクしており、様々な開発支援ツールとの連携とシンプルな操作が心地いい、最近お気に入りのサービスです。
問い合わせの回答や機能改善のスピードが大変高速で、自分にとっては知っている人たちが作っているというのもあって、信頼感も高いです。

### WebHook

このidobataにはWebHookという機能があり、他のサービスから情報をidobataに流しこむことができます。
Jenkins、Pivotal Trackerなどサービス別に用意されているのですが、汎用的な用途のためのGeneric Hookというものがあり、httpでPOSTするだけでidobataに文言を送ることができます。

curl例

````
curl --data-urlencode "source=<h1>hi</h1>" -d format=html https://idobata.io/hook/XXXXXXXXXXXXXXXXXXX
````

//TODO 画像

普段このGeneric Hookを活用して、fluentd-plugin-idobataを使ってidobataにfluentを流れるデータを送りこんだり、システム監視ツールXymonから流し込んだりと大変便利に活用しています。

それなので、golangでidobataのGeneric Hookにアクセスするコマンドを作ることで、golangの理解を深めるのと、普段の生活をよりよくするのと両方を実現しようと思います。

では、さっそくコマンドの仕様を策定します。よくあるパターン通り、下記のようにしましょう。
 - 入力: 環境変数でWeb HookのURLを受け取る。コマンドライン引数で流しこむ文言を受け取る。
 - 出力: Web HookのURLに、流しこむ文言をhttp postする。

### まず出力から

golangでpostするにはどうしたらいいだろう。Rubyならnet/httpとかHttperty
きっとそういうパッケージがあるのではないか。

http://golang.org/pkg/
を見よう

http://golang.org/pkg/net/http/
うん、そのまんまやね
あった。

http://golang.org/pkg/net/http/#Client.Post
あった。

### Webhook URL


http.postform使って、固定で叩こう
Overview通りやってみよう

go build         
# github.com/bash0C7/idobater
./idobatar.go:8: undefined: url


import "net/url"らしい


うごいたー！
https://idobata.io/#/organization/koshiba/room/golang


### まず出力から

環境変数からの読み込みと引数からの読み込みをやろう
http://golang.org/pkg/os/ らしい

引数はflagというのを使う。

そこからリファクタリングしたり、無名関数使ったり、読み込んだり


## 作ったコマンドをゴルーチンを使ってマルチスレッドっぽい動きに変える

ゴルーチンだ

http://qiita.com/cubicdaiya/items/ec8934faf4deb44fea2f


まず、まんまなぞる。

なるほど、まったくわからん

ゴルーチン動かない。
3並行で動かしてみた。
もうちょっと動き方わかるように、printを挟んだりちゃうもん渡したりする。


どういう意味か読み込む。

http://golang.jp/2009/11/198
とか
http://www.slideshare.net/takuyaueda967/goroutinechannelgogolang
とか呼んでみる。なんかややこしい。


なるほど、まったくわからん。

といっててもアレなので、つど読み込みつつ読み込めた分から投げるという想定の元作ってみた。
終了条件は全部読み込んで読み込んだ分が全部投げ終わったら、終わり。それまではこのコマンド実行は待たねばならん。

https://gist.github.com/bash0C7/7387774
なるほどね。

読み込みの戻り値でトータルの件数を取得し、どこまで待つべきかを知る。
読み込みと書き込みの関数間は同じチャネルでやりとり。
書き込みができたら、書き込み完了のチャネルに1を書き込み。
終了判定は、書き込み完了のチャネルに書き込まれたのを読み込み、手元の変数に書き込むことで、いま何回回しているかを把握し、その数とトータルの件数とを比較し、トータル件数以上になってたら終わり。

うむ、むずい。こんなにカジュアルに無限ループっぽいのかくとは思わんかった。
でもこれだと、読み込みと書き込みをそれぞれに非同期にできるし、コマンドそのものの終了条件以外では何件目というのを気にしなくていいのがすばらしい。
しかも余計な仕組みなしにエレガントに書ける。

URLはカンマ区切りで。
どう区切るんやろ？
 -> stringsパッケージ


同時並行数　concurrency　はフラグで。

（スニペット）

動いたー！
コツとしては、無限ループの中でチャネル出し入れ。無限ループは関数の中に書いて、その関数をゴルーチンとして呼び出し。というのが鉄板な書き方なのかな。


## golangの地平線を目指して

実際に動くコードのテスト
- テストダブル
- ゴルーチンを使う場合のテストのこつ
- 
プログラムの配布
- http://qiita.com/futoase/items/73b7ca9fb16ca588ad6f
- 複数ファイルにまたがった時のビルド

クロスコンパイル

http://unknownplace.org/archives/golang-cross-compiling.html

$  GOOS=linux GOARCH=amd64 go build go_world.go
$  ./go_world 
zsh: exec format error: ./go_world
$  GOOS=darwin GOARCH=amd64 go build go_world.go
$  ./go_world 
Go World!

