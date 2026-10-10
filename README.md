# STM32 GPIO LED Demo

STM32F103C8T6 mikrodenetleyicisinde **PC13 pinine bağlı LED'i 500 ms aralıklarla açıp kapatan** C uygulaması. STM32 HAL kullanarak GPIO yapılandırmasını, sistem saatinin hazırlanmasını ve sürekli çalışan bir ana döngüyü gösterir.

## Çalışma mantığı

Başlangıçta HAL, dahili saat kaynağı ve GPIO çıkışı hazırlanır. Ardından ana döngüde şu işlemler tekrarlanır:

```c
HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
HAL_Delay(500);
```

LED'in durumu her 500 ms'de değişir; bir tam açılıp kapanma çevrimi yaklaşık **1 saniye** sürer. `HAL_Delay` ana döngüyü bekleten bir gecikmedir.

## Donanım ve yapılandırma

| Bileşen / ayar | Değer |
| --- | --- |
| Mikrodenetleyici | STM32F103C8T6, ARM Cortex-M3 |
| LED çıkışı | PC13 |
| GPIO modu | Push-pull çıkış, düşük hız, pull-up/pull-down kapalı |
| Saat kaynağı | Dahili HSI, PLL kapalı |
| Durum değiştirme aralığı | 500 ms |
| Programlama bağlantısı | SWD üzerinden ST-LINK veya uyumlu programlayıcı |

PC13'e bağlı yerleşik LED'i olan bir kart kullanılabilir. Karta yükleme için programlayıcının SWDIO, SWCLK ve GND bağlantılarını kartın pin düzenine göre yapın; kartı uygun beslemeyle çalıştırın.

## Geliştirme araçları

- GNU Arm Embedded araç zinciri: `arm-none-eabi-gcc`, `objcopy` ve `size`.
- GNU Make; Windows'ta mevcut VS Code derleme görevi `mingw32-make.exe` kullanır.
- Firmware yüklemek için OpenOCD veya STM32CubeProgrammer.
- İsteğe bağlı STM32CubeMX: `Blink.ioc` içindeki pin ve saat ayarlarını incelemek için.

Linker betiğindeki `READONLY` tanımları nedeniyle GCC 11 veya üzeri bir araç zinciri kullanın.

## Derleme

```bash
git clone https://github.com/muzafferturkoglu/stm32-gpio-led-demo.git
cd stm32-gpio-led-demo
make
```

Windows'ta Make bu adla kuruluysa:

```powershell
mingw32-make
```

Derleyici PATH üzerinde değilse araç zincirinin `bin` dizinini belirtin:

```bash
make GCC_PATH=/arac/zinciri/bin
```

Derleme çıktıları `build/Blink.elf`, `build/Blink.hex` ve `build/Blink.bin` dosyalarıdır. Depo adı değişse de Makefile içindeki derleme hedefi `Blink` olarak korunmuştur.

## Karta yükleme ve hata ayıklama

ST-LINK bağlıyken proje klasöründen OpenOCD ile yükleme:

```bash
openocd -f interface/stlink.cfg -f target/stm32f1x.cfg -c "program build/Blink.elf verify reset exit"
```

Alternatif olarak STM32CubeProgrammer ile `.elf` veya `.hex` dosyasını yükleyin. `.bin` dosyası kullanılacaksa başlangıç adresi `0x08000000`'dır.

VS Code yapılandırmaları `.vscode/` altında bulunur. Hata ayıklama için C/C++ uzantısı, OpenOCD ve uygun GDB kurulumu gerekir. `launch.json` içindeki `miDebuggerPath` değerini kendi bilgisayarınızdaki GDB yoluyla eşleştirin. Yükleme ve hata ayıklama ayarları doğrudan `build/Blink.elf` dosyasını kullanır.

## Dosya yapısı

```text
stm32-gpio-led-demo/
├── README.md
├── Blink.ioc                  # STM32CubeMX yapılandırması
├── Makefile                   # Derleme tanımları
├── STM32F103XX_FLASH.ld        # Flash ve RAM yerleşimi
├── startup_stm32f103xb.s       # Başlangıç kodu
├── Core/
│   ├── Inc/                   # Uygulama başlık dosyaları
│   └── Src/
│       ├── main.c             # Başlatma ve LED kontrol döngüsü
│       └── stm32f1xx_it.c      # Kesme işleyicileri
├── Drivers/                   # STM32 HAL ve CMSIS
└── .vscode/                   # Derleme, yükleme ve hata ayıklama ayarları
```

Yanıp sönme hızını değiştirmek için `Core/Src/main.c` içindeki `HAL_Delay(500)` değerini milisaniye cinsinden düzenleyin ve firmware'i yeniden derleyip yükleyin.

**Geliştirici:** [Muzaffer Türkoğlu](https://github.com/muzafferturkoglu)
