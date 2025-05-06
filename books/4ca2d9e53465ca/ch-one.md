---
title: "環境構築"
---

# 環境構築

VscodeのDevcontainerを使用して、この本の開発環境を構築

なんだかんだで1日かかった

詳しくはこのリポジトリで

https://github.com/XRayZen/rust_os

## 質問プロンプト

OSを１から実装する場合のTODOを教えて
Kernelの実装からユーザー空間の実装初期まで
Rustで実装するので簡易的・抽象的な短いコード例も提示して
x86_64 Qemu上で実行
開発環境は構築済み
ネットワークも実装する


## OSができるまでのTODO
- フリースタンディングRustバイナリを作成
- カスタムターゲットを設定
- ブートローダーを実装
- 基本的なカーネル機能（画面出力、割り込み）を実装
- メモリ管理を実装
- プロセス管理とスケジューラを実装
- システムコールを実装
- ファイルシステムを実装
- シェルなどのユーザー空間アプリケーションを実装


# OSを1から実装するための詳細なTODO

## 1. フリースタンディングRustバイナリの作成

1. 標準ライブラリに依存しないRustバイナリを作成する
   - `#![no_std]`アトリビュートを追加して標準ライブラリのリンクを無効化する
   - `#![no_main]`アトリビュートを追加してRustのエントリポイントを無効化する
   - パニックハンドラを実装する
     ```rust
     #[panic_handler]
     fn panic(_info: &PanicInfo) -> ! {
         loop {}
     }
     ```
   - `Cargo.toml`でパニック時のスタックアンワインドを無効化する
     ```toml
     [profile.dev]
     panic = "abort"
     
     [profile.release]
     panic = "abort"
     ```

2. エントリポイントを定義する
   - `#[no_mangle]`アトリビュートを使用して名前修飾を防ぐ
   - `extern "C"`を使用してCの呼び出し規約を指定する
   - 戻り値が`!`（never型）のエントリポイント関数を実装する
     ```rust
     #[no_mangle]
     pub extern "C" fn _start() -> ! {
         loop {}
     }
     ```

## 2. カスタムターゲットの設定

1. x86_64向けのカスタムターゲット仕様ファイルを作成する
   - JSONファイルでターゲット仕様を定義する
   - 以下の内容を含むx86_64-blog_os.jsonファイルを作成する
     ```json
     {
       "llvm-target": "x86_64-unknown-none",
       "data-layout": "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-i128:128-f80:128-n8:16:32:64-S128",
       "arch": "x86_64",
       "target-endian": "little",
       "target-pointer-width": "64",
       "target-c-int-width": "32",
       "os": "none",
       "executables": true,
       "linker-flavor": "ld.lld",
       "linker": "rust-lld",
       "panic-strategy": "abort",
       "disable-redzone": true,
       "features": "-mmx,-sse,+soft-float"
     }
     ```

2. Rustのnightly版をインストールする
   - `rustup override set nightly`コマンドを実行するか、`rust-toolchain.toml`ファイルを作成する
   ```toml
   [toolchain]
   channel = "nightly"
   components = ["rust-src", "llvm-tools-preview", "rustfmt", "clippy"]
   targets = ["x86_64-unknown-none"]
   ```
   - 必要な実験的機能を有効化するためのfeatureフラグを設定する
   - Rust 1.86.0では多くの機能が安定化されているため、一部の機能はnightlyが不要になっている

3. ビルド設定を構成する
   - `.cargo/config.toml`ファイルを作成してビルドコマンドを統一する
   - ターゲットに応じたリンカ引数を設定する

## 3. ブートローダーの実装

1. bootloaderクレートを利用する
   - `Cargo.toml`に依存関係として追加する
     ```toml
     [dependencies]
     bootloader = "0.11.10"
     ```

2. bootimageツールをインストールする
   - `cargo install bootimage`コマンドを実行する
   - `rustup component add llvm-tools-preview`で必要なコンポーネントを追加する

