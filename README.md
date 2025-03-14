# installing-NVIDIA-drivers

# https://github.com/NVIDIA/open-gpu-kernel-modules

This is a guide/list of problems I found while
installing open source drivers. I made it for myself
that's why stuff are written randomly, if it 
helps you then that's great.
It is old so most things changed and links might not
be available anymore.


## To show the module loaded in kernel
lsmod
lscpi -nvv

## Show installed modules
dkms status

## To install module you need a signed key:

echo -en "[ req ] \n\
default_bits = 4096 \n\
distinguished_name = req_distinguished_name\n\
prompt = no\n\
x509_extensions = myexts\n\

[ req_distinguished_name ]\n\
CN = Modules\n\
 \n\
[ myexts ] \n\
basicConstraints=critical,CA:FALSE \n\
keyUsage=digitalSignature \n\
subjectKeyIdentifier=hash \n\
authorityKeyIdentifier=keyid" > /tmp/x509.genkey

#### Then move the /tmp/x509.genkey file to your certs folder /lib/modules/$(uname -r)/build/certs

#### And use openssl to generate key:

openssl req -new -nodes -utf8 -sha512 -days 36500 -batch -x509 -config x509.genkey -outform DER -out signing_key.x509 -keyout signing_key.pem

### Remember to run "depmod" after modules_install

### Added /etc/modprobe.d/blacklist-nouveau.conf to balcklist nouveau drivers

# Secure boot causes problems with signed certs. Look at
#### dmesg when using modprobe nvidia, it will probably show
#### kernel lockdown
#### coorrection: not a problem

#This should let you see if there is a signature
hexdump -C kernel_module.ko | tail

# Install dkms, utils and cuda

## If device doesn't work check dmesg. Nvidia-smi wasn't working because modules nvidia.ko couldn't load in this early driver version. Enable NVreg_OpenRmEnableUnsupportedGpus=1 as kernel module parameter.

## Make sure all the drivers are the same version for example 515.48.07 and cuda is the latest version
#
## Links Official
#
### I've used deb(network) because the other options didn't have latest drivers:
#
# https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=deb_local
#
### Latest Ubuntu drivers:
#
# https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/
#
# Ubuntu official packages page doesn't have latest drivers. Use the nvidia dev link above.
#
# Links Unofficial
#
### Unable to determine the device handle for GPU 0000:09:00.0 (probably a cuda problem or NVreg parameter)
# https://stdworkflow.com/1479/solve-the-error-unable-to-determine-the-device-handle-for-gpu-0000-09-00-0
#
### Ubuntu packages (has latest drivers) 
# https://ubuntu.pkgs.org/20.04/cuda-amd64/nvidia-utils-515_515.48.07-0ubuntu1_amd64.deb.html


# DPKG Usage
#list drivers
dpkg -l | grep [driver name: ex. nvidia]
#install driver
sudo dpkg -i package_name.deb
#remove driver
sudo dpkg --purge package_name
#forced
sudo dpkg --purge --force-all package_name
#removing all nvidia drivers (not tested)
sudo apt-get --purge remove nvidia-*


# NEW DRIVER 570.124.04 Fedora41
## No open source kernel modules
## Secure Boot Disable because of signing key

1. remove all previous drivers:
``` 
  $ sudo dnf remove nvidia-driver\*
  $ dnf repoqeury --userinstalled | grep nvidia # remove every installed packet 
  # check if modules loaded then rm 
  $ lsmod | grep nvidia
  $ lscpi -nvv | grep nvidia
  $ reboot

```

2. install dependencies



3. go to nvidia official site and download driver .run
```
  $ sh ./NVIDIA*.run
  $ reboot 
```



# STUFF TO READ

### http://download.nvidia.com/XFree86/Linux-x86/361.45.11/README/installdriver.html
### http://download.nvidia.com/XFree86/Linux-x86/361.45.11/README/installdriver.html#modulesigning
### https://docs.oracle.com/en/learn/sboot-module/#prerequisites
### https://www.linuxjournal.com/content/take-control-your-pc-uefi-secure-boot
### https://docs.microsoft.com/it-it/windows-hardware/manufacture/desktop/windows-secure-boot-key-creation-and-management-guidance?view=windows-11
### https://www.kernel.org/doc/html/v4.15/admin-guide/module-signing.html
### https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/signing-kernel-modules-for-secure-boot_managing-monitoring-and-updating-the-kernel
### https://wiki.gentoo.org/wiki/Signed_kernel_module_support
### https://download.nvidia.com/XFree86/Linux-x86_64/510.39.01/README/gsp.html
