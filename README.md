# zivm
This repository contains the source of Hamidreza Mohebbi's M.Sc. thesis project entitled "ZIVM: A Zero-Copy Inter-VM Communication Mechanism. 

Nowadays, virtualization technology (VT) becomes an integral part of academic and industrial researches in distributed systems like cluster and cloud computing, since it provides many substantial benefits for traditional, especially large scale computing environments. In this paper we focused on improving the communication performance of virtualized systems, especially communication between virtualized machines (VM)communication performance. We propose a distributed virtual shared memory nicknamed ZIVM providing near-native inter-VM communication performance preserving programming with minimum modification in existing codes,and transparency. ZIVM is implemented on a hosted VM upon KVM as virtual machine monitor (VMM) using C++ language. Evaluation results demonstrated the superiority of our proposed inter-VM communication approach in terms of lowest network latency and highest network throughput than other major inter-VM communication approaches,as well as than native communication between VMs.

The results of this research is published by the Computer and Information Science Journal on 2011. You can read this paper and cite it using the below link: http://www.ccsenet.org/journal/index.php/cis/article/view/6209

The src directory contains the source code used in this project. Here it is a short description of its exists four subdirectories:

-	banchmarks: benchmarks used for evaluations are in this directory.
-	guest-code: the source code of shared memory device driver and test programs are put it there.
-	Inter-vm-patch: core of shared memory system it is a patch that must be applied to qemu-kvm.
-	qemu-kvm: the qemu-kvm software that inter-vm-patch must applied to it.

Installation Instructions

For installation of inter-vm shared memory communications we need to do these steps:

1-	We are using linux as host operating system, and you must use a linux version (CentOS recommended)
2-	For using this inter-vm mechanism you need to compile your own version of the qemu-kvm executable (qemu-kvm-0.12.1.2):
cd qemu-kvm-0.12.1.2
patch –p1 inter-vm-patch/inter-vm-patch.diff

3-	Make and Install the modified version of kvm:

make
make install

4-	Create an virtual machine image and install the guest OS on it:

qemu-img create –f qcow2 disk.img 5G
qemu-system-x86_64 –cdrom /OSimg/fedora.iso disk.img –m 512

5-	Running guest os:

qemu-system-x86_64 disk.img –m 512

6-	install the guest driver on it (we are using fedora as guest OS), to do this we must get a linux kernel for the guest and compile it with kernel driver module (\guest-code\kernel_module):

tar -xjvf linux-2.6.25.tar.bz2 -C /usr/src
cd /usr/src
make menuconfig
make
make modules
cp kvm_ivshmem.c /usr/src/ linux-2.6.25/
make modules_install
make install

7-	Create a shared memory region using –ivshmem switch on host and boot the guest OS using new kernel that compiled in previous step:
qemu-system-x86_64 fedora12.img -m 512 -ivshmem shmfile 16 (create 16MB shared region)

8-	In this step we can read or write to shared region in host os (for example using Dumpsum test program)
Running this in the host:

./sum_host shmfile 16
And running this in the guest:
num =`cat /proc/devices | grep kvm_ivshmem | awk '{print $1}`
 mknod --mode=666 /dev/ivshmem c $num 0
./dump /dev/ivshmem 16