3. ブートイメージを作成する
   - `cargo bootimage`コマンドを実行してブータブルディスクイメージを生成する
   - 生成されたイメージをQEMUで実行する

## 4. 基本的なカーネル機能の実装

1. VGAテキストモードによる画面出力を実装する
   - VGAバッファを操作するモジュールを作成する
   - 色を表現する列挙型を定義する
   - 画面文字とテキストバッファを表す構造体を実装する
     ```rust
     #[repr(C)]
     struct ScreenChar {
         ascii_character: u8,
         color_code: ColorCode,
     }
     
     #[repr(transparent)]
     struct Buffer {
         chars: [[ScreenChar; BUFFER_WIDTH]; BUFFER_HEIGHT],
     }
     ```
   - 画面出力のためのWriterを実装する
   - `print!`と`println!`マクロを実装する

2. 割り込み処理を実装する
   - 割り込み記述子表（IDT）を設定する
     ```rust
     use x86_64::structures::idt::{InterruptDescriptorTable, InterruptStackFrame};
     use lazy_static::lazy_static;
     use spin::Mutex;
     
     lazy_static! {
         static ref IDT: InterruptDescriptorTable = {
             let mut idt = InterruptDescriptorTable::new();
             idt.breakpoint.set_handler_fn(breakpoint_handler);
             idt.page_fault.set_handler_fn(page_fault_handler);
             unsafe {
                 idt.double_fault
                     .set_handler_fn(double_fault_handler)
                     .set_stack_index(gdt::DOUBLE_FAULT_IST_INDEX);
             }
             idt[InterruptIndex::Timer.as_usize()]
                 .set_handler_fn(timer_interrupt_handler);
             idt[InterruptIndex::Keyboard.as_usize()]
                 .set_handler_fn(keyboard_interrupt_handler);
             idt
         };
     }
     
     pub fn init_idt() {
         IDT.load();
     }
     ```
   - CPU例外ハンドラを実装する
     ```rust
     extern "x86-interrupt" fn page_fault_handler(
         stack_frame: InterruptStackFrame,
         error_code: PageFaultErrorCode,
     ) {
         let addr = Cr2::read();
         println!("PAGE FAULT");
         println!("Accessed Address: {:?}", addr);
         println!("Error Code: {:?}", error_code);
         println!("{:#?}", stack_frame);
         hlt_loop();
     }
     ```
   - ダブルフォルト例外を処理する
   - 割り込みスタックテーブル（IST）を設定する
   - APICを使用した最新の割り込み処理（PICの代わり）
     ```rust
     pub fn init_apic() {
         unsafe {
             // ローカルAPICの初期化
             let mut lapic = LocalApic::new(PhysAddr::new(0xFEE00000));
             lapic.enable();
             lapic.set_timer_divide(TimerDivide::Div16);
             
             // IOAPICの初期化
             let mut ioapic = IoApic::new(PhysAddr::new(0xFEC00000));
             ioapic.init();
             ioapic.enable_irq(1, 0); // キーボード割り込みを有効化
         }
     }
     ```
   - タイマー割り込みとキーボード割り込みのハンドラを実装する
   - 割り込み優先度とネスト処理の実装

## 5. メモリ管理の実装

1. ページングの基本を理解する
   - 仮想メモリと物理メモリの概念を把握する
   - x86_64のページテーブル構造を理解する
   - 5レベルページングの新機能を理解する（Rust 1.86.0対応）

2. 物理メモリへのアクセスを実装する
   - ブートローダーから物理メモリマップ情報を取得する
     ```rust
     fn kernel_main(boot_info: &'static BootInfo) -> ! {
         let phys_mem_offset = VirtAddr::new(boot_info.physical_memory_offset);
         let memory_regions = &boot_info.memory_regions;
         
         // メモリマップの初期化
         memory::init(phys_mem_offset, memory_regions);
     }
     ```
   - 物理アドレスから仮想アドレスへの変換関数を実装する
   - メモリマップBIOSからのメモリ情報を解析する

