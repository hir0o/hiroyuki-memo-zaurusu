# sedコマンド

sedは文字の置換ができるコマンド.

他に、[[trコマンド]] でも文字の置換ができるが、sedの方が新しく、オプションが豊富?

sは、文字を置換するという意味.
最初に発見した文字を置換する

```
$ echo 'shibuya hiroyuki hiroyuki' | sed 's/hiroyuki/tanaka/'
> shibuya tanaka hiroyuki
```

複数回出現する文字を置換するには、gをつける

```
$ echo 'shibuya hiroyuki hiroyuki | sed 's/hiroyuki/tanaka/g'
> shibuya tanaka tanaka
```

拡張正規表現を使う場合はEオプションをつける


### sedで特定の行から行までを抽出する

`sed -n '/regex1/,/regex2/p'` とすると、regix1からregix2までの行を抽出できる

```
$ seq 9 | sed -n '/2/,/4/p'
2
3
4
```
