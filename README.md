# kernel-neko-void
- Workflows of Neko Void for builds kernels
<img width="1280" height="768" alt="image" src="https://github.com/user-attachments/assets/74c14b59-c9c8-4422-b834-cc789f64d738" />




#  MANUAL BUILD ON VOID:

# KERNEL ZEN 
```
xbps-install -Syu --repository=https://repo-de.voidlinux.org/current/ base-devel git bc kmod elfutils-devel bash cpio xz lz4 zstd flex bison openssl-devel curl pahole tar python3 patch wget rsync -y
git clone --depth 1 --branch v7.1.2-zen3 https://github.com/zen-kernel/zen-kernel.git linux-src
cd linux-src
wget -O .config https://raw.githubusercontent.com/void-linux/void-packages/master/srcpkgs/linux7.1/files/x86_64-dotconfig

# Evitar sufijo -dirty (por si acaso)
scripts/config --disable CONFIG_LOCALVERSION_AUTO
scripts/config --set-str CONFIG_LOCALVERSION "-neko"
# Optimizaciones
scripts/config --enable CONFIG_CC_OPTIMIZE_FOR_SIZE
scripts/config --enable CONFIG_MODULE_COMPRESS_ZSTD
          
# Desactivar debug masivo
scripts/config --disable CONFIG_DEBUG_INFO \
                         --disable CONFIG_DEBUG_INFO_DWARF_TOOLCHAIN_DEFAULT \
                         --disable CONFIG_DEBUG_INFO_REDUCED \
                         --disable CONFIG_DEBUG_INFO_BTF \
                         --disable CONFIG_DEBUG_VM \
                         --disable CONFIG_DEBUG_KERNEL \
                         --disable CONFIG_PROVE_LOCKING \
                         --disable CONFIG_LOCK_STAT \
                         --disable CONFIG_LATENCYTOP

# Desactivar firma de módulos
scripts/config --disable CONFIG_MODULE_SIG
scripts/config --set-str CONFIG_SYSTEM_TRUSTED_KEYS ""
          
# Actualizar configuración (incorpora nuevas opciones)
make olddefconfig
make -j$(nproc) prepare
make -j$(nproc) bzImage modules V=1 2>&1 | tee ../build.log
```

# KERNEL CACHYOS 
```bash
# DEPENDENCIES
xbps-install -Syu --repository=https://repo-de.voidlinux.org/current/ base-devel git bc kmod elfutils-devel bash cpio xz lz4 zstd flex bison openssl-devel curl pahole tar python3 patch wget rsync -y

# CLONE REPO
git clone --depth 1 --branch v7.1 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git linux-src
cd linux-src
sed -i 's/^EXTRAVERSION =.*/EXTRAVERSION = -cachy/' Makefile

# Apply config
wget -O .config https://codeberg.org/javiercplus/cachy-void/raw/branch/main/config
wget -O fixdep-largefile.patch https://github.com/void-linux/void-packages/raw/refs/heads/master/srcpkgs/linux7.1/patches/fixdep-largefile.patch
patch -p1 -N --batch < fixdep-largefile.patch
make olddefconfig
          
# Compile
make -j$(nproc) 
make -j$(nproc) bzImage modules 

# Install
make install
``` 
