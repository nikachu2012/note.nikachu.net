+++
title = 'OS作(ろうとす)る #1'
author = 'nikachu2012'
date = 2024-02-11T20:00:00+09:00
description = "importを用いずPythonでSHA256を計算しました。"
tags = [
    "c",
    "os",
    "fullscratch"
]
series = ["OS作(ろうとす)る"]
+++

## 目的
[ゼロからのOS自作入門](https://www.amazon.co.jp/dp/4839975868)を読みながら、UEFI上でHello, world!を表示する。

## まずは機械語でHello, world!
### 機械語を書く
本を読んでいくと、まずはバイナリエディタを用いて機械語を書くとのことだったので、[Okteta](https://apps.kde.org/okteta/)を用いてバイナリを書いていく。 

{{<figure src="image1.png" caption="お怒りのようです" width="50%">}}

大量に行があるのでどこかわからなくなるのが頻繁に起きた。
書き終わったのでsumコマンドを用いてチェックサムが一致するかを調べたが、同じにならない。  

  
どこを間違えたのかと思ったら、1行抜けていたり、見間違えていたりした。
何回か試して直すを繰り返し、やっとチェックサムが一致した。
  
書いた機械語は以下に示す。
```
0000000 4d 5a 00 00 00 00 00 00 00 00 00 00 00 00 00 00 >MZ..............<
0000020 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 >................<
*
0000060 00 00 00 00 00 00 00 00 00 00 00 00 80 00 00 00 >................<
0000100 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 >................<
*
0000200 50 45 00 00 64 86 02 00 00 00 00 00 00 00 00 00 >PE..d...........<
0000220 00 00 00 00 f0 00 22 02 0b 02 00 00 00 02 00 00 >......".........<
0000240 00 02 00 00 00 00 00 00 00 10 00 00 00 10 00 00 >................<
0000260 00 00 00 40 01 00 00 00 00 10 00 00 00 02 00 00 >...@............<
0000300 00 00 00 00 00 00 00 00 06 00 00 00 00 00 00 00 >................<
0000320 00 30 00 00 00 02 00 00 00 00 00 00 0a 00 60 81 >.0............`.<
0000340 00 00 10 00 00 00 00 00 00 10 00 00 00 00 00 00 >................<
*
0000400 00 00 00 00 10 00 00 00 00 00 00 00 00 00 00 00 >................<
0000420 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 >................<
*
0000600 00 00 00 00 00 00 00 00 2e 74 65 78 74 00 00 00 >.........text...<
0000620 14 00 00 00 00 10 00 00 00 02 00 00 00 02 00 00 >................<
0000640 00 00 00 00 00 00 00 00 00 00 00 00 20 00 50 60 >............ .P`<
0000660 2e 72 64 61 74 61 00 00 1c 00 00 00 00 20 00 00 >.rdata....... ..<
0000700 00 02 00 00 00 04 00 00 00 00 00 00 00 00 00 00 >................<
0000720 00 00 00 00 40 00 50 40 00 00 00 00 00 00 00 00 >....@.P@........<
0000740 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 >................<
*
0001000 48 83 ec 28 48 8b 4a 40 48 8d 15 f1 0f 00 00 ff >H..(H.J@H.......<
0001020 51 08 eb fe 00 00 00 00 00 00 00 00 00 00 00 00 >Q...............<
0001040 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 >................<
*
0002000 48 00 65 00 6c 00 6c 00 6f 00 2c 00 20 00 77 00 >H.e.l.l.o.,. .w.<
0002020 6f 00 72 00 6c 00 64 00 21 00 00 00 00 00 00 00 >o.r.l.d.!.......<
0002040 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 >................<
*
0003000
```

0x2000番地以降に文字列のデータがあることぐらいしかわからない。  
  

### USBメモリにコピーして起動
USBメモリに `efi/boot/` フォルダを作り、そこに作ったバイナリデータを`BOOTX64.EFI`という名前で保存する。  
そのUSBメモリからブートすれば`Hello, World!`が表示される。  
USBメモリをサブPCに刺し、起動してブートメニューを開く。しかし表示されない。  
サブPCは4000円で買ったMac Proなのだが、Apple製品は普通じゃないので他のコンピュータで試してみることにした。  
  
{{<figure src="image_macpro.jpeg" caption="Mac Pro 2010" width="50%">}}
  
家に転がっているCore 2 Duo(Centrino2)のPCに刺して起動したが、やはり起動しない。  
ちょうど飯の時間だったので飯を食いながら考えていたが、UEFIのコンピュータではなくBIOSのコンピュータであることに気づいた。
  
編集した後すぐに試せなくなるので、めんどくさかったがUEFIに対応しているメインコンピュータで、再起動してUSBメモリから起動してみた。
今度はしっかり起動し、以下のようになった。

{{<figure src="image2_helloworld.png" caption="機械語でHello, world!">}}
  
### バイナリを変えて文字列を変える
また、バイナリをいじり、少し長い文字列を表示しようとした。機械語は読めないので適当に長くしたら、画面がまずいことになった。  
多分読み取るバイト数の指定がどこかにあるのだろうが、めんどくさいので後にする。  

まずくなった画面は以下に示す。

{{<figure src="image3_bug.png">}}

また、UEFIが日本語に対応していることもわかった。

とりあえず遊んでないで次へ進む。

## C言語でHello, world!
### ソースコード
解説が少ないのでよくわからなかったが、以下のコードをC言語で書き、ビルドすることでHello, world!を表示することができる。

```c
typedef unsigned short CHAR16;
typedef unsigned long long EFI_STATUS;
typedef void *EFI_HANDLE;

struct _EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL;
typedef EFI_STATUS (*EFI_TEXT_STRING)(
  struct _EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL  *This,
  CHAR16                                   *String);

typedef struct _EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL {
  void             *dummy;
  EFI_TEXT_STRING  OutputString;
} EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL;

typedef struct {
  char                             dummy[52];
  EFI_HANDLE                       ConsoleOutHandle;
  EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL  *ConOut;
} EFI_SYSTEM_TABLE;

EFI_STATUS EfiMain(EFI_HANDLE        ImageHandle,
                   EFI_SYSTEM_TABLE  *SystemTable) {
  SystemTable->ConOut->OutputString(SystemTable->ConOut, L"Hello, world!\n");
  while (1);
  return 0;
}
```
C言語なのにメンバ関数が存在しているのがよくわからないが、このコードでも同じように`Hello, world!`を表示することができた。

### 結果
{{<figure src="image_helloworldonc.png" caption="C言語を用いた文字の出力">}}

`\n`はよく改行コードとして用いられているが、本来はLine Feed(次の行に送る)という意味であるため、行頭が戻っていない。行頭を戻すためには`\r` (Carriage Return)を追加する必要がある。  
薄々知ってはいたが、ここでその違いを目にするとは思っていなかった。  
  
また、QEMUを用いることで再起動しなくてもコンピュータ上で試せるようになった。

## C言語とライブラリを用いてHello, world!
[EDK II](https://github.com/tianocore/edk2)というライブラリを用いて、Hello, world!を短いコードで作る。
### ソースコード
```c
#include  <Uefi.h>
#include  <Library/UefiLib.h>

EFI_STATUS EFIAPI UefiMain(
    EFI_HANDLE image_handle,
    EFI_SYSTEM_TABLE *system_table) {
  Print(L"Hello, Mikan World!\n");
  while (1);
  return EFI_SUCCESS;
}
```
### 結果
{{<figure src="image_hellow_c_edk2.png" caption="C言語とEDK IIのライブラリでHello, world!">}}

## 現在と今後
現在は、メモリマップの取得をし、ファイル保存をすることまで行えている。

{{<figure src="image4_memmap.png" caption="C言語とEDK IIのライブラリでHello, world!">}}

{{<figure src="image_memmap_file.png" caption="メモリマップのファイル保存">}}

