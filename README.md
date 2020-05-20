# openvpn-install

このドキュメントは、openvpn-installのフォーク元であるangristan氏の[README.md](https://github.com/angristan/openvpn-install/blob/master/README.md) を翻訳したもので、一部内容は本プロジェクトに合せて改訂されています。翻訳内容の保証はしておりませんのであらかじめご了承ください。

------

openvpn-installは、Debian、Ubuntu、Fedora、CentOS及びArch Linuxで動作するOpenVPNインストーラです。本スクリプトを使用することにより、手軽にセキュアなVPNサーバをセットアップすることができます。

## 使い方

初めにスクリプトをダウンロードし、実行可能にします。

```bash
curl -O https://raw.githubusercontent.com/plum-systems/openvpn-install/master/openvpn-install.sh
chmod +x openvpn-install.sh
```

次にスクリプトを実行します。

```sh
./openvpn-install.sh
```

スクリプトは、rootユーザで実行する必要があります。また、TUNモジュールが有効になっている必要があります。

初めて実行するときは、ウィザードに従っていくつかの質問に答え、VPNサーバをセットアップします。すでにOpenVPNがセットアップされている場合、スクリプトを再び実行すると、次の選択肢を選ぶことができます。

- クライアントの追加
- クライアントの削除
- OpenVPNのアンインストール

ホームディレクトリには、クライアント設定ファイル（`.ovpn`）があります。 これらを サーバからダウンロードし、OpenVPNクライアントを使用して接続します。



質問がある場合は、まず[FAQ](#faq)に進んでください。 [Issues](https://github.com/issues) に登録する前にすべてをお読みください。

**メールでの個別の問い合わせにはお答えしておりませんのでご了承ください。** 質問がある場合は [Issues](https://github.com/issues) をご利用ください。他のユーザが答えてくれるかもしれません。 [Issues](https://github.com/issues) を利用するなら、将来、他のユーザも同じ問題に遭遇する際に役立つ場合もあります。

### ヘッドレスインストール

スクリプトをヘッドレスで実行することで、ウィザードによるユーザ入力を待つことなく、自動的にセットアップすることもできます。

使用例:

```bash
AUTO_INSTALL=y ./openvpn-install.sh

# or

export AUTO_INSTALL=y
./openvpn-install.sh
```

ヘッドレスインストールでは、ユーザ入力の代わりに、デフォルトで設定された変数がパラメータとして渡されます。

変数をカスタマイズしたい場合は、上記のように同じ行に指定するか、変数を `export` することができます。

- `APPROVE_INSTALL=y`
- `APPROVE_IP=y`
- `IPV6_SUPPORT=n`
- `PORT_CHOICE=1`
- `PROTOCOL_CHOICE=1`
- `DNS=1`
- `COMPRESSION_ENABLED=n`
- `CUSTOMIZE_ENC=n`
- `CLIENT=clientname`
- `PASS=1`

サーバがNATの背後にある場合は、 `ENDPOINT` 変数を使用してエンドポイントを指定できます。エンドポイントがグルーバルIPアドレスなら、 `ENDPOINT=$(curl -4 ifconfig.co)` を使用できます（スクリプトのデフォルト）。また、エンドポイントは、IPv4またはドメインで指定することができます。

その他の変数は、選択に応じて設定できます（暗号化、圧縮）。変数はスクリプトの `installQuestions()` 関数で検索できます。

本スクリプトではEasy RSAを使用していますが、Easy RSAはユーザ入力を想定しているため、ヘッドレスインストールではパスワード付きのクライアント証明書の発行はサポートされません。

ヘッドレスインストールは、Ansible / Terraform / Salt / Chef / Puppetなどの構成管理ツールを使用する場合を想定しており、同じパラメータで何回実行しても問題ありません。Easy RSAによるPKIが存在していない場合のみPKIが再構築されます。また、OpenVPNがインストールされていない場合のみ、OpenVPNおよび依存関係にあるパッケージをインストールします。ヘッドレスインストールを実行する度に、すべての設定ファイルは再作成され、クライアント設定ファイルも再生成されます。

### ヘッドレスインストールでのユーザの追加

新しいユーザーの追加を自動化することもできます。 ここで重要なのは、スクリプトを呼び出す前に、 `MENU_OPTION` 変数（文字列）、およびその他の必須変数を設定することです。

次のBashスクリプトは、OpenVPNの新しいユーザ `foo` を追加します。

```bash
#!/bin/bash
export MENU_OPTION="1"
export CLIENT="foo"
export PASS="1"
./openvpn-install.sh
```

## 機能

- 手軽なOpenVPNサーバのインストールと設定
- iptablesの自動設定
- OpenVPNのアンインストールと設定の削除
- 暗号化設定のカスタマイズ、及び強化されたデフォルト設定（以下の [セキュリティと暗号化](#セキュリティと暗号化) を参照）
- OpenVPN 2.4に対応（主に暗号化の改善；以下の [セキュリティと暗号化](#セキュリティと暗号化) を参照）
- 様々な種類のDNSサービスをクライアントにプッシュ可能
- ローカルのUnboundを利用可能
- TCP/UDPの選択
- IPv6サポート
- VORACLE攻撃に対応するためデフォルトで圧縮を無効化
  ※圧縮アルゴリズムはLZ4（v1/v2）及びLZOに対応
- 非特権モードでの実行（ `nobody` / ` nogroup` ）
- Windows 10のDNSリークをブロック
- サーバ証明書名のランダム化
- パスワード付きクライアント証明書（秘密鍵暗号化）
- その他

## 互換性

次のOSとアーキテクチャをサポートしています。

|                 | i386 | amd64 | armhf | arm64 |
| --------------- | ---- | ----- | ----- | ----- |
| Amazon Linux 2  | ❔   | ✅    | ❔    | ❔    |
| Arch Linux      | ❔   | ✅    | ❔    | ✅    |
| CentOS 7        | ❔   | ✅    | ❌    | ✅    |
| CentOS 8        | ❌   | ✅    | ❔    | ❔    |
| Debian 8        | ✅   | ✅    | ❌    | ❌    |
| Debian >= 9     | ❌   | ✅    | ✅    | ✅    |
| Fedora >= 27    | ❔   | ✅    | ❔    | ❔    |
| Ubuntu 16.04    | ✅   | ✅    | ❌    | ❌    |
| Ubuntu >= 18.04 | ❌   | ✅    | ✅    | ✅    |

注記:

- Debian 8以降とUbuntu 16.04以降で動作します。 上記の表にないバージョンは正式にサポートされていません。
- 本スクリプトの動作には  `systemd` が必要です。
- 本スクリプトは `amd64` のみでテストされています。

## フォーク

本スクリプトは [Nyr氏を始めとする多くの貢献者](https://github.com/Nyr/openvpn-install) の成果をベースにしています。

2016年に本スクリプトはフォークしましたが、内部は大幅に変わっています。 当初の目的はセキュリティの強化でしたが、それ以来、スクリプトは完全に書き直され、多くの機能が追加されました。 本スクリプトは最近のディストリビューションのみと互換性があるため、非常に古いサーバまたはクライアントを使用する必要がある場合は、Nyr氏のスクリプトを使用することをお勧めします。

## FAQ

詳しいQ&Aは [FAQ.md](FAQ.md) を参照してください。

**Q:** どのVPNプロバイダーを推奨しますか？

**A:** 以下を推奨します。

- [Vultr](https://goo.gl/Xyd1Sc): 世界中のロケーション、IPv6サポート、$3.50/月から
- [PulseHeberg](https://goo.gl/76yqW5): フランス、無制限の帯域幅、€3/月から
- [Digital Ocean](https://goo.gl/qXrNLK): 世界中のロケーション、IPv6サポート、$5/月から

---

**Q:** どのOpenVPNクライアントを推奨しますか？

**A:** 可能なら、公式のOpenVPN 2.4 クライアントを使用してください。

- Windows: [OpenVPN GUI（公式）](https://www.openvpn.jp/download/)、 [vpnux Client](http://www.plum-systems.co.jp/vpnux-client/)
- Linux: `openvpn` パッケージ
  ※Debian/Ubuntuベースのディストリビューション用の [公式APTリポジトリ](https://community.openvpn.net/openvpn/wiki/OpenvpnSoftwareRepos) もあります。
- macOS: [Tunnelblick](https://tunnelblick.net/)、[Viscosity](https://www.sparklabs.com/viscosity/)
- Android: [OpenVPN Coonect（公式）](https://play.google.com/store/apps/details?id=net.openvpn.openvpn)
  - iOS: [OpenVPN Connect（公式）](https://itunes.apple.com/us/app/openvpn-connect/id590379981)

---

**Q:** このスクリプトはNSAから守ることができますか？

**A:** OpenVPNはセキュリティの強化と最先端の暗号への対応を常に考慮していますが、NSAから通信を完全に隠したいならVPNを使用するべきではありません。リスク分析した上でVPN利用の検討を行ってください。

---

**Q:** OpenVPNのドキュメントはありますか？

**A:** はい。[OpenVPNマニュアル](https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage) にすべてのオプションについての説明が記載されています。

---

詳しいQ&Aは [FAQ.md](FAQ.md) を参照してください。

## AWS向けワンストップ・ソリューション

AWSを利用している場合、Terraformを利用した次のスクリプトにより、OpenVPNサーバを一度にプロビジョニングすることができます。

- [`openvpn-terraform-install`](https://github.com/dumrauf/openvpn-terraform-install)

## コードフォーマット

Bashのコーディング規則に合わせるために、 [shellcheck](https://github.com/koalaman/shellcheck) と [shfmt](https://github.com/mvdan/sh) を利用しています。これらはGitHub Actionsによりコミットされる度に実行されます。

## セキュリティと暗号化

OpenVPNの暗号化のデフォルト設定は弱いため、本スクリプトではそれを改善しています。

OpenVPN 2.4では、暗号化に多くのアップデートがなされ、 ECDSA、ECDH、AES GCM、NCP、tls-cryptがサポートされました。オプションの詳細については、 [OpenVPNマニュアル](https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage) を参照してください。

OpenVPNの暗号化関連の設定のほとんどは、 [Easy-RSA ](https://github.com/OpenVPN/easy-rsa) によって管理されています。 デフォルトのパラメータは [vars.example](https://github.com/OpenVPN/easy-rsa/blob/v3.0.6/easyrsa3/vars.example) にあります。

### 圧縮

本スクリプトは、LZO及びLZ4 (v1/v2) アルゴリズムをサポートしており、後者の方が圧縮率に優れています。しかし、 [VORACLE攻撃](https://protonvpn.com/blog/voracle-attack/) が圧縮を悪用するため、デフォルトでは圧縮を有効にしておらず、このオプションの有効化は推奨しません。

### TLSバージョン

OpenVPNはデフォルトでTLS1.0を受け付けます。このバージョンは [20年前](https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_1.0) に規定された古いものです。

`tls-version-min 1.2` ではTLS1.2以上を受け付けます。TLS1.2は、現在OpenVPNで利用できる最も良いプロトコルであり、OpenVPN 2.3.3以降サポートされています。

### 証明書

OpenVPNはデフォルトで2048ビットのRSA証明書をします。

OpenVPN 2.4ではECDSAのサポートが追加されています。楕円曲線暗号は、より速く、軽く、安全です。

本スクリプトでは、次のオプションを提供しています。

- ECDSA: `prime256v1`/`secp384r1`/`secp521r1` 楕円曲線
- RSA: `2048`/`3072`/`4096` ビット鍵

デフォルトは、ECDSAの `prime256v1` です。

OpenVPNはデフォルトの署名ハッシュとして `SHA-256` を使用し、現時点で他の方法は提供されていません。

### データチャンネル

デフォルトで、データチャンネルの暗号化に `BF-CBC` を使用します。Blowfishは1993年に開発された古いアルゴリズムで、今では脆弱性があり、OpenVPNの公式ドキュメントもその点を認めています。

> The default is BF-CBC, an abbreviation for Blowfish in Cipher Block Chaining mode.
>
> Using BF-CBC is no longer recommended, because of its 64-bit block size. This small block size allows attacks based on collisions, as demonstrated by SWEET32. See https://community.openvpn.net/openvpn/wiki/SWEET32 for details.

> Security researchers at INRIA published an attack on 64-bit block ciphers, such as 3DES and Blowfish. They show that they are able to recover plaintext when the same data is sent often enough, and show how they can use cross-site scripting vulnerabilities to send data of interest often enough. This works over HTTPS, but also works for HTTP-over-OpenVPN. See https://sweet32.info/ for a much better and more elaborate explanation.
>
> OpenVPN's default cipher, BF-CBC, is affected by this attack.

AESはスタンダードな暗号で、現在利用可能な最速かつセキュアな暗号です。[SEED](https://en.wikipedia.org/wiki/SEED) や [Camellia](<https://en.wikipedia.org/wiki/Camellia_(cipher)>) はセキュアですが、AESより速度が遅く、信頼性で劣ります。

> Of the currently supported ciphers, OpenVPN currently recommends using AES-256-CBC or AES-128-CBC. OpenVPN 2.4 and newer will also support GCM. For 2.4+, we recommend using AES-256-GCM or AES-128-GCM.

AES-256は、AES-128よりも40%遅く、AESで128ビット鍵よりも256ビット鍵を使用する理由はありません（出典: [1](http://security.stackexchange.com/questions/14068/why-most-people-use-256-bit-encryption-instead-of-128-bit), [2](http://security.stackexchange.com/questions/6141/amount-of-simple-operations-that-is-safely-out-of-reach-for-all-humanity/6149#6149)）。さらにAES-256は [タイミング攻撃](https://en.wikipedia.org/wiki/Timing_attack) に弱い欠点があります。

AES-GCMは [認証付き暗号](https://en.wikipedia.org/wiki/Authenticated_encryption) の一種であり、データの機密性、完全性、信頼性を保証します。

本スクリプトでは、次のオプションを提供しています。

- `AES-128-GCM`
- `AES-192-GCM`
- `AES-256-GCM`
- `AES-128-CBC`
- `AES-192-CBC`
- `AES-256-CBC`

OpenVPNのデフォルトは `AES-128-GCM` です。

OpenVPN 2.4は、"NCP（_Negotiable Crypto Parameters_）"と呼ばれる機能を追加しました。これは、HTTPSと同様にクライアントにネゴシエート可能な暗号スイートを提供できることを意味しています。デフォルトでは、 `AES-256-GCM:AES-128-GCM` が設定されており、OpenVPN 2.4クライアントを利用すると、 `--cipher`  パラメータは上書きされます。本スクリプトでは簡略化のために、 `--cipher` と `--ncp-cipher` の両方が設定されており、前述の暗号が指定されています。

### コントロールチャンネル

OpenVPN 2.4では、デフォルトで利用可能な最も良い暗号でネゴシエートします。（例：ECDHE+AES-256-GCM）

本スクリプトでは、次のオプションを提供しています。

- ECDSA:
  - `TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256`
  - `TLS-ECDHE-ECDSA-WITH-AES-256-GCM-SHA384`
- RSA:
  - `TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256`
  - `TLS-ECDHE-RSA-WITH-AES-256-GCM-SHA384`

デフォルトは、 `TLS-ECDHE-*-WITH-AES-128-GCM-SHA256` です。

### DH鍵交換

OpenVPNはデフォルトで2048ビットのDH鍵を使用します。

OpenVPN 2.4 では、ECDH鍵がサポートされました。楕円曲線暗号は、より速く、軽く、安全です。また、従来DH鍵の生成には時間がかかる場合がありましたが、ECDH鍵は一時的に利用されるため生成時間が短いメリットがあります。

本スクリプトでは、次のオプションを提供しています。

- ECDH: `prime256v1`/`secp384r1`/`secp521r1` 楕円曲線
- DH: `2048`/`3072`/`4096` ビット鍵

デフォルトは、 `prime256v1` です。

### HMACダイジェスト認証アルゴリズム

OpenVPNマニュアルの `--auth` について：

> Authenticate data channel packets and (if enabled) tls-auth control channel packets with HMAC using message digest algorithm alg. (The default is SHA1 ). HMAC is a commonly used message authentication algorithm (MAC) that uses a data string, a secure hash algorithm, and a key, to produce a digital signature.
>
> If an AEAD cipher mode (e.g. GCM) is chosen, the specified --auth algorithm is ignored for the data channel, and the authentication method of the AEAD cipher is used instead. Note that alg still specifies the digest used for tls-auth.

本スクリプトでは、次のオプションを提供しています。

- `SHA256`
- `SHA384`
- `SHA512`

デフォルトは、 `SHA256` です。

### `tls-auth` ＆ `tls-crypt`

OpenVPNマニュアルの `tls-auth` について：

> Add an additional layer of HMAC authentication on top of the TLS control channel to mitigate DoS attacks and attacks on the TLS stack.
>
> In a nutshell, --tls-auth enables a kind of "HMAC firewall" on OpenVPN's TCP/UDP port, where TLS control channel packets bearing an incorrect HMAC signature can be dropped immediately without response.

`tls-crypt` について：

> Encrypt and authenticate all control channel packets with the key from keyfile. (See --tls-auth for more background.)
>
> Encrypting (and authenticating) control channel packets:
>
> - provides more privacy by hiding the certificate used for the TLS connection,
> - makes it harder to identify OpenVPN traffic as such,
> - provides "poor-man's" post-quantum security, against attackers who will never know the pre-shared key (i.e. no forward secrecy).

従って、どちらもセキュリティの追加レイヤを提供し、DoS攻撃を軽減することができます。これらはデフォルトでは使用されません。

`tls-crypt` は認証に加えて暗号化することができ、OpenVPN 2.4から実装された機能です（ `tls-auth` とは異なります）。

本スクリプトでは両方をサポートしており、デフォルトは `tls-crypt` です。

## クレジット & ライセンス

[貢献者の皆さん](https://github.com/Angristan/OpenVPN-install/graphs/contributors) と オリジナル開発者のNyr氏、及びフォーク元のangristan氏に感謝いたします。

このプロジェクトは [MIT Licence](https://raw.githubusercontent.com/Angristan/openvpn-install/master/LICENSE) の下に公開されています。
