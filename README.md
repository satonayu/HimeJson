# HimeJson
JSON library for Kuin.


## パースの方法
### JSONファイルからパース
@makeJsonF(path: []char): @Json を使います。
```
var json: \json@Json :: \json@makeJsonF("res/test_JsonParser.json")
```


### テキストデータ（[]char）からパース
@makeJsonT(text: []char): @Json を使います。
```
var text: []char :: "{\"a\":\"b\"}"
var json: \json@Json :: \json@makeJsonT(text)
```


## JSONデータへのアクセス
@Json.at(path: []char): @Json を使って欲しい要素までのパスを文字列で指定します。

test_JsonParser.json
```json
{
  "a":{"b":1,"c": 2, "d" :3 ,"e" : 4 , "f":5},
  "b": {  "b":1,"c": 2, "d" :3 ,"e" : 4 , "f":5  },
}
```
```
var json: \json@Json :: \json@makeJsonF("res/test_JsonParser.json")
do cui@print( json.at("a/d").toStr() ) { => 3 }
```


## HimeJsonの型
JSONの値をKuinで扱えるように、７つのクラスが準備してあります。

・@Json
  すべてのJson型の親となるクラスです。  
  基本的に、各メソッドが返してくるインスタンスは @Json に型変換されています。  
  必要に応じて、以下の子クラスに変換して使用して下さい。

・@JsonBool
  真偽値（true/false）を扱うクラスです。

・@JsonNumeric
  数値を扱うクラスです。内部では float でデータを管理しています。

・@JsonString
  文字列値を扱うクラスです。

・@JsonArray
  配列値を扱うクラスです。内部では list でデータを管理しています。

・@JsonObject
  オブジェクト型を扱うクラスです。内部では dict でデータを管理しています。

・@JsonError
  エラー用の特殊なクラスです。  
  メソッド処理中にエラーが発生した際に、@JsonErrorにメッセージを追加して返します。  
  発生したエラーは toStr() メソッドで確認する事ができます。

### @Json から子クラスへの変換
Kuinのキャスト演算子（$）、または@Jsonの to**メソッドで本来の型に変換することができます。  
ただし、元が@JsonBool⇢@Json⇢@JsonNumericのような型変換はKuinの仕様通り例外が発生します。
```
var json: \json@Json :: \json@makeJsonF("res/test_JsonParser.json")

var num1: \json@JsonNumeric :: json.at("a/d").toNumeric()
var num2: \json@JsonNumeric :: json.at("a/d") $ \json@JsonNumeric
var bool: \json@JsonBool :: json.at("a/d").toBool() { error! "
```

#### @Jsonの正体は…？
@Jsonの中身が何者なのか、それは @Json.type() メソッドで判断することができます。
```
var json: \json@Json :: \json@makeJsonF("res/test_JsonParser.json")

if (json.at("a/b").type = %jnum)
  do cui@print("a/b => @JsonNumeric")
else
  do cui@print("a/b => ???")
end if
```


