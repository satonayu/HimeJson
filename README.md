# HimeJson
JSON library for Kuin.

## ◆ License
[くいなちゃんライセンス](http://kuina.ch/others/license)

* 改変OK！  
* 再配布OK！  
* 二次創作OK！  
* 著作権表示義務なし！  
* 「くいなちゃんライセンス」のものを使った作品は「くいなちゃんライセンス」にする必要なし！


## ◆ JSONからKuinへのパースの方法
#### パースする方法は２つあります。  
1. JSON形式のファイルから直接パース
2. JSONのテキストデータ（[]char）からパース

2の方法はURL等のストリームから取得した文字列をパースする際に使います。  
もちろん、URL等のストリームからパースするメソッドを作っても良いでしょう。
```kuin
var json: \json@Json :: \json@makeJsonF("res/sample.json")

var text: []char :: "{\"a\":\"b\"}"
var json2: \json@Json :: \json@makeJsonT(text)
```


## ◆ JSONデータへのアクセス
#### 1. ネストされた配列値、オブジェクト値に保管されているデータを取得
`@Json.at(path: []char): @Json`  
path -> 欲しい要素までの配列のインデックスやオブジェクトのキーを `/` で繋いだパスを指定。  
戻り値 -> @Jsonクラスのインスタンス

sample.json
```json
{
  "a" : { "b" : 1, "c" : 2, "d" : 3, "e" : 4, "f" : 5 },
  "b" : [ 1, 2, 3, { "name" : "satochi", "id" : "satonayu" } ]
}
```
```kuin
var json: \json@Json :: \json@makeJsonF("res/sample.json")
do cui@print( json.at("b/3/name").toStr() ) { => "satochi" }
```

また、逆に親や祖先の要素にアクセスするには以下のメソッドを使用します。
```kuin
@Json.parent(): @Json
@Json.root(): @Json
```


#### 2. 取得したデータをKuinの型に変換
```kuin
@Json.toBool(): bool
@Json.toInt(existed: &bool): int
@Json.toFloat(existed: &bool): float
@Json.toStr(): []char
@Json.toStrFmt(fmt: []char): []char
```
これらのメソッドはKuinの組み込みメソッド同様にある程度同様に動作します。  
それぞれのJSON型がどのような値に変換して返すかはソースコードを確認して下さい。

また、HimeJsonでは数値を簡単に取得できるよう、以下のメソッドを追加で用意しています。  
ただし、Kuinの`toInt`や`toFloat`が変換できない値を保持していた場合、想定していない数値を取得している事に気づくチャンスを失います。
（`toInt`と`toFloat`の仕様変更により、存在価値なくなったかな。)
```kuin
@Json.toi(): int
@Json.tof(): float
```

#### 3. 取得したデータを別のJSON型にキャスト
```kuin
@Json.toJBool(): @JsonBool
@Json.toJNum(): @JsonNumeric
@Json.toJStr(): @JsonString
@Json.toJAry(): @JsonArray
@Json.toObj(): @JsonObject
@Json.toErr(): @JsonError
```
ただし、値が持つ本来の型以外にキャストしようとすると、Kuinの仕様通り例外が発生するので注意して下さい。  
これらのメソッドは内部的にKuinのキャスト演算子（$）を使用しているだけです。  
`json $ \json@JsonBool` のようにしても全く同じ動作となります。


#### 4. 取得したデータが持つ本来のJSON型を確認
```kuin
@Json.type(): @JsonType
  ;-> %jerr, %jnull, %jbool, %jnum, %jstr, %jary, %jobj
```
HimeJsonではKuinの列挙型を使い、７つのタイプを準備しています。  
この中で`%jerr`と`%jnull`は特別な状態を表します。  

`%jerr` -> 各JSON型のメソッドの処理中にエラーが発生した場合、本来の値ではなくエラーメッセージを内包した@JsonError型が返されます。  
@JsonErrorが返された場合、適切なエラー対応処理を実行して下さい。

`%jnull` -> @Json型の中でも値を保持していない純粋な@Json型を表します。  
他のJSON型へはキャストできないため注意して下さい。


#### 5. 取得したデータの値を書き換え
各JSON型の値を書き換えるには`setValue(value)`メソッドを使用します。  
このメソッドは@Jsonには実装されていないため、本来の型にキャストしてから使用して下さい。  
例えば、真偽値の値を書き換えるには以下のようにします。
```kuin
var jbool: \json@JsonBool :: json.toJBool()
do jbool.setValue(true)
```

