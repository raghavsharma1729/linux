# CMPE_283 Assignment2

## Contributors
Raghav Sharma
Sakshi Kekre

## Work Distribution

Raghav:
1. Setup GCP VM and build and install kernel, compilation of code, seting up github
2. Modified the code in the cpuid.c and vmx.c file to implement the functionality for returning the total number of all exit types in %eax when the register value for %eax is 0x4FFFFFFC for the CPUID instruction call
3. Setup the inner-VM inside a GCP Ubuntu instance.

Sakshi:
1. Modified the code in the cpuid.c and vmx.c file to implement the functionality for returning the high 32 bits in %ebx and the low 32 bits in %ecx of the total time spent in processing all exits, when the register value for %eax is 0x4FFFFFFD for the CPUID instruction call
2. Research about how to test the changes on the Inner-VM and figured out the dependencie. Wrote the tests.

Common
Pairing in debugging and running code

## setup steps

1. Setup gcloud account 

2. Loign to console

3. Create a VM 
   ( for this project i created with following features 
    machine type: n2-standard-2
    OS: Ubntu
    )
4. Connect to machine using either by Browser based SSH or your own machine

5. Install gcloud CLI
  <br> https://cloud.google.com/sdk/docs/install

6. Once intsalled initialize gcloud CLI
   <br> `gcloud init`

7. To enable nested virtualization
  <br> https://cloud.google.com/compute/docs/instances/nested-virtualization/enabling

8. Run the following command to create VM
```
gcloud compute instances create instance-1 --project=shaped-manifest-367908 --zone=us-west1-b --machine-type=n2-standard-16 --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=22441749854-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --create-disk=auto-delete=yes,boot=yes,device-name=instance-1,image=projects/ubuntu-os-cloud/global/images/ubuntu-1804-bionic-v20221201,mode=rw,size=250,type=projects/shaped-manifest-367908/zones/us-west1-b/diskTypes/pd-ssd --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any --min-cpu-platform "Intel Cascade Lake" --enable-nested-virtualization
```

9. To check if nested virtualization is enabled, connect to VM and run the following command,it should any number other than 0
    <br> `grep -cw vmx /proc/cpuinfo`

10. Fork the linux  https://github.com/torvalds/linux

11. Connect to VM 

12. Update apt-get
``` sudo apt-get update ```

13. Clone the repository
```
git clone  git@github.com:raghavsharma1729/linux.git
```

14. Setup git ssh as well 
```
https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
```
if facing error The authenticity of host 'github.com (140.82.113.4)' can't be established then run
```ssh-keyscan github.com >> ~/.ssh/known_hosts```

15. Install dependency:
```sudo apt-get install gcc make build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache bison flex libelf-dev```

16. Find the configuration of current OS and use that to build linux kernel
 ``` cd linux ```
 ``` uname -r ```
 ``` cp /boot/config-5.4.0-1096-gcp  .config ```
 ``` make oldconfig ```
 just keep enter pressed for newer configuration asked in above command
 
 17. Prepare the linux
 ``` make prepare ```
 
18.  Build all the modules
 ```  make -j 8 modules ```
run it a couple of times if stops and if issue regarding some certificiation then run following and run again
 ``` 
  scripts/config --disable SYSTEM_TRUSTED_KEYS 
  scripts/config --disable SYSTEM_REVOCATION_KEYS 
  ```
  
 19. Build kernel:
``` make -j 8 ``` 

20. Package the build
``` sudo make INSTALL_MOD_STRIP=1 modules_install ```

21. Install built linux kernel 
``` sudo make install ```

22. Reboot the kernel
``` sudo reboot ```

23. Connect back to VM and check the version it should be updated

24. Now do the required changes in the code for adding a new leafnode for CPUID 

25. Rebuild and install the modules:
 ``` 
    make -j 8 modules
    sudo lsmod | grep kvm
    sudo rmmod kvm_intel
    sudo rmmod kvm
    sudo modprobe kvm
    sudo modprobe kvm_intel
    sudo lsmod | grep kvm
   ```
    
26. Install tools for KVM,check kvm-ok, and reboot:

 ``` 
    sudo apt-get install cpu-checker    
    sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
    sudo kvm-ok
    sudo reboot 
   ```
    
 27.  Steps to Do for seting up inner VM
 
 ```
  sudo adduser 'spartan' libvirt
  sudo adduser 'spartan' kvm
  sudo systemctl status libvirtd (this should show status running)
 ```

 28. Set up Chrome Remote Desktop for Linux on Compute Engine : https://cloud.google.com/architecture/chrome-desktop-remote-on-compute-engine

29.  To enable GUI provided by GCP Enable Display Device for your VM

30. Create our inner VM using virt-manager: https://www.tecmint.com/create-virtual-machines-in-kvm-using-virt-manager/

31. On the inner-VM install dependency
``` 
    sudo apt-get update
    sudo apt-get install build-essential
    sudo apt-get install cpuid
 ```
    
32. Test your Kernel functionality with your written tests
