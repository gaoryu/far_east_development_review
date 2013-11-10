# さぁ、唄ってgolang...

筆者は普段はRubyistとして、Rubyでアプリケーションを書いたりfluentdのプラグインを作ったりしている。
新しい言語を使えるようになって芸事の幅を広げようと思い、最近話題を聞くことが増えてきたgolangにこの度挑戦してみた。

この記事では、golangに興味はあるけど、どこから始めたらいいだろうか、即物的に役に立てるものを作るにはどうしたらいいだろうかという悩みを持つプログラマーを対象として、golangをHello Worldからはじめて、テスト実行、普段から使える実用コマンドを作るに至る道筋と、その中で困った事の試行錯誤について解説する。

## 目次

+ 早速goをインストールして実行してみる
+ テストを書きながら敗北を知る
+ golangの書き方を覚える
+ 実務と同じくパッケージ分けしてみる
+ 普段から使っているWebサービスを叩く実用的なコマンドを作る
+ 作ったコマンドをゴルーチンを使ってマルチスレッドっぽい動きに変える
+ golangの地平線を目指して

## 早速goをインストールして実行してみる

brew install go
で終わり。

なんか出た。
````
$ go --help
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

http://golang.jp/install#osx
のあと、インストールの確認
````
$ cat go_world.go 
package main

import "fmt"

func main() {
    fmt.Printf("Go World!\n")
}
$  go run go_world.go 
Go World!
````

クロスコンパイル

http://unknownplace.org/archives/golang-cross-compiling.html

$  GOOS=linux GOARCH=amd64 go build go_world.go
$  ./go_world 
zsh: exec format error: ./go_world
$  GOOS=darwin GOARCH=amd64 go build go_world.go
$  ./go_world 
Go World!

IDE http://www.winsoft.sk/go.htm

## テストを書きながら敗北を知る

http://golang.jp/code#Testing

テスト対象ファイル名_test.goって名前をつける

go test   

- テスト対象と同じパッケージ
- testingをimport
-  TestXxx(t *testing.T)
-  失敗の場合はt.Errorを呼び出す

ただし、エラー用の出力があるだけなので、ifで調べなあかん。

やりにくいから、assertion作ろう。

testingをもとに、AssertEqualsを実装するぞ
http://golang.jp/go_faq#testing_framework　によるといらんがなという話ではあるが、いちいちifは書きたくない。


func AseertEquals(expect, actual, t *testing.T) {
	if expect != actual {
		t.Errorf("%s =! %s", expect, actual)
	}
}
ってすればいけるかな？

え、引数の型必須？
どうすんのこれ。stringかもしれず、intergerかもしれず。


型？
戻り値？
どうやんねん？

わからないので、golangのソースコードを読もう。
例えば、おなじみコレを読めば。

// Errorf is equivalent to Logf followed by Fail.
func (c *common) Errorf(format string, args …interface{}) {
	c.log(fmt.Sprintf(format, args…))
	c.Fail()
}

なにこれわからん。これをほっとくのは気持ちが悪い。




## golangの書き方を覚える

http://golang.jp/install/source

// Errorf is equivalent to Logf followed by Fail.
func (c *common) Errorf(format string, args …interface{}) {
	c.log(fmt.Sprintf(format, args…))
	c.Fail()
}

なにこれわからん。これをほっとくのは気持ちが悪い。
ということで。

関数名の後ろに引数と戻り値を書くのはわかった。では関数名の前にあるやつは？
http://golang.jp/effective_go#methods

これにはまず、メソッドと結びつけるために新たに名前付きの型を宣言し、メソッドにこの型のレシーバを定義します。

type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // 本体部分は前と全く同じ
}

何に対して生やすかか。で、このメソッドは当該ネームスペースにあるから、そう呼び出せるのね。

次。引数の最後にあるinterface{}ってなんぞ？
http://golang.jp/effective_go#printing

Printfのシグネチャの最後の引数には…interface{}型が使われているため、フォーマット指定以降には、いくつでも、どんな型のパラメータでも指定できます。

