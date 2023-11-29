# 	操作系统思考题



1. SSD采用何种文件系统？

   （我在网上查找的资料基本都关于操作系统层面的文件系统，如ext4或者是NTFS。似乎并没有专门针对SSD提供的文件系统）。

   1. 在ROM的分类中，SSD属于Flash以来，是当前最为广泛使用的一种ROM设备。

   2. 目前SSD(solid state memory)常用的文件系统，包括ext4（Linux）和NTFS（Windows）。

   3. NTFS文件系统是Microsoft在Windows NT3.1 之后推出的一种文件系统.

   4. 下面介绍ext4文件系统

      1. ![image-20231129113650079](https://raw.githubusercontent.com/moonchildink/image/main/imgs202311291136144.png)

         在WSL中可以看到当前使用的文件系统为`ext4`。Linux最初使用 MINIX1 文件系统，受限于该系统的文件名和文件大小限制，后来推出了`ext`文件系统，而`ext4`文件系统`ext`系统的改良版。*Modern Operating System*一书中对`ext4`文件系统做出了详细的介绍：支持journaling system

2. fat32文件系统如何支持长文件名？
   1. 一般来说，FAT32文件系统支持8.3 format，这种格式一般应用于DOS,Windows 95以及Windows NT之前的操作系统。这种格式要求文件名包含8字符，文件拓展名为3字符。但FAT32文件系统支持长文件名后来更新了长文件名的支持。
   2. 在[NTFS.com](https://www.ntfs.com/fat-filenames.htm)中，我找到了关于长文件名的更详细的说明：在Windows NT3.5之后，文件重命名使用属性位(attribute bits)来支持长文件名。当用户创建长文件名文件时，Windows系统都会创建一个8.3 filename，此外，还会创建一个或者多个辅助文件夹条目，每13个字符一个。使用Unicode形式存储相应部分。
   3. 下图中说明了文件`Thequi~1.fox`的存储方式：

<img src='https://www.ntfs.com/images/screenshots/recover-ROOT-structure.gif'>