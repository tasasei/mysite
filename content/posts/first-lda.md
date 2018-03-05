---
title: "Rのldaの使い方"
date: 2018-03-04T21:18:53+09:00
draft: true
---

先日 lda の使い方を説明する機会があったので、その時書いたものを晒します。
大した量は無いですが、無断使用OKと一応書いておきます。

# ldaを実行する
R が標準で内蔵するライブラリの1つ、MASS に含まれる線形判別を行う関数である lda に
ついて説明を行います。動作確認に使うのは iris (アヤメ) データ。
これもデフォルトで R に入ってます。こんなデータ↓

```R
head(iris)
##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
## 1          5.1         3.5          1.4         0.2  setosa
## 2          4.9         3.0          1.4         0.2  setosa
## 3          4.7         3.2          1.3         0.2  setosa
## 4          4.6         3.1          1.5         0.2  setosa
## 5          5.0         3.6          1.4         0.2  setosa
## 6          5.4         3.9          1.7         0.4  setosa
unique(iris$Species)
## [1] setosa     versicolor virginica 
## Levels: setosa versicolor virginica
nrow(iris)
## [1] 150
```

2クラスに絞ったirisデータをこのままldaに入れてみます。
実行して、出てきた変数を確認します。

```R
library(MASS)
iris2 <- iris[1:100,]
iris2$Species <- factor(iris2$Species)

mdl <- lda(iris2[,1:4],iris2$Species)
res <- predict(mdl,iris2[1:100,1:4])

## 推定されたクラス。
head(res$class)
## [1] setosa setosa setosa setosa setosa setosa
## Levels: setosa versicolor

## 線形判別の係数。入力ベクトルが4次元なので4係数。
print(mdl$scaling)
##                    LD1
## Sepal.Length -0.300458
## Sepal.Width  -1.773845
## Petal.Length  2.142260
## Petal.Width   3.035726

## 係数 * 変数。0 が決定境界になるよう調整されている。
## as.matrix(iris[,1:4]) %*% mdl$scaling に定数を加えたもの。
head(res$x)
##         LD1
## 1 -5.508619
## 2 -4.561605
## 3 -5.070508
## 4 -4.434626
## 5 -5.655958
## 6 -5.058471

## 事後確率。
head(res$posterior)
##   setosa   versicolor
## 1      1 4.927872e-25
## 2      1 7.437990e-21
## 3      1 4.225398e-23
## 4      1 2.702481e-20
## 5      1 1.102840e-25
## 6      1 4.775094e-23

## 事前確率。教師データに含まれる各クラスの割合で算出されるがlda や predict.lda の
## 引数で与えることもできる。これとxを使って事後確率が計算される。
print(mdl$prior)
## setosa versicolor 
##    0.5        0.5 
```

3クラスでも確認してみます。大体おなじ。

```R
library(MASS)
mdl <- lda(iris[,1:4],iris$Species)
res <- predict(mdl,iris[,1:4])

tail(res$class)
## [1] virginica virginica virginica virginica virginica virginica
## Levels: setosa versicolor virginica

## 3クラスなので2軸
print(mdl$scaling)
##                     LD1         LD2
## Sepal.Length  0.8293776  0.02410215
## Sepal.Width   1.5344731  2.16452123
## Petal.Length -2.2012117 -0.93192121
## Petal.Width  -2.8104603  2.83918785

head(res$x)
##           LD1        LD2
## [1,] 8.061800  0.3004206
## [2,] 7.128688 -0.7866604
## [3,] 7.489828 -0.2653845
## [4,] 6.813201 -0.6706311
## [5,] 8.132309  0.5144625
## [6,] 7.701947  1.4617210

head(res$posterior)
##      setosa   versicolor    virginica
## [1,]      1 3.896358e-22 2.611168e-42
## [2,]      1 7.217970e-18 5.042143e-37
## [3,]      1 1.463849e-19 4.675932e-39
## [4,]      1 1.268536e-16 3.566610e-35
## [5,]      1 1.637387e-22 1.082605e-42
## [6,]      1 3.883282e-21 4.566540e-40

print(mdl$prior)
##    setosa versicolor  virginica 
## 0.3333333  0.3333333  0.3333333 
```

## 結果をプロットする

結果をプロットしてみます。

```R
library(MASS)
mdl <- lda(iris[,1:4],iris$Species)
res <- predict(mdl,iris[,1:4])
png("iris.png",height=960, width=960, res=144)
plot(iris$Sepal.Length,iris$Sepal.Width,col=iris$Species,pch=as.numeric(res$class))

## 誤判別だけマーキング
miss.row <- which(res$class!=iris$Species)
points(iris$Sepal.Length[miss.row],iris$Sepal.Width[miss.row],cex=2.3,col="blue",lwd=1.5)

leg <- unique(iris$Species)
legend("topleft",legend=leg,col=leg,pch=as.numeric(leg))
dev.off()
```

![image](/mysite/images/iris.png)
