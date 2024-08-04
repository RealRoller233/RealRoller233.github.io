一、导入CMCC管理界面cfg文件：
将路由器lan口连接电脑网口输入192.168.10.1进入移动路由器后台，找到“管理”，然后“导出配置文件”，之后得到一个conf文件，这是一个加密的归档文件，下一步进入linux系统进行解密
![[Pasted image 20240721153047.png]]
进入conf文件所在文件夹
输入
```bash
openssl aes-256-cbc -d -pbkdf2 -k $CmDc#RaX30O0M@\!$ -in cfg_export_config_file.conf -out - | tar -zxvf -
```
解压后得到etc文件
修改解压出的etc（注意不是linux的etc!）中etc/config/dropbear和etc/shadow两个文件，将dropbear中option enable从0改成1，将etc/shadow中的root密码去掉
![[2024-07-21_15-11.png]]
![[2024-07-21_15-11_1.png]]
![[2024-07-21_15-12.png]]
然后重新打包
输入
```bash
tar -zcvf - etc | openssl aes-256-cbc -pbkdf2 -k $CmDc#RaX30O0M@\!$ -out cfg_export_config_file_new.conf
```
cfg_export_config_file_new.conf是打包后的文件名
然后进入移动路由器后台，导入新的conf文件

二、登陆ssh进入后台
在shell中输入
```bash
ssh root@192.168.10.1
```
之后非常重要！
先下载五个文件
![[Pasted image 20240621223528.png]]
将这五个文件通过scp传入192.168.10.1
由于RAX3000M不支持sftp，所以需要加上-O选项
```bash
scp -O /path/to/your/file root@192.168.10.1:/tmp
```
之后进入RAX3000M后台/tmp进行以下操作(可以替换相应文件)￼￼Shell￼
root@RAX3000M:/tmp#ls #检验一下文件是不是都传上去了
## 注意一定要有红框里面的输出！！
```bash
root@RAX3000M:/tmp#ls #检验一下文件是不是都传上去了
root@RAX3000M:/tmp#insmod /tmp/mtd-rw.ko i_want_a_brick=1
root@RAX3000M:/tmp#mtd erase BL2
root@RAX3000M:/tmp#mtd write openwrt-mediatek-filogic-cmcc_rax3000m-nand-preloader.bin BL2
root@RAX3000M:/tmp#mtd erase FIP
root@RAX3000M:/tmp#mtd write openwrt-mediatek-filogic-cmcc_rax3000m-nand-bl31-uboot.fip FIP
```
![[2024-07-21_15-53 1.png]]
之后校验一下
```bash
root@RAX3000M:/tmp#mtd verify openwrt-mediatek-filogic-cmcc_rax3000m-nand-preloader.bin BL2
root@RAX3000M:/tmp#mtd verify openwrt-mediatek-filogic-cmcc_rax3000m-nand-bl31-uboot.fip FIP
```
![[2024-07-21_15-55.png]]
之后断电重启
进入windows操作系统，将电脑有线网卡IP设置为192.168.1.254
打开tfptd,将recovery文件版本号去掉，放在tftpd所在文件夹中，之后打开tftpd，有如图显示则成功
![[屏幕截图 2024-07-21 081320.png]]
![[屏幕截图 2024-07-21 081401.png]]![[屏幕截图 2024-07-21 082824.png]]
之后重启路由器，浏览器登陆192.168.1.1,上传sysupgrade固件，之后大功告成
![[IMG_9982.png]]
![[IMG_0327.png]]