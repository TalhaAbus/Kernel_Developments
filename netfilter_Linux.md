**Ağ paketleri işleme üzerine bir linux kernel modülü geliştirelim. Netfilter altyapısını kullnarak IP paketlerini yakalayalım ve bunları işleyelim.**

# Ortamı Hazırla
- Netfilter ile ilgili başlık dosyası eksikse bunu yükleyelim:

```bash
sudo apt-get install linux-headers-$(uname -r) build-essential
```

# Netfilter Hook Modülü Kodunu Yazalım:

```C
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/netfilter.h>
#include <linux/netfilter_ipv4.h>
#include <linux/ip.h>
#include <linux/inet.h>

static struct nf_hook_ops nfho;   //netfilter hook option struct

unsigned int hook_func(void *priv, struct sk_buff *skb, const struct nf_hook_state *state) {
    struct iphdr *ip_header = ip_hdr(skb);  //ip header struct
    if (!ip_header) return NF_ACCEPT;

    printk(KERN_INFO "Paket yakalandı: Kaynak IP = %pI4, Hedef IP = %pI4\n",
           &ip_header->saddr, &ip_header->daddr);

    return NF_ACCEPT;  //paketleri normal şekilde akıtmaya devam edin
}

static int __init packet_filter_init(void) {
    nfho.hook = hook_func;                    //hook fonksiyonumuzu ayarlayın
    nfho.hooknum = NF_INET_PRE_ROUTING;       //ilk paket yakalama noktası
    nfho.pf = PF_INET;                        //IPv4 paketleri için
    nfho.priority = NF_IP_PRI_FIRST;          //yakalama zincirindeki en yüksek öncelik

    nf_register_net_hook(&init_net, &nfho);   //hook'u kaydet
    return 0;
}

static void __exit packet_filter_exit(void) {
    nf_unregister_net_hook(&init_net, &nfho);  //hook'u kaldır
}

module_init(packet_filter_init);
module_exit(packet_filter_exit);

MODULE_LICENSE("GPL");
```

# Makefile oluşturalım:

```bash
obj-m += netfilter_module.o

all:
  make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
  make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

```

# Modülü derle ve test et

```bash
make
sudo insmod netfilter_module.ko
```

# Gözlemle
```bash
dmesg | tail
```





















