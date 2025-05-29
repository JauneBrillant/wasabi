## 環境

- QEMU(Qucik EMUlator)について
  仮想マシン/ハードウェアエミュレータ
  仮想のcpu、メモリ、ディスク、ネットワークなどを完全にエミュレートする

- ovmf(open virtual maschine firmware)
  UEFIを実装したファームウェアバイナリ(EDK2ベース)
  QEMUに'-biso'オプションで読み込ませることで、仮想マシンがUEFI起動できるようになる

QEMU (仮想マシン)
└─ OVMF（UEFIファームウェア）
└─ EFIアプリ（BOOTX64.EFI） ← あなたのRustで作ったコード

## UEFI

- EFI_GUID
  UEFIでは多くの「プロトコル」や「サービス」がGUIDで識別される

- GUID(Globally unique identifier)
  世界で一意なIDで、128bit(16バイト)
  例: 9042A9DE-23DC-4A38-9FFB-7ADED080516A

```rust
// グラフィック出力に関するGUIDをrustの構造体で表現
const EFI_GRAPHICS_OUTPUT_PROTOCOL_GUID: EfiGuid = EfiGuid {
    data0: 0x9042a9de,
    data1: 0x23dc,
    data2: 0x4a38,
    data3: [0x96, 0xfb, 0x7a, 0xde, 0xd0, 0x80, 0x51, 0x6a],
};
```

- EfiBootServicesTable
  UEFIが提供するブートサービス関数群のテーブル（構造体)
  今回はefi_system_table.boot_services.locate_protocolで指定されたGUIDのプロトコルを取得

  マザーボードにUEFIのファームウェアが搭載されており、その不揮発メモリに関数が定義してある。それらを呼び出すために、UEFIアプリケーションは、EfiBootServicesTableを介してアクセスする。

- EfiSystemTable
  UEFIアプリケーションのエントリーポイント(efi_main)に引数として渡される構造体

マザーボードのROMに保存されたUEFIバイナリが、pc起動時にRAMにコピーされる。
rustで書かれたUEFIアプリケーションも、RAM上のUEFI関数テーブルを通じて、ROMの機能を間接的に使う
UEFIの機能を使うには、マザーボードの不揮発メモリ上にある関数やデータ構造に、正確にアクセスする必要がある。
UEFIファームウェアの内部には、例えばEfiSystemTableやEfiBootServicesTableのような関数テーブルや情報構造が、C言語の構造体の形で物理メモリに配置されている
今回rustでそれにピッタリ一致する構造体を#[repr(C)]で定義し、正しいサイズとオフセットで各フィールドにアクセスすることで、C側の関数ポインタを安全に呼び出せるようにしている。
