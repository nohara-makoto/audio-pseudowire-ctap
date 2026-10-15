# APW-CTAP：4極オーディオ経路によるFIDO2通信の検証案

作成日：2026-10-15（JST）  
状態：検証計画。以下の組合せでの動作は未検証。

## 1. 目的

PCとAndroidの間をアナログオーディオケーブルで接続し、音声モデムでCTAP2の要求・応答を運ぶ。AndroidをFIDO2認証器として動かし、登録・認証まで成立させる。

最初は音楽なしで通信を確認し、その後、音楽の高域を通信時だけ減衰させ、空けた帯域にモデム波形を重ねる。将来は、モデム部分の安価なMCUへの移植と、リモートデスクトップの音声経路での動作を検討する。

1,200bps程度でも、普段の認証が数秒程度で終わるなら実用候補とする。登録時の証明書などによる長い転送は、日常認証と分けて評価する。

## 2. 構成と役割

| 位置 | 使用ソフトウェア／部品 | 役割 |
|---|---|---|
| PC | libfido2 | CTAP2のGetInfo、MakeCredential、GetAssertion等を発行 |
| PC | 自作APW-CTAPテストクライアント | libfido2と音声通信を接続、計測・ログ・進捗表示 |
| PC | libquiet（Quiet） | バイト列の変調、PCM波形の復調 |
| PC | PortAudio、またはOSの音声API | オーディオ機器への録音・再生 |
| Android | 自作APW-CTAP認証器アプリ | 音声送受信、CTAP処理、承認画面、計測 |
| Android | rauth-android | 認証器処理、鍵管理、ユーザー確認 |
| Android | org.quietmodem.Quiet、またはlibquiet＋JNI/NDK | QuietのAndroid側組込み |
| 接続 | USB-Cの4極ヘッドセット対応DAC/ADCアダプター | Androidのステレオ出力とマイク入力 |
| 接続 | PC側の録音・再生対応オーディオ機器 | 対向するアナログ入出力 |
| 接続 | 分岐・交差配線、レベル調整、DCカット回路 | 出力から相手の入力への接続 |

これらは完成済みアプリをインストールするだけで連携する構成ではない。PC側テストクライアントとAndroid側アプリ、および両者の接続処理を作る。

### 主要リポジトリ

- libfido2：https://github.com/Yubico/libfido2
- Quiet：https://github.com/quiet/quiet
- Quiet Android：https://github.com/quiet/org.quietmodem.Quiet
- rauth-android：https://github.com/WIOsense/rauth-android
- quiet-lwip（TCP/UDP方式を選ぶ場合）：https://github.com/quiet/quiet-lwip

各リポジトリは使用時のコミットSHAを記録する。既存Androidライブラリの説明は古いビルド環境を含むため、現在のAndroid SDK/NDK、端末、権限、ネイティブライブラリ要件への適合を最初に確認する。

## 3. 論理経路と配線

PCでは要求を音声へ変換して出力する。AndroidではUSBアダプターのマイク入力から受信し、認証器処理へ渡す。応答はAndroidのイヤホン出力からPCの録音入力へ返す。

| 方向 | 送信端 | 受信端 |
|---|---|---|
| PC→認証器 | PCの音声出力 | AndroidアダプターのMIC入力 |
| 認証器→PC | AndroidアダプターのLまたはR出力 | PCの録音入力 |
| 基準 | GND | GND |

CTIAの4極配列は、Tip=L、Ring1=R、Ring2=GND、Sleeve=MIC。両端が通常のヘッドセット用端子なら、ストレートの4極ケーブルをそのまま使う構成ではない。送信出力を相手のMIC入力へ交差させる必要がある。

出力同士を接続しない。ヘッドホン出力とマイク入力の信号レベル、マイクバイアス、入力インピーダンス、マイク接続検出を考慮して回路を決める。具体的な抵抗値等は使用機器を決定してから確定する。

最初の試験ではL/Rの片側で十分。音楽のステレオ配信や実際のヘッドセット併用の配線は、基本通信の成立後に扱う。