## ◆ JSON型からJSONの文字列を出力
JSON型で木構造に構成されたデータをJSONの文字列にシリアライズするには以下のメソッドを使用します。
```kuin
@Json.dump()
  ; -> {"a":{"b":1,"c":2,"d":3,"e":4,"f":5},"b":[1,2,3,{"name":"satochi","id":"satonayu"}]}
```

## ◆ @JsonArray型特有のメソッド
#### 1. @JsonArray.len(): int
配列値の要素が保管している子要素の数を取得
```kuin
var cnt: int :: json.len() { -> 2 }
```


#### 2. @JsonArray.add(item: @Json)
配列値の最後に新しい要素`item`を追加
```kuin
var new_item: \json@JsonBool :: #\json@JsonBool
do new_item.setValue(true)

do json.add(new_item)
```


#### 3. @JsonArray.addAt(index: int, item: @Json)
`index`で指定した位置に`item`を追加
```kuin
var new_item: \json@JsonBool :: #\json@JsonBool
do new_item.setValue(true)

do json.addAt(1, new_item)
```


#### 4. @JsonArray.del(item: @Json): bool
配列の中から`item`を探し、最初に見つけた要素を削除。  
戻り値は、削除した場合は`true`、要素が見つからなかった場合は`false`を返します。
```
if (json.del(del_item))
  do cui@print("delete!")
else
  do cui@print("del_item not found.")
end if
```


#### 5. @JsonArray.delAt(index: int): bool
`index`で指定した位置の要素を削除。  
戻り値は、削除した場合は`true`、指定した位置が存在しなかった場合は`false`を返します。
```kuin
if (json.delAt(1))
  do cui@print("delete!")
else
  do cui@print("Index does not exist.")
end if
```


#### 6. @JsonArray.forEach(callback: func<(@Json, kuin@Class): bool>, data: kuin@Class): bool
要素が保管している全要素を走査。  
コールバック関数には、各要素と任意で渡したデータ`data`が渡されるため、その関数内で目的の処理を行います。  
以下の例では、配列値の中から最大の要素を探しています。
```kuin
class Item()
  +var maxValue: int
end class
func main()
  var json: \json@Json :: \json@makeJsonT("[1,2,3,4,5,4,3,2,1]")
  var item: @Item :: #@Item
  
  do item.maxValue :: 0
  do json.toJAry().forEach(max_search, item)
  do cui@print("max value -> \{item.maxValue}") {max value -> 5}
  
  func max_search(json: \json@Json, data: @Item): bool
    if (data.maxValue < json.toi())
      do data.maxValue :: json.toi()
    end if
    ret true
  end func
end func
```


#### 7. @JsonArray.hasItem(item: @Json): bool
配列値が指定の要素`item`を保管しているかを取得
```kuin
if (json.hasItem(find_item))
  do cui@print("Item found.")
else
  do cui@print("Item not found.")
end if
```

## ◆ @JsonObject型特有のメソッド
#### 1. @JsonObject.len(): int
オブジェクト値の要素が保管している子要素の数を取得
```kuin
var cnt: int :: json.len() { -> 2 }
```


#### 2. @JsonObject.add(key: []char, item: @Json)
`key`と`item`のペアで要素を追加
```kuin
var new_item: \json@JsonBool :: #\json@JsonBool
do new_item.setValue(true)

do json.add("new_key", new_item)
```


#### 3. @JsonObject.del(key: []char): bool
※未実装


#### 4. @JsonObject.forEach(callback: func<([]char, @Json, kuin@Class): bool>, data: kuin@Class): bool
#### 5. @JsonObject.exist(key: []char): bool
#### 6. @JsonObject.toArrayKey(): [][]char
#### 7. @JsonObject.toArrayValue(): []@Json
これらのメソッドはdict型の組み込みメソッドのラッパーです。  


## ◆ @JsonError型特有のメソッド
#### 1. @JsonError.addMessage(message: []char)
新しいエラーメッセージを追加
```kuin
do json.addMessage("エラー発生")
```
#### 2. @JsonError.getMessage(): \[]\[]char
エラーメッセージの配列を取得
```kuin
var messages: [][]char :: json.getMessage()
```
