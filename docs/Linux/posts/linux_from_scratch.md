# Learning OS from scratch

this article is based on [闪客：#linux源码趣读]([#Linux 源码趣读 (qq.com)](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjE3NDE0Ng==&action=getalbum&album_id=2123743679373688834&scene=173&from_msgid=2247499226&from_itemidx=1&count=3&nolastread=1#wechat_redirect))

![overview](D:\typora_img\image-20231106193521026.png)

## Section 1

### Power up & Load data

![image-20231106193511445](D:\typora_img\image-20231106193511445.png)

The primordial task of boosting an OS is loading the "OS" (code, data, etc.) into memory based on the functionality of BIOS:

- BIOS loads the **first sector** of disk (512 bytes) into the memory at address 0x7c00 (magic number)
    - actually it is the binary code compiled of **bootsect.s**

#### Bootsect.s

- **bootsect.s** will set some registers (basically, code, segment, and stack) in CPU to specific pre-defined number to following loading
- **bootsect.s** then will load the following 4 + 240 sectors on disk to specific address in memory
- then will jump to address **0x90200**, which is corresponding to **setup.s** file

![image-20231106194216641](D:\typora_img\image-20231106194216641.png)

#### setup.s

- in **setup.s**, OS will read some information of **devices**
    - memory size, video-card data, display config, hd(hard disk) data
- then move the data from [0x10000, 0x90000] to 0x0
    - ![image-20231106195359521](D:\typora_img\image-20231106195359521.png)
- now, the layout of memory is
    - ![image-20231106195526423](D:\typora_img\image-20231106195526423.png)

#### Mode convertion

- real mode (16 bit) -> protected mode (32 bit)

- not core function, just skip this

#### segmentation

<center class="half">
    <img src="D:\typora_img\image-20231107145146268.png" width="300">
    <img src="D:\typora_img\image-20231107145411697.png" width="300">
</center>    

- CPU has 2 registers:
    - idt(interrupt discriptor table) -> record the address of interrupt handler functions
    - gdt(global discriptor table) -> record some segment registers (code, data)

#### paging

<img src="D:\typora_img\image-20231107145754353.png" alt="image-20231107145754353" style="zoom:67%;" />

