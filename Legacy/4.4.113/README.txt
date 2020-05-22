HOWTO: make Mellanox OFED driver work with the upstream 4.4.113 kernel for
Linux VM on Hyper-V.

1. apply the below patches to make SR-IOV work with the 4.4.113 kernel

1.1 checkout the upstream 4.4.113 kernel
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git/commit/?h=v4.4.113&id=f0d0a93b0e81278e86c7d81c25a54ac4f4b739d2

1.2 apply the 1 patch
0001-hyperv-backport-vPCI-and-SRIOV-support-for-Linux-4.4.patch

1.3 use the "kernel-config" as the .config and build & install the kernel/drivers; then shutdown the VM.

1.4 enable SRIOV to the vNIC of this VM, then boot up the VM with the new kernel.

1.5 run "lspci" in this VM, and you will get:
0002:00:02.0 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]

2. enable the OFED driver

2.1 Download the related OFED driver
http://www.mellanox.com/page/products_dyn?product_family=26&mtag=linux_sw_drivers
e.g. if the VM's distro is Ubuntu 16.04, we can use
Download | Current Versions: 4.2-1.2.0.0 | Ubuntu 16.04 | x86_64 |  MLNX_OFED_LINUX-4.2-1.2.0.0-ubuntu16.04-x86_64.iso.

2.2 Copy the .iso into the VM and run “./mlnxofedinstall”.
We can get an error about missing DKMS support. Let's run “apt-get install dkms” to fix it.

2.3 Re-run the script and we'll get another error “DKMS is disabled for
4.4.113+ kernel, MLNX_OFED_LINUX does not have non-DKMS driver packages
available for this kernel”. It means we need to build the OFED driver from
source code.

2.4 Uncompress the src/MLNX_OFED_SRC-4.2-1.2.0.0.tgz, and run
src/MLNX_OFED_SRC-4.2-1.2.0.0/install.pl. After installing the necessary
prerequisite packages, we'll start to build the OFED drivers and this can take ~30
minutes, and finally the generated drivers will be installed automatically with
no error.  

2.5 Reboot the VM, and the MLX OFED driver should load fine.

root@u1604:~# lspci -s 0002:00:02.0 -vvv
0002:00:02.0 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
        Subsystem: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
        Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx-
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 0
        Region 2: Memory at fe0800000 (64-bit, prefetchable) [size=8M]
        Capabilities: [60] Express (v2) Endpoint, MSI 00
                DevCap: MaxPayload 512 bytes, PhantFunc 0, Latency L0s <64ns, L1 <1us
                        ExtTag- AttnBtn- AttnInd- PwrInd- RBE- FLReset+
                DevCtl: Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
                        RlxdOrd- ExtTag- PhantFunc- AuxPwr- NoSnoop- FLReset-
                        MaxPayload 128 bytes, MaxReadReq 128 bytes
                DevSta: CorrErr- UncorrErr- FatalErr- UnsuppReq- AuxPwr- TransPend-
                LnkCap: Port #0, Speed 8GT/s, Width x8, ASPM not supported, Exit Latency L0s <64ns, L1 <1us
                        ClockPM- Surprise- LLActRep- BwNot- ASPMOptComp-
                LnkCtl: ASPM Disabled; RCB 64 bytes Disabled- CommClk-
                        ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
                LnkSta: Speed unknown, Width x0, TrErr- Train- SlotClk- DLActive- BWMgmt- ABWMgmt-
                DevCap2: Completion Timeout: Range ABCD, TimeoutDis+, LTR-, OBFF Not Supported
                DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis-, LTR-, OBFF Disabled
                LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete-, EqualizationPhase1-
                         EqualizationPhase2-, EqualizationPhase3-, LinkEqualizationRequest-
        Capabilities: [9c] MSI-X: Enable+ Count=24 Masked-
                Vector table: BAR=2 offset=00002000
                PBA: BAR=2 offset=00003000
        Capabilities: [40] Power Management version 0
                Flags: PMEClk- DSI- D1- D2- AuxCurrent=0mA PME(D0-,D1-,D2-,D3hot-,D3cold-)
                Status: D0 NoSoftRst- PME-Enable- DSel=0 DScale=0 PME-
        Kernel driver in use: mlx4_core
        Kernel modules: mlx4_core

