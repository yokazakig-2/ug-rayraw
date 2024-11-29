# Software

Here, the author describes the hul-common-lib and the rayraw-soft.

## hul-common-lib

[Github repository](https://github.com/spadi-alliance/hul-common-lib)

Since the basic structure of The RAYRAW firmware is the same that of AMANEQ's firmware, the RAYRAW board can be controlled by hul-common-lib.
Before reading this section, see the [AMANEQ user guide](https://spadi-alliance.github.io/ug-amaneq/software/software/).
In this user guide, the author additionally explains about programs for RAYRAW.

## rayraw-soft

[Github repository](https://github.com/spadi-alliance/rayraw-soft)

`rayraw-soft`はRAYRAWのファームウェア専用の機能を提供します。
コンパイルするためには`hul-common-lib`が必要です。

### Executable files

#### TrgFW/daq

Triggered-type ADC multi-hit TDC専用のデータ収集用プログラムです。
引数で指定したサーチ窓をADCとTDCに設定し、指定したイベント数データを取得します。
指定したイベント数取得した後自動的にプログラムは終了しますが、`Ctr-C`で割り込んで止める事も可能です。
プログラムを実行した場所に`data`ディレクトリがある必要があります、あらかじめ作成しておいてください。

このプログラムは1台のボードからデータを取得するための簡易的なDAQプログラムです。
**このプログラムを使って物理実験を行う事は想定してません。必ずお使いのDAQソフトウェアへ機能を移植してください。**
このプログラム内では

- トリガー種類の選択
- トリガー入力をNIM-IN1へアサイン
- TDCとADCへのサーチ窓の設定
- イベントカウンタのリセット
- ADCブロックの初期化状況の確認

を行ってからTCP接続を行いデータ読み出しを開始します。
上記はRAYRAWボードの制御であり、データ読み出しのシーケンスとは独立のため1つのプログラム内で全て行う必要は本来ありません。
このプログラムでは簡単のために全てを1つの実行体内で行っていますが、移植の際には制御部は適宜分離してください。

```shell
    [IP]           [Run No] [Num of events] [Window max] [Window min]
daq 192.168.10.16  1        10000           500              300
```

上記の例ではサーチ窓を300から500にセットし、10000イベントを`./data/run1.dat`というファイルへ保存します。
サーチ窓のLSB精度およびビット幅はファームウェアのシステムクロック周期に依存します。
ファームウェアの項目を参照してください。
ここで指定した値がTDCとADC両方に対して使用されます。
サーチ窓値のLSB精度をA nsとすると、上記例ではコモンストップ信号からみて`300*A - 500*A`秒過去のデータが取得できます。


## rayraw-control

ここにrayraw-controlの使い方を書く