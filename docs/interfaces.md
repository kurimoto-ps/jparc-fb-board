# 母基板および外部基板との接続

以下の J7～J10 は本子基板のコネクタである。母基板 TEP521C の CN1A～CN1D では同じ接点番号に対応する。母基板の外部コネクタには「母 J」を付けた。信号の方向は子基板を基準とする。

| 子基板 | 母基板 | 主な信号と接続先 |
| --- | --- | --- |
| J8（80極） | CN1A | PLC 用 GPIO。GPIO0～15 は母 J4 経由で PLC DI へ、GPIO16～31 は母 J3 経由で PLC DO から入力。汎用入出力、RS-232C、RESET も収容。 |
| J7（80極） | CN1B | DA0～24 は母 J1 からの旧パタン入力、DB0～24 は母 J2 へのデータ出力。75 番は光クロック、77 番は LEMO クロック、79 番はトリガ入力。 |
| J10（100極） | CN1C | DF 系は母 J10 の D/A 基板、DG 系は母 J11 の O/E 基板。DC0～16、DD0～16 は DutyOut1/2。 |
| J9（100極） | CN1D | DE0～19 は DutyOut3。DC17～19、DD17～19 もここを通る。DJ 系は母 J13 から E/O 系へ向かう。DH 系の対応接点は子基板側で未接続。 |

各コネクタの 1～4 番は +3.3 V、5～8 番は未接続。GND は別の G 接点・取付端子に接続する。J8 の 66 番と 68 番の接続は、母基板の RS-232C フロー制御を折り返すための意図した短絡である。DutyOut1/2/3 はそれぞれ 20 本の信号群で、数値の符号・スケーリング・ラッチ条件はこの本数からは決まらない。

## 2 種類の光 ADC 受信