## 4. Quietの使い方

### 4.1 フレーム送受信を基本とする

Quietはencoderとdecoderを独立に持つ。

| 方向 | API | 処理 |
|---|---|---|
| 送信 | quiet_encoder_send() | データフレームを送信キューへ投入 |
| 送信 | quiet_encoder_emit() | 送信用PCMを取得 |
| 受信 | quiet_decoder_consume() | 録音PCMを復調処理へ投入 |
| 受信 | quiet_decoder_recv() | 復号されたフレームを取得 |

両端でencoderとdecoderを動かす。物理的には送受信が別の線であり、機器・OSが対応すれば同時送受信可能。ただし、最初のCTAP試験は要求・応答の交互通信でよい。

Quiet本体のフレーム送受信には、ACK・再送を含む信頼できる接続を自動で提供する前提を置かない。CRC、誤り訂正、同期等はプロファイルとビルドに依存する。

### 4.2 Androidの既存ラッパー

org.quietmodem.QuietはFrameTransmitter / FrameReceiverによるフレーム送受信と、quiet-lwipによる独立したTCP/UDPスタックを提供する。

最初はフレーム方式を選ぶ。TCP方式は分割・再送を既存スタックへ任せられる代替案だが、IP/TCPのヘッダー、接続確立、ACKなどの通信量と遅延が増える。Auto IP利用時のアドレス確定待ちも計測対象になる。

AndroidラッパーでUSB入出力の選択や音楽ミックスを十分に制御できない場合は、libquietのPCM APIをJNI経由で使い、AudioRecord / AudioTrack等で音声経路を管理する。

### 4.3 プロファイル

| プロファイル | 設定上の中心周波数 | 方式 | 2019年研究の速度評価 |
|---|---:|---|---:|
| audible | 4.2kHz | GMSK | 約1.30kbps |
| ultrasonic-whisper | 19.5kHz | GMSK | 約0.35kbps |
| ultrasonic | 19kHz | GMSK | 約1.04kbps |
| ultrasonic-3600 | 18.5kHz | OFDM | 約7.23kbps |
| cable-64k | 10.2kHz | OFDM／1024-QAM | 最大約118.05kbps |

周波数・変調は確認したquiet-profiles.json、速度はB-DTN7論文に基づく。異なる版の設定と評価である可能性があるため、そのまま同じ数値を再現すると仮定しない。

論文の速度評価は生成した音声ファイルの時間から求めたもので、実機の損失・再送を含む保証スループットではない。プロファイル名の3600や64kを厳密な速度として扱わない。

中心周波数と占有帯域は別物。特にultrasonic-3600が18.5～20kHzの1.5kHz幅に収まるとは仮定しない。FFT等で生成信号の帯域を測り、音楽側に空ける帯域を決める。

GMSKはFFT不要の受信方式としてQuietで説明されており、将来のMCU化の候補。ただし同期・誤り訂正を含めた必要RAMと演算量は未評価。

## 5. CTAPの接続方式

### 5.1 初期案：HIDレポートを運ぶ

PCではfido_dev_set_io_functions()でopen/close/read/writeを差し替える。libfido2が作るHIDレポートをAPW側で運び、AndroidのTransactionManager.handleReport()へ渡す。返された応答レポートを逆方向に返す。

libfido2の書込み側のレポートIDの扱い、Android側が期待するレポート長・形式は実装で合わせる。音声をUSB HIDとして物理接続するのではなく、既存のCTAPHIDフレーム処理を再利用する方式。

#### Quietフレームへの分割

確認した既定設定ではultrasonicのframe_lengthは34B、ultrasonic-whisperは16B。64BのHIDレポートはそのまま1フレームへ入らない。

APWヘッダー分を引いたサイズで分割し、受信側でレポート全体を再構成してからTransactionManagerへ渡す。単純にframe_lengthを64以上に変更する場合も、遅延・誤り率・同期安定性を再評価する。

レポートの途中まで届いただけで相手へ渡さない。再構成待ちの間にCTAPHIDの継続パケット待ちタイムアウトが発生しないよう、バッファリングとタイムアウトを確認する。

