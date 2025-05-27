- QEMU(Qucik EMUlator)について
  仮想マシン/ハードウェアエミュレータ
  仮想のcpu、メモリ、ディスク、ネットワークなどを完全にエミュレートする

- ovmf(open virtual maschine firmware)
  UEFIを実装したファームウェアバイナリ(EDK2ベース)
  QEMUに'-biso'オプションで読み込ませることで、仮想マシンがUEFI起動できるようになる

QEMU (仮想マシン)
└─ OVMF（UEFIファームウェア）
└─ EFIアプリ（BOOTX64.EFI） ← あなたのRustで作ったコード