3. ページテーブル操作を実装する
   - 現在のページテーブルにアクセスする関数を実装する
     ```rust
     pub fn active_level_4_table(physical_memory_offset: VirtAddr) -> &'static mut PageTable {
         let (level_4_table_frame, _) = x86_64::registers::control::Cr3::read();
         let phys = level_4_table_frame.start_address();
         let virt = physical_memory_offset + phys.as_u64();
         let page_table_ptr: *mut PageTable = virt.as_mut_ptr();
         unsafe { &mut *page_table_ptr }
     }
     ```
   - 仮想アドレスを物理アドレスに変換する関数を実装する
   - 新しいマッピングを作成する関数を実装する
   - ページテーブルフラグの最適化（実行保護、書き込み保護など）

4. ヒープ割り当てを実装する
   - ヒープメモリ領域を作成する
     ```rust
     pub fn init_heap(
         mapper: &mut impl Mapper<Size4KiB>,
         frame_allocator: &mut impl FrameAllocator<Size4KiB>,
     ) -> Result<(), MapToError<Size4KiB>> {
         let page_range = {
             let heap_start = VirtAddr::new(HEAP_START as u64);
             let heap_end = heap_start + HEAP_SIZE - 1u64;
             let heap_start_page = Page::containing_address(heap_start);
             let heap_end_page = Page::containing_address(heap_end);
             Page::range_inclusive(heap_start_page, heap_end_page)
         };
         
         for page in page_range {
             let frame = frame_allocator
                 .allocate_frame()
                 .ok_or(MapToError::FrameAllocationFailed)?;
             let flags = PageTableFlags::PRESENT | PageTableFlags::WRITABLE;
             unsafe {
                 mapper.map_to(page, frame, flags, frame_allocator)?.flush();
             }
         }
         
         // グローバルアロケータの初期化
         unsafe {
             ALLOCATOR.lock().init(HEAP_START, HEAP_SIZE);
         }
         
         Ok(())
     }
     ```
   - Rustのアロケーションインターフェースを実装する
   - 最新のアロケータクレート（linked_list_allocator 0.10.5など）を設定する
   - メモリリークトラッキングとデバッグ機能の実装

## 6. プロセス管理とスケジューラの実装

1. プロセス構造体を定義する
   - タスクID、状態、スタック情報を保持する構造体を実装する
     ```rust
     pub struct Task {
         id: TaskId,
         state: TaskState,
         stack: [u8; STACK_SIZE],
         stack_pointer: usize,
     }
     ```

2. コンテキスト切り替え機能を実装する
   - レジスタ状態を保存・復元する関数を実装する
   - プロセス間の切り替えを行うメカニズムを実装する
     ```rust
     extern "C" fn app_main() -> ! {
         hprintln!("App").unwrap();
         unsafe { asm!("svc 0"::::"volatile"); }
         loop {}
     }
     ```

3. スケジューラを実装する
   - ラウンドロビン方式のスケジューラを実装する
     ```rust
     pub struct Scheduler {
         list: LinkedList<Task>,
     }
     ```
   - プロセスの実行順序を管理する機能を実装する
   - タイマー割り込みと連携したプロセス切り替えを実装する

## 7. システムコールの実装

1. システムコールインターフェースを定義する
   - システムコール番号を定義する
   - システムコールハンドラを実装する
     ```rust
     extern "x86-interrupt" fn syscall_handler(stack_frame: InterruptStackFrame) {
         let syscall_number = unsafe { core::arch::asm!("mov {}, rax", out(reg) let num: u64) };
         
         match syscall_number {
             1 => sys_write(),
             2 => sys_read(),
             _ => println!("Unknown syscall: {}", syscall_number),
         }
     }
     ```