### 5.2 最適化案：コマンドとデータをまとめて運ぶ

fido_dev_set_transport_functions()を使えば、CTAPHIDコマンドとそのデータを直接受け渡す接続を実装できる。HID固定長レポートのパディングと分割ヘッダーを削減できる。

ただしCBORだけを送れば自動で接続できるわけではない。INIT、CBOR、CANCEL、KEEPALIVE、ERROR等の対応を定義する。Android側もTransactionManagerのHID処理を分離するか、Authenticatorへのディスパッチ層を実装する。

初期の動作確認と転送量最適化は分けて行う。

### 5.3 ブラウザー接続

最初の成功条件は、自作テストクライアントからlibfido2を使って登録・署名・検証を行えること。

通常のブラウザーが、自作プロセスで設定したlibfido2のコールバックを自動利用することはない。ブラウザーとの統合には、仮想HID、OS側連携等の追加設計が必要で、基本PoCとは別段階にする。

## 6. APWフレームと再送

最初は1要求ずつ処理し、並列取引を避ける。最低限、以下を扱う。

| 項目 | 目的 |
|---|---|
| メッセージ種別 | データ、ACK、キャンセル、処理中、エラー等の識別 |
| 取引ID | 要求と応答、古い再送の区別 |
| 分割番号／全長 | 分割・再構成 |
| データ | HIDレポートまたはCTAPコマンドと内容 |

ヘッダーのビット配置は実装時に確定する。1フレームごとのACKは往復同期の負担が大きい可能性があるため、まとめてACKする方式との比較も行う。

再送を受けても同じ認証器コマンドを二重実行しないよう、取引IDと完了応答の短期保持を使う。CRCエラー、欠落、重複、切断、キャンセルを明示的に扱う。

KEEPALIVEは通常USBで使う送信周期をそのまま音声に載せると負担になる。認証器の待機状態を保持しつつ、必要ならPC側で待機通知を生成する設計を検討する。独自経路でもキャンセルや実際の応答期限は守る。

## 7. データ量と時間の目安

### 7.1 普段の認証

RP ID=example.com（11B）、credential ID=64Bを1件、ES256署名=71B、応答にもcredentialを含み、PIN関連情報・ユーザー情報・拡張なしを仮定したCBORの計算例：

| 内容 | CTAPコマンド／ステータスを含む量 | 64B HIDレポートで運ぶ量 |
|---|---:|---:|
| GetAssertion要求 | 約138B | 192B |
| GetAssertion応答 | 約203B | 256B |
| 合計 | 約341B | 448B |

これは実測ではない。credentialの省略条件、ID長、候補件数、署名方式、拡張、ユーザー情報で増減する。GetInfo、INIT、PIN/UV手順、処理中通知、APWヘッダー、音声フレーム同期、再送は別途。

実効1,200bpsを仮定した純転送時間は341Bで約2.3秒、448Bで約3.0秒。実効1,040bpsなら約2.6秒、約3.4秒。いずれも承認待ちと追加処理は含まない。

### 7.2 登録・PIN等

| 処理 | 転送量の見通し |
|---|---|
| GetInfo | 要求1B、応答は実装・対応機能で変動 |
| MakeCredential要求 | 短い名前・少数のアルゴリズム・除外リストなしなら数百B程度を想定 |
| MakeCredential応答 | credential ID、公開鍵、attestation方式で変動。証明書なしなら数百B、証明書チェーン付きでは数KiBも想定 |
| PIN/UVトークン取得 | 鍵交換等の追加往復があり、数百B規模以上の追加になり得る |
| 大きいallowList／excludeList | credential IDの件数に応じ増加 |

attestation証明書は基本的に登録時の情報であり、通常ログインで毎回送るものではない。登録時も証明書を含まない方式がある。

初期検証では拡張機能やlargeBlob転送を加えず、標準的な登録・認証を先に確認する。正式な認証試験ではUP/UVの要求と結果フラグも検証する。

## 8. 段階的な検証手順

