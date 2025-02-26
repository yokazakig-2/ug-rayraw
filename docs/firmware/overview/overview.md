# Overview

[RAYRAWのファームウェア](https://github.com/spadi-alliance/RAYRAW-FW)は[AMANEQ](https://spadi-alliance.github.io/ug-amaneq/)同様、[Hadron Universal Logic module (HUL)](https://hul-official.gitlab.io/hul-ug/)を元にしている。
RAYRAWファームウェアの基本構造を[下図](#RAYRAW-FIRMWARE)に示す。
四角で囲まれた文字はファームウェアモジュールを、モジュールと矢印で繋がっている囲いのない文字はハードウェアを表す。
ファームウェアの監視・制御 (温度の読み出しや電圧調整など) およびデータ読み出しは[SiTCP](https://www.sitcp.net/doc/SiTCP.pdf)を通じてPCから行われる。
前者はスローコントロールと呼ばれ、各モジュールとの信号の送受信はLocal Busを通して、Local Bus Controller (BCT) により制御される。
データ取得はTrigger Manager (TRM) が発行するトリガーを機に始まり、ADCおよびMulti-Hit TDC (MTDC) で記録された信号がBuilder Busを通じてEvent Builder (EVB) によって集約され、1イベント分のデータが組み立てられる。
トリガーの発行はDAQ Controller (DCT) から送られるDAQ gateの値が1である場合に限り可能で、0になると停止する。
2025年2月時点でファームウェアのシステムクロックは75 MHzであるため、データ取得時のTime WindowのLSB精度は13.3 nsである。
また、TDCのサンプリングに用いられるクロックは300 MHz×4相の実質1.2 GHzであるので、時間測定はLSB精度0.833 nsで行われる。

![RAYRAW-FIRMWARE](rayraw-firmware-fig-v3.png "RAYRAW firmware structure (toplevel.vhd)"){: #RAYRAW-FIRMWARE width="90%"}

## Local Bus Modules
AMANEQ同様、RAYRAWファームウェアではLocal Bus経由でアクセスできるモジュールのことをLocal Bus Moduleと呼ぶ。
これらのうち、BCT、FMP、SDS、TRMについては[HUL User Guide](https://hul-official.gitlab.io/hul-ug/)、C6Cについては[AMANEQ User Guide](https://spadi-alliance.github.io/ug-amaneq/)に説明があるのでここでの説明は省く。
ただし、TRMに関しては、HULと異なりRMおよびJ0 busが存在しないため、NIMINからの外部トリガー (Ext) に関連する箇所のみが意味を持つ。
BCTやFMP等の制御はHULと共通であるため、HULのソフトウェアの基本ライブラリ[hul-common-lib](https://github.com/spadi-alliance/hul-common-lib)により行われる一方、YAENAMI Slow Control (YSC) のようなRAYRAW固有の制御はhul-common-lib+[rayraw-soft](https://github.com/spadi-alliance/rayraw-soft)により行われる ([Software](../../software/software.md)の項目も参照) 。

Local Busは32ビットアドレス空間を持ち、その構造は以下のようになっている：
```
| module ID (4-bits) | local address (12-bits) | reserved (16-bits) |
```
Module IDは各Local Bus Moduleを区別するために用いる。
Local Addressはモジュール内部のアドレス値であり、レジスタや動作を指す。
前者の場合、`Reserved`はレジスタに読み書きするデータを指すが、後者の場合、`Reserved`は意味を持たず指定されたアドレスに信号を送ること自体が動作開始の命令となる。

以下の各節では、Module IDのリストを示した後、RAYRAW固有のLocal Bus Modulesとそのレジスタ・動作を列挙する。
以下の表に出てくるレジスタ名は、実際にファームウェアのコード中で使用されている変数名ではなく、定義ファイル`def*.vhd`中でLocal Addressを格納するために用いられている定数（例えば`kRegName`）から先頭の`k`を除いたもの (`RegName`) である。

### Module ID
Module IDとLocal Bus Moduleとの対応は以下の通り。

|Module ID|Local Bus Module|
|:----:|:----|
|0x0|YAENAMI Slow Control (YSC)|
|0x1|Trigger Manager (TRM)|
|0x2|DAQ Controller (DCT)|
|0x3|TDC|
|0x4|I/O Manager (IOM)|
|0x5|ADC|
|0x9|MAX1932 Controller (APD)|
|0xB|CDCE62002 Controller (C6C)|
|0xC|Self Diagnosis System (SDS)|
|0xD|Flash Memory Programmer (FMP)|
|0xE|Local Bus Controller (BCT)|
|0xF|SiTCP予約領域|


### YAENAMI Slow Control (YSC)
|レジスタ名|Local Address|読み書き|ビット長|機能・備考|
|:----|:----:|:----:|:----:|:----|
|WriteData|0x000|W|8|未使用 (`Reserved`の上位8ビットを使用し常に更新される)|
|BusyFlag|0x200|R|1|SPI通信のビジー信号|
|StartCycle|0x300|W|1|SPI通信の送信サイクル (FPGA→YAENAMI) を開始する|
|ChipSelect|0x400|W|4|4つのASICのどれにデータを送るかを選択する (複数可)|

### DAQ Controller (DCT)
|レジスタ名|Local Address|読み書き|ビット長|機能・備考|
|:----|:----:|:----:|:----:|:----|
|DaqGate|0x000|W/R|1|DAQ gateのON/OFF (TRMのトリガー出力の有効化/無効化)|
|ResetEvb|0x010|W|1|EVB内のEventBuffer (FIFO) のリセット信号|

DaqGateはマニュアルで有効化/無効化することもできるが、[rayraw-soft](https://github.com/spadi-alliance/rayraw-soft)を用いて取得したいイベント数を指定し、ソフトウェアによって自動的に有効化/無効化することを推奨する ([Software](../../software/software.md)の項目も参照)。

### TDC
|レジスタ名|Local Address|読み書き|ビット長|機能・備考|
|:----|:----:|:----:|:----:|:----|
|EnBlock|0x000|W/R|2|TDCBlock (Leading/Trailing) を有効化する|
|OfsPtr|0x010|W/R|11|リングバッファの読み出しポインタのオフセット|
|WinMax|0x020|W/R|11|Time Windowの最大値|
|WinMin|0x030|W/R|11|Time Windowの最小値|

### I/O Manager (IOM)

FPGA内部の信号線をNIMOUTへ割り当てたり、NIMINの信号をFPGA内部の信号線に接続したりする機能を持つ。
HULのものを流用しているが、入出力ポートの数がHULと異なるため若干の違いが存在する。

|レジスタ名|Local Address|読み書き|ビット長|機能・備考|
|:----|:----:|:----:|:----:|:----|
|NimOut1|0x000|W/R|4|NIMOUT1へ割り当てる内部信号線の選択|
|NimOut2|0x010|W/R|4|NIMOUT2へ割り当てる内部信号線の選択|
|ExtL1|0x040|W/R|3|内部信号線extL1へ割り当てる入力ポートの選択|
|ExtL2|0x050|W/R|3|内部信号線extL2へ割り当てる入力ポートの選択|
|ExtClr|0x060|W/R|3|内部信号線extClrへ割り当てる入力ポートの選択|
|ExtBusy|0x070|W/R|3|内部信号線extBusyへ割り当てる入力ポートの選択|

上記`NimOut1/2`の値に応じて以下の信号線がNIMOUT1/2に割り当てられる。

|NimOut1/2の値|内部信号線|備考|
|:----:|:----:|:----|
|0x0|moduleBusy|TRMが発行するModule busyを出力|
|0x1|daqGate|DAQ gateを出力|
|0x2|clk1MHz|1 MHzのクロックを出力|
|0x3|clk100kHz|100 kHzのクロックを出力|
|0x4|clk10kHz|10 kHzのクロックを出力|
|0x5|clk1kHz|1 kHzのクロックを出力|
|0xE|0|Lowを出力|

また、`ExtL1, ExtL2, ExtClr, ExtBusy` (表中`Ext*`とする) の値に応じて以下の入力ポートが対応する内部信号線に割り当てられる。
ただしExtBusyはどこにも繋がっていないため、外からVeto信号を入力したとしてもRAYRAWではTriggerを止めることはできない。そのため、RAYRAWに入力するNIM信号自体にVetoをかけなければならない。

|Ext*の値|NIM入力ポート|
|:----:|:----:|
|0x0|NIMIN1|
|0x1|NIMIN2|
|0x6|0|

IOM内のデフォルトの入出力の割り当ては次の通りである。
ただし、2025年2月時点ではIOMはNIMOUTと接続されておらず、かわりに`toplevel.vhd`内でNIMOUT1とNIMOUT2にそれぞれシステムクロックとModule busyが割り当てられている。

|NIM出力ポート|デフォルト内部信号線|
|:----:|:----:|
|NIMOUT1|moduleBusy|
|NIMOUT2|clk1kHz|

|内部信号線|デフォルトNIM入力ポート|
|:----:|:----:|
|extL1|NIMIN1|
|extL2|0|
|extClr|0|
|extBusy|NIMIN2|

### ADC
|レジスタ名|Local Address|読み書き|ビット長|機能・備考|
|:----|:----:|:----:|:----:|:----|
|OfsPtr|0x000|W/R|11|リングバッファの読み出し位置を示すポインタ|
|WinMax|0x010|W/R|11|Time Windowの最大値|
|WinMin|0x020|W/R|11|Time Windowの最小値|
|AdcRoReset|0x030|W/R|1|ADC読み出し部 (FIFO等) のリセット信号 (デフォルトは1)|
|IsReady|0x040|R|4|各ASICからのADC読み出し準備状況|

`AdcRoReset`について少し補足をしておく。
このready信号は`YaenamiAdc.vhd`内で値の設定が行われており、IDELAYにより適切な遅延がかかりISERDESのbitslipが適切な値になったタイミングで1となる ([TRG-ADC-TDC](../trg-adc-tdc/trg-adc-tdc.md/)のADCの項目参照) 。

### MAX1932 Controller (APD)
|レジスタ名|Local Address|読み書き|ビット長|機能・備考|
|:----:|:----:|:----:|:----:|:----|
|Txd|0x000|W|8|MAX1932への8ビットDAC入力|
|ExecWrite|0x100|W|-|SPI通信(送信)を開始する|

## Data structure
EVBにより生成される1イベントのデータ構造は以下の通りである (データは全て32ビット長) 。
```
Header 1: | 0xFFFF0160 |
Header 2: | 0xFF0 | 0 | overflow (1-bit) | event size (18-bits) |
Header 3: | 0xFF | 0 (enRM) | 000 | 0000 (trigger tag) | event number (16-bits) |
TDCL   1: | 0xCC | 0 | TDC ch (7-bits) | 0 | TDC data (15-bits) |
                                   ...
TDCL  n1: | 0xCC | 0 | TDC ch (7-bits) | 0 | TDC data (15-bits) |
TDCT   1: | 0xCD | 0 | TDC ch (7-bits) | 0 | TDC data (15-bits) |
                                   ...
TDCT  n2: | 0xCD | 0 | TDC ch (7-bits) | 0 | TDC data (15-bits) |
ADC    1: | 0xA | ADC ch (5-bits) | 00 | coarse TDC (11-bits) | ADC (10-bits) |
                                   ...
ADC   n3: | 0xA | ADC ch (5-bits) | 00 | coarse TDC (11-bits) | ADC (10-bits) |
```

まず、イベントの区切りを表すHeader 1が送られる。
次に、イベントに含まれるTDCL (TDC Leading)、TDCT (TDC Trailing)、ADCからのワード数の合計`n1+n2+n3`をイベントサイズとしてHeader 2が送られる。
Header 2内の`overflow`は1イベント中にTDCでカウントできる最大のヒット数を超えた場合に1となるフラグである。
続いて現在のイベント番号がHeader 3として送られた後、TDCL、TDCT、ADCからのデータがこの順に送られる。
15ビットTDCデータの上位11ビットはシステムクロック単位で測られる粗い時間情報 (coarse TDC) であり、ADCはこのcoarse TDCを通じてより細い時間分解能を持つTDCデータと紐付けられる。

なお、上のブロック中の`enRM`および`trigger tag`は、HULにおいてExt/J0/RMのどのトリガーでデータ取得をしたかを識別するために用いられていたが、RAYRAWにおいては常に値0となる。

## RAYRAW上のスイッチ・LEDの機能

### SW1の機能

|スイッチ番号|機能|詳細|
|:----:|:----:|:---|
|1||not used|
|2|SiTCP force default|OFFでSiTCPのデフォルトモードで起動します。電源投入前に設定している必要があります。|
|3|C6C reset|Jitter Cleaner (CDCE62002)のHardware Resetです。|
|4||not used|
|5||not used|
|6||not used|
|7||not used|
|8||not used|

### SW2

全てのモジュールをリセットするためのHardware Reset。

### LED
|番号|備考|
|:----:|:----|
|1|点灯中はTCP接続が張られています。|
|2|RAYRAWがBUSY信号を出しています。|
|3|TDCやADCに用いるCoarse、Fine Clockを作るMMCMがロックしています。|
|4|Jitter Cleaner (CDCE62002)がロックしています。|