root@u1604:~# dmesg |grep -i -E 'mlx|netvsc'
[    2.298464] hv_vmbus: registering driver hv_netvsc
[    2.306160] hv_netvsc: hv_netvsc channel opened successfully
[    2.327703] hv_netvsc vmbus_15: Send section size: 6144, Section count:2560
[    2.329292] hv_netvsc vmbus_15: Device MAC 00:15:5d:af:0c:20 link state up
[    2.344797] hv_netvsc: hv_netvsc channel opened successfully
[    2.366896] hv_netvsc vmbus_16: Send section size: 6144, Section count:2560
[    2.369889] hv_netvsc vmbus_16: Device MAC 00:15:5d:af:0c:21 link state up
[    6.829611] mlx_compat: loading out-of-tree module taints kernel.
[    6.832030] mlx_compat: module verification failed: signature and/or required key missing - tainting kernel
[    8.533853] mlx4_core: Mellanox ConnectX core driver v4.2-1.2.0
[    8.533876] mlx4_core: Initializing 0002:00:02.0
[    8.537312] mlx4_core 0002:00:02.0: Detected virtual function - running in slave mode
[    8.537414] mlx4_core 0002:00:02.0: Sending reset
[    8.537557] mlx4_core 0002:00:02.0: Sending vhcr0
[    8.539031] mlx4_core 0002:00:02.0: HCA minimum page size:512
[    8.539653] mlx4_core 0002:00:02.0: Timestamping is not supported in slave mode
[    8.539655] mlx4_core: device is working in RoCE mode: Unknown
[    8.539656] mlx4_core: gid_type 0 for UD QPs is not supported by the device, gid_type 3 was chosen instead
[    8.539657] mlx4_core: UD QP Gid type is: Unknown
[    8.664339] mlx4_en: Mellanox ConnectX HCA Ethernet driver v4.2-1.2.0
[    8.665205] mlx4_en 0002:00:02.0: Activating port:1
[    8.688888] mlx4_en: 0002:00:02.0: Port 1: Using 8 TX rings
[    8.688891] mlx4_en: 0002:00:02.0: Port 1: Using 8 RX rings
[    8.689086] mlx4_en: 0002:00:02.0: Port 1: Initializing port
[    8.692996] mlx4_core 0002:00:02.0 eth2: joined to eth1
[    8.693001] hv_netvsc vmbus_16 eth1: VF registering: eth2
[    8.701125] mlx4_core 0002:00:02.0 enP2p0s2: renamed from eth2
[    9.716385] <mlx4_ib> mlx4_ib_add: mlx4_ib: Mellanox ConnectX InfiniBand driver v4.2-1.2.0
[    9.719254] <mlx4_ib> mlx4_ib_add: counter index 6 for port 1 allocated 1
[    9.720253] ib_query_pkey failed (-1) for mlx4_0 (index 0)
[    9.722457] mlx4_core 0002:00:02.0: mlx4_ib: multi-function enabled
[    9.722460] mlx4_core 0002:00:02.0: mlx4_ib: operating in qp1 tunnel mode
[    9.939433] mlx4_en: enP2p0s2: Steering Mode 2
[    9.969773] hv_netvsc vmbus_16 eth1: VF up: enP2p0s2
[    9.969777] hv_netvsc vmbus_16 eth1: Data path switched to VF: enP2p0s2
[    9.973777] mlx4_en: enP2p0s2: Link Up

And "ifconfig" should show the VF interface "enP2p0s2".

Note: the patchset backported the transparent SR-IOV VF patch, so we should set up the IP to 
the ethX interface rather than the enP2p0s2 interface.