2. ユーザー空間からのシステムコール呼び出しを実装する
   - x86_64の`syscall`命令を使用するためのラッパーを実装する
   - システムコール引数の受け渡し方法を実装する

3. 基本的なシステムコール関数を実装する
   - ファイル操作（open, read, write, close）
   - プロセス管理（fork, exec, exit）
   - メモリ管理（brk, mmap）

## 8. ファイルシステムの実装

1. ファイルシステムの抽象化レイヤーを設計する
   - ファイルとディレクトリを表す構造体を定義する
     ```rust
     pub struct FileSystem {
         root_directory: Directory,
     }
     
     pub struct Directory {
         name: &'static str,
         files: Vec,
         subdirectories: Vec,
     }
     
     pub struct File {
         name: &'static str,
         content: &'static [u8],
     }
     ```

2. シンプルなファイルシステムを実装する
   - FATファイルシステムの基本構造を理解する
   - ディスクのパーティション情報を読み取る
   - ファイルシステムのメタデータを解析する

3. ファイル操作APIを実装する
   - ファイルの作成、読み取り、書き込み、削除機能を実装する
     ```rust
     impl FileSystem {
         pub fn new() -> Self {
             FileSystem {
                 root_directory: Directory {
                     name: "/",
                     files: Vec::new(),
                     subdirectories: Vec::new(),
                 }
             }
         }
         
         pub fn create_file(&mut self, path: &str, content: &'static [u8]) -> Result {
             // ファイル作成処理
             Ok(())
         }
         
         pub fn read_file(&self, path: &str) -> Option {
             // ファイル読み込み処理
             None
         }
     }
     ```
   - ディレクトリ操作機能を実装する

## 9. ネットワークの実装

1. ネットワークデバイスドライバを実装する
   - QEMUのe1000/virtio-netネットワークカードドライバを実装する
     ```rust
     pub struct E1000Device {
         base_addr: PhysAddr,
         rx_ring: RxRing,
         tx_ring: TxRing,
         mac_address: [u8; 6],
         irq: u8,
     }
     
     impl NetworkDevice for E1000Device {
         fn init(&mut self) -> Result<(), DeviceError> {
             // デバイスの初期化処理
             self.reset();
             self.init_rx_ring();
             self.init_tx_ring();
             self.enable_interrupts();
             Ok(())
         }
         
         async fn receive(&mut self) -> Result<Packet, DeviceError> {
             // 非同期パケット受信処理
             let rx_desc = self.rx_ring.next_to_check();
             if !rx_desc.is_done() {
                 // 受信完了まで待機
                 self.wait_for_rx_interrupt().await;
             }
             
             let packet = self.rx_ring.get_packet(rx_desc);
             self.rx_ring.return_descriptor(rx_desc);
             Ok(packet)
         }
         
         async fn send(&mut self, packet: &Packet) -> Result<(), DeviceError> {
             // 非同期パケット送信処理
             let tx_desc = self.tx_ring.get_free_descriptor()?;
             self.tx_ring.fill_descriptor(tx_desc, packet);
             self.tx_ring.submit_descriptor(tx_desc);
             
             // 送信完了まで待機
             self.wait_for_tx_complete(tx_desc).await;
             Ok(())
         }
     }
     ```
   - PCI設定空間からネットワークカードを検出する
   - 割り込み駆動の非同期ネットワーク処理を実装する
   - DMAバッファ管理の最適化

