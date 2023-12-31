---
title: SOLARISでUSDアセットを作ろう
tags:
    - USD
    - SOLARIS
    - Houdini
    - HoudiniAdventCalendar
---

!!! info

    現在（Houtini19）では、ComponentBuilderを使用することで
    以下の内容をもう少し簡単につくることができます。
    ComponentBuilderについては :fa-external-link: [こちら](./16_component_builder.md)

[Houdini Advent Calendar 2020](https://qiita.com/advent-calendar/2020/houdini) 12日目は **SOLARISでUSDアセットを作ろう** です。

## ゴール

今回は、
**「USDの汎用アセット」をSOLARISのネットワークでこねくり回して作ってみます。** 


どういうことかというと、USDのシーングラフは基本使う人間の設計によって
自由にシーングラフやレイヤー構造などを構築できます。
つまり、何でもできてしまうわけですね。

ですが、それだと何からどうしていいかもわかりにくいですし
どういうふうなアプローチを取っていいのかもわかりません。
何をどう組み合わせると使いやすい物ができるのかも想像しにくいのではないかと
ずっと思っていました。

ので、まずは割と汎用的に使える
SOLARIS上でレイアウトするもよし、ゲームエンジンに持っていくもよし
別のツールに持っていって使うもよし　というような
汎用アセットの構造を、現在公開されている各種サンプルHip/USD・サンプルコード
をもとに整理して、それをHoudiniのSOLARIS上でセットアップしていきます。

このセットアップ方法を通じて、
USDのコンポジションの理屈と合わせてまとめていければと思います。

## ここでいうアセットとは

これは分野や会社によっても定義は変わるかと思いますが、
ここではショットワークで使用する各種モデルなどの素材を指します。

![](https://gyazo.com/1e324b32c176801299477fa8238123aa.png)

ものすごいざっくりと書き出すとこういう関係になります。
映像作品などを作る場合は、各カットごとにアニメーションをつけてエフェクトをつけたりして
映像を作成します。
アセットはここで使用するキャラクターだったりBGだったり、小物等
あるいは汎用アニメーション・汎用エフェクト（共通素材）なども
アセットに含まれます。

なので、ここでは各Shotで使うアセット類をUSD化して管理できるようにします。

アセットには色々種類がありますが、今回は BG/Prop をベースにして構築します。
（キャラ、アニメーション、エフェクト周りは現状だと作りにくい故。。。）
ただし、基本的な考え方は共通で大丈夫なはずです。

!!!info
    このあたりはプロジェクトや会社によって大きく扱いが変わるとはおもいますので
    今回はあくまでこういうものとして見ていただけると。。。

## 基本構造を作る

では早速、やっていきましょう。
まずは、BGやプロップなどのアセットをどういうふうに作っていけばいいか
毎度おなじみ、Pixarのサンプル「Kitchen_set」を使用して確認していきます。

### 最小単位の構成を作る

まず、USDの大きな特徴として
USDはファイルを好きに分割することができます。
分割したファイルはコンポジションアークを利用して１つのシーングラフに合成することができます。
ので、まずUSDでアセットを構築する場合
どういうファイル単位にするのか、どういうふうにコンポジションを構築すると
扱いやすいかを考える必要があります。

この場合のセオリーは、個人的には２つあると思っていて

1. 担当者が変わるところは分ける
2. 共通素材になるところは分ける

だと思います。

![](https://gyazo.com/52e201c1d7f3a3244d3b4ccce7960053.png)

キッチンセットは、 assetsフォルダ以下に複数のアセットが保存されています。
その中のKitchenTableを開いてみると

![](https://gyazo.com/ba44d0885069324d93a5b8f5b3d94dd1.png)

その下には KitchenTable.geom.usd KitchenTable_payload.usd KitchenTable.usd
という３つのモデルが入っています。

![](https://gyazo.com/a823dbf6db30a31f3839c46f0712a091.png)

そのTableが、「キッチンセット」という１つのアセットにリファレンスで
読み込まれているのがわかります。
（payloadの階層などはこのあとで説明）

これから読み取れることは、
Kitchen_setというアセット（カテゴリ的にはこれはBGに当たる）の中には
数多くの別のアセット（小物等）がリファレンスで読み込まれています。

![](https://gyazo.com/8460327ed096a587d4ea8305fd0448aa.png)

１つのBGを構築しようとしたときに、共通の小物
（家具、机の上の小物、街路樹、信号などなどBGを構成する要素）は
**共通素材となるパーツ** となります。
また、そういった小物を作るときは複数の担当者で同時並行して作業をすることが
可能でしょう。

なので、まずはアセットを構築する場合は
最小単位はまずほかのセット（BG）でも使い回せる小物類にしておくのが良さそうです。

### SOLARISで構築開始

とりあえず、SOLARIS上で組み立ててみます。

![](https://gyazo.com/6efa7045e9576ef4d5695eef54d57913.png)

いつものように豚さんを召喚します。

![](https://gyazo.com/6a69984a208888699126c53d312d8e0d.png)

ネットワークはこちら。
召喚方法はいろいろありますが、今回は sopimportを使用して
指定のモデルを指定のネームスペース以下に生成します。

![](https://gyazo.com/e5e0d129754b7b6b06174c67ad89dc52.png)

なにもしないとこうなります。
これじゃあ何がなんだかわからないので、ちゃんと整理してみます。

![](https://gyazo.com/34964b14ee5a7055bf44a2c4e1a5af1c.png)

こうなりました。

![](https://gyazo.com/ef82df203693df5e84cec84d70771fb6.png)

これは、アセットを作る場合はPixarが提供しているサンプルをみるとわかりますが
トップノードはアセット名がルート以下に来るようにしています。
そしてこのアセット名Primを **「defaultPrim」** に指定します。

![](https://gyazo.com/1bed7bdb3c6fa0888aece11637be9ce9.png)

最近ずっと調べていた ConfigureLayerノードを作り

![](https://gyazo.com/ccc8c77b6b44e05300fdff2345bec17e.png)

Default Primitive を、ルート以下につくった アセット名のPrimに指定します。

```
#sdf 1.4.32
(
    defaultPrim = "Pig"
    doc = """Generated from Composed Stage of root layer 
"""
    framesPerSecond = 24
    metersPerUnit = 1
    timeCodesPerSecond = 24
)
```
こうすると、レイヤーに defaultPrimが指定できました。
なぜこれを指定するかというと、この作成したusdファイルを
どこかのPrimに対してリファレンスしたい場合、このデフォルト指定をしておかないと
エラーになってしまう（リファレンス時にPrimを指定しなければいけなくなる）からです。

### マテリアルの設定

これだけですと、メッシュがあるだけでマテリアルも何もないので
次にマテリアルを設定していきます。

![](https://gyazo.com/98bfe657cafaa2adc89f00e4762c460e.png)

マテリアルは、SOP側ですでに指定されている場合は、MaterialLibraryを使用して

![](https://gyazo.com/a2aca5c90ad3a4190a56ba9181c319d5.png)

shopnetのマテリアルをSOLARIS上に読み込みます。

![](https://gyazo.com/07a6bc9ca2889a9ce24d9b0032331b48.png)

MaterialLibraryはこんなかんじです。
SOLARIS側でマテリアルを作るのならば [UsdPreviewSurface](./12_usd_preview_surface.md)を使用して
作る方法もあります。

![](https://gyazo.com/f4461f100f42e4b350d25e1ba2152b06.png)

結果。
注意点として、デフォルトだとマテリアルは /materials に作られますが
それだとPig以下にならないので、 Material Path Prefix にアセットのPrim以下の
Material用の階層（この場合はMaya準拠のLooks）を作成します。

![](https://gyazo.com/c30d254fc20af49568bb5284b15be863.png)

そして最後にUSD ROPを使用して USDファイルに出力します。

この段階だと、 pig.usda という USDファイルが１つあり
その中にアセットが入っているという状態になります。
(とくにコンポジションなどの処理もなにもされていない)

これが、最もシンプルなUSDアセットの構造になります。

### マテリアルを別レイヤー化する例

上の構成でもOKですが、もう少しレイヤーを分割する例も軽く触れておきます。
それはマテリアルを分割したい場合。

![](https://gyazo.com/0b5c37920c6462a8885b82013c4b90be.png)

豚のマテリアルを共有化したいケースはまぁさすがに無いとは思いますが
汎用的なマテリアルは他のモデルでも使いたい場合が出てくると思います。
そういう場合などは、マテリアルのUSDは別途出力しておいて、
それをアセットのレイヤーに合成して使用する...といった構造も作ることができます。

![](https://gyazo.com/b03bac2d737f690e2a075803a41f4428.png)

この、Materialを作る　で作ったレイヤーは、
こんなふうにルート以下にMaterialがあるような構造になっています。
これをリファレンスでMeshのシーンに合成して
合成後にアサインすることで、Material部分を共通化することができました。

[参考Hip](https://1drv.ms/u/s!AlUBmJYsMwMhhOcEoihzTbjgh3KnoQ?e=pwZQR2)

参考はこちらのHipのコメントなどを参考にしてください。

以降は、この基本構造をもとにして
さらにUSDのコンポジションを活用した構造をつくっていきます。

## USD的な構造を追加する

一応ここまでの構造でも完結していましたが、
USD的に見ると今ひとつというか、もう少し改善する余地があります。
それが、上で軽くだけ触れましたが「ペイロード」の仕組みであったり
「バリアント」を使ったモデルのバリエーションの作成であったりの
**コンポジションアークを駆使したUSDアセット** です。

次からは、このコンポジションアークを使った構造を、キッチンセットと [こちら](https://github.com/PixarAnimationStudios/USD/blob/release/extras/usd/examples/usdMakeFileVariantModelAsset/usdMakeFileVariantModelAsset.py)の
USDリポジトリ以下にあるサンプルPythonをベースに
SOLARIS上で構築していきつつ説明していきます。

### ペイロード

#### ペイロードとは

まずは **ペイロード** について。
ペイロードとはなにかというと、基本的な動作はリファレンスと共通ですが
「アンロード」指定でUSDを開いた場合、ペイロード化しているレイヤーはロードされなくなります。

```
usdview D:\USDsample\Kitchen_set\Kitchen_set.usd --unloaded
```
例として、キッチンセットをアンロード状態で開いてみます。
usdviewの引数に --unloaded して実行します。

![](https://gyazo.com/ac8092319fd8396164a40b8ad24cb7d0.png)

実行すると、開いてもモデルが何も表示されなくなりました。
そのかわり、USDViewがものすごい高速で起動します。

これは、キッチンセットで使用している大量のアセットは、かならずルートに
ペイロードの構造を持っているからです。

![](https://gyazo.com/2a90779e47cb545577271df7313f9fb3.png)

たとえば、この中のKitchen TableをLoadしてみます。

![](https://gyazo.com/74dfafe0ba62762639d4aa5fa5011c6c.png)

すると、Tableのみがロードされました。

この機能を使うと何が良いかというと、
たとえばあるショットが超巨大なビル群などの背景に大量のキャラクターが配置され、
エフェクトもバンバン乗っているようなものがあったとします。
ファイルサイズは合計何十ギガ。
これをまともに開こうとするととんでもない時間がかかってしまいます。
けど、直したいのはカメラ近くにある邪魔なオブジェクトを消したいだけだったり。

そういう場合は、 **アンロードでシーンを開き（ロードされないので一瞬で開く）**
**作業したい場所のみピンポイントでロードし、**
**編集して保存** すれば、でかいシーンをロードしないですむのでとてもエコです。

なので、使用するアセットのルートにはペイロードの階層を
必ず入れることで、アンロードしたあとアセット単位でのロードが可能になります。

#### ペイロードの構造をつくる

![](https://gyazo.com/0ad9748545265d3bf5123123df8e704c.png)

以上を踏まえて、最初の基本サンプルを変更しました。

結果。
このレイヤー構成はまだまだ考える余地はあるのですが、
まず、最後にペイロードの階層をいれます。
ペイロード前までのレイヤーは pig.geom.usda として出力するようにしました。
そのため、この段階で

* Pig.material.usda マテリアルのみのレイヤー
* Pig.geom.usda MeshとそのMeshにマテリアルアサインをするレイヤー
* Pig.usda ペイロードの構造を追加するレイヤー

という３つのレイヤーが作成されました。

![](https://gyazo.com/c3693928ee8f287a13d7775ba5439fd3.png)

別案。
先程のに＋して、マテリアルアサインとモデル部分を分離した場合。
この場合は

* Pig.geom.usda マテリアルがアサインされていないMeshのみのレイヤー
* Pig.material.usda マテリアルのみのレイヤー
* Pig.payload.usda マテリアルをアサインするレイヤー
* Pig.usda ペイロードの構造を追加するロード用のレイヤー

という構成になります。
  
こうすると、テクスチャがアサインされていない素のメッシュだけのレイヤーが
作成できるので、例えばテクスチャとかは何もいらないガイドとしてだけ使いたい
みたいなケースがある場合は、この メッシュのみのレイヤーが使えるようになるので
ちょっと便利になります。

### バリアント

次にバリアントの構造を作っていきます。
この参考には、[こちら](https://github.com/PixarAnimationStudios/USD/blob/release/extras/usd/examples/usdMakeFileVariantModelAsset/usdMakeFileVariantModelAsset.py)のコードを参考にします。

![](https://gyazo.com/c1daae2dc47f07077ba7b1971dacb8d0.png)

バリアントが入った場合も、基本的な考え方は同じです。
バリアントで切り替えるモデル自体も、最小単位のアセットとして構築しておきます。
（このサンプルの場合は、面倒なのでマテリアルは分離していません）

そして、切り替えしたいモデルを **Add Variant** につなぎます。

![](https://gyazo.com/f489e662268bc108014ac20349f416a4.png)

PrimitivePathは、このアセットの名前（HoudiniFriends）
バリアントの切り替えはこのルートPrimに対して作りたいので Variant Primitive は / にします。

![](https://gyazo.com/82b4ce46fed1edbe312b1a25f6343598.png)

結果、ルートの HoudiniFriendsPrim に対して、バリアントセットが追加されます。
これでモデルの切り替えができるようになりました。

![](https://gyazo.com/0b3b1d7d0b3ab09990ffae36f1177811.png)

あとは、このモデルに対してペイロードの構造を追加し、
USDROPでエクスポートします。

バリアントを使えば、複数のアセットを１つのアセットとして切り替えて使用できるように
できます。
たとえば、木だったり草だったり、あとは室内の本や小物などなど。
まずは大量に配置して切り替えて調整したりしたい場合など。
キャラクターだと、服装違いや髪型違いをバリアントとして保持すると
色々コントロールがしやすくなります。

今回は１つのHIP内でモデルの作成とセットアップ内でバリアントの構築をしていますが、
(SOPでプロシージャルモデリングで複数バリエーション作るならこの作り方が有効)
すでに別途作っておいたアセットを、LoadLayerなどでロードして
Add Variantにそのレイヤーをペイロードで入力することで
単独でも使えるし、バリアントセットで切り替え可能なアセットとしても
使用することができます。
(Houdiniと合わせて、別のDCCツールで作ったものをオーサリングするときに有効)

### 継承

最後がこの **継承（Inherits）**
Houdiniのページでもあまり言及がないのと、いまいち使い方がピンとこない
（Referenceとの違いは？など）ところはあると思いますが、
個人的にはコンポジションアークで最も興味深い機能がこの継承です。

継承に関しては、[Pixar公式のDownloads](http://graphics.pixar.com/usd/downloads.html) にある USD Composition の内容がわかりやすいですので、あわせて参考にしてもらえれば。

#### 基本的な継承構造の作り方

まず、継承の特徴はあるPrimを対象のPrimに内容を「継承させる」ことができます。

![](https://gyazo.com/116a34743780ccef395210b1fb9f5a15.png)

USDの定義には Define 以外に Class が存在します。（あとはOver）
このClassは、いわゆるプログラミングのClasと同様で
あくまでも型であり実体ではありません。
ので、このClassで定義しただけではなにも作られません。

![](https://gyazo.com/15194aa46f037529c8d23e21100eecdb.png)

このClass定義は Primitive ノードで作成できます。

![](https://gyazo.com/c4eab92bb7a4ddafb9ac2162c5457c90.png)

このClassを継承（Referenceノードの Reference Typeを Inherit From First Input にする）
で継承します。

![](https://gyazo.com/8e439cf6284b99f089165846b338ef28.png)

すると、ClassのPrimを継承したPrimができあがりました。

![](https://gyazo.com/dd80d26350d1246d251db41b6ca84d6c.png)

BaseClassを複数継承してくる...のような使い方も可能です。
このとき、ベースクラスのPrimを編集すれば
継承先のPrimも変更されます。

#### USDアセット的 継承 の使い方

これだけだと、Referenceと代わりがありません。
ですが、他のコンポジションアークと組み合わせたとき、この継承は意味をなしていきます。

この継承も、[こちら](https://github.com/PixarAnimationStudios/USD/blob/release/extras/usd/examples/usdMakeFileVariantModelAsset/usdMakeFileVariantModelAsset.py)のコードを参考に組み立ててみます。

![](https://gyazo.com/94612c12719c6d8230facd3858ed26f3.png)

全貌はこちら。

![](https://gyazo.com/c2afe1153bff8e075c6e2dcb7e90f0d6.png)

アセットの構築部分はこのあたりになります。

![](https://gyazo.com/afa89c339886b0bddae6537af0331b21.png)

その結果、こんなシーングラフができあがります。
この __class__Pig を、Pigが継承しています。

![](https://gyazo.com/308f91dc3ae4f52836de020c79740dc7.png)

この作成したアセットを、Referenceで配置します。

![](https://gyazo.com/6fca8e8eef1cf817c50ab5914c0a2175.png)

ブタさん２匹が配置されました。

![](https://gyazo.com/2c73191c6298ee0ff8bbff0f5244f4ba.png)

シーングラフはこのとおり。
先程作った __class__Pig は見当たりませんが
LayoutPrim以下にPigが２匹召喚されていることがわかります。

![](https://gyazo.com/9e311e424ba6d149667836495e22b69d.png)

それとは別の系統で、このような構造を作ります。

![](https://gyazo.com/27365d217d0cfb9247e86484eb565ecf.png)

この構造では、最初のPigアセットの構造と同じ
__class__Pig 以下に、 Pig以下にあるマテリアルの構造 Looks/Pig を作っています。
このMaterialは「赤くする」マテリアルです。
このマテリアルを、 __class__Pig 以下に作っているわけです。

![](https://gyazo.com/1e9f1a383efe990d2581a1e2fdef0e6a.png)

これを、レイアウトした構造とサブレイヤーで合成してみます。

![](https://gyazo.com/dae67cdaaaabc6b5a6b44f55d1f2ab4a.png)

結果。
二匹とも赤くなってしまいました。

ルート以下の __class__Pig 1つに対して設定していたはずなのに
配置していたPigが両方とも変わってしまう...というのが継承の特徴です。

これだけだと何が起きたのが全くわかりません。
ので、図解してみます。

#### 継承のしくみ

![](https://gyazo.com/53fb5ac97ec3752f0988a7e10f63dfe5.png)

今の状況を図にするとこうなっています。

コンポジションが１つのPrimに対して複数ある場合は、 **「LIVRPS」の原則** によって
順番が解決していきます。
それを踏まえて詳しく見ていきます。

!!!info 
    LIVRPSとは、コンポジションアークの解決順序の原則です。
    **L** Local
    **I** Inherits    
    **V** Variant
    **R** Reference
    **P** Payload
    **S** Specialize
    のそれぞれの頭文字をとって LIVRPS という。
    １つのPrimに複数のコンポジションがある場合、 Lに近いほど優先されます。

まず、読み込んでいるアセットの構造は左上のようになっています。
PigPrimが Root 以下にあるPrimを「継承」しています。
しかし、この段階だと継承もとのPrimは何も定義されていないのでなにも起きません。

そして、サブレイヤーで合成された構造は右上です。

この合成を順番に（弱い順に）見ていくと、

![](https://gyazo.com/6f09120fad5ea86a81a69da233e66da6.png)

まず、リファレンスで、「Pigアセット」が配置されます。

![](https://gyazo.com/bce8ced76e7e7a302b30df62b7cf737f.png)

アセットの段階で、 __class__Pig という空のClassを defaultPrimの「Pig（リファレンス元）」
が継承していたので、このようになっています。
（ Layout/__class__Pig ではないのがミソ）

![](https://gyazo.com/0e66b976a10100bf2b10461671dc9ddc.png)

この「リファレンス元で継承しておいた Root以下の __class__Pig に
サブレイヤーで「赤いマテリアルにする」というMaterialプリムを合成します。

![](https://gyazo.com/33864c65bc96e48c322d2b30d34ce154.png)

すると、リファレンスの段階で仕込んでいた「継承」の構造は
サブレイヤーで合成された __class__Pig 以下のPrimを合成します。

Pig1とPig2というネームスペース以下にリファレンスされた「Pigアセット」は、
両方とも __class__Pig を継承しています。
そのため、両方に「赤いマテリアルのマテリアル」が継承されて「最終的に赤く」なっているのです。

この場合「継承」のほうが「リファレンス」より強いので
「赤くするマテリアル」のほうが「Pig.usdaのマテリアル」より強いので
結果Pigが赤くなっています。

「ローカル」のほうが「継承」より強いはずなのに、
なんでローカル（サブレイヤー）されたものが継承で使われるんだろう？となるかもしれませんが
あくまでもサブレイヤーで合成しているのは、
__class__Pig に対してであって /Layout/Pig2/Looks/Pig に対してサブレイヤーしているわけでは
ありません。
なので、この場合は、継承→継承した先のローカル (再帰的なコンポジションの解決)
という扱いになっています。

#### さらなる使いみち

見ての通り、この継承を使用すると
**「１つのレイヤーに読み込んだ別のアセットをまとめて編集できる」** 
わけです。

例えば、あるシーンに本（Bookアセット）が１０００冊おいあり、
この本は５色（赤・青・黃・緑・黒＿のバリアントを持っていて、それがシーンにレイアウトされています。
**この本のうち「赤い本」だけを調整したい**
となった場合。
すべての赤い本だけをサブレイヤーで書き換えをしたいとすると、

/Layout/Book1/Looks/RedBooksMaterial
/Layout/Book2/Looks/RedBooksMaterial

のように **すべてのネームスペース以下のマテリアルを編集しなければいけません。**
これはさすがに冗長です。

ですが、

/__class__Books というClassを Bookアセットのルートプリムに対して継承しておけば

/__class__Books/Looks/RedBooksMaterial
に修正したマテリアルを定義すれば、すべての赤い本を調整することができるわけです。

このとき、マテリアルには 継承・バリアント・リファレンス が関係しているわけですが、
コンポジションの順序は、元の本の「リファレンス」より「赤い本」を選ぶバリアントが強く、
その「バリアント」よりも「継承」のほうが強いので
バリアントで選択された「赤い本」を継承によって編集することができるわけです。

これですべての赤い本が一律で調整できました。
しかし、これのうち１つの赤い本をピンポイントで調整したくなった場合はどうでしょうか。
その場合はサブレイヤー（Local）で値を編集すれば良いです。
そうすることで、この個別の編集が最も強いのでピンポイントの調整ができます。

直感的にわかりにくいかもしれませんが、わかるととても便利なので
ぜひとも活用してほしい機能です。

:fa-download:[サンプルHip](https://1drv.ms/u/s!AlUBmJYsMwMhhOcDFYU8SlEw6WsFyQ?e=B1qh1X)

!!!info
    継承順が
    Local -> Inherits -> Varint -> Reference -> Paylaods -> Specialize
    なのは、考えれば考えるほどうまくできてるよなぁ...と思うのが
    バリアントと継承の位置です。
    バリアントよりリファレンスが強いと、リファレンスの切り替えができなくなるし
    継承がリファレンスより弱いと一律で編集ができなくなります。
    それが、このアセット構築をするだけでもとても良くわかります。

## AssetInfo

一通りの構造は完成したので、最後にこの作成するアセットの情報を仕込みます。
この情報とは、例えば「名前」だったり「バージョン」だったり
それ以外にもアセットに紐づく様々な情報を、usda内に埋め込むことができます。

![](https://gyazo.com/d859caca4afb093ee7f0193b85070533.png)

ペイロードの手前に ConfigurePrimitive と ConfigureLayer を追加します。

![](https://gyazo.com/435f39b7065172f04e229c6e888a5796.png)

ConfigureLayerは、FlattenInputを **Flatten Input Layers** に変更します。
これを入れることで、メタ情報を入れるUSDファイルを pig.usda になるようにします。

次に ConfigurePrimitive

![](https://gyazo.com/95b8dcc2396c85b59053627dd11b7bbc.png)

Primitivesは、メタ情報を入れたいPrimのPathです。
今回は /Pig （Root以下にあるアセット名のPrim）に指定します。

ここは必要な値を入れれば良いですが、今回はアセット名とバージョン、Kind（今回は割愛）
を指定します。
ここに無いパラメーターは、 Custom Data に仕込むことができて

![](https://gyazo.com/c6eb843866344eb42e36eedfa1236d27.png)

こんなかんじに入れると

```
def "Pig" (
    assetInfo = {
        string name = "Pig"
        string version = "1.0"
    }
    customData = {
        double Age = 1
        string whoAreYou = "HoudiniPig"
    }
    kind = "component"
    prepend payload = @anon:0000000126126B00:rootlayer@
)
{
}
```
usd のメタ情報が追加されます。
このようにメタ情報を入れるようにすれば、どのバージョンのファイルなのかを明確化
することができます。

## 完成

![](https://gyazo.com/5bb6d311a44b34c6f9206b2b63cb7a43.png)

いろいろやってきましたが、こんなかんじになりました。
できあがった hip ファイルは :fa-download:[こちら](https://1drv.ms/u/s!AlUBmJYsMwMhhOcoB7mqcctbgbO2GA?e=hGuenm)

これに必要に応じてバリアントで切り替えを追加すれば基本的なUSDの構造を含んだ
アセットができあがります。

重要な点は

1. アセット名のPrim以下にすべてをまとめる
2. １のPrimをデフォルトPrimにする
3. ルート以下に __class__アセット名 のクラスを定義して 1がこれを「継承」
4. １を「ペイロード」する構造をつくる
5. メタ情報を入れておく

がワンセットです。
あとは必要に応じてこの構造を「バリアント」で切り替えできるようにまとめます。

このように作ったアセットは、
複数リファレンスでロードした場合でも「アンロード」で開けば高速に開いて
一部だけロードが必ずできるようになります。
また継承することで、大量にリファレンスされた場合でもまとめて調整が可能になります。

## まとめ

長々とやっていきましたが、
これで一通り基本的なUSDの構造を活用したアセットが出来上がりました。

USDの構造は理解していても、じゃあ実際Houdini上でどうやってやったものかと
結構色々調べていたのですが
調べたおかげでようやく自分の思い描いた構造がSOLARIS上で構築できるようになりました。

ここ最近
[EditLayerの記事](./10_edit_layer.md)とか [LayerFlattenの記事](./11_edit_layer_flatten.md)とか[PreviewSurfaceの記事](./12_usd_preview_surface.md)とかは、
このUSDの構造を構築する方法を調べる過程で理解した内容をまとめたものになります。

一応できはしましたが、実際に運用する場合に考慮しなければいけないことや
まだまだ最適化できることとか、良さげなコンポジション構造をつくるとか
どういうレイヤー分割にするかだったり、工夫する点はたくさんあるとおもうので
今後は色々と試していきたいです。

とくに、マテリアル周りは正直まだ考える余地は多分にあると思います。

あとは、VEXも理解できたので
このあたりの処理をHDA化したりPDGで処理したいみたいなこともいずれは
試してみようかなと思います。

## 参考

* http://graphics.pixar.com/usd/downloads.html
* [usdMakeFileVariantModelAsset.py](https://github.com/PixarAnimationStudios/USD/blob/release/extras/usd/examples/usdMakeFileVariantModelAsset/usdMakeFileVariantModelAsset.py)
* https://www.sidefx.com/docs/houdini/solaris/tutorials.html