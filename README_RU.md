# chia-plotter (pipelined multi-threaded)

Это новая реализация плоттера Chia, который предназначен для многопоточности,
пример тому, как работает GPU, только «ядра» - это обычные программные потоки CPU.

В результате этот плоттер способен полностью использовать максимум пропускной способности устройства хранения,
просто путем увеличения количества «ядер», то есть. потоков.

Русская поддержка [MIGPOOL.COM](https://migpool.com/)

## Использование

Присоединяйтесь к Discord для поддержки: https://discord.gg/JfHDJ3dtkc

```
Для <poolkey> и <farmerkey> смотрите вывод команды `chia keys show`.
Контракт для пулов, укажите <contract> адрес через аргумент -c вместо <poolkey>, смотрите `chia plotnft show`.
<tmpdir> Необходимо около 220 GiB пространства, она будет обрабатывать около 25% всех записей. (Пример: './', '/mnt/tmp/')
<tmpdir2> Нужно около 110 GiB пространства, в идеале - это RAM диск (ОЗУ), будет обрабатывать около 75% всех записей.
Объединенный (tmpdir + tmpdir2) Использование при пике диска составляет менее 256 GiB.
В случае <count>! = 1 вы можете нажать Ctrl-C для корректного завершения после завершения текущего графика, 
или дважды нажмите Ctrl-C для немедленного завершения.

Использование:
  chia_plot [ОПЦИИ...]

  -k, --size arg       K size (стандартно = 32, k <= 32)
  -x, --port arg       Сетевой порт (стандартно = 8444, chives = 9699)
  -n, --count arg      Количество участков для создания (стандартно = 1, -1 = infinite)
  -r, --threads arg    Количество потоков (стандартно = 4)
  -u, --buckets arg    Количество сегментов (стандартно = 256)
  -v, --buckets3 arg   Количество сегментов при фазе 3+4 (стандартно = buckets)
  -t, --tmpdir arg     Временный каталог, нужно ~220 GiB (стандартно = $PWD)
  -2, --tmpdir2 arg    Временный каталог 2, нужно ~110 GiB [RAM] (стандартно = <tmpdir>)
  -d, --finaldir arg   Окончательный каталог (стандартно = <tmpdir>)
  -w, --waitforcopy    Ждать, пока скопируется предыдущий участок перед началом создания следующего
  -p, --poolkey arg    Публичный ключ пула (48 bytes)
  -c, --contract arg   Адрес контракта пула (62 chars)
  -f, --farmerkey arg  Открытый ключ фермера (48 bytes)
  -G, --tmptoggle      Альтернативные tmpdir/tmpdir2 (стандартно = false)
  -K, --rmulti2 arg    Многопоточность для P2 (стандартно = 1)
      --help           Показать справку
```

Не забудьте активировать `<threads>`, если у вас много ядер, по умолчанию - 4.
В зависимости от фазы больше потоков будут запущены, настройка является лишь множителем.

Использование ОЗУ зависит от `<threads>` и `<buckets>`.
С новым значением по умолчанию в 256 сегментов это не более 0.5 ГБ на поток.

`-G` опция будет чередовать TEMP Dirs, используемые во время создания, чтобы дать каждому, TMPDIR и TMPDIR2, равноправное использование. Первое создание плота будет использовать TMPDIR и TMPDIR2, как и ожидалось. Следующий запуск, если -n равен 2 или более, поменятся на TMPDIR2 и TMPDIR. Следующее создание плота снова сводится к TMPDIR и TMPDIR2. Это произойдет до тех пор, пока число созданных участков не будет достигнуто или до остановки.

### Настройка диска RAM на Linux
`sudo mount -t tmpfs -o size=110G tmpfs /mnt/ram/`

Примечание: Минимальный минимум системы 128 GiB требуется для RAM Disk.

## Как поддержать первичного разработчика

XCH: xch1w5c2vv5ak08pczeph7tp5xmkl5762pdf3pyjkg9z4ks4ed55j3psgay0zh

XFX: xfx1succfn2z3uwmq50ukztjanrvs9kw294mqn4lv22rk6tzmcu7e2xsyxyaa5

XCC: xcc16j65y35fs8u289nq6krcyehsmp5eqd4we493rxf36pg7eymcqrqqltsrat

ETH-ERC20: 0x97057cdf529867838d2a1f7f23ba62456764e0cd

LTC: MNUnszsX2srv5EJpu9YYHAXb19MqUpuBjD

BTC: 15GSE5ymStxXMvJ58hyosEVm4FXFxUyJZg

## Результаты

На двойном процессоре Xeon® E5-2650v2 <span>@</span> 2,60 ГГц R720 с 256 ГБ ОЗУ и 3x800 ГБ SATA SSD RAID0 с использованием tmpfs 110 ГБ для `<tmpdir2>`:

<details>
  <summary>Нажмите, чтобы расширить</summary>
  
  ```
  Number of Threads: 16
  Number of Buckets: 2^8 (256)
  Working Directory:   /mnt/tmp3/chia/tmp/ 
  Working Directory 2: /mnt/tmp3/chia/tmp/ram/
  [P1] Table 1 took 17.2488 sec
  [P1] Table 2 took 145.011 sec, found 4294911201 matches
  [P1] Table 3 took 170.86 sec, found 4294940789 matches
  [P1] Table 4 took 203.713 sec, found 4294874801 matches
  [P1] Table 5 took 201.346 sec, found 4294830453 matches
  [P1] Table 6 took 195.928 sec, found 4294681297 matches
  [P1] Table 7 took 158.053 sec, found 4294486972 matches
  Phase 1 took 1092.2 sec
  [P2] max_table_size = 4294967296
  [P2] Table 7 scan took 15.5542 sec
  [P2] Table 7 rewrite took 37.7806 sec, dropped 0 entries (0 %)
  [P2] Table 6 scan took 46.7014 sec
  [P2] Table 6 rewrite took 65.7315 sec, dropped 581295425 entries (13.5352 %)
  [P2] Table 5 scan took 45.4663 sec
  [P2] Table 5 rewrite took 61.9683 sec, dropped 761999997 entries (17.7423 %)
  [P2] Table 4 scan took 44.8217 sec
  [P2] Table 4 rewrite took 61.36 sec, dropped 828847725 entries (19.2985 %)
  [P2] Table 3 scan took 44.9121 sec
  [P2] Table 3 rewrite took 61.5872 sec, dropped 855110820 entries (19.9097 %)
  [P2] Table 2 scan took 43.641 sec
  [P2] Table 2 rewrite took 59.6939 sec, dropped 865543167 entries (20.1528 %)
  Phase 2 took 620.488 sec
  Wrote plot header with 268 bytes
  [P3-1] Table 2 took 73.1018 sec, wrote 3429368034 right entries
  [P3-2] Table 2 took 42.3999 sec, wrote 3429368034 left entries, 3429368034 final
  [P3-1] Table 3 took 68.9318 sec, wrote 3439829969 right entries
  [P3-2] Table 3 took 43.8179 sec, wrote 3439829969 left entries, 3439829969 final
  [P3-1] Table 4 took 71.3236 sec, wrote 3466027076 right entries
  [P3-2] Table 4 took 46.2887 sec, wrote 3466027076 left entries, 3466027076 final
  [P3-1] Table 5 took 70.6369 sec, wrote 3532830456 right entries
  [P3-2] Table 5 took 45.5857 sec, wrote 3532830456 left entries, 3532830456 final
  [P3-1] Table 6 took 75.8534 sec, wrote 3713385872 right entries
  [P3-2] Table 6 took 48.8266 sec, wrote 3713385872 left entries, 3713385872 final
  [P3-1] Table 7 took 83.2586 sec, wrote 4294486972 right entries
  [P3-2] Table 7 took 56.3803 sec, wrote 4294486972 left entries, 4294486972 final
  Phase 3 took 733.323 sec, wrote 21875928379 entries to final plot
  [P4] Starting to write C1 and C3 tables  
  [P4] Finished writing C1 and C3 tables   
  [P4] Writing C2 table
  [P4] Finished writing C2 table
  Phase 4 took 84.6697 sec, final plot size is 108828428322 bytes
  Total plot creation time was 2530.76 sec 
  ```
</details>

## Как проверить

Чтобы убедиться, что участки действительны `ProofOfSpace`, вы можете использовать инструмент [chiapos](https://github.com/Chia-Network/chiapos):

```bash
git clone https://github.com/Chia-Network/chiapos.git
cd chiapos && mkdir build && cd build && cmake .. && make -j8
./ProofOfSpace check -f plot-k32-???.plot [num_iterations]
```

## Как обновить до последней версии

```bash
cd chia-plotter
git checkout master
git pull
git submodule update --init
./make_devel.sh
```

## Зависимости

- cmake (>=3.14)
- libsodium-dev

## Установка

<details>
  <summary>Windows</summary>
  
  Бинарные файлы, созданные [MIGPOOL] (https://github.com/MIGPOOL), можно найти здесь
https://github.com/MIGPOOL/chia-plotter/releases

</details>

<details>
  <summary>Arch Linux</summary>

  First, install dependencies from pacman:
  ```bash
  sudo pacman -S cmake libsodium gmp gcc11
  ```
  Then, clone and compile the project:
  ```bash
  # Checkout the source and install
  git clone https://github.com/MIGPOOL/chia-plotter.git
  cd chia-plotter

  git submodule update --init
  ./make_devel.sh
  ./build/chia_plot --help
  ```
</details>

<details>
  <summary>CentOS 7</summary>
  
  ```bash
  git clone https://github.com/MIGPOOL/chia-plotter.git
  cd chia-plotter

  git submodule update --init
  sudo yum install epel-release -y
  sudo yum install cmake3 libsodium libsodium-static -y
  ln /usr/bin/cmake3 /usr/bin/cmake
  # Install a package with repository for your system:
  # On CentOS, install package centos-release-scl available in CentOS repository:
  sudo yum install centos-release-scl -y
  # Install the collection:
  sudo yum install devtoolset-7 -y
  # Start using software collections:
  scl enable devtoolset-7 bash
  ./make_devel.sh
  ./build/chia_plot --help
  ```
</details>

<details>
  <summary>Clear Linux</summary>
  
  ```bash
  sudo swupd update
  sudo swupd bundle-add c-basic devpkg-libsodium git wget

  echo PATH=$PATH:/usr/local/bin/ # for statically compiled cmake if not already in your PATH

  # Install libsodium
  cd /tmp
  wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
  tar -xvf LATEST.tar.gz
  cd libsodium-stable
  ./configure
  make && make check
  sudo make install
  # Checkout the source and install
  cd ~/
  git clone https://github.com/MIGPOOL/chia-plotter.git 
  cd ~/chia-plotter
  git submodule update --init
  ./make_devel.sh
  ./build/chia_plot --help
  ```
</details>

<details>
  <summary>Ubuntu 20.04</summary>
  
  ```bash
  sudo apt install -y libsodium-dev cmake g++ git build-essential
  # Checkout the source and install
  git clone https://github.com/MIGPOOL/chia-plotter.git 
  cd chia-plotter

  git submodule update --init
  ./make_devel.sh
  ./build/chia_plot --help
  ```

  The binaries will end up in `build/`, you can copy them elsewhere freely (on the same machine, or similar OS).
</details>

<details>
  <summary>Debian 10 ("buster")</summary>

  Make sure to add buster-backports to your sources.list otherwise the installation will fail because an older cmake version. See the [debian backport documentation](https://backports.debian.org/Instructions/) for reference.

  ```bash
  # Install cmake 3.16 from buster-backports
  sudo apt install -t buster-backports cmake
  sudo apt install -y libsodium-dev g++ git
  # Checkout the source and install
  git clone https://github.com/MIGPOOL/chia-plotter.git 
  cd chia-plotter

  git submodule update --init
  ./make_devel.sh
  ./build/chia_plot --help
  ```
  The binaries will end up in `build/`, you can copy them elsewhere freely (on the same machine, or similar OS).
</details>

<details>
  <summary>macOS</summary>
  
  First you need to install the [Brew](https://brew.sh/) package manager and [Xcode](https://apps.apple.com/app/xcode/id497799835) OR [Xcode Command Line Tools](https://developer.apple.com/download/).
  ```bash
  # Download Xcode Command Line Tools (skip if you already have Xcode)
  xcode-select --install

  # Now download chia-plotter's dependencies
  brew install libsodium cmake git autoconf automake libtool wget
  brew link cmake
  git clone https://github.com/MIGPOOL/chia-plotter.git 
  cd chia-plotter
  git submodule update --init
  ./make_devel.sh
  ./build/chia_plot --help
  ```

  Linking libsodium should be performed automatically, but in case of failure please follow these instructions:
  ```
  # If you downloaded Xcode run these:
  sudo ln -s /usr/local/include/sodium.h /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/
  sudo ln -s /usr/local/include/sodium /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/

  # If you downloaded CommandLineTools run these:
  sudo ln -s /usr/local/include/sodium.h /Library/Developer/CommandLineTools/usr/include
  sudo ln -s /usr/local/include/sodium /Library/Developer/CommandLineTools/usr/include

  ```

  Confirm which directory you have on YOUR Mac before applying following commands
  ```
  # For x86_64 Macs
  wget https://raw.githubusercontent.com/facebookincubator/fizz/master/build/fbcode_builder/CMake/FindSodium.cmake -O /usr/local/opt/cmake/share/cmake/Modules/FindSodium.cmake
  ```
   or
  ``` 
  # For ARM64 (M1) Macs
  wget https://raw.githubusercontent.com/facebookincubator/fizz/master/build/fbcode_builder/CMake/FindSodium.cmake -O /opt/homebrew/Cellar/cmake/*/share/cmake/Modules/FindSodium.cmake
  ```

  If a maximum open file limit error occurs (as default OS setting is 256, which is too low for default bucket size of `256`), run this before starting the plotter
  ```
  ulimit -n 3000
  ```
  This file limit change will only affect the current session.
</details>

<details>
  <summary>Running in a Docker container</summary>

  In some setups and scenarios, it could be useful to run your plotter inside a Docker container. This could be potentially useful while running `chia-plotter` in Windows.

  To do so, [install Docker](https://docs.docker.com/get-docker/) on your computer and them run the following command:

  ```sh
  docker run \
    -v <path-to-your-tmp-dir>:/mnt/harvester \
    -v <path-to-your-final-dir>:/mnt/farm \
    odelucca/chia-plotter \
      -t /mnt/harvester/ \
      -d /mnt/farm/ \
      -p <pool-key> \
      -f <farm-key> \
      -r <number-of-CPU-cores>
  ```
  > 💡 You can provide any of the plotter arguments after the image name (`odelucca/chia-plotter`)

  In a Linux benchmark, we were able to find that running in Docker is only 5% slower than running in native OS.

  For Windows users, you should check if your Docker configuration has any RAM or CPU limits. Since Docker runs inside HyperV, that could potentially constrain your hardware usage. In any case, you can set the RAM limits with the `-m` flag (after the `docker run` command).

  ### Regarding multithread in Docker

  While running in Windows, you may need to proper configure your Docker to allow multi CPUs. You can do so by following [this article](https://www.thorsten-hans.com/docker-container-cpu-limits-explained/)

  In a nutshell, you could also pass the `--cpus` flag to your `docker run` command in order to achieve the same result.

  So, for example, the following command...
  ```sh
  docker run \
    -v <path-to-your-tmp-dir>:/mnt/harvester \
    -v <path-to-your-final-dir>:/mnt/farm \
    -m 8G \
    --cpus 8 \
    odelucca/chia-plotter \
      -t /mnt/harvester/ \
      -d /mnt/farm/ \
      -p <pool-key> \
      -f <farm-key> \
      -r 8
  ```

  ...would run your plotter with 8 CPU cores and 8GB of RAM.

  ### Building a Docker container
  Make sure your submodules are up-to-date by running `git submodule update --init`, then simply build with `docker build .`
</details>

---

## Известные проблемы

- Нуждается по крайней мере Cmake 3.14 (because of bls-signatures)

[How to install latest cmake on Ubuntu 18.04](https://askubuntu.com/questions/1203635/installing-latest-cmake-on-ubuntu-18-04-3-lts-run-via-wsl-openssl-error)
