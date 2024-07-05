- **Bu projede "timer_setup" ve "mod_timer" fonksiyonları ile bir zamanlayıcı ayarladık. Bir linux zamanlayıcısı kullanarak belirli aralıklarla bir callback fonksiyonu çalıştırdık ve kernel'e çıktı verdik ve bu çıktıyı gözledik.**

- Bir linux kernel zamanlayıcısı kullanarak belirli aralıklarla bir callback fonksiyonu çalıştıralım. .

# Modülü başlatırken kullanıcıdan parametre alacak şekilde yazalım.

- hello.c dosyamızı güncelleyelim:

```C
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/timer.h>

static int delay = 500;
module_param(delay, int, S_IRUSR | S_IWUSR);
MODULE_PARM_DESC(delay, "Zamanlayıcı gecikmesi milisaniye cinsinden");

static struct timer_list my_timer;

void my_timer_callback(struct timer_list *t) {
    printk(KERN_INFO "my_timer_callback called (%ld).\n", jiffies);
    mod_timer(&my_timer, jiffies + msecs_to_jiffies(delay));
}

static int __init hello_start(void) {
    printk(KERN_INFO "Merhaba dünya: delay = %d\n", delay);

    // Zamanlayıcıyı başlat
    timer_setup(&my_timer, my_timer_callback, 0);
    mod_timer(&my_timer, jiffies + msecs_to_jiffies(delay));

    return 0;
}

static void __exit hello_end(void) {
    printk(KERN_INFO "Güle güle dünya\n");
    del_timer(&my_timer);
}

module_init(hello_start);
module_exit(hello_end);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Sizin Adınız");
MODULE_DESCRIPTION("Gelişmiş bir test modülü");
```

# Modülü derleyip test edelim:
- Makefile ouşturup derleyelim. (makefile içeriği aynı sadece dosya ismi değişik)

```bash
make
sudo insmod hello.ko delay=1000
```
> modül 1000 milisaniyede bir mesaj yazar. Çıktıyı görelim:

```bash
dmesg
```

# Modülü Kaldıralım:

```bash
sudo rmmod sys
dmesg | tail
```



















