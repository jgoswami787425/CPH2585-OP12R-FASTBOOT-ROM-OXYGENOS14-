name: Build Custom ROM

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install Dependencies
        run: |
          sudo apt update
          sudo apt install -y git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib \
            libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev \
            libxml2-utils xsltproc unzip fontconfig python-is-python3

      - name: Sync ROM source
        run: |
          mkdir rom && cd rom
          repo init -u https://github.com/LineageOS/android.git -b lineage-21
          repo sync -j$(nproc)

      - name: Clone device, vendor, kernel trees
        run: |
          cd rom
          git clone https://github.com/YourName/device_xiaomi_codename device/xiaomi/codename
          git clone https://github.com/YourName/vendor_xiaomi_codename vendor/xiaomi/codename
          git clone https://github.com/YourName/kernel_xiaomi_codename kernel/xiaomi/codename

      - name: Build ROM
        run: |
          cd rom
          source build/envsetup.sh
          lunch lineage_codename-userdebug
          make bacon -j$(nproc)

      - name: Upload Output
        uses: actions/upload-artifact@v4
        with:
          name: rom-output
          path: rom/out/target/product/codename/*.zip
