## APIについて
#### Web APIってなんや？
- [APIとは？初心者向けに分かりやすく解説します。](https://products.sint.co.jp/topsic/blog/api)
- APIを、簡単に説明すると「決まった方法でアクセスをすれば決まった結果を返してくれるもの」。決まった形式でアクセスを受け付けて、仕様通りの結果を返すインターフェースとなるのがAPI。

### Roads API
[Snap to Roads  \|  Roads API  \|  Google Developers](https://developers.google.com/maps/documentation/roads/snap#maps_http_roads_snaptoroads_interpolation-txt)
Roads APIは、ルートに沿って収集された最大100個のGPSポイントを受け取り、車両が走行していた可能性の高い道路にポイントがスナップされた同様のデータセットを返します。オプションとして、ポイントの補間を要求することができ、道路の形状に沿った滑らかなパスを得ることができます。


#### リクエスト
snap to roadへのリクエストは、HTTPSで送信する必要があり、次のような形式になります。
```
https://roads.googleapis.com/v1/snapToRoads?parameters&key=YOUR_API_KEY
```

#### 必須パラメーター 
##### path
スナップされるパスです。pathパラメータには、緯度と経度のペアのリストを指定します。緯度と経度の値はカンマで区切ってください。座標は、パイプ文字 "|". For example: path=60.170880,24.942795|60.170879,24.942796|60.170877,24.942796.
#### 任意のパラメーター 
##### interpolate
完全な道路形状を形成するすべての点を含むようにパスを補間するかどうか。true の場合、追加の補間ポイントも返され、結果として、コーナーやトンネルを通過する際にも、道路のジオメトリにスムーズに沿ったパスが生成されます。デフォルトはfalse。

#### 例
リクエスト。GET。
```
https://roads.googleapis.com/v1/snapToRoads?interpolate=false&key=AIzaSyDC4lRXp8WvphHA76VX90sAcPyFFvfKM6Q&path=-33.865863%2C151.193183%7C-33.866655%2C151.193537%7C-33.867297%2C151.193601%7C-33.867867%2C151.194073
```
レスポンス
```
{
  "snappedPoints": [
    {
      "location": {
        "latitude": -33.8658612307971,
        "longitude": 151.19319514586903
      },
      "originalIndex": 0,
      "placeId": "ChIJB7-UDDauEmsRYIxz5md9ARM"
    },
    {
      "location": {
        "latitude": -33.866677051354564,
        "longitude": 151.19345674386255
      },
      "originalIndex": 1,
      "placeId": "ChIJL4MoZTauEmsRkzbR3v9NneY"
    },
    {
      "location": {
        "latitude": -33.867257966825434,
        "longitude": 151.1937142818249
      },
      "originalIndex": 2,
      "placeId": "ChIJa5zkiTauEmsRoBGD5Wd9ARM"
    },
    {
      "location": {
        "latitude": -33.867873447676253,
        "longitude": 151.19405736995711
      },
      "originalIndex": 3,
      "placeId": "ChIJFcWK7DauEmsRIsO935KEEiw"
    }
  ]
}

```
#### レスポンスの中身
- snappedPoints
スナップポイントの配列です。
- location
10進法の緯度と経度の組み合わせ
- placeId
ある場所の一意の識別子。
- originalIndex
元のリクエストの対応する値を示す整数です。リクエストの各値は、レスポンスの対応する値にマッピングする必要があります。

#### 今日の残った疑問
このレスポンスをどうやって扱うの？
