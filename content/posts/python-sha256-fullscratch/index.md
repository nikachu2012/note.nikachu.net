+++
title = 'PythonでSHA256をフルスクラッチした'
author = 'nikachu2012'
date = 2024-02-11T00:27:48+09:00
description = "importを用いずPythonでSHA256を計算しました。"
tags = [
    "python",
    "sha256",
    "fullscratch"
]
+++

どうも金ないニカチュウと申します。
Pythonでimportを用いずにSHA256のハッシュを計算するプログラムを書きました。  

## hashlib
PythonでSHA256のハッシュを計算するには偉大な `hashlib` ライブラリを用いるのが簡単。

```python
>>> import hashlib
>>> hashlib.sha256(b"note.nikachu.net").hexdigest()
'f3bb10e2172099a8a6425ef9c77ccfe5101e440c713e3b8e2362c0d748342af0'
```

まあこんなライブラリで楽するのは良くないですね。  
私の学校の教員も言っていました。
> 楽しちゃダメだから  
> — 某高専某教員


## フルスクラッチ
**以下に紹介するプログラムを用いて何が起きても責任は負いません**  
**信頼性が必要なら`hashlib`を用いてください**


SHA256の仕様は以下のPDFに記載されている。
https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf


まずはH, Kを用意する。

```py
H: list[int] = [
    0x6A09E667, 0xBB67AE85, 0x3C6EF372, 0xA54FF53A, 0x510E527F, 0x9B05688C, 0x1F83D9AB, 0x5BE0CD19
]

K: list[int] = [
    0x428A2F98, 0x71374491, 0xB5C0FBCF, 0xE9B5DBA5, 0x3956C25B, 0x59F111F1, 0x923F82A4, 0xAB1C5ED5, 0xD807AA98, 0x12835B01, 0x243185BE, 0x550C7DC3, 0x72BE5D74, 0x80DEB1FE, 0x9BDC06A7, 0xC19BF174, 0xE49B69C1, 0xEFBE4786, 0x0FC19DC6, 0x240CA1CC, 0x2DE92C6F, 0x4A7484AA, 0x5CB0A9DC, 0x76F988DA, 0x983E5152, 0xA831C66D, 0xB00327C8, 0xBF597FC7, 0xC6E00BF3, 0xD5A79147, 0x06CA6351, 0x14292967, 0x27B70A85, 0x2E1B2138, 0x4D2C6DFC, 0x53380D13, 0x650A7354, 0x766A0ABB, 0x81C2C92E, 0x92722C85, 0xA2BFE8A1, 0xA81A664B, 0xC24B8B70, 0xC76C51A3, 0xD192E819, 0xD6990624, 0xF40E3585, 0x106AA070, 0x19A4C116, 0x1E376C08, 0x2748774C, 0x34B0BCB5, 0x391C0CB3, 0x4ED8AA4A, 0x5B9CCA4F, 0x682E6FF3, 0x748F82EE, 0x78A5636F, 0x84C87814, 0x8CC70208, 0x90BEFFFA, 0xA4506CEB, 0xBEF9A3F7, 0xC67178F2,
]
```

次に使う関数を定義する。
```python
def Ch(x, y, z) -> int:
    return (x & y) ^ (~x & z)


def Maj(x, y, z) -> int:
    return (x & y) ^ (x & z) ^ (y & z)


def Rotr(x: int, n: int) -> int:
    return (x << (32 - n)) & 0xFFFFFFFF | (x >> n)


def Shr(x: int, n: int) -> int:
    return x >> n


def SmallSigma0(x: int) -> int:
    return Rotr(x, 7) ^ Rotr(x, 18) ^ Shr(x, 3)


def SmallSigma1(x: int) -> int:
    return Rotr(x, 17) ^ Rotr(x, 19) ^ Shr(x, 10)


def LargeSigma0(x: int) -> int:
    return Rotr(x, 2) ^ Rotr(x, 13) ^ Rotr(x, 22)


def LargeSigma1(x: int) -> int:
    return Rotr(x, 6) ^ Rotr(x, 11) ^ Rotr(x, 25)
```
右ローテートや右シフトなどは普通だが、加算や左シフトを行う場合気を付ける点がある。  
Pythonの整数型は精度が無限(何桁でも保持できる)ので、32ビットで表せる範囲を超えてしまう可能性があるため、`0xFFFFFFFF`(`0b1`が32ビット分)でマスクして範囲を超えないようにしている。

