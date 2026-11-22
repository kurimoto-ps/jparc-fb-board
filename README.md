# J-PARC MR 主電磁石電源フィードバック制御子基板

J-PARC メインリング（MR）の主電磁石電源で、電流パタンに沿って電磁石電流を制御するための子基板です。既設のフィードバック母基板 **TEP521C** を使い、その上の **HUMANDATA ACM-205** を、STM32H745 と MAX 10 を搭載した本基板に置き換えます。

既設の光受信、D/A 出力、PLC、変換器盤への Duty 指令は母基板のコネクタを通ります。新たに Ethernet と SDRAM を子基板に持たせ、これまで独立したパタンメモリ基板が担っていた電流指令の受信・保持を取り込むことが設計の狙いです。

試作機では ACM-205 を本子基板に交換し、STM32 の起動、SDRAM、Ethernet、FPGA との通信、実 ADC 入力 8 ch と対応する DAC 出力の波形まで確認しました。現行基板では J4 の書き込みケーブルと J5 の RJ45 ジャックに個別の対処を行っています。[実機試験の結果](verification/bench-tests.md) · [既知不具合 3 件と次回改版](docs/errata.md)

```mermaid
flowchart TD
    Host["パタン作成システム"] -->|Ethernet| MCU
    subgraph Board["本基板"]
        MCU["STM32H745"] <-->|FMC 16 bit| RAM["SDRAM 8 MiB"]
        MCU <-->|SPI 4系統| FPGA["MAX 10"]
    end
    Mother["既設フィードバック母基板 TEP521C"]
    MCU <-->|PLC用GPIOなど| Mother
    Mother -->|受信データ・外部同期| FPGA
    FPGA -->|DAC制御・Duty指令など| Mother
    OE["6ch O/E基板"] -->|帰還信号| Mother
    Mother --> DAC["16ch D/A基板"]
    Mother --> Power["変換器盤への指令"]
```

## 設計を読む

| 知りたいこと | 資料 |
| --- | --- |
| 回路全体と部品の接続 | [回路図 PDF 全6ページ](docs/drawings/schematic.pdf) |
| 基板の配線と両面の部品配置 | [配線・配置図 PDF 全10ページ](docs/drawings/pcb-patterns.pdf) |
| 回路図・PCB の編集 | [KiCad プロジェクト](hardware/kicad/JPARC_PS_FB_BOARD.kicad_pro) |
| プロセッサの役割、電源、メモリ、クロック | [ハードウェア構成](docs/hardware.md) |
| 母基板との対応、ADC/DAC の接点と FPGA ボール | [外部インターフェース](docs/interfaces.md) |
| ランド単位のネット名、部品座標 | [ピン・ネット対応 CSV](hardware/tables/pin-net-mapping.csv) · [部品配置 CSV](hardware/tables/component-locations.csv) |
| 基板製造・部品実装に実際に渡したデータ | [PCBWay の裸基板 Gerber・ドリル、P板.com の実装指示書・部品表・マウントデータ・ステンシル](manufacturing/README.md) |
| 接続・DRC の検査結果と原本の識別 | [設計検証記録](verification/README.md) |

このリポジトリの対象は子基板のハードウェアです。MCU の制御ソフトウェア、MAX 10 の動作用 HDL、閉ループでの追従性能・遅延・ジッタの実測値は収録していません。図は設計上の信号経路を示します。
