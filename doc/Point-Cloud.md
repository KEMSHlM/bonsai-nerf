# OVERVIEW

点群の処理を学んだ際の備忘録．

以下の文献を参考にした．

> [詳細3次元点群](https://www.amazon.co.jp/%E8%A9%B3%E8%A7%A3-3%E6%AC%A1%E5%85%83%E7%82%B9%E7%BE%A4%E5%87%A6%E7%90%86-Python%E3%81%AB%E3%82%88%E3%82%8B%E5%9F%BA%E7%A4%8E%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0%E3%81%AE%E5%AE%9F%E8%A3%85-KS%E7%90%86%E5%B7%A5%E5%AD%A6%E5%B0%82%E9%96%80%E6%9B%B8-%E9%87%91%E5%B4%8E/dp/406529343X)
> 詳細とは思わない, むしろ紹介が近い．広く理解するには良い．点群の数理的な処理に関する点はほとんど載っていない．

> [DIGITAL IMAGE PROCESSING](https://www.amazon.co.jp/%E3%83%87%E3%82%A3%E3%82%B8%E3%82%BF%E3%83%AB%E7%94%BB%E5%83%8F%E5%87%A6%E7%90%86-%E6%94%B9%E8%A8%82%E7%AC%AC%E4%BA%8C%E7%89%88-%E3%83%87%E3%82%A3%E3%82%B8%E3%82%BF%E3%83%AB%E7%94%BB%E5%83%8F%E5%87%A6%E7%90%86%E7%B7%A8%E9%9B%86%E5%A7%94%E5%93%A1%E4%BC%9A/dp/490347464X)
> 2次元画像の処理に関する文書．点群に関する処理はないが，colmapなどの2次元画像から3次元復元に関する知識が得られる．良書の匂いがする．

<!--toc:start-->

- [OVERVIEW](#overview)
- [点群処理の基礎](#点群処理の基礎)
  - [ファイル入出力](#ファイル入出力)
  - [点群の可視化(レンダリング)](#点群の可視化レンダリング)
  - [サンプリング](#サンプリング)
  - [外れ値の除去](#外れ値の除去)
  - [法線推定](#法線推定)
  - [点群の回転](#点群の回転) - [オイラー角による回転](#オイラー角による回転) - [ジンバルロック](#ジンバルロック) - [固定角](#固定角) - [クォータニオンによる回転](#クォータニオンによる回転) - [クォータニオンとは](#クォータニオンとは) - [任意軸回転](#任意軸回転) - [クォータニオンによる回転操作](#クォータニオンによる回転操作) - [複素数の拡張としてのクォータニオン](#複素数の拡張としてのクォータニオン) - [クォータニオンの利点](#クォータニオンの利点)
  <!--toc:end-->

# 点群処理の基礎

## ファイル入出力

点群データの拡張子には，`.ply`, `.pcd`, `.xyz`などがある．  
点群データは，3次元座標(x, y, z), 色の情報(RGB), 受光強度, 反射強度などが含まれる．

- `.ply`
  3Dスキャンデータやその他の形状を格納するために設計された柔軟なファイル形式である．色情報，法線ベクトル，テクスチャ座標などを含むことができる．  
  また，点のデータのみならず，面のデータも保持することができる．

- `.las`
  LiDARデータを格納するための公式ファイル形式で，地理情報システム(GIS)などで一般的に利用される．高度，受光強度，反射強度などの情報を含むことができる．  
  ここで，受光強度とは点がどの程度光を受けっとたかを表し，反射強度は点がどの程度光を反射したかを表す．

- `.pts`
  主に3Dスキャンデータの保存に用いられるシンプルなテキストベースのファイル形式で，点の位置情報の他に色や強度情報を持つことができる．

## 点群の可視化(レンダリング)

点群を可視化するライブラリには，Open3D, PCL(Point Cloud Library)などがある．

- Open3D
  3Dデータ処理用のOSSライブラリ，実装はC++であるが，pythonラッパーから利用することができる．

以下の疑問を解決したい．  
点って体積持たないのに，どうやって可視化してるの？それに，可視化のやり方によって見え方変わらない??

## サンプリング

2次元画像と3次元群データの最も大きな違いはデータ構造にあると行っても過言ではない．  
2次元画像では，画像を構成する点(ピクセル)が等間隔に並んでおり，そのすべての点が輝度やRGBカラー値を持っている．  
3次元点群データは，任意この計測点の3次元座標データの集合である．そのため，計測器などによってデータを取得した際は，計測器に近い物体ほど点が密に取得することができるのに対して，遠い物体では疎の点群が得られる．

点群の点数を減らし，圧縮する手続きを **サンプリング** という．
サンプリングは以下のような目的で行われる．

- データの圧縮
- ノイズの除去
- 特徴の強調

### ランダムサンプリング

名前の通り，ランダムに点を選択する方法．当然，ばらつきが大きい．

### ボクセルによるフィルタリング（サンプリングではない）

以下は，ボクセルとメッシュの違い．  
<img src="https://gamemakers.jp/cms/wp-content/uploads/2022/07/4a426bf19014ea1a078bbfde51996023.png" width=500>

立方体のグリッドを配置し，そのグリッド内の重心を求めて，一点に置き換えることで，点の数を減らし，その分布を一様にすることができる．  
この等間隔に整列した点群を得ることを，ボクセル化と呼ぶ．  
このボクセル化はもともと点が取得できていない箇所では，ボクセルが空であるため，データ構造が疎であることに注意する必要がある．

<img src="https://tech-deliberate-jiro.com/wp-content/uploads/2022/05/image-15-1024x367.png" width=500>

## 外れ値の除去

- 統計的外れ値除去法
  各点とその近傍点との距離の平均値を算出し，この平均値に基づいてある閾値以上になる点を外れ値として除去する．

- 半径的外れ値除去法

## 法線推定

点群の法線ベクトルを推定する．さまざまな点群処理において，法線ベクトルはとても重要な情報になる．  
特に，点群の面と面の位置合わせをするには，法線ベクトルの情報が必要となる．

点群の法線を求めるには，各点の近傍てんを求めて，そして，近傍点群の3次元座標に対して主成分分析を行う．  
主成分分析は，近傍点群の共分散を求めて，その固有値分解を行う分析手法．

通常の場合，例えば次元削除を行うために，主成分分析を行う場合は，固有値の大きものから従運に任意個の固有ベクトルを抽出する．  
固有値の大きい固有ベクトルは，すなわちサンプルの分散が大きい軸を表わしている．  
一方で，最小固有値を持つ固有ベクトルを選べば，これが法線ベクトルとなる．

3次元点群を行列$M$で表すと，

$$
M =
\left[
\begin{matrix}
r_{11} & r_{12} & r_{13} \\ r_{21} & r_{22} & r_{23} \\ r_{31} & r_{32} & r_{33} \\ \vdots & \vdots & \vdots \\ r_{N1} & r_{N2} & r_{N3}
\end{matrix}
\right]
$$

中心座標を原点に移動させた$M'$で表すと，

$$
M' =
\left[
\begin{matrix}
r_{11}-\overline{r}_1 & r_{12}-\overline{r}_2 & r_{13}-\overline{r}_3 \\ r_{21}-\overline{r}_1 & r_{22}-\overline{r}_2 & r_{23}-\overline{r}_3 \\ r_{31}-\overline{r}_1 & r_{32}-\overline{r}_2 & r_{33}-\overline{r}_3 \\ \vdots & \vdots & \vdots \\ r_{N1}-\overline{r}_1  r_{N2}-\overline{r}_2 & r_{N3}-\overline{r}_3
\end{matrix}
\right]
$$

共分散行列$C$(3x3)は，

$$
C = M'^T M'
$$

この共分散行列$C$の固有値を求めることで，法線ベクトルを求めることができる．

$$
U^{T} C U = \Lambda
$$

$$
\Lambda =
\left[
\begin{matrix} \lambda_1 & 0 & 0 \\ 0 & \lambda_2 & 0 \\ 0 & 0 & \lambda_3 \end{matrix}
\right]
$$

ここで，$\lambda_1, \lambda_2, \lambda_3$は固有値であり，$U$は固有ベクトルである．  
$\lambda_1 > \lambda_2 > \lambda_3$が成立するならば，$\lambda_3$に対応する固有ベクトルが法線ベクトルになる．

以下は，2次元点群を用いたイメージ図．

<img src="http://www.sanko-shoko.net/note.php?img=ynvx" width=500>

固有ベクトルはそれぞれ垂直であるため，上の図を3次元点群で考えると  
法線ベクトルが最も小さい固有値に対応する固有ベクトルとなるのは，容易に想像できる．

## 点群の回転

回転を数学的に表現する上で, オイラー角,回転行列とクォータニオンがある．

> [参考:回転行列、クォータニオン(四元数)、オイラー角の相互変換](https://qiita.com/aa_debdeb/items/3d02e28fb9ebfa357eaf)

### オイラー角による回転

オイラー角は，3つの角度の組で表される．  
一方の座標系を(x, y, z)で表し，他方を(X, Y，Z)で表す．

以下の図の例では，

1. (x, y, z)をz軸周りに角度をα回転させ，(x\', y\', z\')とする．
2. (x\', y\', z\')をx\'軸周りに角度をβ回転させ，(x\'\', y\'\', z\'\')とする．
3. (x\'\', y\'\', z\'\')をz\'\'軸周りに角度をγ回転させ，(X, Y, Z)とする．

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/a1/Eulerangles.svg/300px-Eulerangles.svg.png" width=500>

上記の定義では，z軸-x\'軸-z\'\'軸の順で回転する．この順番をオイラー角の回転順序といい，  
この場合z-x-z系のオイラー角と呼ばれる．(左に書いてある方が内側)  
この回転順序には，i-j-kタイプとi-j-iタイプがある．そのため，3! + 3C1 = 12通りのオイラー角が存在する．

逆を言い返せば，(x, y, z)から(X, Y, Z)へ変換するには，どんな変換であれ，3回の手続きで変換可能であり，その回転方法は12通りあるということだ．

回転行列の積で表すと，

$$
{\begin{align}
\boldsymbol{R} _{xyz} (\alpha, \beta, \gamma)
& = \boldsymbol{R} _x (\alpha) \boldsymbol{R} _y (\beta) \boldsymbol{R} _z (\gamma)  \\
& = \begin{pmatrix}
1 & 0 & 0 \\
0 & \cos \alpha& -\sin \alpha\\
0 & \sin \alpha& \cos \alpha
\end{pmatrix}
\begin{pmatrix}
\cos \beta& 0 & \sin \beta\\
0 & 1 & 0 \\
-\sin \beta& 0 & \cos \beta
\end{pmatrix}
\begin{pmatrix}
1 & 0 & 0 \\
0 & \cos \gamma& -\sin \gamma\\
0 & \sin \gamma& \cos \gamma
\end{pmatrix}
\end{align}
}
$$

アニメーションは以下の通り．  
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/85/Euler2a.gif/220px-Euler2a.gif" width=500>

#### ジンバルロック

三つの軸のうち，二つの軸が一致してしまうと，本来ある自由度３が，自由度が２になってしまう現象．  
代数的な説明で${\frac{\partial R}{\partial \theta}, \frac{\partial R}{\partial \phi}, \frac{\partial R}{\partial \psi}}$が線形従属になる時を，ジンバルロックという．  
オイラー角X-Y-Z系で，Y軸周りの回転が90度の時にジンバルロックが発生する．

<img src="https://camo.qiitausercontent.com/27d9828e2b985b72e686721aad67caa046eff37e/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3332333537362f35343636343731372d373562342d316336622d393362352d3530313037373734623039372e676966" width=500>

> [参考:ジンバルロックDEMO](https://arihide.github.io/demos/gimbal/)  
> [参考:ジンバルロック解説](https://qiita.com/Arihi/items/4b306feb3d9e6cd93204)

<img src="../Threejs/.img/qrcode_arihide.github.io.png">

#### 固定角

**オイラー角X-Y-Z系の場合**

1. 初期の座標系を(x-y-z)とする．
2. (x, y, z)をx軸周りに角度をα回転させ，(x\', y\', z\')とする．
3. (x\', y\', z\')をy\'軸周りに角度をβ回転させ，(x\'\', y\'\', z\'\')とする．
4. (x\'\', y\'\', z\'\')をz\'\'軸周りに角度をγ回転させ，(X, Y, Z)とする．

**固定角X-Y-Z系の場合**

1. 初期の座標系を(x-y-z)とする．
2. (x, y, z)をx軸周りに角度をα回転させ，(x\', y\', z\')とする．
3. (x\', y\', z\')をy軸周りに角度をβ回転させ，(x\'\', y\'\', z\'\')とする．
4. (x\'\', y\'\', z\'\')をz軸周りに角度をγ回転させ，(X, Y, Z)とする．

固定角は回転行列で表現すると，以下のようになる．

$$
{\begin{align}
\boldsymbol{R} _{xyz} (\alpha, \beta, \gamma)
& = \boldsymbol{R} _z (\gamma) \boldsymbol{R} _y (\beta) \boldsymbol{R} _x (\alpha)  \nonumber \\
& =
\begin{pmatrix}
\cos \gamma& -\sin \gamma& 0  \\
\sin \gamma& \cos \gamma& 0  \\
0 & 0 & 1
\end{pmatrix}
\begin{pmatrix}
\cos \beta& 0 & \sin \beta\\
0 & 1 & 0 \\
-\sin \beta& 0 & \cos \beta
\end{pmatrix}
\begin{pmatrix}
1 & 0 & 0 \\
0 & \cos \alpha& -\sin \alpha\\
0 & \sin \alpha& \cos \alpha
\end{pmatrix}
\end{align}
}
$$

### クォータニオンによる回転

#### クォータニオンとは

クォータ二オンは,任意軸回転させる．

ここで，留意しておくべき点は，回転と姿勢の定義である．  
<img src="https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F182963%2F746087b2-4d80-4403-d7e6-c6b4e37fc5eb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&w=1400&fit=max&s=75a4e3e1761e480baf71e4a7ad1a2940" width=500>

方向すなわち回転軸は，三次元ベクトルの大きさを１とすると2次元で表される．  
姿勢は，方向+1次元の3次元で表される．

三次元空間にて，

方向ベクトル$(\lambda_x, \lambda_y, \lambda_z)$を回転軸として, 角度$\theta$回転させるという「回転クォータニオン」は，  
四次元ベクトル$(\cos\frac{\theta}{2}, \lambda_x \sin\frac{\theta}{2}, \lambda_y \sin\frac{\theta}{2}, \lambda_z \sin\frac{\theta}{2})$

なぜ，$\frac{\theta}{2}$ を用いるのかは，後述する．

#### 任意軸回転

<img src="https://storage.googleapis.com/zenn-user-upload/p4v2a4qfpu55wf2lglayn2nuf64w" width=500>

回転軸となる正規化ベクトルを$\vec{n}$として，角度$\theta$回転を行うことする．  
回転の対象になる位置ベクトル$\vec{r}$を回転後の位置ベクトルを$\vec{r'}$とする．

$$
{\begin{align}
\vec{r_{\|}} = (\vec{n}\cdot\vec{r})\vec{n} \nonumber \\
\vec{r_{\perp}} = \vec{r} - \vec{r_{\|}} \nonumber
\end{align}}
$$

ここで，回転軸$\vec{n}$ と垂直成分$\vec{r_{\perp}}$に直行している回転面にあるベクトル$\vec{v}$を導入する．

$$
\vec{r_{\perp}^{'}} = \cos{\theta}\vec{r_{\perp}} + \sin{\theta}\vec{v}
$$

あとは，垂直成分ベクトル$\vec{r_{\|}}$との合成により，

$$
{\begin{align}
\vec{r_{\perp}^{'}} &= \vec{r_{\|}} + \vec{r_{\perp}} \nonumber \nonumber \\
&= \vec{r_{\|}} + (\cos\theta)\vec{r_\perp} + (\sin\theta)\vec{v} \nonumber \\
&= (\cos\theta)\vec{r} + (1-\cos\theta)(\vec{n}\cdot\vec{r})\vec{n} + (\sin\theta)(\vec{n}\times\vec{r})
\end{align}}
$$

#### クォータニオンによる回転操作

以下は，クォータニオンによる外積の式である．

$$
{\begin{align}
q \otimes p = (
&q_w p_x - q_z p_y + q_y p_z + q_x p_w, \nonumber \\
&q_z p_x + q_w p_y - q_x p_z + q_y p_w, \nonumber \\
&-q_y p_x + q_x p_y + q_w p_z + q_z p_w, \nonumber \\
&-q_x p_x - q_y p_y - q_z p_z + q_w p_w)
\end{align}
}
$$

行列式を使うと，

$$
{q \otimes p =
\begin{pmatrix}
q_w & -q_z & q_y & q_x \\
q_z & q_w & -q_x & q_y \\
-q_y & q_x & q_w & q_z \\
-q_x & -q_y & -q_z & q_w
\end{pmatrix}
\begin{pmatrix}
p_x \\
p_y \\
p_z \\
p_w
\end{pmatrix}
}
$$

クォータ二オンによる回転操作は，非常にシンプルである．  
三次元ベクトル$\vec{v}$に回転$q$を実施すると，$q\otimes (0, \vec{v}) \otimes \overline{q}(=q^{-1})$ ここで，$\overline{q}$は$q$の逆回転である．  
姿勢クォータニオン$p$に回転$q$を実施すると，$q\otimes p$.
クォータニオンのw成分を0にして掛け算したものは，ベクトルの外積に対応している．

そのため，$p=(0, \vec{v})$として，以下を計算すると

$$
{\begin{align}
p^{'} &= q \otimes p \otimes q^{-1} \nonumber \\
&= (w, \vec{v}) (0, \vec{r}) (w, -\vec{v}) \nonumber \\
&= (0, \cos{2\phi\vec{r}}) + (1-\cos{2\phi})(\vec{n}\cdot\vec{r})\vec{n} + \sin2\phi(\vec{n}\times\vec{r})) \\
\end{align}}
$$

となり，先ほどの任意軸回転の式と近い形になる．

ここで，回転角が$2\phi$であることを考慮して，

$$
q = (\cos{\frac{\phi}{2}}, \sin{\frac{\phi}{2}}\vec{n})
$$

#### 複素数の拡張としてのクォータニオン

クォータニオンは複素数の拡張と言える．  
| | 対応ベクトル | 実現する回転 | 回転行列 |
|-------------- | -------------- | -------------- | -------------- |
| 複素数 $x + yi$ | 二次元ベクトル | 二次元回転 | 2x2 |
| 四元数 $w + xi + yj+ zk$ | 四次元ベクトル | 三次元回転 | 3x3 |

複素数と同様な性質を持つ．  
$i^2 = j^2 = k^2 = ijk = -1$のルールが定まれば，以下のような表が作れる．  
| | i | j | k |
|---------------- | --------------- | --------------- | --------------- |
| i | -1 | k | -j |
| j | -k | -1 | i |
| k | j | -i | -1 |

また，複素数と同様な定義を持つ．

- ノルム
  $$
  |q| = \sqrt{{q_w}^2 + {q_x}^2 + {q_y}^2 + {q_z}^2}
  $$
- 共役四元数
  $$
  |\overline{q}| = q_w - q_x\bf{i} - q_y\bf{j} - q_z\bf{k}
  $$
- 逆数
  $$
  q^{-1} = \frac{\overline{q}}{|q|^2}
  $$

逆数を持つため，体である．

#### クォータニオンの利点

- メモリ的に効率的
- 計算時間が短い
- 回転行列の性質を引き継いでるので，回転の結合法則も成り立つ．
- ジンバルロックが発生しない.
- 計算時に数値誤差が生じにくいらしい．
- 回転お連続的変化で補間計算が容易．
- 結合法則が成り立ち，複数の回転の操作でも扱いやすい．