### メッセージブロックを生成

まずは、データの次に入力直後の`0x80`、末尾8バイトにデータのビット数を入れ、総バイト数が64バイトの倍数になるように調整する関数を作る。
```python
def padding(message: bytes) -> bytes:
    meslen = len(message)

    nullPadding = b"\x00" * (55 - (meslen % 64))

    return message + b"\x80" + nullPadding + (meslen * 8).to_bytes(8, byteorder="big")
```

次に64バイトごとの配列に分割する関数を作る。
```python
def split(input: bytes) -> list[bytes]:
    buf: list[bytes] = []

    for i in range(-(-len(input) // 64)):
        buf.append(input[64 * i : (64 * i) + 64])

    return buf
```

このようにしてメッセージブロックを生成する。
```py
inputData = "note.nikachu.net"

pad = padding(inputData.encode())
messageBlocks = split(pad)
```

### メッセージスケジュールを生成
次はメッセージブロックからメッセージスケジュールを生成する。

まずはバイト列を4バイト(32ビット)ごとint配列に変換する関数を実装する。
```py
def toIntArray(b: bytes) -> list[int]:
    temp: list[int] = []

    for i in range(len(b) // 4):
        temp.append(int.from_bytes(b[i * 4 : i * 4 + 4], byteorder="big"))
    return temp
```

また、以下からのコードはメッセージブロックごとに行う。
```python
for i in messageBlocks:
```

メッセージスケジュールは、本来32bit*64=256バイトとなる。  
メッセージブロックをint配列に変換しても、64バイト分しかないため、残りを計算する。

```python
    words = toIntArray(i)

    for j in range(16, 64):
        w = (SmallSigma1(words[j - 2]) + words[j - 7] + SmallSigma0(words[j - 15]) + words[j - 16]) & 0xFFFFFFFF
        words.append(w)
```

これでメッセージスケジュールが生成できた。

### ローテーション処理
次にローテーション処理を行う。
```py
    a, b, c, d, e, f, g, h = H

    for t, w in enumerate(words):
        t1 = (h + LargeSigma1(e) + Ch(e, f, g) + K[t] + w) & 0xFFFFFFFF
        t2 = (LargeSigma0(a) + Maj(a, b, c)) & 0xFFFFFFFF

        h = g
        g = f
        f = e
        e = (d + t1) & 0xFFFFFFFF
        d = c
        c = b
        b = a
        a = (t1 + t2) & 0xFFFFFFFF
```

### 中間ハッシュ値の更新
```python
    H[0] = (a + H[0]) & 0xFFFFFFFF
    H[1] = (b + H[1]) & 0xFFFFFFFF
    H[2] = (c + H[2]) & 0xFFFFFFFF
    H[3] = (d + H[3]) & 0xFFFFFFFF
    H[4] = (e + H[4]) & 0xFFFFFFFF
    H[5] = (f + H[5]) & 0xFFFFFFFF
    H[6] = (g + H[6]) & 0xFFFFFFFF
    H[7] = (h + H[7]) & 0xFFFFFFFF
```
ここまでの操作を全メッセージブロックに対して行う。

### ハッシュ値の出力
```python
for i in H:
    print(f"{i:04x}", end="")
```

実行すると、以下のようにハッシュを取得できた。

```
f3bb10e2172099a8a6425ef9c77ccfe5101e440c713e3b8e2362c0d748342af0
```
---

## 全コード

{{< gist nikachu2012 61173991e5c395f6afbc68a75ce4b65e main.py >}}