### 段階0：ビルドと認証器単体

- Androidのバージョン、端末、USBアダプター、PCのオーディオ機器を記録。
- QuietとAndroidラッパーのビルド・起動・権限・録音再生を確認。
- rauth-androidでGetInfo、鍵生成・署名、ユーザー承認の基本処理を確認。
- StrongBox必須設定は端末対応を確認して選ぶ。未対応端末で必須にして失敗させない。

### 段階1：音声ファイル内の往復

- 既知のバイト列をQuietでPCM化し、同じPCMをdecoderへ渡す。
- フレーム長、分割、再構成、内容一致を確認。
- 生成PCMの長さから、実効ペイロード速度とフレーム起動時間を求める。
- 無信号時、途中で切れたフレーム、CRC不一致も確認。

### 段階2：有線の双方向通信

- 最初はaudible等で配線とレベルを確認し、次にultrasonicへ切り替える。
- PC→Android、Android→PCを別々に測定。
- 16～22kHz付近のテスト信号等で周波数応答を確認。USBマイク入力側の高域が残るかを特に確認。
- 録音・再生の同時動作、実サンプルレート、入力先・出力先を確認。
- ノイズ抑制・AGC・エコー除去等を可能な範囲で無効化し、実際の経路を記録。
- 16/64/256/1024Bの既知データで往復時間、誤り、欠落、再送を測定。

### 段階3：CTAP GetInfo

- libfido2のI/OコールバックをAPWへ接続。
- HIDレポートの分割・再構成と、INITおよびGetInfoの往復を確認。
- Androidが返す対応バージョン・機能をPCで表示。
- 接続初期化と再接続の時間を、認証自体の時間と分けて記録。

### 段階4：登録・認証

- テスト用RP IDとchallengeでMakeCredentialを実行。
- 登録された公開鍵を使い、GetAssertionの署名をPCで検証。
- challenge、RP ID、UP/UVフラグ、カウンター等を適切に確認。
- タッチ／指紋等の承認、拒否、キャンセル、タイムアウトを確認。
- 証明書なし／証明書あり、PIN/UV追加往復を別ケースで計測。
- 取引の再送で二重実行しないことを確認。

### 段階5：音楽との共存

- 実測したモデムの占有帯域に合わせ、送信元の音楽をフィルター処理。
- 通信前に滑らかに減衰させ、帯域が空いてから同期・データを送信。
- 最後の送信波形が出終わってから音楽の高域を戻す。キュー投入完了を送信完了と扱わない。
- 金属打楽器、高域の多い曲、静かな曲、曲間などで誤り率と聴感を比較。
- 通信あり／なしをランダム化して聞き分け試験を行う。

開始案は音楽を18.5kHzまでに十分減衰させ、18.5～20kHzをデータ用にする構成。ただし既定プロファイルが収まらなければフィルターと帯域を変更する。

低感度な高域の利用と聴覚マスキングは、同じ効果ではない。高域の音楽を除いた条件での不可聴性は未確認。音量を上げればよいという設計にはせず、復調可能レベルと聴取検出レベルを別々に測る。

### 段階6：高速化・リモート音声経路

- ultrasonic-3600で認証時間を測り、占有帯域・再送率・必要CPUを確認。
- CTAPコマンド単位の転送にしてHIDの固定長パディングを削減。
- リモートデスクトップの音声出力とマイク転送を経由させる。
- 高域が削られる経路ではaudibleへの切替を試す。
- PCMでは通るが圧縮で通らない場合、コーデック、ビットレート、録音モード、音声処理を記録。

audibleならすべての音声コーデックで通るとは限らない。会話向け圧縮、雑音抑制、エコー除去等への耐性は別途の実測課題。

### 段階7：MCU化

- Androidで記録した正解PCMとパケットを、MCU復調器の基準入力にする。
- ADC帯域、サンプルレート、RAM、CPU、固定小数点化、同期、FECを評価。
- MCUに認証器機能まで載せる案と、音声モデム／別認証器への橋渡しだけを載せる案を分ける。
- 100円程度のMCUで成立するかは、型番決定と実測後に判断する。

