# Software

Here, the author describes the hul-common-lib and the cirasame-soft.

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

This program accesses the AD9220 block in the FPGA and reads out the ADC data for the specified number of events.
The data that has just been read is written to the file specified by the argument.
As described in the firmware section, no header or trailer words are added to indicate the end of an event.
Process every 132 words.
**This program should not be terminated by Ctrl-C.**
If users stop it by Ctrl-C, data in the next run will be broken.

Example to read 100 events and store them to hoge.dat.
```shell
       [IP]          [file]   [Events]
ad9220 192.168.10.16 hoge.dat 100
```

#### hgddelay

Please read the Hold Generator sub-section in RAYRAW skeleton firmware section.
In this program, the register value given at 2nd argument is transformed to the mask pattern.
The value of 1 provides the smallest delay amount, and the delay amount increases as the given value increases.
**Please set the register value before calling ad9220 program**

## rayraw-control

ここにrayraw-controlの使い方を書く