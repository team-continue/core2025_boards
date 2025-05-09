# センサレスブラシレスモータドライバの基板
FETドライブ基板、MCU基板を基板対基板コネクタでつなげることでモータシステムを構成している。
この構成の利点としては、故障の可能性が高いFETドライブ基板を切り離すことで運用コストの低下、高さ方向に基板をつなげることで下方占有面積の削減を目的としている。
特に、ヒートシンクや電源コネクタ等を基板に搭載すると高さ方向にデッドスペースが多く発生するためその有効利用となる。

## FETドライブ基板サイズ
58.5 x 78.4

## 取付穴ピッチ
52.5 x 49 M3

## 対応ヒートシンク
ヒートシンクの取り付けを前提とした設計をしており優れた放熱効果を実現する。
* ALPHA DCC5923Lシリーズ

## MCU基板インターフェース
* CAN通信用コネクタは2種類基板上に搭載しており、用途に応じて柔軟に対応可能としている。特にRJ45コネクタではCAN以外に12V電源ラインを有しておりLANケーブルのみでの通信電源確保を可能としている。<br>
* 過去にMDを設計した際、組み込みで使うIFのみしか用意しなかった結果ファームウェアの開発が非常にめんどくさくなったため、ロボット組み込み時は使わないがデバッグ用のUARTを装備している。<br>
* UART用の5ピンコネクタには、USB-UART変換によくついている5V電源を入力するピンがあり、5V出力を有するUSB-UART変換であればこれ単体でMCUの起動が可能である。<br>
* MCU基板単体でのデバッグ作業が可能なように電源入力用のコネクタを用意している。さらにFETドライブ基板側にも電源出力があり、GNDラインの絶縁を犠牲にすることでMCU側への電源供給が可能<br>
* MCU基板は5V系IC、3.3V系ICの2系が存在し、5V生成にスイッチング電源を利用し、電源入力を高電圧としバスの電流負荷を軽減することを狙っている。<br>

### 通信ライン
CAN,UART<br>
### 電源
PowerSupply 12V

### RJ45 ピンアサイン
POEのピンアサインに似せている
|Pin num|Function|
|---|---|
|1||
|2||
|3|CANL|
|4|Power_Supply|
|5|Power_Supply|
|6|CANH|
|7|GND|
|8|GND|

## MCUぺりふぇらるあさいん
|Pin num|Pin name|peripheral|note|
|----|----|----|----|
|1|RA7|GPIO IN|CAN ID1|
|2|RB14|NONE||
|3|RB15|QEICMP1|Unsupported|
|4|MCLR|||
|5|VSS|||
|6|VDD|||
|7|RA12|NONE||
|8|RA11|NONE||
|9|RA0|AN0|ADC W_CURRENT|
|10|RA1|NONE||
|11|RB0|AN2|ADC U_CURRENT|
|12|RB1|NONE||
|13|RB2|PGC1||
|14|RB3|PGD1||
|15|AVDD|||
|16|AVSS|||
|17|RC0|QEA1|Unsupported|
|18|RC1|INDX1|Unsupported|
|19|RC2|HOME1|Unsupported|
|20|RC11|NONE||
|21|VSS|||
|22|VDD|||
|23|RA8|QEB1|Unsupported|
|24|RB4|C1TX|CAN_TX|
|25|RA4|C1RX|CAN_RX|
|26|VDD|||
|27|RC12|OSCI||
|28|RC15|OSCO||
|29|VSS|||
|30|RD8|GPIO IN|MD_FAULT|
|31|RB5|GPIO OUT|STAT_LED|
|32|RB6|U1RX|UART_RX|
|33|RC10|NONE||
|34|RB7|U1TX|UART_TX|
|35|RC13|GPIO OUT|FF1_LED(bord err "only input")|
|36|RB8|GPIO OUT|FF2_LED(bord err "only input")|
|37|RB9|NONE||
|38|RC6|PWM6H|MCPWM AHI|
|39|RC7|PWM6L|MCPWM ALO|
|40|RC8|PWM5H|MCPWM BHI|
|41|RC9|PWM5L|MCPWM BLO|
|42|VSS|||
|43|VDD|||
|44|RB10|PWM3H|MCPWM CHI|
|45|RB11|PWM3L|MCPWM CLO|
|46|RB12|GPIO IN|CAN ID4|
|47|RB13|GPIO IN|CAN ID3|
|48|RA10|GPIO IN|CAN ID2|

* VSS IS GND,VDD IS POWER SUPPLY(3.3V),"NONE" MUST BE "INTERNAL PULLUP" OR "GPIO OUT"

## ファームウェアについて
### マイコンについて
MCUにはPIC32MK0512MCJ048を採用しており、32BitPICのなかモータ制御向けとして販売されているものである。<br>
特徴として、専用の直行エンコーダインターフェース(QEI)、モータ制御用PWM(MCPWM)、CANFDが搭載されている。<br>
今回は利用していないがQEIはカウンタパルスのほかにカウンタパルス間の測定も可能となっており、高速域、低速域の領域でエンコーダを有効に利用できるペリフェラルである。<br>
さらに、前回のカウンタリード値と現在のカウンタリード値の差分を記憶するレジスタもあり速度計算も非常に用意に実装可能である。<br>
**QEI便利だね**<br>
モータ制御用PWMではPWMのカウンタマッチ割り込みがあり、これをADCの変換フラグに利用することが可能である。<br>
つまり、センターアラインPWMを利用すればPWM波形の一番安定した場所でADCのデータをペリフェラルのみで取得可能であるということだ。<br>
**これは本当に便利**<br>
トリガポイントはさらに柔軟に設定可能なので、割り込みハンドラを処理するときにはすでに変換データが容易されている状態を作り出すことすら可能である。<br>

### 制御について
6パルスIPDを実装し、磁器突極性を問わず初期位置推定を可能としている。<br>
IPDによって、初期位置を検出し任意の進角で動作を開始し、オープンループで十分なBEMFを取得できる速度域まで動作させる。（ここまではできた）<br>
そこから、FOCに遷移する。予定だった。ort

## コメント
いろいろ能書き書いたけどファーム未完成<br>
現代制御なんもわからん
