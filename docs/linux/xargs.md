# xargsコマンド

縦の出力を横に並べられる.

これは、xargsにコマンドを渡さなかったらデフォルトでechoを渡すという挙動を利用している。

```
$ seq 4 | xargs
1 2 3 4
```

パイプで渡ってきたものをコマンドの引数として渡せる

```
$ seq 4 | xargs mkdir
$ ls
1/ 2/ 3/ 4/
```

-n オプションで、指定した個数分を引数として渡せる。

```
$ mkdir 1 3
$ seq 4 | xargs -n 2 mv
$ ls
2/ 4/
```

-I オプションで、文字を後続のコマンドにわせる. (変数化?)

```
$ seq 4 | xargs -I DIR_NAME mkdir dir_DIR_NAME
```


### 並列実行

-Pオプションで、並列実行する数を指定できる。
デフォルトだと並列実行されない。

```
$ ls *.png | sed 's/\.png$//' | xargs -P 2 -I @ convert @.png @.jpg
```

nprocコマンドで並列実行できる数を調べられる

```
$ nproc
10
```

[[コマンド置換]]でnprocを-Pオプションに渡すと、そのマシンの最大の並列実行数でコマンドを実行できる

```
$ ls *.png | sed 's/\.png$//' | xargs -P $(nproc) -I @ convert @.png @.jpg
```