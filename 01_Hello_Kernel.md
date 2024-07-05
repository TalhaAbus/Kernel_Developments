**Bu projede basit bir kernel modülü yazacağız ve çıktısını gözlemleyeceğiz.**

# Gerekli paketleri kuralım:

```bash
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r)
```

#  hello.c adında bir dosya oluşturalım:

```C
#include <linux/module.h>
#include <linux/kernel.h>

int init_module(void) {
    printk(KERN_INFO "Merhaba dünya 1.\n");
    // 0 dönerse modül başarıyla yüklenmiş demektir.
    return 0;
}

void cleanup_module(void) {
    printk(KERN_INFO "Güle güle dünya 1.\n");
}

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Sizin Adınız");
MODULE_DESCRIPTION("Basit bir test modülü");
```
> Bu modül, yüklendiğinde "Merhaba dünya 1." ve kaldırıldığında "Güle güle dünya 1." mesajını yazdıracaktır.

#  Makefile oluşturalım:
- Derlemek için bir makefile gerekir.

```makefile
obj-m += hello.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```
#  Modülü derleyip test edelim:
```bash
make
sudo insmod hello.ko
```
> insmod ile linux çekirdeğine modül ekledik. ko uzantılı dosya kernel object'tir.

#  Mesajları kontrol edelim.
```bash
dmesg | tail
```
> dmesg ile kerneldeki mesajı gördük. "merhaba dünya 1."

#  Modülü silelim:
```bash
sudo rmmod hello
```
> Yüklediğimiz modülü kerneldan sildik. "güle güle dünya" çıktısı aldık







