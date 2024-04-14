### Installation
```
uclient-fetch -P /etc/opkg/keys/ https://repo.openwrt.eleventwentytwo.com/c54a05f852ed0b42
echo 'src/gz upnatom_repo https://repo.openwrt.eleventwentytwo.com' >> /etc/opkg/customfeeds.conf
opkg update
```

### or add-to Image Builder
```
src/gz upnatom_repo https://repo.openwrt.eleventwentytwo.com to repositories.conf
```

### or self-build
```
cd $HOME
wget https://downloads.openwrt.org/releases/21.02.3/targets/x86/64/openwrt-sdk-21.02.3-x86-64_gcc-8.4.0_musl.Linux-x86_64.tar.xz
tar Jxz openwrt-sdk-21.02.3-x86-64_gcc-8.4.0_musl.Linux-x86_64.tar.xz
git clone https://git.openwrt.org/openwrt/openwrt.git
cd openwrt
git checkout v21.02.3
./scripts/feeds update -a
./scripts/feeds install -a
wget -O ./target/linux/x86/patches-5.4/600-bnx2x-warpcore-8727-2g5.patch https://raw.githubusercontent.com/JAMESMTL/snippets/master/bnx2x/patches/bnx2x_warpcore_8727_2_5g_sgmii_txfault.patch
wget -O .config https://downloads.openwrt.org/snapshots/targets/x86/64/config.buildinfo
make defconfig
make download
rsync -a $HOME/openwrt-sdk-21.02.3-x86-64_gcc-8.4.0_musl.Linux-x86_64/staging_dir/ ./staging_dir/
export PATH=$HOME/openwrt/staging_dir/host/bin:$PATH
make target/linux/compile
make package/kernel/linux/compile
usign -G -s ./key-build -p ./key-build.pub -c "OpenWrt usign key of up-n-atom"
usign -F -p key-build.pub
mkdir ./staging_dir/repo
cp ./bin/targets/x86/64/packages/kmod-bnx2x_5.4.188-1_x86_64.ipk ./staging_dir/repo/
./scripts/ipkg-make-index.sh ./staging_dir/repo 2>&1 > ./staging_dir/repo/Packages.manifest
grep -vE '^(Maintainer|LicenseFiles|Source|SourceName|Require|SourceDateEpoch)' ./staging_dir/repo/Packages.manifest > ./staging_dir/repo/Packages
usign -S -m ./staging_dir/repo/Packages -s ./key-build
gzip -fk ./staging_dir/repo/Packages
```
