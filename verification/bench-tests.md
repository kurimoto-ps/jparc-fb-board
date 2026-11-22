# ACM-205 置き換え後の実機試験

既設フィードバック母基板 TEP521C 上の HUMANDATA ACM-205 を本子基板に置き換え、STM32 の起動・メモリ・通信に加え、既設の入力系から DAC アナログ出力までを確認した。下表は試験日・構成ごとの結果である。原記録へのリンクは確認時のコミットに固定している。[測定記録の一覧](https://github.com/kurimoto-ps/digital_power_control/tree/2a581d19a357db6020fd6056bd86994f189acb45/targets/jparc_rev58/docs/measurements)

| 記録 | 対象 | 実機で確認したこと |
| --- | --- | --- |
| 2026-10-25　子基板受入れ | STM32H745、MAX 10 v1.3 | ST-LINK/V3SET による SWD 書き込み・照合、M7 の 400 MHz 起動、SDRAM 8 MiB の試験、Ethernet の PHY 読み出し・リンク・TCP 通信、SPI4 の FPGA レジスタ書き込み・読み戻し。電源再投入後もデバッガなしで起動した。 |
| 同日　ADC ダミーデータ | MAX 10 → SPI1 → STM32 DMA | コールドブート後の 1 秒試験は 234,794 レコードを受信。FPGA の送信数と一致し、CRC・通番等のエラーは 0。試験値は FPGA が生成した。 |
| 2026-11-16～17　16 bit 実入力 | ADS8568、8 ch | CH1～8 にノコギリ波を 1 ch ずつ入力し、対応する DAC CH1～8（試験時の BNC13～20）で各波形を確認した。 |
| 同日程　24 bit 実入力 | ADC24-1/2（DCCT P/N） | 300 A 相当の一定入力を与え、各値の上位 16 bit を出す DAC CH9/10（BNC21/22）でそれぞれ約 −3.3 V／+3.3 V を観測した。ADC24-3 → DAC CH11 は設計上の割り当てで、今回の実入力波形試験の対象外。 |

10 月の対象基板は STM32 UID で識別されている。11 月の受領データには UID と書き込みイメージのハッシュがないため、2 回の試験を同一個体・同一ファームウェアでの測定として扱うことはできない。[10 月の受入れ試験・デバッグログへの入口](https://github.com/kurimoto-ps/digital_power_control/blob/2a581d19a357db6020fd6056bd86994f189acb45/targets/jparc_rev58/docs/measurements/2026-10-25-board-00250028-35335115-39383331/README.md)

## 11 月の波形と受信ログ

FPGA 内の ADC→DAC 連動試験では、更新周期を 100 µs に設定し、`phase=0`（0°）と `phase=2`（90°）の両方で 8 ch の対応を確認した。両条件とも `delay=9`、`conversion=68`、`negate=1`。この 2 条件は良好な波形を得た設定で、最終的な既定値は未選定である。[条件別のチャンネル対応・波形の原記録](https://github.com/kurimoto-ps/digital_power_control/blob/2a581d19a357db6020fd6056bd86994f189acb45/targets/jparc_rev58/docs/measurements/2026-11-17-mixed-adc/README.md)

FPGA→SPI1→STM32 DMA→Ethernet の取得経路は、波形撮影に先立つ遅延探索ログで確認した。**218 件、計 2,182,406 フレーム**で、全件が `pass`、CRC・通番・DMA エラー等は 0。これらはすべて `negate=0`、ADC16 の期待値指定なしで取得している。アナログ波形と同時に取得したログではなく、アナログ精度の判定にも用いていない。[受信ログ集計 CSV](https://github.com/kurimoto-ps/digital_power_control/blob/2a581d19a357db6020fd6056bd86994f189acb45/targets/jparc_rev58/docs/measurements/2026-11-17-mixed-adc/capture_summary.csv)

この記録で実証したのは、置き換えた子基板で実入力を取得し、対応する DAC に出力できることまでである。DCCT のゲイン・極性の校正、100 µs 周期ごとの遅延・ジッタ、電源の閉ループ電流追従および長時間安定性は、ここでの合格項目に含まれない。
