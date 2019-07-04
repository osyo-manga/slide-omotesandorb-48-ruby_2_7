## 表参道.rb #48
- - -

## 5分ぐらいでわかる Ruby 2.7

---

#### 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* Rails 歴 1年半
* 趣味で Ruby にパッチを投げています
* 好きなメソッド：tap
* 好きな gem：rspec
* 好きなエディタ：Vim

---

### 今日話すこと
- - -

## 5分ぐらいでわかる Ruby 2.7


---

# ！！！注意！！！
- - -
### 2019/07/04 時点での仕様です
### リリースまでに仕様が変わる可能性があります！！

---


### [Enumerable#filter_map](https://bugs.ruby-lang.org/issues/15323)
- - -

* `filter + map` を行うメソッドを追加
* `#select_collect` はない

```ruby
# map しつつ、結果が偽(nil or false)の要素を取り除く
p (1..10).filter_map { |i| i * 2 if i.even? }
#=> [4, 8, 12, 16, 20]

p [nil, 1, 10, nil, 100, nil].filter_map { |it| it&.to_s(2) }
# => ["1", "1010", "1100100"]
```

---

#### [Enumerable#tally](https://bugs.ruby-lang.org/issues/11076)
- - -

* 要素の数を数えるメソッド
* 要素をキーとして要素数を値とする Hash を返す
* `tally` は日本でいう `正` という意味

```ruby
pp [1, 1, 2, 2, 2, 3, 3, 4].tally
# => {1=>2, 2=>3, 3=>2, 4=>1}

pp ["homu", "homu", "mami", "mado", "mado", "mado"].tally
# => {"homu"=>2, "mami"=>1, "mado"=>3}
```

---

#### [Time#floor](https://bugs.ruby-lang.org/issues/15653) / [Time#ceil](https://bugs.ruby-lang.org/issues/15772)
- - -

* ミリ秒を丸め込む #round はあった
* ミリ秒を切り捨てる #floor の追加
* ミリ秒を切り上げる #ceil の追加
* わしが追加した

```ruby
time = Time.utc(2010,3,30, 5,43,"25.123456789".to_r)
p time.iso8601(10)          # => "2010-03-30T05:43:25.1234567890Z"

p time.round(3).iso8601(10) # => "2010-03-30T05:43:25.1230000000Z"
p time.round(7).iso8601(10) # => "2010-03-30T05:43:25.1234568000Z"

p time.floor(3).iso8601(10) # => "2010-03-30T05:43:25.1230000000Z"
p time.floor(7).iso8601(10) # => "2010-03-30T05:43:25.1234567000Z"

p time.ceil(3).iso8601(10)  # => "2010-03-30T05:43:25.1240000000Z"
p time.ceil(7).iso8601(10)  # => "2010-03-30T05:43:25.1234568000Z"

```

---

#### [.: での method 呼び出し](https://bugs.ruby-lang.org/issues/12125)
- - -

* method(:hoge) を self.:hoge という構文で書ける
* .:hoge とは書けないので注意

```ruby
(1..10).map(&method(:p))

# が

(1..10).map(&self.:p)

# と書ける

(1..10).map(&.:p)

# とは書けない
```

---

#### [先端無限Range](https://bugs.ruby-lang.org/issues/14799)
- - -

* Ruby 2.6 で終端無限Range が入った
  * (1..) or (1..nil)
* Ruby 2.7 では先端無限Range が入る
  * (..10) or (nil..10)

```ruby
def check(age)
  case age
  when (..10)
    "10歳以下です"
  when (20..)
    "20歳以上です"
  else
    "11歳〜19歳です"
  end
end
```

---

#### [パターンマッチ](https://bugs.ruby-lang.org/issues/14912)
- - -

* case in でパターンマッチ構文が書ける

```ruby
def validation user
  case user
  in { name: /^[a-z]+$/, age: Integer }
    :ok
  else
    :ng
  end
end

p validation(name: "homu", age: 14)
# => :ok
p validation(name: "42", age: 14)
# => :ng
p validation(name: "mami", age: "14")
# => :ng
```

>>>

