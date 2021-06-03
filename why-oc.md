# Tại sao nên chọn OpenCore

Phần này tóm tắt ngắn gọn về lý do tại sao cộng đồng lại chuyển đổi sang OpenCore và nhằm mục đích xóa tan một vài lầm tưởng phổ biến trong cộng đồng. Ai không cần thì có thể bỏ qua trang này.

* [Tại sao nên chọn OpenCore](#tai-sao-nen-chon-opencore)
  * Các tính năng của OpenCore
  * Nhiều phần mềm hỗ trợ
  * Kext injection
* [Khiếm khuyết của OpenCore](#khiem-khuyet-cua-opencore)
* [Lầm tưởng chung](#common-myths)
  * Có phải OpenCore không ổn định vì nó là bản beta?
  * Có phải OpenCore luôn inject SMBIOS và ACPI vào các hệ điều hành (OSes) khác phải không?
  * Có phải OpenCore yêu cầu phải cài mới, cài vanilla không?
  * Có phải OpenCore chỉ hỗ trợ rất ít phiên bản (versions) macOS?

## Tính năng của OpenCore

* Hỗ trợ nhiều OSes hơn!
  * OpenCore hiện hỗ trợ nhiều versions của OS X and macOS một cách tự nhiên mà không cần các vụ hack khó khăn của Clover và Chameleon
  * OpenCore hỗ trợ các OSes từ 10.4, Tiger trở lại đây, thâm chs là phiên bản mới nhất 11, Big Sur!
* Trung bình, các hệ thống chạy OpenCore boot nhanh hơn Clover vì ít các patch không cần thiết hơn
* Tính ổn định tổng thể tốt hơn vì các bản patches chính xác hơn nhiều
  * [macOS 10.15.4 update](https://www.reddit.com/r/hackintosh/comments/fo9bfv/macos_10154_update/)
  * AMD OSX patches không cần phải được cập với mọi bản Secủity update nhỏ
* Tổng quan thì bảo mật tốt hơn ở nhiều mặt:
  * Không cần tắt System Integrity Protection (SIP)
  * Hỗ trợ FileVault 2
  * [Vaulting](https://viopencore.github.io/OpenCore-Post-Install/universal/security.html#Vault) giúp tạo "ảnh chụp" của EFI để ngăn những sửa đổi không mong muốn
  * Thực sư hỗ trợ secure-boot
    * Both UEFI and Apple's variant
* BootCamp switching and boot device selection are supported by reading NVRAM variables set by Startup Disk, just like a real Mac.
* Supports boot hotkey via `boot.efi` - hold `Option` or `ESC` at startup to choose a boot device, `Cmd+R` to enter Recovery or `Cmd+Opt+P+R` to reset NVRAM.

## Nhiều phần mềm hỗ trợ

Đây chính là nguyên nhân chính làm cho ai đó chuyển qua OpenCore từ các bootloaders khác chính là vì OpenCore có nhiều phần mềm hỗ trợ hơn:

* Kexts không còn được test cho Clover:
  * Gặp lỗi với 1 kext? Một developers bao gồm nhóm [Acidanthera](https://github.com/acidanthera) (người viết hầu hết những kexts yêu thích nhất của bạn) sẽ không hỗ trợ trừ khi bạn sử dụng OpenCore
* Một số firmware drivers đã được gộp vào OpenCore:
  * [APFS Support](https://github.com/acidanthera/AppleSupportPkg)
  * [FileVault support](https://github.com/acidanthera/AppleSupportPkg)
  * [Firmware patches](https://github.com/acidanthera/AptioFixPkg)
* [AMD OSX patches](https://github.com/AMD-OSX/AMD_Vanilla/tree/opencore):
  * Bạn đang sử máy chạy AMD? Kernel patches phục vụ việc boot macOS không còn hỗ trợ Clover – chúng chỉ còn hỗ trợ OpenCore.

## Kext Injection

Để hiểu rõ hơn hệ thống kext injection của OpenCore, chúng ta trước tiên nên tìm hiểu cách Clover hoạt động:

1. Patches SIP tắt đi
2. Patches để kích hoạt zombie code của XNU để inject kext injection
3. Patches race condition with kext injection
4. Injects kexts
5. Patches SIP mở trở lại

Things to note with Clover's method:

* Sử dụng zombie code của XNU (đã không được sử dụng từ 10.7, điều này thật là ấn tượng khi Apple đã không loại bỏ code này
  * OS updates thường phá vỡ patch này, gần nhất là 10.14.4 và 10.15
* Tắt SIP and attempts to mở nỏ lại, don't think much needs to be said
* Likely to bị thất bại bại macOS 11.0 (Big Sur)
* Hỗ trợ tất cả OS X đến 10.5

Bây giờ thì hãy xem phương pháp của OpenCore:

1. Takes existing prelinked kernel and kexts ready to inject
2. Rebuilds the cache in the EFI environment with the new kexts
3. Adds this new cache in

Những điều cần lưu ý với phương pháp của OpenCore:

* OS agnostic as the prelinked kernel format has stayed the same since 10.6 (v2), far harder to break support.
  * OpenCore also supports prelinked kernel (v1, found in 10.4 and 10.5), cacheless, Mkext and KernelCollections, meaning it also has proper support for all Intel versions of OS X/macOS
* Far better stability as there is far less patching involved

# Khiếm khuyết của OpenCore

Hầu hết các Clover's functionality is actually supported in OpenCore in the form of some quirk, however when transitioning you should pay close attention to OpenCore's missing features as this may or may not affect yourself:

* Does not support booting MBR-based operating systems
  * Work around is to chain-load rEFInd once in OpenCore
* Does not support UEFI-based VBIOS patching
  * This can be done in macOS however
* Does not support automatic DeviceProperty injection for legacy GPUs
  * ie. InjectIntel, InjectNvidia and InjectAti
  * This can be done manually however: [GPU patching](https://viopencore.github.io/OpenCore-Post-Install/gpu-patching/)
* Does not support IRQ conflict patching
  * Can be resolved with [SSDTTime](https://github.com/corpnewt/SSDTTime)
* Does not support P and C state generation for older CPUs
* Does not support Target Bridge ACPI patching
* Does not support Hardware UUID Injection
* Does not support auto-detection for many Linux bootloader
  * Can be resolved by adding an entry in `BlessOverride`
* Does not support many of Clover's XCPM patches
  * ie. Ivy Bridge XCPM patches
* Does not support hiding specific drives
* Does not support changing settings within OpenCore's menu
* Does not patch PCIRoot UID value
* Does not support macOS-only ACPI injection and patching

# Common Myths

## Có phải OpenCore không ổn định vì nó là bản beta?

Câu trả lời ngắn: Không

Câu trả lời dài: Không

Số phiên bản của OpenCore không tái hiện chất lượng của dự án. Thay vào đó, it's more of a way to see the stepping stones of the project. Acidanthera still has much they'd like to do with the project including overall refinement and more feature support.

For example, OpenCore goes through proper security audits to ensure it complies with UEFI Secure Boot, and is the only Hackintosh bootloader to undergo these rigorous reviews and have such support.

Phiên bản 0.6.1 vốn dĩ được thiết kê là bản ra mắt chính thức OpenCore vì nó có UEFI/Apple Secure Boot ổn định, và đánh dấu tròn 1 năm OpenCore được ra mắt như là bộ công cụ công khai. However, due to circumstances around macOS Big Sur and the rewriting of OpenCore's prelinker to support it, it was decided to push off 1.0.0 for another year.

Bản đồ các giai đoạn:

* 2019: Năm của Beta
* 2020: Năm của Secure Boot
* 2021: Năm của sự sàng lọc

So please do not see the version number as a hindrance, instead as something to look forward to.

## Does OpenCore always inject SMBIOS and ACPI data into other OSes

By default, OpenCore will assume that all OSes should be treated equally in regards to ACPI and SMBIOS information. The reason for this thinking consists of three parts:

* This allows for proper multiboot support, like with [BootCamp](https://viopencore.github.io/OpenCore-Post-Install/multiboot/bootcamp.html)
* Avoids poorly made DSDTs and encourages proper ACPI practices
* Avoids edge cases where info is injected several times, commonly seen with Clover
  * i.e. How would you handle SMBIOS and ACPI data injection once you booted boot.efi, but then get kicked out? The changes are already in memory and so trying to undo them can be quite dangerous. This is why Clover's method is frowned upon.

However, there are quirks in OpenCore that allow for SMBIOS injection to be macOS-limited by patching where macOS reads SMBIOS info from. The `CustomSMIOSGuid` quirk with `CustomSMBIOSMode` set to `Custom` can break in the future and so we only recommend this option in the event of certain software breaking in other OSes. For best stability, please disable these quirks.

## Does OpenCore require a fresh install

Not at all in the event you have a "Vanilla" installation – what this refers to is whether the OS has tampered in any way, such as installing 3rd party kexts into the system volume or other unsupported modifications by Apple. When your system has been heavily tampered with, either by you or 3rd party utilities like Hackintool, we recommend a fresh install to avoid any potential issues.

Special note for Clover users: please reset your NVRAM when installing with OpenCore. Many of Clover variables can conflict with OpenCore and macOS.

* Note: Thinkpad laptops are known to be semi-bricked after an NVRAM reset in OpenCore, we recommend resetting NVRAM by updating the BIOS on these machines.

## Có phải OpenCore chỉ hỗ trợ rất ít phiên bản (versions) macOS?

Bắt đầu từ phiên bản OpenCore 0.6.2, bạn có thể khởi động bất kỳ phiên bản Intel của macOS cho đến phiên bản OS X 10.4! Sự hỗ trợ thích hợp tuy nhiên sẽ tuỳ vào phần cứng của bạn, nên hãy tự xác nhận: [Hardware Limitations](macos-limits.md)

::: details macOS Install Gallery

Acidanthera has tested many versions, and I myself have run many versions of OS X on my old HP DC 7900 (Core2 Quad Q8300). Here's just a small gallery of what I've tested:

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.4-Tiger.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.5-Leopard.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.6-Snow-Loepard.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.7-Lion.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.8-MountainLion.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.9-Mavericks.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.10-Yosemite.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.12-Sierra.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.13-HighSierra.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/10.15-Catalina.png)

![](./images/installer-guide/legacy-mac-install-md/dumpster/11-Big-Sur.png)

:::

## Does OpenCore support older hardware

As of right now, the majority of Intel hardware is supported so long as the OS itself does! However please refer to the [Hardware Limitations page](macos-limits.md) for more info on what hardware is supported in what versions of OS X/macOS.

Currently, Intel's Yonah and newer series CPUs have been tested properly with OpenCore.

## Does OpenCore support Windows/Linux booting

OpenCore works in the same fashion as any other boot loader, so it respects other OSes the same way. For any OSes where their bootloader has an irregular path or name, you can simply add it to the BlessOverride section.

## Legality of Hackintoshing

Where hackintoshing sits is in a legal grey area, mainly that while this is not illegal we are in fact breaking the EULA. The reason this is not illegal:

* We are downloading macOS from [Apple's servers directly](https://github.com/acidanthera/OpenCorePkg/blob/0.6.9/Utilities/macrecovery/macrecovery.py#L125)
* We are doing this as a non-profit origination for teaching and personal use
  * People who plan to use their Hackintosh for work or want to resell them should refer to the [Psystar case](https://en.wikipedia.org/wiki/Psystar_Corporation) and their regional laws

While the EULA states that macOS should only be installed on real Macs or virtual machines running on genuine Macs ([sections 2B-i and 2B-iii](https://www.apple.com/legal/sla/docs/macOSBigSur.pdf)), there is no enforceable law that outright bans this. However, sites that repackage and modify macOS installers do potentially risk the issue of [DMCA takedowns](https://en.wikipedia.org/wiki/Digital_Millennium_Copyright_Act) and such.

* **Note**: This is not legal advice, so please make the proper assessments yourself and discuss with your lawyers if you have any concerns.

## Does macOS support Nvidia GPUs

Due to issues revolving around Nvidia support in newer versions of macOS, many users have somehow come to the conclusion that macOS never supported Nvidia GPUs and don't at this point. However, Apple actually still maintains and supports Macs with Nvidia GPUs in their latest OS, like the 2013 MacBook Pro models with Kepler GPUs.

The main issue has to do with any newer Nvidia GPUs, as Apple stopped shipping machines with them and thus they never had official OS support from Apple. Instead, users had to rely on Nvidia for 3rd party drivers. Due to issues with Apple's newly introduced Secure Boot, they could no longer support the Web Drivers and thus Nvidia couldn't publish them for newer platforms limiting them to mac OS 10.13, High Sierra.

For more info on OS support, see here: [GPU Buyers Guide](https://viopencore.github.io/GPU-Buyers-Guide/)
