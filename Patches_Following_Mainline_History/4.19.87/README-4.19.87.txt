HOWTO: These patches will enable accelerated networking on the upstrem stable 4.19.87 kernel for Linux VM on Azure.

1. Apply the below patches to make SR-IOV work with the 4.19.87 kernel


1.1 checkout the upstream 4.19.87 kernel and set HEAD to 4.19.87
git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
cd ~/linux-stable
git reset --hard v4.19.87
git clean -dxf

1.2 Apply the patches 
First change to top of the 4.19.87 kernel tree:
git am -k ~/azure-linux-kernel/4.19.87/*.patch

2. Confirm Mellanox card is seen

2.1 run "lspci" in this VM, and you will get:
0002:00:02.0 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]

2.2 The following command should give you similar output:
'sudo ifconfig -a' should show the VF interface starting with "en".
enP41257s1: flags=6211<UP,BROADCAST,RUNNING,SLAVE,MULTICAST>  mtu 1500

'ethtool -S eth0 | grep vf_' should show activity on the VF (virtual function):
 vf_rx_packets: 8941
     vf_rx_bytes: 65642040
     vf_tx_packets: 34601
     vf_tx_bytes: 9600462
     vf_tx_dropped: 0
     cpu0_vf_rx_packets: 1828
     cpu0_vf_rx_bytes: 15852293
     cpu0_vf_tx_packets: 5509
     cpu0_vf_tx_bytes: 1570508
     cpu1_vf_rx_packets: 2782

