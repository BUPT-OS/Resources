# evl-5.13��װ
## �������
![](_v_images/20220315103311509_8806.png)

[ubuntu-18.04.6-live-server-amd64.iso](https://releases.ubuntu.com/18.04/ubuntu-18.04.6-live-server-amd64.iso)
## ��װ����
### 1.�������װ
���Ͱ�װ��ʽ����  
![](_v_images/20220315102952353_2206.png)

![](_v_images/20220315103015238_8419.png)

![](_v_images/20220315103054083_21028.png)
�������֮�����ubuntu�İ�װ
ע���ڷ�����ʱ�����Ŀ¼������40g
![](_v_images/20220315102900549_4281.png)

### 2.���ð�װ����

```bash
sudo apt upgrade #�ȸ��������
sudo apt-get install libncurses5-dev gcc make git exuberant-ctags bc libssl-dev flex bison libelf-dev # ��װ�����Ҫ�İ�
```
### 3.evl������������

```bash
git clone https://source.denx.de/Xenomai/xenomai4/linux-evl.git #����evl��
cd linux-evl
git checkout -b evl-5.13 origin/v5.13-evl-rebase #�л���5.13��֧
```

����evl
```bash
make menuconfig
```
![](_v_images/20220315111502713_2504.png)

![](_v_images/20220315111426626_1675.png)

�޸�.config
![](_v_images/20220315111637962_30126.png)

��ʼ���벢��װ
```bash
sudo make
sudo make modules_install install
```
֮��reboot��������Linux�ں�5.13�汾
![](_v_images/20220315151256231_7943.png)

### 4.evl-5.13����libevl r27
��Ϊװ����С��R29�İ汾�����԰�װ�̳�������
[LEGACY LIBEVL BUILD](https://evlproject.org/core/build-libevl-legacy/)

[Git �л���Tag��Branch��֧](https://www.choupangxia.com/2019/11/21/git-tag-branch/)
����libevl���л���r27 tag
```git
git clone https://source.denx.de/Xenomai/xenomai4/libevl.git
git checkout r27
```

��װg++
```command
sudo apt install g++
```

����libevl
```command
$ mkdir /tmp/build-native && cd /tmp/build-native
$ sudo make -C ~/libevl O=$PWD UAPI=~/linux-evl DESTDIR=/usr/evl install
```

Ȼ��������µ�~/.bashrc��
```command
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/evl/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib
export PATH=$PATH:/usr/evl/bin
```

��ɺ�ִ��
```command
source ~/.bashrc
```
[Can't access shared libraries when running with sudo](https://raspberrypi.stackexchange.com/questions/583/cant-access-shared-libraries-when-running-with-sudo)
Ȼ�����sudo bash������evl
```bash
sudo bash
evl test
```

![](_v_images/20220316005714187_4726.png)

����������װ�ɹ�������

### 5.����.c�ļ�
���뻷������C_INCLUDE_PATH��CPLUS_INCLUDE_PATH��Ŀ��Ŀ¼��libevl��include
����������䵽~/.bashrc��
```bash
export C_INCLUDE_PATH=$C_INCLUDE_PATH:/usr/evl/include
export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/usr/evl/include
```
Ȼ��
```bash
source ~/.bashrc
```

[evl����init��������](https://evlproject.org/core/user-api/init/)
�������´��뵽test.c
```c
#include <error.h>
#include <evl/evl.h>

int main(int argc, char *const argv[])
{
	int ret;

	ret = evl_init();
	if (ret)
		error(1, -ret, "evl_init() failed");
	...
	return 0;
}
```
ʹ����������ɳɹ�����
```bash
gcc test.c /usr/evl/lib/libevl.so -o t
```


**���ˣ�evl��libevlȫ���ɹ���װ����**

## ����������
### error��kernel does not meet out ABI requirements��libevl�汾ѡ�����
һ��ʼѡ����r32������meson���а�װ��������������´���
![](_v_images/20220315180740724_25605.png)

![](_v_images/20220315185337243_27510.png)

���Կ�����EVL_ABI_PREREQ���������[EVL_ABI_BASE��EVL_ABI_LEVEL]��Ҫ��
�����ڲ�[evl�ĵ�](https://evlproject.org/devprocess/)�﷢������仰
![](_v_images/20220315184523056_4815.png)
EVL_ABI_PREREQ�������[EVL_ABI_BASE��EVL_ABI_LEVEL]֮��
1. ��libevl�ҵ�EVL_ABI_PREREQ��ֵ
2. ��linux-evlԴ�����ҵ�EVL_ABI_BASE��EVL_ABI_LEVEL��ֵ
3. �Ա�֮���������EVL_ABI_PREREQ����[EVL_ABI_BASE��EVL_ABI_LEVEL]֮�䣬��ô˵�����libevl�İ汾�ǿ��õ�

EVL_ABI_PREREQ������libevl/include/evl/evl.h����
[r27](https://source.denx.de/Xenomai/xenomai4/libevl/-/blob/r27/include/evl/evl.h#L25)
![](_v_images/20220315184457211_17131.png)

EVL_ABI_BASE��EVL_ABI_LEVEL��/include/uapi/evl/control.h����
![](_v_images/20220315184713373_25039.png)

[evl-5.13](https://source.denx.de/Xenomai/xenomai4/linux-evl/-/blob/v5.13-evl-rebase/include/uapi/evl/control.h#L14)��[EVL_ABI_BASE��EVL_ABI_LEVEL] =  [23,26]
���Աȣ�evl-5.13���õ�libevl�汾Ϊ[r17](https://source.denx.de/Xenomai/xenomai4/libevl/-/blob/r17/include/evl/evl.h)����[r27](https://source.denx.de/Xenomai/xenomai4/libevl/-/blob/r27/include/evl/evl.h)

���ϣ�5.13��evlҪѡ��r27�汾��libevl

### ����ʱ�Ҳ����ļ���evl/evl.h: No such file or directory��
[gccָ��ͷ�ļ�����̬���ӿ�·��](https://blog.csdn.net/trochiluses/article/details/48491539)
���뻷���������༭~/.bashrc�������������
```bash
export C_INCLUDE_PATH=$C_INCLUDE_PATH:/usr/evl/include
export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/usr/evl/include
```
֮��
```bash
source ~/.bashrc
```

### unknown type name ��useconds_t��; did you mean ��suseconds_t��?
![](_v_images/20220316111306887_4903.png)
���ݴ�����ʾ�޸ļ���

### undefined reference to `evl_init'
[GCC ����̬���ӿ����ӵ���ִ���ļ�](http://c.biancheng.net/view/2385.html)


 ����ʱδָ����̬���ӿ�libevl.soӦ��ʹ������������붯̬���ӿ�
```bash
gcc test.c /usr/evl/lib/libevl.so -o t
```

## �����װ�˸߰汾��evl���ɲο�����libevl��װ��ʽ
### 1��װpython3.7
[ubuntu18��װPython3.7](https://zhuanlan.zhihu.com/p/129867344)
```bash
sudo apt update
sudo apt upgrade -y
```

```bash
sudo apt install build-essential zlib1g-dev libbz2-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev wget
```
���ؽ�ѹ���벢��װ
```bash
wget https://www.python.org/ftp/python/3.7.4/Python-3.7.4.tgz
tar -xzvf Python-3.7.4.tgz
cd Python-3.7.4
./configure --prefix=/usr/local/src/python37  # ���ð�װλ��
sudo make
sudo make install
```

����������
```bash
sudo ln -s /usr/local/src/python37/bin/python3.7 /usr/bin/python3.7
sudo ln -s /usr/local/src/python37/bin/pip3.7 /usr/bin/pip3.7
```

### 2��װninja
[Ninja��װ�ͻ���ʹ��](https://zhuanlan.zhihu.com/p/321882707)
```bash
sudo apt install ninja-build
```
### 3��װmeson
[Meson�ٷ���װ](https://mesonbuild.com/Getting-meson_zh.html)

```bash
pip3.7 install --user meson
reboot
```
ע��Ҫ��������������meson���Ƿ�װ�ɹ�
### 4.libevl
����libevl
```git
git clone https://source.denx.de/Xenomai/xenomai4/libevl.git
```

�������л�Ŀ¼������ע��Ҫʹ�þ���·��������ʹ��~��Ϊ/home/evl
```bash
mkdir /tmp/build-native && cd /tmp/build-native
meson setup -Dbuildtype=release -Dprefix=/opt/evl -Duapi=/home/evl/linux-evl . /home/evl/libevl
```

�����������´����Ҳ���~/Ŀ¼�µ��ļ�
![](_v_images/20220315175935261_14435.png)

����ֱ����sh��������~/linux-evl�ֿ��Գɹ��ҵ���ӦĿ¼����bugδ��
![](_v_images/20220315180052257_19236.png)

֮��
```bash
meson compile
```

δ���......