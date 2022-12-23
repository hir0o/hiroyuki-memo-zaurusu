# 画像をbase64に変換したい

```ts
const getBase64Image = async (imageUrl: string) => {
  const image = await fetch(imageUrl)

  return new Promise<string>((resolve) => {
    image.blob().then((blob) => {
    const reader = new FileReader()
    reader.readAsDataURL(blob)
    reader.onloadend = () => {
      const base64data = reader.result
      resolve(base64data as string)
    }
    })
  })
}
```

[JavaScriptで画像ファイルをbase64文字列に変換する方法！ | コードライク](https://codelikes.com/javascript-image-to-base64/)