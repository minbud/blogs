---
title: Boot_Singlecore
date: 2018-07-03 22:51:09
tags: evmc6678l
---

## 1. EEPROM BOOT & POST(Power On Self Test)  
　　evmc6678评估板上有一块128KB的EEPROM，这块内存主要用于烧写IBL（Intermediate BootLoader）即二次引导的程序，或者烧写POST上电自检程序。其中在烧写程序时有两个总线地址，一个是0x50，一个是0x51，两个地址分别代表寻址128kB的其中64KB，前者默认用来烧写POST程序，后者一般用来烧写IBL。两种方式通过SW5的PIN4来区分（on：0x50，off：0x51），可以同时存在，不互相影响，启动时根据开关设置从相应位置读取程序。

<!--more-->
### POST
　　POST例程位于MCSDK安装目录下，从 mcsdk >tools >post >docs >readme.txt中查看到具体的将POST程序烧写到EEPROM中的步骤。下面简单说明一下步骤：
> tips: ti文档中的txt文件直接用windows的记事本打开会有格式问题，可以用vscode或者notepad++打开。  
> 1. 重新编译生成POST工程文件。即clean project >> build project。(不是必要的)  
> 2. 由于POST需要利用EEPROM烧写工具进行烧写，将 tools >post >evmc6678l >bin >post_i2crom.bin 拷贝到 tools　>writer >eeprom >evmc6678l >bin下。  
> 3. 修改该目录下的eepromwriter_input.txt文件，设置file_name为post_i2crom.bin，bus_addr为0x50，其它为0.  
> 4. 将tools >writer >eeprom >evmc6678l 工程impot到CCS中，重新生成.out文件。  
> 5. 连接开发板，launch目标文件，连接core0，使用evmc6678.gel初始化内存,将生成的.out文件load到core0。  
> 6. view >memory browser,在打开的窗口中输入0x0C000000，找到该内存，在该处右击鼠标，load memory,选择post_i2crom.bin文件，type选择ti raw data，点击next，确认地址是0x0C000000，为32bit，swap没有勾选，点击finish。  
> 7. console窗口输出successfully时，terminate debug。确保bootmode管脚为I2C post boot,按下REST_FULL按钮,可以在串口中看到如下数据：  
> <div align=center><img src="post_result.png" height=400px width=600px alt="img"></div>
> 说明一下烧写程序的工作（包括EEPROM，NAND，NOR烧写程序），我们在CCS中的Memory Browser窗口将程序烧写到DDR内存中，另外烧写程序读取硬盘bin目录下（*input.txt中指定的文件）的程序文件，并解析该文件是否符合要求。若符合则直接将在DDR内存中文件烧写到指定内存（EEPROM，NAND，NOR）中。

