Building
========

### macOS ⇩

```bash
mkdir -p $HOME/src && cd $HOME/src
git clone -q https://github.com/CloverHackyColor/CloverBootloader
cd CloverBootloader && git submodule update --init --recursive
cd OpenCorePkg && git checkout master && git pull
cd ..
./buildme
```

### Ubuntu 19.10 ⇩

```bash
sudo apt install build-essential nasm uuid-dev genisoimage
cd ~/Desktop/
git clone \--recurse https://github.com/CloverHackyColor/CloverBootloader.git
cd CloverBootloader
../edksetup.sh
./ebuild.sh -fr
```
* * *

[Back on top ↑](https://github.com/CloverHackyColor/Clover-Documentation?tab=readme-ov-file#welcome-to-the-cloverbootloader-documentation-clover-documentation-site)
