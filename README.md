# Compile-Lustre-client-with-Generic-O2IB-on-Fedora-39
Compile Lustre client(version ```2.16.61```) with generic ```o2ib``` on Fedora 39(``kernel 6.11.9-100.fc39``), ``Aarch64``, ``without MLNX_OFED``.

1. Download the source file from https://github.com/lustre/lustre-release, or from WhamCloud.
2. Ensure that you have Infiniband tool packages  installed: ```infiniband-diags perftest qperf  opensm, rdma-core-devel```
   The generic Infiniband driver comes with the kernel: ```kernel-modules-6.11.9-100.fc39```
   The needed ```o2ib``` header files comes with package ```rdma-core-devel```, the location of these files is: ```/usr/include/infiniband```
3. Untar the source file, go to the lustre folder:

   ```
   ./configure --disable-server --enable-client --disable-tests --enable-gss=no --disable-gss-keyring --with-o2ib=/usr/include/infiniband  --with-linux=/usr/src/kernels/6.11.9-100.fc39.aarch64
   ```
   If this goes smoothly, we can go to compile and make the rpm files. Before that, we need to make some modifications to let it work:

4. Modify ```lustre.spec```  (please note, each time you run "```./configure```" command, it will overwite ```lustre.spec``` file using the ```lustre.spec.in```. So you can change it there too)

         
         #Change line 748 to:
         WITH_O2IB="--with-o2ib=/usr/include/infiniband/"
         # comment it's following block of %if-%endif lines
         # from line 749 to 764.

#### These lines in ```lustre.spec``` are hardcoded to check ```MLNX_OFED``` package, which we do not have. The ```MLNX_OFED``` does not support ```Fedora 39```, ```kernel 6.11``` at present.

5. Modify 2 KMP files:
  
        #vi rpm/kmp-lustre.preamble
        #vi rpm/kmp-lnet-o2iblnd.preamble  
        # Comment the following line:
   
        # Feng BuildRequires: mlnx-ofa_kernel-devel
  
Now:        
```
make rpms
```


6. Install the Lustre client
```
rpm -ivh --nodeps  kmod-lustre-client-2.16.61-1.fc39.aarch64.rpm lustre-client-2.16.61-1.fc39.aarch64.rpm
```
Here we use the "```--nodeps```", otherwise it will fail due to error:
```
 Problem 1: conflicting requests
  - nothing provides /usr/sbin/weak-modules needed by kmod-lustre-client-2.16.61-1.fc39.aarch64 from @commandline
```
Which we do not need, since the "```weak-modules```" is for ```DKMS```, and here we are using ```KMOD```(Kenel module).

```
modprod lustre
```

Make sure the IB device works fine, like IP over IB is set, etc. And module "```ko2iblnd```" has been loaded into kernel as well.

```
lctl list_nids
192.168.0.100@o2ib

mount -t lustre 192.168.0.100@o2ib:/lustre /mnt/

```
