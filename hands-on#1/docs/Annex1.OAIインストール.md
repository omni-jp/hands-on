hands-on用のコンテナイメージ準備でOpenAirInterface(OAI) RANをインストールした手順です。

## 1. OAI RANソフトダウンロード

OAI RAN (eNBとUE) ソースコードはEurecom [gitlab repository](https://gitlab.eurecom.fr/oai/openairinterface5g/)からダウンロードできます。

``` 
$ git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
``` 

* **master** : ユーザへのリリースブランチ. 2021年6月8日現在：最新[v1.2.2](https://gitlab.eurecom.fr/oai/openairinterface5g/-/releases#v1.2.2)
* **develop** : 開発本線のブランチ. Merge RequestはCI自動テストとTechnical committeeメンバのレビューをクリアするとdevelop branchへmergeされる

branchの考え方 [https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/policy/branch-policy](https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/policy/branch-policy)

Merge Request Policy [https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/policy/merge-request-policy](https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/policy/merge-request-policy)


## 2. 初回ビルド

ダウンロードしたソースコードを以下のコマンド(無線機なし)でビルドすることでeNBとUEそれぞれの実行ファイルを作成できます。

``` 
$ cd <your oai installation directory>/openairinterface5g/
$ source oaienv
$ cd cmake_targets/
## スーパーユーザ権限でないユーザの場合は、sudoを付与する
$ ./build_oai -I --eNB --UE
``` 

- `-I` option : OAIをビルド・実行するために必要なパッケージをインストールしますオプションです。OAIをダウンロードしたマシンの初回ビルド時にのみ必要です。
- `--eNB` : eNBの実行ファイルである`lte-softmodem` を生成します。
- `--UE` :  UEの実行ファイルである`lte-uesoftmodem`を生成します。

無線機(USRP)ありの場合は以下のコマンドでビルドしてください。

- `-w` option : 使用する無線機を選択するオプションです。USRPは、CI自動試験で現在テストされている唯一のハードウェアです。

``` 
$ cd <your oai installation directory>/openairinterface5g/
$ source oaienv
$ cd cmake_targets/
## スーパーユーザ権限でないユーザの場合は、sudoを付与する
$ ./build_oai -I -w USRP --eNB --UE
``` 

## 3. linux-headersパッケージ追加

linuxヘッダーがインストールされていない場合、UE(nasmesh)でのビルドに失敗します。
kernel versionに応じた、linux-headersのパッケージを追加ください。

``` 
$ apt-get install -y linux-headers-$(uname -r)
```


## 4. 再ビルド

linuxヘッダーを追加後に再ビルドを実施します。

``` 
$ cd <your oai installation directory>/openairinterface5g/
$ source oaienv
$ cd cmake_targets/
## スーパーユーザ権限でないユーザの場合は、sudoを付与する
$ ./build_oai -c --eNB --UE
## "-c"オプションを必ず付与してください。
``` 

- `-c` option : 前回のビルドで生成されたファイルを削除して再ビルドを開始するオプションです。

その他オプションはビルドスクリプト[build_oai](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/master/cmake_targets/build_oai)に記載されています。

``` 
## build_oai抜粋
This program installs OpenAirInterface Software
You should have ubuntu 16.xx or 18.04 updated
Options
-h
   This help
-c | --clean
   Erase all files to make a rebuild from start
-C | --clean-all
   Erase all files made by previous compilations, installations
--clean-kernel
   Erase previously installed features in kernel: iptables, drivers, ...
-I | --install-external-packages
   Installs required packages such as LibXML, asn1.1 compiler, freediameter, ...
   This option will require root password
--install-optional-packages
   Install useful but not mandatory packages such as valgrind
-g | --run-with-gdb
   Add debugging symbols to compilation directives. It also disables any compiler optimization. Only for debugging. Do not use in normal operation!
-h | --help
   Print this help
--eNB
   Makes the LTE softmodem
--UE
   Makes the UE specific parts (ue_ip, usim, nvram) from the given configuration file
--UE-conf-nvram [configuration file]
   Specify conf_nvram_path (default \"$conf_nvram_path\")
--UE-gen-nvram [output path]
   Specify gen_nvram_path (default \"$gen_nvram_path\")
-a | --agent
   Enables agent for software-defined control of the eNB
-r | --3gpp-release
   default is Rel14,
   Rel8 limits the implementation to 3GPP Release 8 version
   Rel10 limits the implementation to 3GPP Release 10 version
-w | --hardware
   EXMIMO, USRP, BLADERF, LMSSDR, IRIS, None (Default)
   Adds this RF board support (in external packages installation and in compilation)
--phy_simulators
   Makes the unitary tests Layer 1 simulators
--core_simulators
   Makes the core security features unitary simulators
-s | --check
   runs a set of auto-tests based on simulators and several compilation tests
--run-group 
   runs only specified test cases specified here. This flag is only valid with -s
-V | --vcd
   Adds a debgging facility to the binary files: GUI with major internal synchronization events
--install-system-files
   Install OpenArInterface required files in Linux system
   (will ask root password)
--verbose-compile
   Shows detailed compilation instructions in makefile
--cflags_processor
   Manually Add CFLAGS of processor if they are not detected correctly by script. Only add these flags if you know your processor supports them. Example flags: -msse3 -msse4.1 -msse4.2 -mavx2
--build-doxygen
   Builds doxygen based documentation.
--build-coverity-scan
   Builds Coverity-Scan objects for upload
--disable-deadline
   Disables deadline scheduler of Linux kernel (>=3.14.x).
--enable-deadline
   Enable deadline scheduler of Linux kernel (>=3.14.x). 
--disable-cpu-affinity
   Disables CPU Affinity between UHD/TX/RX Threads (Valid only when deadline scheduler is disabled). By defaulT, CPU Affinity is enabled when not using deadline scheduler. It is enabled only with >2 CPUs. For eNB, CPU_0-> Device library (UHD), CPU_1->TX Threads, CPU_2...CPU_MAX->Rx Threads. For UE, CPU_0->Device Library(UHD), CPU_1..CPU_MAX -> All the UE threads
--disable-T-Tracer
   Disables the T tracer.
--disable-hardware-dependency
   Disable HW dependency during installation
--ue-autotest-trace
    Enable specific traces for UE autotest framework
--ue-trace
    Enable traces for UE debugging
--ue-timing
    Enable traces for timing
--build-eclipse
   Build eclipse project files. Paths are auto corrected by fixprj.sh
--build-lib <libraries>
   Build optional shared library, <libraries> can be one or several of $OPTIONAL_LIBRARIES or \"all\"
--usrp-recplay
   Build for I/Q record-playback modes

Usage (first build):
 NI/ETTUS B201  + COTS UE : ./build_oai -I  --eNB --install-system-files -w USRP
Usage (Regular):
 Eurecom EXMIMO + OAI ENB : ./build_oai --eNB  
 NI/ETTUS B201  + OAI ENB : ./build_oai --eNB -w USRP"

``` 

OAIのビルド手順 [https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/master/doc/BUILD.md](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/master/doc/BUILD.md)