2. 最新のネットワークプロトコルスタックを実装する
   - イーサネットフレームの処理を実装する
   - ARPプロトコルを実装する
     ```rust
     #[derive(Debug, Clone, Copy)]
     pub struct ArpPacket {
         hardware_type: u16,
         protocol_type: u16,
         hardware_size: u8,
         protocol_size: u8,
         operation: u16,
         sender_mac: [u8; 6],
         sender_ip: Ipv4Addr,
         target_mac: [u8; 6],
         target_ip: Ipv4Addr,
     }
     
     impl ArpPacket {
         pub fn new_request(sender_mac: [u8; 6], sender_ip: Ipv4Addr, target_ip: Ipv4Addr) -> Self {
             Self {
                 hardware_type: 1, // Ethernet
                 protocol_type: 0x0800, // IPv4
                 hardware_size: 6,
                 protocol_size: 4,
                 operation: 1, // Request
                 sender_mac,
                 sender_ip,
                 target_mac: [0; 6],
                 target_ip,
             }
         }
         
         pub fn to_bytes(&self) -> [u8; 28] {
             // パケットのシリアライズ処理
             let mut bytes = [0u8; 28];
             // ...
             bytes
         }
     }
     ```
   - IPv4/IPv6デュアルスタックの実装
   - ICMPv4/ICMPv6プロトコルの実装
   - UDPプロトコルの実装
   - TCPプロトコルの実装（輻輳制御、フロー制御、再送メカニズム）
   - DNSクライアントの実装

3. 最新のネットワークインターフェース抽象化
   - ネットワークインターフェース構造体を定義する
     ```rust
     pub struct NetworkInterface {
         name: String,
         mac_address: [u8; 6],
         ipv4_config: Ipv4Config,
         ipv6_config: Option<Ipv6Config>,
         mtu: u16,
         driver: Arc<Mutex<dyn NetworkDevice>>,
         stats: NetworkStats,
     }
     
     pub struct Ipv4Config {
         address: Ipv4Addr,
         subnet_mask: Ipv4Addr,
         gateway: Option<Ipv4Addr>,
         dns_servers: Vec<Ipv4Addr>,
     }
     ```
   - 複数のネットワークインターフェースを管理する機能を実装する
   - DHCPクライアントによる自動設定
   - ルーティングテーブルの実装

4. 非同期ソケットAPIを実装する
   - 非同期ソケット抽象化を定義する
     ```rust
     pub struct AsyncSocket<T: SocketProtocol> {
         protocol: T,
         state: SocketState,
         rx_buffer: RingBuffer,
         tx_buffer: RingBuffer,
     }
     
     impl<T: SocketProtocol> AsyncSocket<T> {
         pub async fn connect(&mut self, addr: SocketAddr) -> Result<(), SocketError> {
             // 非同期接続処理
             self.protocol.connect(addr).await
         }
         
         pub async fn send(&mut self, data: &[u8]) -> Result<usize, SocketError> {
             // 非同期送信処理
             self.protocol.send(data).await
         }
         
         pub async fn recv(&mut self, buf: &mut [u8]) -> Result<usize, SocketError> {
             // 非同期受信処理
             self.protocol.recv(buf).await
         }
     }
     ```
   - 非同期ソケット操作の実装（async/await対応）
   - ゼロコピーデータ転送の最適化
   - TLS/SSLサポートの実装

## 10. シェルなどのユーザー空間アプリケーションの実装

1. シンプルなシェルを実装する
   - コマンドライン入力を処理する機能を実装する
     ```rust
     pub struct Shell {
         input_buffer: [u8; 256],
         position: usize,
     }
     
     impl Shell {
         pub fn new() -> Self {
             Shell {
                 input_buffer: [0; 256],
                 position: 0,
             }
         }
         
         pub fn process_keypress(&mut self, key: u8) {
             if key == b'\n' {
                 self.execute_command();
                 self.position = 0;
             } else {
                 self.input_buffer[self.position] = key;
                 self.position += 1;
                 print!("{}", key as char);
             }
         }
         
         fn execute_command(&self) {
             let cmd = core::str::from_utf8(&self.input_buffer[0..self.position]).unwrap_or("");
             match cmd {
                 "help" => println!("利用可能なコマンド: help, echo, ls, ping"),
                 "ping" => self.ping_command(),
                 _ => println!("不明なコマンド: {}", cmd),
             }
         }
         
         fn ping_command(&self) {
             // pingコマンドの実装
         }
     }
     ```