### RBL直接读取EEPROM程序
　　根据实验结果以及[搜索结果](https://e2e.ti.com/support/dsp/c6000_multi-core_dsps/f/639/t/388490?Can-c6678-be-booted-from-directly-i2c-eeprom-)，程序文件只能烧到0x50对应的64KB中并且启动执行，而0x51对应的EEPROM只是保留给IBL启动程序的。 RBL对读取的程序文件格式有一定的要求，生成的.out文件需要经过工具链的处理。相应的处理流程和SPI nor flash boot 一样，这个留在后面叙述。  
　　由于程序是烧在0x50处，因而流程和POST是一致的，启动时开关设置也一致。我烧写的是Led_play例程，能够成功运行。  

## 2. IBL NAND|NOR FLASH BOOT
　　由于EPPROM大小只有128kB（64kB可用于用户程序），当用户程序文件较大时，就需要将文件放在FLASH中，此时一种可选的方法就是通过二级引导程序（IBL）来将FLASH中的文件搬运到RAM中运行，而IBL文件就是放在EEPROM中的。另外由于NAND FLASH的特点（不支持随机读取，读取时是一块一块的），因而不能直接在NAND FLASH上运行代码。因此RBL也不支持直接的NAND FLASH BOOT。  
　　将IBL烧写到EEPROM中的操作和上面类似，但是需要将bus_addr设置为0x51，烧写文件为i2crom_0x51_c6678_le.bin，同时在烧写完成后需要配置boot param table。具体操作如下：
> 1. 打开 mcsdk >tools >boot_loader >ibl >src >make >bin >i2cConfig.gel,找到setConfig_c6678_main函数，将  
> ibl.bootModes[0].u.norBoot.bootFormat	= ibl_BOOT_FORMAT_BBLOB;　　　　　改写为 
> ibl.bootModes[0].u.norBoot.bootFormat	= ibl_BOOT_FORMAT_ELF;
> 其中BBLOB表示二进制文件即.bin，而ELF文件指生成的.out文件，这样修改后烧写时load memory操作就可以使用生成的.out文件，更加方便，不用利用工具链转换成.bin。缺点就是.out文件中包含比较多无用的数据，文件较大。  
> 2. 接着连接开发板，将mcsdk >tools >boot_loader >ibl >src >make >bin >i2cparam_0x51_c6678_le_0x500.out load 到core0，并执行。同时执行load gel操作，将上面修改的gel文件load进来。注意前面evmc6678.gel还是要load。然后 scripts >EVM c6678 IBL >setConfig_c6678_main。等待几秒钟，在console窗口中按回车健，看到I2c table write complete 表示写启动参数表成功了。  
> tips：在第二步中，需要先resume即先执行程序，然后执行gel文件的setConfig_c6678_main，不然会显示出错。  
> 3. 这样IBL就烧写好了，以后如果没有需要修改boot param的话，就不用在执行这个过程了。  
> 4. 接着需要将自己的程序烧写到NOR FLASH中，具体步骤可以查看mcsdk >tools >writer >nor >docs >readme.txt。与烧写EEPROM类似，主要差别就是memory browser处地址为0x80000000。
> 4. 按照IBL NOR BOOT设置开关，重启，可以发现程序跑起来了~_~。  


## 3. SPI NOR FLASH BOOT
　　evmc6678l也可直由RBL引导，从NOR FLASH处启动。RBL只能识别boot table format，而且只能使用大端模式。因此我们主要的工作是将生成的.out文件经过工具链进行处理，转换为能够被RBL识别的文件格式。此处的资料来源主要是[TI论坛](https://e2e.ti.com/support/dsp/c6000_multi-core_dsps/f/639/t/447109?C6678-multicore-booting-from-SPI-Nor-flash)。文档为*Booting from the SPI NOR on C6670/C6678 EVM*。[更新资料](https://www.google.com.hk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=7&ved=0ahUKEwjLroyPy4TcAhVDa94KHWJ-BFkQFghNMAY&url=%68%74%74%70%3a%2f%2f%77%77%77%2e%64%65%79%69%73%75%70%70%6f%72%74%2e%63%6f%6d%2f%63%66%73%2d%66%69%6c%65%2e%61%73%68%78%2f%5f%5f%6b%65%79%2f%63%6f%6d%6d%75%6e%69%74%79%73%65%72%76%65%72%2d%64%69%73%63%75%73%73%69%6f%6e%73%2d%63%6f%6d%70%6f%6e%65%6e%74%73%2d%66%69%6c%65%73%2f%35%33%2f%33%36%31%37%2e%4b%65%79%73%74%6f%6e%65%2d%31%2d%53%50%49%2d%4e%4f%52%2d%5f%32%46%35%34%41%38%35%32%36%35%36%42%41%34%39%41%45%35%34%45%43%41%35%33%45%38%36%43%30%46%36%31%38%42%34%45%37%39%39%38%5f%2e%70%64%66&usg=AOvVaw1QyyLgyoSvds9ox4LGi3lZ)  
#### hex6x.exe
　　这个工具的主要作用是将.out转换为Boot table format,使得RBL能够读取到程序二进制文件的各个段。该工具的输入是一个.rmd文件。包括以下部分：
> C:\Users\DarkLing\Desktop\KeystoneI_bootloader_workshop\boot_image\SPI_Bootloader\led_play.out  
> -a  
> -boot  
> -e _c_int00  
>   
> ROMS  
> {  
>	ROM1:  org = 0x0C000000, length = 0x100000, memwidth = 32, romwidth = 32  
>	files = { C:\Users\DarkLing\Desktop\led_test\led_play.btbl>}  
>}  

　　创建.rmd文件后执行命令:  
> .\hex6x.exe .\led_play.rmd  

　　生成了led_play.btbl文件。boot table format如下所示：
<div align=center><img src="boot_table_format.png" height=500px width=450px alt="img"></div>

#### b2i2c.exe
　　将.btbl文件以0x80字节分块，并附加长度和校验和信息，以符合RBL的需要。  
> .\b2i2c.exe .\led_play.btbl .\led_play.i2c

　　生成led_play.i2c

#### b2ccs.exe
　　将.i2c转为CCS能够接收的格式。
> .\b2ccs .\led_play.i2c .\led_play.ccs  

　　生成led_play.ccs  

#### romparse.exe
　　将boot table 和boot parameter table 结合，需要有.map文件作为输入:
> section {  
> boot_mode = 50  
> param_index = 0  
> options = 1  
> core_freq_mhz = 1000  
> exe_file = "led_play.ccs"  
> next_dev_addr_ext = 0x0  
> sw_pll_prediv = 5  
> sw_pll_mult = 32  
> sw_pll_postdiv = 2  
> sw_pll_flags = 1  
> addr_width = 24  
> n_pins = 4  
> csel = 0  
> mode = 0  
> c2t_delay = 0  
> bus_freq_mhz = 0  
> bus_freq_khz = 500  
> }

> .\romparse.exe .\nysh.spi.map


　　生成i2crom.ccs.修改生成的i2crom.ccs将第九行的51改为00。这个是I2c的地址,需要改为00，[原因未知](https://e2echina.ti.com/question_answer/dsp_arm/c6000_multicore/f/53/p/136365/382193#382193)。个人理解是，由于是使用SPI连接flash启动，不需要I2c，而0x51则是使用EEPROM作二级引导使用的，因而需要设置为0，表示不使用。  

#### byteswapccs.exe
　　将小端文件转为大端。

> .\byteswapccs.exe .\i2crom.ccs .\app.dat

　　生成的app.dat文件可以通过nor writer烧写到nor flash中。  
tips：  
　　注意到此时生成的是app.dat文件，也可以通过ccs2bin.exe将i2crom.ccs转为.bin文件：
> .\ccs2bin.exe  -swap .\i2rom.ccs .\app.bin  

　　ccs2bin.exe可以使用-swap参数表示大小端转换。 

但是操作文档中说到:  
The EVM uses 24 bit NOR connected on CHIP select 0. While flashing using the NOR writer use the .dat as is and don’t convert the .dat into a bin file.   
实际操作时，发现app.bin文件烧写时如果开关不是no boot mode 烧写后无法启动。而app.dat文件则不存在这个问题。
