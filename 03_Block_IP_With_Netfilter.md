**Netfliter altyapısı ile belirli IP adresinden gelen trafiği bloklayacak bir kernel modülü yazalım**

# Ortam Hazırla
- **linux-heasders** ve **build-essential** paketlerini kontrol et.

# Netfilter hook modülü kodunu yazalım

```C
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/netfilter.h>
#include <linux/netfilter_ipv4.h>
#include <linux/ip.h>
#include <linux/inet.h>

static struct nf_hook_ops nfho;
static char *block_ip = "192.168.1.100";  // Bloklanacak IP adresi
module_param(block_ip, charp, 0000);
MODULE_PARM_DESC(block_ip, "Bloklanacak IP adresi");

unsigned int block_ip_hook(void *priv, struct sk_buff *skb, const struct nf_hook_state *state) {
    struct iphdr *ip_header = ip_hdr(skb);  // IP başlık yapısını al
    if (!ip_header) return NF_ACCEPT;

    // IP adresini kontrol et
    if (ip_header->saddr == in_aton(block_ip)) {
        printk(KERN_INFO "Bloklanan paket: %pI4\n", &ip_header->saddr);
        return NF_DROP;  // Bu paketi engelle
    }

    return NF_ACCEPT;  // Diğer paketlere izin ver
}

static int __init filter_init(void) {
    nfho.hook = block_ip_hook;
    nfho.hooknum = NF_INET_PRE_ROUTING;
    nfho.pf = PF_INET;
    nfho.priority = NF_IP_PRI_FIRST;
    nf_register_net_hook(&init_net, &nfho);
    return 0;
}

static void __exit filter_exit(void) {
    nf_unregister_net_hook(&init_net, &nfho);
}

module_init(filter_init);
module_exit(filter_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A network packet filter module");
```

# Makefile oluşturup modülü deneyelim

```bash
make
sudo insmod block_ip_with_netfilter.ko block_ip="192.168.1.100"
```

# Test
- dmesg ile bloklanan paketi gözlemleyelim.

```bash
dmesg | grep "Bloklanan paket"
```