## 9. 計測と表示

### 必須ログ

| 指標 | 記録内容 |
|---|---|
| ソフトウェア | 各コミットSHA、ビルド条件、Quietプロファイル |
| 音声経路 | 機器名、選択デバイス、録音再生サンプルレート、バッファ量 |
| データ量 | CTAP本体、HID化後、APW化後、再送分を分けて記録 |
| 時間 | 初期化、要求転送、承認待ち、応答転送、総所要時間 |
| 信頼性 | CRC失敗、欠落、再送、成功率、重複抑止 |
| 聴感 | 曲、再生レベル、通信信号レベル、聞き分け結果 |

開発ログは原則としてコマンド種別・長さ・時刻・結果コードを記録する。PIN、認可トークン、秘密鍵等をログへ出さない。

### ユーザー表示

- 「認証器へ送信中」：受信確認済みバイト数を基準に進捗を表示。
- 「スマートフォンで承認してください」：転送進捗と分けた待機表示。
- 「認証応答を受信中」：全長が分かれば進捗、分からなければ受信量を表示。
- 「再送中」「キャンセル」「切断」を区別。

送信キューへ積んだ量を、相手の受信完了量として表示しない。

## 10. 初期の合格条件案

以下は検証開始時の目標であり、保証仕様ではない。

1. GetInfoの往復が成立する。
2. 音楽なしの有線接続で、登録→認証→署名検証を完了する。
3. 同じ条件の認証を100回試行し、成功率、中央値、95パーセンタイルを記録する。
4. 通常認証の総時間について、承認待ちを除く5秒程度を初期目標にする。
5. 通信時の音楽フィルター処理で、認証成功率が維持される。
6. audibleを使った別経路について、少なくともパケット往復を確認する。

最初の成果物は、PCのテストクライアント、Androidの試験アプリ、回路／配線図、プロファイル、計測ログ。一般ブラウザー対応とMCU完成は次段階の成果物とする。

## 11. 先行例と根拠

### ライブラリの接続箇所

- libfido2独自I/O／transport API：  
  https://github.com/Yubico/libfido2/blob/main/man/fido_dev_set_io_functions.3
- Quiet公開API：  
  https://github.com/quiet/quiet/blob/master/include/quiet.h
- Quietプロファイル：  
  https://github.com/quiet/quiet/blob/master/quiet-profiles.json
- Quiet Androidのフレーム／TCP/UDP API：  
  https://github.com/quiet/org.quietmodem.Quiet/blob/master/README.md
- rauth-androidのAuthenticator／TransactionManager：  
  https://github.com/WIOsense/rauth-android/blob/master/README.md

### 仕様・評価・関連技術

- CTAP 2.1仕様（認証器コマンド、PIN/UV、CTAPHID）：  
  https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html
- WebAuthn Level 2（authenticatorData、公開鍵、署名等）：  
  https://www.w3.org/TR/webauthn-2/
- Android USB-Cオーディオアダプター仕様：  
  https://source.android.com/docs/core/interaction/accessories/headset/usb-adapter
- B-DTN7（2019年、Quietプロファイル別の生成音声長による速度評価、図12付近）：  
  https://dtn7.github.io/assets/baumgaertner2019bdtn7.pdf
- Quietの構成とプロファイル調整：  
  https://quiet.github.io/docs/quiet/configuration/  
  https://quiet.github.io/quiet-blog/2016/03/30/quiet-profile-lab-build-modem-learn-dsp.html
- US5612943A／WO1998008218A1：音楽の18kHz以上を減衰させ、18.5/19.5kHzのFSK等で制御データを重畳する先行記載。必要時だけの動的切替を同じ形で実装しているとまでは確認していない：  
  https://patents.google.com/patent/US5612943A/en  
  https://patents.google.com/patent/WO1998008218A1/en

この計画書は会話で検討した構成をまとめたものであり、既存のAPW-CTAP仕様書を読んで改訂したものではない。フレーム形式や既存仕様との整合は実装開始時に確認する。