混在構成の MAX 10 は **24 bit × 3 系統**と **16 bit × 8 ch（ADS8568）**を扱う。24 bit 側では外部 ADC がクロックを送り、16 bit 側では MAX 10 がクロックと変換指令を送る。受信データはいずれも O/E 基板 TEP523A → 母 J11 → 子 J10 を通る。[混在ファームウェア仕様 §1～5](https://github.com/kurimoto-ps/digital_power_control/blob/2a581d19a357db6020fd6056bd86994f189acb45/targets/jparc_rev58/docs/hardware/mixed_v2/FIRMWARE_SPEC_v2.md)

### 24 bit　各 ADC からクロック・データを受信

ADC24-1～3 は、それぞれ DRDY、DATA、SCLK の 3 本が FPGA への入力。MAX 10 は系統ごとに DRDY の立ち上がりを検出し、SCLK の立ち下がりごとに MSB から 24 bit を受信する。旧 ACM-205 の配線では同じ形式を最大 6 系統分収容できる。下表の 4～6 行はその旧割り当てで、混在構成で使う 24 bit 受信器は 1～3 のみである。

各セルは信号が通る順に **TEP523A の光受信素子 / 子 J10 の接点 / U5（MAX 10）のボール**を示す。TEP523A の回路図で各系統の出力ラベルは、FS/DRDY が `CHn-1`、DATA/DOUT が `CHn-2`、SCLK が `CHn-3`（`n` は系統番号）。例えば ADC24-1 のデータは IC2 の CH1-2 出力から OE1・O/E 基板 CN1-b1 を経て、子 J10-20、U5-F18 に届く。[TEP523A 回路図](https://github.com/kurimoto-ps/digital_power_control/blob/2a581d19a357db6020fd6056bd86994f189acb45/targets/jparc_rev58/docs/hardware/circuit/6chO_E基板_TEP523A回路図.pdf) · [24 bit 3 系統の光受信素子・接点対応](https://github.com/kurimoto-ps/digital_power_control/blob/2a581d19a357db6020fd6056bd86994f189acb45/targets/jparc_rev58/docs/hardware/rev58_adc_dac/daughter_adc_mapping.json) · [現基板の接続記録](../verification/native_audit.json)

| 系統 | FS/DRDY（CHn-1） | DATA/DOUT（CHn-2） | SCLK（CHn-3） |
| --- | --- | --- | --- |
| ADC24-1 | IC1 / 18 / E19 | IC2 / 20 / F18 | IC3 / 22 / E21 |
| ADC24-2 | IC4 / 26 / G17 | IC5 / 28 / F21 | IC6 / 30 / F22 |
| ADC24-3 | IC7 / 34 / H22 | IC8 / 36 / AB10 | IC9 / 38 / J22 |
| 旧 ADC4 | IC10 / 42 / L19 | IC11 / 44 / V13（混在時は ADDATA16） | IC12 / 46 / L22 |
| 旧 ADC5 | IC13 / 50 / AB14 | IC14 / 52 / N22 | IC15 / 54 / P18 |
| 旧 ADC6 | IC16 / 58 / P21 | IC17 / 60 / R18 | IC18 / 62 / R20 |

### 16 bit　ADS8568 の同時変換 8 ch をシリアル受信

MAX 10 から子 J9 → 母 J13 → E/O 基板 TEP526A を通じて SCLK・FS・CONVSTART を送る。戻りデータには旧 ADC4 の DATA 線を使う。ADDATA16 は物理的には 1 本で、A0、A1、B0、B1、C0、C1、D0、D1 の順に各 16 bit、計 128 bit を受け取る。

| 信号 | MAX 10 から見た方向 | 子基板接点 / U5 ボール | 光変換基板上の割り当て |
| --- | --- | --- | --- |
| SCLK | 出力 | J9-58 / R2 | E/O IC24 |
| FS（Low 有効） | 出力 | J9-60 / R1 | E/O IC25 |
| CONVSTART | 出力 | J9-62 / AA15 | E/O IC26 |
| ADDATA16 | 入力 | J10-44 / V13 | O/E IC11 |

### STM32 への転送と DAC モニタ

ADS8568 の 8 ch 読み出しが完了すると、MAX 10 はその 8 値と、独立に届いた ADC24-1～3 の各最新値を **48 バイトの 1 フレーム**にまとめ、SPI1 で STM32 に渡す。100 µs 周期の設定では毎秒 10,000 フレームとなる。STM32 へは 24 bit 値を全桁転送し、各値の通番、受信後の経過時間、更新フラグも添える。8 ch の同時変換に対し、24 bit の 3 値の取得時刻はそれぞれ独立である。

DAC 自動モニタでは、ADS8568 の 8 値を CH1～8 に、ADC24-1～3 の**上位 16 bit**を順に CH9～11 に割り当てる。[SPI1 フレーム仕様と DAC 割り当て](https://github.com/kurimoto-ps/digital_power_control/blob/2a581d19a357db6020fd6056bd86994f189acb45/targets/jparc_rev58/docs/hardware/mixed_v2/FIRMWARE_SPEC_v2.md) · [実入力で確認したチャンネル](../verification/bench-tests.md)

## DAC 通信　MAX 10 ↔ 子 J10 ↔ 母 J10

TEP522A は AD5764 を 4 個搭載した 16ch D/A 基板。各 IC の SCLK、/SYNC、SDIN は子基板から出力し、**SDO は DAC からの戻り線**として子基板側を入力または Hi-Z にする。旧トップ HDL にある SDO の固定 Low 出力を移植すると、DAC 側の出力と競合し得る。

| D/A 基板上の IC | SCLK 出力 | /SYNC 出力 | SDIN 出力 | SDO 入力 |
| --- | --- | --- | --- | --- |
| DAC1（IC11） | 17 / E22 | 19 / F20 | 21 / G19 | 23 / G20 |
| DAC2（IC13） | 27 / P13 | 29 / H21 | 31 / R13 | 33 / L18 |
| DAC3（IC15） | 37 / M18 | 39 / M20 | 41 / M21 | 43 / M22 |
| DAC4（IC17） | 47 / N20 | 49 / N21 | 51 / P20 | 53 / V14 |

旧送信器は 20 MHz SCLK を用い、IC ごとに 8 bit コマンドと 16 bit データからなる 24 bit フレームを 4 回送る。これは外付け DAC のシリアル通信であり、STM32 と MAX 10 を結ぶ SPI1～4 のフレーム仕様は別に定める。

母 J13 は設備図では CLK 送信系として整理されるが、引継ぎ対象の旧 HDL では状態・学習モニタに使っている。上表の DAC クロックは子 J10 経由である。

## 配線を追う

KiCad の [Interface シート](../hardware/kicad/Interface.kicad_sch)と [MAX 10 シート](../hardware/kicad/max10.kicad_sch)で回路をたどれる。コネクタの接点から調べるには、[ピン・ネット対応 CSV](../hardware/tables/pin-net-mapping.csv) の `Reference` と `Pin` で検索する。信号の用途は母基板 TEP521C、O/E 基板 TEP523A、D/A 基板 TEP522A の回路図と ACM-205 引継ぎ資料に基づく。