2. 基本的なコマンドを実装する
   - `help`、`echo`、`ls`などの基本コマンドを実装する
   - ネットワーク関連コマンド（`ping`、`ifconfig`）を実装する
   - コマンド実行機能を実装する

3. ユーザープログラムのロードと実行機能を実装する
   - ELFファイルの解析と実行機能を実装する
   - ユーザープログラムの実行環境を設定する

## 11. 開発環境の整備とテスト

1. QEMUでの実行環境を設定する
   - QEMUの起動スクリプトを作成する
     ```bash
     qemu-system-x86_64 -drive format=raw,file=target/x86_64-blog_os/debug/bootimage-blog_os.bin -netdev user,id=u1 -device e1000,netdev=u1 -serial mon:stdio
     ```

2. デバッグ機能を実装する
   - シリアルポート出力によるログ機能を実装する
   - GDBを使用したリモートデバッグ環境を設定する

3. 単体テストと統合テストを実装する
   - テスト用のハーネスを実装する
   - 各コンポーネントの単体テストを作成する
   - システム全体の統合テストを作成する
   - ネットワーク機能のテストを実装する

これらのステップを順番に進めることで、基本的な機能を持つネットワーク対応のOSを構築できます。各ステップは複雑なので、一つずつ確実に実装していくことをお勧めします。

## 12. 非同期プログラミングとEmbassyフレームワーク

Rust 1.86.0では非同期プログラミングのサポートが大幅に向上しており、組込みシステムでも効率的な非同期処理が可能になっています。特にEmbassyフレームワークは、ベアメタル環境での非同期プログラミングを簡素化します。

1. Embassyフレームワークの導入
   - `Cargo.toml`に依存関係を追加する
     ```toml
     [dependencies]
     embassy-executor = { version = "0.3.0", features = ["nightly", "arch-x86_64"] }
     embassy-time = { version = "0.1.0", features = ["nightly"] }
     embassy-sync = { version = "0.2.0" }
     ```

2. 非同期ランタイムの設定
   - 非同期エグゼキュータの初期化
     ```rust
     #[embassy_executor::main]
     async fn main(_spawner: embassy_executor::Spawner) {
         // 非同期タスクを実行
     }
     ```
   - タスクスポーニングの実装
     ```rust
     #[embassy_executor::task]
     async fn my_task() {
         loop {
             // 非同期タスクのロジック
             embassy_time::Timer::after(embassy_time::Duration::from_millis(1000)).await;
         }
     }
     ```

3. 非同期デバイスドライバの実装
   - 割り込み駆動の非同期I/O
   - ポーリングなしでのハードウェア待機
   - 電力効率の高い待機状態の活用

4. 非同期通信プリミティブ
   - チャネルを使用したタスク間通信
     ```rust
     use embassy_sync::channel::Channel;
     use embassy_sync::blocking_mutex::raw::CriticalSectionRawMutex;
     
     static CHANNEL: Channel<CriticalSectionRawMutex, u32, 1> = Channel::new();
     
     #[embassy_executor::task]
     async fn producer() {
         let mut counter = 0;
         loop {
             CHANNEL.send(counter).await;
             counter += 1;
             embassy_time::Timer::after(embassy_time::Duration::from_millis(1000)).await;
         }
     }
     
     #[embassy_executor::task]
     async fn consumer() {
         loop {
             let value = CHANNEL.receive().await;
             println!("Received: {}", value);
         }
     }
     ```

5. 非同期ネットワークスタックの実装
   - 非ブロッキングソケット操作
   - 効率的なパケット処理
   - 複数接続の同時処理

Embassyフレームワークを活用することで、リソース制約のある環境でも効率的なマルチタスク処理が可能になり、割り込み処理やI/O操作を簡素化できます。また、非同期プログラミングモデルにより、コードの可読性と保守性も向上します。

