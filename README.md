# VME/VPX 协议包 — 工控背板总线，需 VME 桥接

> [English](README.en.md)

VME/VPX 协议包 — 工控背板总线，需 VME 桥接。需专用硬件，通过内核 Bridge 层桥接至厂商 SDK 或网关设备。

## 安装

```bash
composer require erikwang2013/industrial-protocols-kernel erikwang2013/industrial-protocols-vme
```

> 本包依赖 [erikwang2013/industrial-protocols-kernel](https://github.com/erikwang2013/industrial-protocols-kernel)，内核提供硬件桥接层、连接管理、协议注册等基础设施。

## 设计说明

本协议需要专用硬件芯片或接口卡，PHP 无法直接在应用层实现协议栈。本包通过内核的 Bridge 层桥接至硬件 SDK 或网关设备。BridgeConnector 实现 ConnectorInterface，内部委托给 BridgeInterface（ExternalProcessBridge 通过 proc_open 启动本地 C/C++ SDK 子进程，TcpGatewayBridge 通过 TCP/UDP Socket 连接远程网关硬件）。

## 架构

Bridge 桥接模式：BridgeConnector（实现 ConnectorInterface）→ BridgeInterface（open/close/execute/isReady）→ ExternalProcessBridge（本地 SDK 子进程，proc_open stdin/stdout 通信）或 TcpGatewayBridge（远程网关，TCP/UDP Socket 通信）。上层应用通过 ConnectionManager 统一调用 connect/read/write/getHealth。

## 支持的框架

本包通过内核的框架适配器兼容以下 6 种 PHP 运行时环境：Laravel (ServiceProvider+Facade+artisan)、Webman (config/plugin 自动发现+ProtocolProcess)、Hyperf (ConfigProvider+DI+KernelFactory)、ThinkPHP (services.php+IndustrialProtocolsService)、Yii2 (Bootstrap+组件注册)、Plain PHP (直接实例化 Kernel)

## 使用说明

```php
use Erikwang2013\IndustrialProtocols\Kernel;
use IndustrialProtocols\Bridge\TcpGatewayBridge;
use IndustrialProtocols\Bridge\ExternalProcessBridge;

$kernel = new Kernel(['config_path' => __DIR__ . '/config.php']);
$kernel->boot();

// 方式 1: 通过 TCP 网关连接
$bridge = new TcpGatewayBridge('192.168.1.200', 502);
$conn = $kernel->getConnectionManager()->connect('device-id', [
    'protocol' => 'vme',
    'bridge'   => $bridge,
]);
$result = $conn->read('address');

// 方式 2: 通过厂商工厂一键创建
$bridge = $kernel->getVendorBridgeFactory()->create('hilscher', 'netX 90', '');
$conn = $kernel->getConnectionManager()->connect('device-id', [
    'protocol' => 'vme', 'bridge' => $bridge,
]);

// 方式 3: 通过 C/C++ SDK 子进程
$bridge = new ExternalProcessBridge('/opt/sdk/bin/master');
$conn = $kernel->getConnectionManager()->connect('device-id', [
    'protocol' => 'vme', 'bridge' => $bridge,
]);
```

## 适配厂商

VME 桥接模块（Tundra Universe II, IDT Tsi148）

## 系统要求

- PHP >= 8.1
- Composer
- erikwang2013/industrial-protocols-kernel
- VME/VPX 桥接模块或总线接口

## 相关链接

- [Industrial Protocols 主项目](https://github.com/erikwang2013/industrial-protocols)
- [Kernel 内核](https://github.com/erikwang2013/industrial-protocols-kernel)
- [全部 42 个协议包](https://github.com/erikwang2013/industrial-protocols#支持的协议)

## License

MIT — Copyright (c) 2026 erik <erik@erik.xyz> — https://erik.xyz