```ruby
def fizzbuzz a
  case [a % 5 == 0, a % 3 == 0]
  in [true, true]
    "FizzBuzz"
  in [_, true]
    "Fizz"
  in [true, _]
    "Buzz"
  else
    a
  end
end

# p (1..15).map &method(:fizzbuzz)
p (1..15).map &self.:fizzbuzz
```

#### [Pattern matching - New feature in Ruby 2.7](https://speakerdeck.com/k_tsj/pattern-matching-new-feature-in-ruby-2-dot-7)
#### [Ruby の trunk に（実験的に）パターンマッチ構文が入った！](http://secret-garden.hatenablog.com/entry/2019/04/17/175435)

---

#### [Numbered parameters](https://bugs.ruby-lang.org/issues/4475)
- - -

* ブロックの引数を暗黙的にキャプチャする機能
  * 略してナンパラ
* 炎上中

```ruby
%w(homu mami mado).map(&:upcase)
[1 ,2, 16].map { |it| it.to_s(16) }
[9, 7, 10, 11, 8].sort { |a, b| a.to_s <=> b.to_s }

## ↓↓↓↓

%w(homu mami mado).map { @1.upcase }
[1 ,2, 16].map { @1.to_s(16) }
[9, 7, 10, 11, 8].sort { @1.to_s <=> @2.to_s }
```

#### 参照：[Ruby 2.7.0-preview1 Numbered parameters 完全攻略！ - SmartHR Tech Blog](https://tech.smarthr.jp/entry/2019/06/06/120104)

---

#### [pipeline operator](https://bugs.ruby-lang.org/issues/15195)
- - -

* . よりも優先順位が低いメソッド呼び出し
* () を省略してメソッドチェーンが出来る
* |> はメソッドではなくて構文
* 大炎上中

```ruby
(1..).take(10).map { |e| e * 2 }

# を

1.. |> take 10 |> map { |e| e * 2 }

# と書くことが出来る
```

参照:[【速報】Ruby の trunk に pipeline operator がはいった - Secret Garden(Instrumental)](http://secret-garden.hatenablog.com/entry/2019/06/14/002651)

>>>

ちなみに最初は

```ruby
1|>|5
2|>>>4
3|><3
4|><=>2
5|>=~1
```

みたいに書けましたが、<br>
今は規制されました(´・ω・`)
これは `a.|b` が有効なので `a|>|b` も有効になっていたため




---

#### irb 強化
- - -

* irb がめっちゃ強化
* 複数行編集対応
* 複数行履歴
* シンタックスハイライト
* コード補完
* ドキュメント表示


---

### まとめ
- - -

* [Enumerable#filter_map](https://bugs.ruby-lang.org/issues/15323)
* [Enumerable#tally](https://bugs.ruby-lang.org/issues/11076)
* [Time#floor](https://bugs.ruby-lang.org/issues/15653) / [Time#ceil](https://bugs.ruby-lang.org/issues/15772)
* [先端無限Range](https://bugs.ruby-lang.org/issues/14799)
* [.: での method 呼び出し](https://bugs.ruby-lang.org/issues/12125) 13581
* [パターンマッチ](https://bugs.ruby-lang.org/issues/14912)
* [Numbered parameters](https://bugs.ruby-lang.org/issues/4475)
* [pipeline operator](https://bugs.ruby-lang.org/issues/15195)
* irb ぱねえ


---

### 参照
- - -

* [ruby/NEWS](https://github.com/ruby/ruby/blob/master/NEWS)
* [Ruby 2.7.0-preview1 リリース](https://www.ruby-lang.org/ja/news/2019/05/30/ruby-2-7-0-preview1-released/)

---

### 最新の Ruby に興味がある人は！！
- - -

* 毎月 Ruby Hack Challenge Holiday というイベントをやっている
  * 次回 [Ruby Hack Challenge Holiday #5](https://rhc.connpass.com/event/131026/)
* みんなでモクモクしながら Ruby の開発を行うのが目的
* 自前で Ruby をビルドして最新版の Ruby を触ってみよう！



---


## ご清聴
## ありがとうございました
