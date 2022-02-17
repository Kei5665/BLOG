## APIについて（前回の続き）
- レスポンスをどう扱うのか調べる

### デモコード
こちらにデモコードがあるのでもし良ければ一緒に見てください。
[Snap to Roads  \|  Roads API  \|  Google Developers](https://developers.google.com/maps/documentation/roads/snap#maps_http_roads_snaptoroads_interpolation-txt)

### やりたいこと再度確認
レスポンスについての話の前にやりたいことを再度確認。

まず自分がやりたいのは地図上に線を作ること。

わかりやすい形で地図上に線を作る実装がされているウェブサイト見つけた[Polylineで線を引く:Geekなぺーじ](https://www.geekpage.jp/web/google-maps-api/v3/polyline.php)
```
      map = new google.maps.Map(document.getElementById("map"), opts);

      var patharray = new Array();
      patharray[0] = new google.maps.LatLng(38, 137);
      patharray[1] = new google.maps.LatLng(39, 137);
      patharray[2] = new google.maps.LatLng(40, 138);
      patharray[3] = new google.maps.LatLng(37, 139);

      // Polylineの初期設定
      var polylineOpts = {
        map: map,
        path: patharray
      };
      // 直前で作成したPolylineOptionsを利用してPolylineを作成
      var polyline = new google.maps.Polyline(polylineOpts);
    }

   // Polylineを構成する線の各点は、LatLngの配列を用意してPalylineOptionsにセットすることで設定されます。
```
まず最後をみると以下が書いてある
```
var polyline = new google.maps.Polyline(polylineOpts);
```
大体の意味はGoogleマップのポリラインオブジェクトを作ってPolylineという変数に格納している
Polylineとは折れ線、連続直線という意味
Polylineメソッドの引数にはpathを指定できて、それはPolylineを構成する線の各点の配列のことで、LatLng（緯度と経度）の配列の配列になっている。

つまり、polylineを作るにはLatLngの配列が必要って事らしい。

### デモページのコード

デモページの場合は以下のコードで実現されている。
```
snappedPolyline.setMap(map);
```

スナップされた折れ線をマップにセットする的な意味合い

ここでデモコードを見てみる
```
// スナップされたポリラインを描画する（snap-to-roadレスポンス処理後)
function drawSnappedPolyline() {
  var snappedPolyline = new google.maps.Polyline({
    path: snappedCoordinates,
    strokeColor: '#add8e6',
    strokeWeight: 4,
    strokeOpacity: 0.9,
  });

  snappedPolyline.setMap(map);
  polylines.push(snappedPolyline);　// 多分これは何回も繰り返す用のコード
}
```
メソッドの中でsnappedPolylineが定義されていて、`new google.maps.Polyline`で折れ線のオブジェクトを作って格納している。

これも引数にpathがある
pathに指定されている`snappedCoordinates`を見てみる
```
// snap-to-roadサービスによって返されたスナップされたポリラインを格納する。
function processSnapToRoadResponse(data) {
  snappedCoordinates = [];
  placeIdArray = [];
  for (var i = 0; i < data.snappedPoints.length; i++) {
    var latlng = new google.maps.LatLng(
        data.snappedPoints[i].location.latitude,
        data.snappedPoints[i].location.longitude);
    snappedCoordinates.push(latlng);
    placeIdArray.push(data.snappedPoints[i].placeId);
  }
}
```
ほうほう。
`snappedCoordinates`がLatLngの配列であることは間違いなさそう。

ただ、これがわからん。
```
        data.snappedPoints[i].location.latitude,
        data.snappedPoints[i].location.longitude);
```
あれ、でも`snappedPoints`って見たことあるかも？

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
```
うん、間違いない。
`data.snappedPoints[i].location.latitude,`は
レスポンスの`location.latitude`のこと言っとるわ。

### まとめ
とりあえずレスポンスがどこでどう扱われてるかは大体わかった。
あとはデータベースに道をどうやって保存できるかを調べる必要がある。