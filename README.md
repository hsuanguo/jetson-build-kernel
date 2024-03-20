# jetson-build-kernel

This repository contains instrcutions on how to build the kernel for the NVIDIA Jetson devices.

Here we take Jetson Linux 35.5.0 on Jetson Xavier NX as an example.

## Prepare the toolchain

```bash
mkdir $HOME/l4t-gcc

```

Download the toolchain from the following [link](https://developer.nvidia.com/embedded/jetson-linux-r3550), and put it in the directory `$HOME/l4t-gcc`.

```bash
cd $HOME/l4t-gcc
tar xf aarch64--glibc--stable-final.tar.gz
CROSS_COMPILE_AARCH64_PATH=$HOME/l4t-gcc
```

## Prepare the kernel source

Download the BSP source code.

```bash
mkdir -p $HOME/l4t-sources
cd $HOME/l4t-sources
wget https://developer.nvidia.com/downloads/embedded/l4t/r35_release_v5.0/sources/public_sources.tbz2
tar -xvf public_sources.tbz2

```

### Build the kernel

Install the following packages.

```bash
sudo apt install flex bison libssl-dev
```

Build the kernel.


```bash
cd $HOME/l4t-sources
cd Linux_for_Tegra/source/public
mkdir output
./nvbuild.sh -o $HOME/l4t-sources/Linux_for_Tegra/source/public/output
```

### Customize the Device Tree

In my case, the carrier board(BOXER-8251AI), sdmmc 1 is connected to internal 16GB eMMC, and sdmmc 3 is connected to sdcard reader, while On the Jetson NX development board, sdmmc 1 is enabled and connected to sdcard reader, and sdmmc 3 is disabled. So I need to modify the device tree to enable sdmmc 3 so that I can use the sdcard reader on the carrier board.

Note: If you have a different jetson device, you will need to change the file names mentioned below accordingly.

We will need to modify 2 files: 

- `./hardware/nvidia/platform/t19x/jakku/kernel-dts/common/tegra194-fixed-regulator-p3668.dtsi`
- `./hardware/nvidia/platform/t19x/jakku/kernel-dts/common/tegra194-p3668-all-p3509-0000.dtsi`

In the file `tegra194-fixed-regulator-p3668.dtsi`, we need to add the following lines:

```dts
    p3668_vdd_sdmmc3_sw: regulator@103 {
        compatible = "regulator-fixed";
        reg = <103>;
        regulator-name = "vdd-sdmmc3-sw";
        regulator-min-microvolt = <3300000>;
        regulator-max-microvolt = <3300000>;
        gpio = <&tegra_main_gpio TEGRA194_AON_GPIO(CC, 0) 0>;
        enable-active-high;
    };
```

This will enable the regulator for sdmmc 3.

In the file `tegra194-p3668-all-p3509-0000.dtsi`, we need to add the following lines:

```dts
	sdmmc3: sdhci@3440000 {
		mmc-ocr-mask = <0x0>;
		cd-inverted;
		cd-gpios = <&tegra_main_gpio TEGRA194_MAIN_GPIO(Q, 2) 0>;
		nvidia,cd-wakeup-capable;
		mmc-ocr-mask = <0>;
		cd-inverted;
		vmmc-supply = <&p3668_vdd_sdmmc3_sw>;
		status = "okay";
	};
```

This will enable the sdmmc 3.

After modifying the device tree, we need to rebuild the kernel.

```bash
cd $HOME/l4t-sources/Linux_for_Tegra/source/public
./nvbuild.sh -o $HOME/l4t-sources/Linux_for_Tegra/source/public/output
```

You can find the updated device tree file `tegra194-p3668-all-p3509-0000.dtb` under the directory `$HOME/l4t-sources/Linux_for_Tegra/source/public/output/arch/arm64/boot/dts/nvidia`.