func Printf(format string, v …interface{}) (n int, errno os.Error) {
Printf関数の中で、vは、[]interface{}型の変数のように振る舞いますが、これを他の可変変数関数に渡すときは、通常の引数リストのように振る舞います。下は、以前使用した関数log.Printlnの実装です。ここでは、実際にフォーマット処理を行うために、fmt.Sprintlnに引数をそのまま渡しています。

// Printlnは、fmt.Printlnに沿って標準ロガーに出力します。
func Println(v …interface{}) {
    std.Output(2, fmt.Sprintln(v…))  // Outputはパラメータ (int, string)を取る
}
入れ子にしたSprintlnの呼び出しで、vの後ろに記述した…は、vを引数のリストとして扱うようコンパイラに指示するためです。こうしないと、vは、単にひとつのスライス引数として渡されます。

ここで説明した範囲は出力機能のほんの一部です。より詳しい説明はパッケージfmtのgodocドキュメントを参照ください。

話題が変わりますが、…パラメータには型を指定します。たとえば、次のmin関数の…intです。この関数は、整数リスト内から最小値を抽出します。

func Min(a …int) int {
    min := int(^uint(0) >> 1)  // intの最大値
    for _, i := range a {
        if i < min {
            min = i
        }
    }
    return min
}

### だいたいわかったからリベンジ


func AseertEquals(expect interface{}, actual interface{}, t *testing.T) {
	if expect != actual {
		t.Errorf("%s =! %s", expect, actual)
	}
}

func TestDouble1(t *testing.T) {
        AseertEquals(Double(2), 3, t);
}

できたー！
でも、testing.Tってのが気持ち悪い。元のTestingに生やしたいよね。

どうやればいいんだろう。
http://qiita.com/Jxck_/items/509459ee7be940290130
オーバーライド。読み込んでみよう。自分の場合ではどないや？
オープンクラスじゃなくて継承になるの？
http://qiita.com/Jxck_/items/fb829b818aac5b5f54f7


なんかできた。

type tt struct{
        *testing.T // 'testing' inherits all testing.T methods
}

func (t tt) AseertEquals(expect interface{}, actual interface{}) {
	if expect != actual {
		t.Errorf("%s =! %s", expect, actual)
	}
}

で

func TestDouble1(t *testing.T) {
        hoge := tt{t}
        hoge.AseertEquals(Double(2), 3);

	if 3 != Double(2) {
		t.Errorf("")
	}
}

でもエレガントじゃないね。でもいまのだんかいでエレガントもとめても完全主義者のわなにはまるので、次へ進もう。



## 実務と同じくパッケージ分けしてみる

それなりに書き方とか意味とかわかってきたので、
http://golang.org/doc/code.html
を頭からやりましょう。
パッケージこうつくるとか、これたたくとか懇切丁寧にかいてる。英弱でも読みやすい。

やった！動いた！

commit 5b502834f5073aaec7d50adfc8980cfcde215445
Author: Toshiaki Koshiba <koshiba@qg8.so-net.ne.jp>
Date:   Tue Oct 29 20:54:05 2013 +0900

    assert.Equalsが通った！！！



## 普段から使っているWebサービスを叩く実用的なコマンドを作る

idobata

idobataとは


# http

http://golang.org/pkg/
を見よう

http://golang.org/pkg/net/http/
うん、そのまんまやね


http://golang.org/pkg/net/http/#Client.Post

### Webhook URL


http.postform使って、固定で叩こう
Overview通りやってみよう

go build         
# github.com/bash0C7/idobater
./idobatar.go:8: undefined: url


import "net/url"らしい


うごいたー！
https://idobata.io/#/organization/koshiba/room/golang


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



## golangの地平線を目指して

実際に動くコードのテスト
- テストダブル
- ゴルーチンを使う場合のテストのこつ
- 
プログラムの配布
- http://qiita.com/futoase/items/73b7ca9fb16ca588ad6f
- 複数ファイルにまたがった時のビルド
