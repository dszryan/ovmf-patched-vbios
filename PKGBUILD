# Maintainer: dszryan
pkgname=edk2-ovmf-vbios-1050-ti-max-q-10de-1c8c
pkgver=edk2.stable201905
epoch=1
pkgrel=1
arch=('x86_64')
pkgdesc="Tianocore UEFI firmware for qemu with vbios patched in."
url="http://sourceforge.net/apps/mediawiki/tianocore/index.php?title=EDK2"
license=('custom')
makedepends=('git' 'gcc5' 'python2' 'iasl' 'nasm' 'subversion' 'perl-libwww' 'vim' 'dos2unix')
source=("edk2::git+https://github.com/tianocore/edk2#tag=${pkgver/\./-}"
        "file://$startdir/vBIOS.bin"
        "file://$startdir/70-edk2-ovmf-patched-x86_64.json"
        "file://$startdir/IntelIGD.patch"
        "file://$startdir/QemuFwCfgAcpi.c.patch"
        "file://$startdir/ssdt.asl")
sha256sums=('SKIP'
            'SKIP'
            'd289eb3ccda1e431d59622956b87dfb2c9d384140946dba643d8e95d0030385e'
            '59954884e0f565a4569cfd58995a748ddefe48f1b6b2f42928cd8db506699d2a'
            'a1717291ade527ddc7221c9448c15fe187b30e5fe831a2570d5ead851e57dd03'
            '10a8758c07a675134ea1a66eed26513ec38991832a7087ee3724a6a6ebee7c4b')
validpgpkeys=(8657ABB260F056B1E5190839D9C4D26D0E604491)
options=(!makeflags)
_toolchain_opt=GCC5

#pkgver() {
#  cd "${srcdir}"/edk2
#  printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
#}

prepare() {
  cd "${srcdir}"
  xxd -i vBIOS.bin vrom.h
  sed -i 's/vBIOS_bin/VROM_BIN/g; s/_len/_LEN/g' vrom.h
  rom_len=$(grep VROM_BIN_LEN vrom.h | cut -d' ' -f5 | sed 's/;//g')
  sed -i "s/103936/$rom_len/g" ssdt.asl
  iasl -f ssdt.asl
  xxd -c1 Ssdt.aml | tail -n +37 | cut -f2 -d' ' | paste -sd' ' | sed 's/ //g' | xxd -r -p > vrom_table.aml
  xxd -i vrom_table.aml | sed 's/vrom_table_aml/vrom_table/g' > vrom_table.h

  mv "$srcdir/vrom.h" "${srcdir}/edk2/OvmfPkg/AcpiPlatformDxe/"
  mv "$srcdir/vrom_table.h" "${srcdir}/edk2/OvmfPkg/AcpiPlatformDxe/"

  cd "$srcdir/edk2"
  git submodule update --init --recursive

  dos2unix OvmfPkg/AcpiPlatformDxe/QemuFwCfgAcpi.c
  patch -N -p1 < "$srcdir/QemuFwCfgAcpi.c.patch"
  unix2dos OvmfPkg/AcpiPlatformDxe/QemuFwCfgAcpi.c

#  dos2unix OvmfPkg/IgdAssignmentDxe/IgdAssignment.c OvmfPkg/IgdAssignmentDxe/IgdAssignment.inf OvmfPkg/Include/IndustryStandard/AssignedIgd.h OvmfPkg/OvmfPkgIa32.dsc
#  patch -N -p1 < "$srcdir/IntelIGD.patch"
#  unix2dos OvmfPkg/IgdAssignmentDxe/IgdAssignment.c OvmfPkg/IgdAssignmentDxe/IgdAssignment.inf OvmfPkg/Include/IndustryStandard/AssignedIgd.h OvmfPkg/OvmfPkgIa32.dsc

  cd "${srcdir}"
  # edk2 uses python everywhere, but expects python2
  mkdir -p bin
  ln -sf /usr/bin/python2 bin/python
  ln -sf /usr/bin/gcc-5   bin/gcc
  ln -sf /usr/bin/g++-5	  bin/g++
}

build() {
  export PATH="${srcdir}/bin:$PATH"
  cd "${srcdir}/"edk2
  make -C BaseTools
  export EDK_TOOLS_PATH="${srcdir}"/edk2/BaseTools
  . edksetup.sh BaseTools

  # Set RELEASE target, toolchain and number of build threads
  sed "s|^TARGET[ ]*=.*|TARGET = RELEASE|; \
       s|TOOL_CHAIN_TAG[ ]*=.*|TOOL_CHAIN_TAG = ${_toolchain_opt}|; \
       s|MAX_CONCURRENT_THREAD_NUMBER[ ]*=.*|MAX_CONCURRENT_THREAD_NUMBER = $(nproc)|;" -i Conf/target.txt

  # Build OVMF for x64
  sed "s|^TARGET_ARCH[ ]*=.*|TARGET_ARCH = X64|; \
       s|^ACTIVE_PLATFORM[ ]*=.*|ACTIVE_PLATFORM = OvmfPkg/OvmfPkgX64.dsc|;" -i Conf/target.txt

  ./BaseTools/BinWrappers/PosixLike/build
}

package() {
  install -D -m644 "${srcdir}"/edk2/Build/OvmfX64/RELEASE_${_toolchain_opt}/FV/OVMF_CODE.fd "${pkgdir}"/usr/share/edk2-ovmf/x64/OVMF_CODE_PATCHED_1050-ti-max-q-10de-1c8c.fd
  install -D -m644 "${srcdir}"/edk2/Build/OvmfX64/RELEASE_${_toolchain_opt}/FV/OVMF_VARS.fd "${pkgdir}"/usr/share/edk2-ovmf/x64/OVMF_VARS_PATCHED_1050-ti-max-q-10de-1c8c.fd
  install -D -m644 "${srcdir}"/70-edk2-ovmf-patched-x86_64.json                             "${pkgdir}"/usr/share/qemu/firmware/70-edk2-ovmf-patched-x86_64.json
  install -D -m644 "${srcdir}"/edk2/OvmfPkg/License.txt                                     "${pkgdir}"/usr/share/licenses/ovmf/License.txt
}
