文件(File)是操作系统中的一个重要概念。在系统运行时，计算机以进程为基本单位进行资源的调度和分配；而在用户进行的输入、输出中，则以文件为基本单位。大多数应用程序的输入都是通过文件来实现的，其输出也都保存在文件中，以便信息的长期存及将来的访问。当用户将文件用于应用程序的输入、输出时，还希望可以访问文件、修改文件和保存文件等，实现对文件的维护管理，这就需要系统提供一个文件管理系统，操作系统中的文件系统(File System)就是用于实现用户的这些管理要求。

# 文件

文件是指由创建者所定义的一组相关信息的集合，逻辑上可分为有结构文件和无结构文件两种。在有结构文件中，文件由一组相似记录组成，如报考某学校的所有考生的报考信息记录，又称记录式文件；而无结构文件则被看成是一个字符流，比如一个二进制文件或字符文件，又称流式文件。

虽然上面给出了结构化的表述，但实际上关于文件并无严格的定义。通常在操作系统中将程序和数据组织成文件。文件可以是数字、字母或二进制代码，基本访问单元可以是字节、行或记录。文件可以长期存储于硬盘或其他二级存储器中，允许可控制的进程间共享访问，能够被组织成复杂的结构。

## 文件的属性

文件有一定的属性，这根据系统的不同而有所不同，但是通常都包括如下属性：

* 名称：文件名称唯一，以容易读取的形式保存。
* 标识符：标识文件系统内文件的唯一标签，通常为数字，它是对人不可读的一种内部名称。
* 类型：被支持不同类型的文件系统所使用。
* 位置：指向设备和设备上文件的指针。
* 大小：文件当前大小（用字节、字或块表示），也可包含文件允许的最大值。
* 保护：对文件进行保护的访问控制信息。
* 时间、日期和用户标识：文件创建、上次修改和上次访问的相关信息，用于保护、 安全和跟踪文件的使用。

所有文件的信息都保存在目录结构中，而目录结构也保存在外存上。文件信息当需要时再调入内存。通常，目录条目包括文件名称及其唯一标识符，而标识符定位其他属性的信息。

文件属于抽象数据类型。为了恰当地定义文件，就需要考虑有关文件的操作。操作系统提供系统调用，它对文件进行创建、写、读、定位和截断。

* 创建文件：创建文件有两个必要步骤，一是在文件系统中为文件找到空间；二是在目录中为新文件创建条目，该条目记录文件名称、在文件系统中的位置及其他可能信息。
* 写文件：为了写文件，执行一个系统调用，指明文件名称和要写入文件的内容。对于给定文件名称，系统搜索目录以查找文件位置。系统必须为该文件维护一个写位置的指针。每当发生写操作，便更新写指针。
* 读文件：为了读文件，执行一个系统调用，指明文件名称和要读入文件块的内存位置。同样，需要搜索目录以找到相关目录项，系统维护一个读位置的指针。每当发生读操作时，更新读指针。一个进程通常只对一个文件读或写，所以当前操作位置可作为每个进程当前文件位置指针。由于读和写操作都使用同一指针，节省了空间也降低了系统复杂度。
* 文件重定位（文件寻址）：按某条件搜索目录，将当前文件位置设为给定值，并且不会读、写文件。
* 删除文件：先从目录中找到要删除文件的目录项，使之成为空项，然后回收该文件所占用的存储空间。
* 截断文件：允许文件所有属性不变，并删除文件内容，即将其长度设为0并释放其空间。

这6个基本操作可以组合执行其他文件操作。例如，一个文件的复制，可以创建新文件、 从旧文件读出并写入到新文件。

# 共享文件

文件共享使多个用户（进程）共享同一份文件，系统中只需保留该文件的一份副本。如果系统不能提供共享功能，那么每个需要该文件的用户都要有各自的副本，会造成对存储空间的极大浪费。为解决文件的共享使用，Linux 系统引入了两种链接：硬链接 (hard link) 与软链接（又称符号链接，即 soft link 或 symbolic link）。

## 硬链接

在树形结构的目录中，当有两个或多个用户要共享一个子目录或文件时，必须将共享文件或子目录链接到两个或多个用户的目录中，才能方便地找到该文件，如下图所示。

![][1]

在这种共享方式中，诸如文件的物理地址及其他的文件属性等信息，不再是放在目录项中，而是放在索引结点中；在文件目录中只设置文件名及指向相应索引结点的指针。在索引结点中还应有一个链接计数count，用于表示链接到本索引结点（亦即文件）上的用户目录项的数目。当count=2时，表示有两个用户目录项链接到本文件上，或者说是有两个用户共享此文件。

当用户A创建一个新文件时，它便是该文件的所有者，此时将count置为1。当有用户B要共享此文件时，在用户B的目录中增加一个目录项，并设置一指针指向该文件的索引结点。此时，文件主仍然是用户A，count=2。如果用户A不再需要此文件，不能将文件直接删除。因为，若删除了该文件，也必然删除了该文件的索引结点，这样便会便用户B的指针悬空，而用户B则可能正在此文件上执行写操作，此时用户B会无法访问到文件。因此用户A不能删除此文件，只是将该文件的count减1，然后删除自己目录中的相应目录项。用户B仍可以使用该文件。当Count=0时，表示没有用户使用该文件，系统将负责删除该文件。下图给出了用户B链接到文件上的前、后情况。

![][2]

由于硬链接是有着相同 inode 号仅文件名不同的文件，因此硬链接存在以下几点特性：

1. 文件有相同的 inode 及 data block；
2. 只能对已存在的文件进行创建；
3. 不能交叉文件系统进行硬链接的创建；
4. 不能对目录进行创建，只可对文件创建；
5. 删除一个硬链接文件并不影响其他有相同 inode 号的文件。

硬链接可由命令 link 或 ln 创建，`ln source_file target_file`，如下例子：

    $ ln demo.o link_demo.o
    $ ls -li demo.o link_demo.o
    35104097 -rwxr-xr-x  2 feizhao  staff  15476 Apr 15 19:15 demo.o
    35104097 -rwxr-xr-x  2 feizhao  staff  15476 Apr 15 19:15 link_demo.o

## 软链接

还可以由系统创建一个 LINK 类型的新文件，并将 LINK 文件写入用户B的目录中，以实现用户B的目录与文件的链接。在新文件中只包含被链接文件的路径名，这样的链接方法被称为符号链接。（软连接是建立了一个iNode，专门用来指向实际文件的iNode，有点像Win下的快捷方式）。

在利用符号链方式实现文件共享时，只有文件的拥有者才拥有指向其索引结点的指针。而共享该文件的其他用户则只有该文件的路径名，并不拥有指向其索引结点的指针。这样，也就不会发生在文件主删除一共享文件后留下一悬空指针的情况。当文件的拥有者把一个共享文件删除后，其他用户通过符号链去访问它时，会出现访问失败，于是将符号链删除，此时不会产生任何影响。

在符号链的共享方式中，当其他用户读共享文件时，需要根据文件路径名逐个地查找目录，直至找到该文件的索引结点。因此，每次访问时，都可能要多次地读盘，使得访问文件的开销。此外，符号链的索引结点也要耗费一定的磁盘空间。但是符号链方式有一个很大的优点，即网络共享只需提供该文件所在机器的网络地址以及该机器中的文件路径即可。

软链接与硬链接不同，若文件用户数据块中存放的内容是另一文件的路径名的指向，则该文件就是软连接。软链接就是一个普通文件，只是数据块内容有点特殊。软链接有着自己的 inode 号以及用户数据块，因此软链接的创建与使用没有类似硬链接的诸多限制：

* 软链接有自己的文件属性及权限等；
* 可对不存在的文件或目录创建软链接，但直到这个名字对应的文件被创建后，才能打开其链接。
* 软链接有自己的inode，并在磁盘上有一小片空间存放路径名。因此，软链接能够跨文件系统，也可以和目录链接！
* 创建软链接时，链接计数 i_nlink 不会增加；
* 删除软链接并不影响被指向的文件，但若被指向的原文件被删除，则相关软连接被称为死链接（即 dangling link，若被指向路径文件被重新创建，死链接可恢复为正常的软链接）。

软链接可以用 ln -s 选项创建，`ln -s source_file target_file`，如下例子：

    $ ln -s demo.o softlink
    $ ls -li demo.o softlink
    35104097 -rwxr-xr-x  1 feizhao  staff  15476 Apr 15 19:15 demo.o
    35392402 lrwxr-xr-x  1 feizhao  staff      6 Apr 27 11:39 softlink -> demo.o

上述两种链接方式都存在一个共同的问题，有链接的文件中，文件有两个或者多个路径。查找一指定目录的全部文件的程序将多次定位到被连接的文件。那么一个将某一目录及其子目录下的全部文件转储到磁带上的程序有可能多次复制一个被链接的文件。

［[硬链接和软连接解读](http://www.nowcoder.com/questionTerminal/1b695f9055ed4017a9fe578ef8b02c34)］

# 文件保护

为了防止文件共享可能会导致文件被破坏或未经核准的用户修改文件，文件系统必须控制用户对文件的存取，即解决对文件的读、写、执行的许可问题。为此，必须在文件系统中建立相应的文件保护机制。

文件保护通过口令保护、加密保护和访问控制等方式实现。其中，口令保护和加密保护是为了防止用户文件被他人存取或窃取，而访问控制则用于控制用户对文件的访问方式。

对文件的保护可以从限制对文件的访问类型中出发。可加以控制的访问类型主要有以下几种：

* 读：从文件中读。
* 写：向文件中写。
* 执行：将文件装入内存并执行。
* 添加：将新信息添加到文件结尾部分。
* 删除：删除文件，释放空间。
* 列表清单：列出文件名和文件属性。

此外还可以对文件的重命名、复制、编辑等加以控制。这些高层的功能可以通过系统程序调用低层系统调用来实现。保护可以只在低层提供。例如，复制文件可利用一系列的读请求来完成。这样，具有读访问用户同时也具有复制和打印的权限了。

解决访问控制最常用的方法是根据用户身份进行控制。而实现基于身份访问的最为普通的方法是为每个文件和目录增加一个访问控制列表(Access-Control List, ACL)，以规定每个用户名及其所允许的访问类型。

这种方法的优点是可以使用复杂的访同方法。其缺点是长度无法预期并且可能导致复杂的空间管理，使用精简的访问列表可以解决这个问题。精简的访问列表釆用拥有者、组和其他三种用户类型。

* 拥有者：创建文件的用户。
* 组：一组需要共享文件且具有类似访问的用户。
* 其他：系统内的所有其他用户。

这样只需用三个域列出访问表中这三类用户的访问权限即可。文件拥有者在创建文件时，说明创建者用户名及所在的组名，系统在创建文件时也将文件主的名字、所属组名列在该文件的FCB中。用户访问该文件时，按照拥有者所拥有的权限访问文件，如果用户和拥有者在同一个用户组则按照同组权限访问，否则只能按其他用户权限访问。UNIX操作系统即釆用此种方法。

口令和密码是另外两种访问控制方法。口令指用户在建立一个文件时提供一个口令，系统为其建立FCB时附上相应口令，同时告诉允许共享该文件的其他用户。用户请求访问时必须提供相应口令。这种方法时间和空间的开销不多，缺点是口令直接存在系统内部，不够安全。

密码指用户对文件进行加密，文件被访问时需要使用密钥。这种方法保密性强，节省了存储空间，不过编码和译码要花费一定时间。口令和密码都是防止用户文件被他人存取或窃取，并没有控制用户对文件的访问类型。

注意两个问题：

* 现代操作系统常用的文件保护方法，是将访问控制列表与用户、组和其他成员访问控制方案一起组合使用。
* 对于多级目录结构而言，不仅需要保护单个文件，而且还需要保护子目录内的文件, 即需要提供目录保护机制。目录操作与文件操作并不相同，因此需要不同的保护机制。

# 文件系统的实现

// TODO

# 磁盘管理

磁盘(Disk)是由表面涂有磁性物质的金属或塑料构成的圆形盘片，通过一个称为磁头 的导体线圈从磁盘中存取数据。在读/写操作期间，磁头固定，磁盘在下面高速旋转。如下图所示：

![磁盘结构][3]

磁盘的盘面上的数据存储在一组同心圆中，称为磁道。每个磁道与磁头一样宽, 一个盘面有上千个磁道。磁道又划分为几百个扇区，每个扇区固定存储大小（通常为512B), 一个扇区称为一个盘块。相邻磁道及相邻扇区间通过一定的间隙分隔开，以避免精度错误。

硬盘的存取访问时间分为三个部分：寻道时间Ts，旋转延迟时间Tr和传送时间Tt。

* 寻道时间Ts：活动头磁盘在读写信息前，将磁头移动到指定磁道所需要的时间。这个时间除跨越n条磁道的时间外，还包括启动磁臂的时间s。
* 旋转延迟时间Tr：磁头定位到某一磁道的扇区（块号）所需要的时间。
* 传输时间Tt：从磁盘读出或向磁盘写入数据所经历的时间，这个时间取决于每次所读/写的字节数b和磁盘的旋转速度

由于存储介质的特性，磁盘本身存取就比主存慢很多，再加上机械运动耗费，磁盘的存取速度往往是主存的几百分分之一，因此为了提高效率，要尽量减少磁盘I/O。为了达到这个目的，磁盘往往不是严格按需读取，而是每次都会预读，即使只需要一个字节，磁盘也会从这个位置开始，顺序向后读取一定长度的数据放入内存。这样做的理论依据是计算机科学中著名的局部性原理：

* 当一个数据被用到时，其附近的数据也通常会马上被使用。
* 程序运行期间所需要的数据通常比较集中。

由于磁盘顺序读取的效率很高（不需要寻道时间，只需很少的旋转时间），因此对于具有局部性的程序来说，预读可以提高I/O效率。

## RAID 

独立磁盘冗余阵列(RAID，Redundant Array of Independent Disks)，基本思想就是把多个相对便宜的硬盘组合起来，成为一个硬盘阵列组，使性能达到甚至超过一个价格昂贵、容量巨大的硬盘。RAID通常被用在服务器电脑上，使用完全相同的硬盘组成一个逻辑扇区，因此操作系统只会把它当做一个硬盘。RAID分为不同的等级，各个不同的等级均在数据可靠性及读写性能上做了不同的权衡。 在实际应用中，可以依据自己的实际需求选择不同的RAID方案。

1. RAID 0：无差错控制的带区组
    
    要实现RAID0必须要有两个以上硬盘驱动器，RAID0实现了带区组，数据并不是保存在一个硬盘上，而是分成数据块保存在不同驱动器上。在所有的级别中，RAID 0的速度是最快的。但是RAID 0没有冗余功能的，如果一个磁盘(物理)损坏，则所有的数据都无法使用。

2. RAID 1：镜像结构

    当主硬盘损坏时，镜像硬盘就可以代替主硬盘工作。镜像硬盘相当于一个备份盘，可想而知，这种硬盘模式的安全性是非常高的，RAID 1的数据安全性在所有的RAID级别上来说是最好的。但是其磁盘的利用率却只有50%，是所有RAID级别中最低的。

3. RAID5：分布式奇偶校验的独立磁盘结构

    RAID5最大的好处是在一块盘掉线的情况下，RAID照常工作，相对于RAID0必须每一块盘都正常才可以正常工作的状况容错性能好多了。因此 RAID5是RAID级别中最常见的一个类型。RAID5校验位即P位是通过其它条带数据做异或(xor)求得的。计算公式为 P=D0xorD1xorD2…xorDn，其中p代表校验块，Dn代表相应的数据块，xor是数学运算符号异或。

4. RAID10：高可靠性与高效磁盘结构

    RAID 10是先镜像再分区数据。是将所有硬盘分为两组，视为是RAID 0的最低组合，然后将这两组各自视为RAID 1运作。RAID 10有着不错的读取速度，而且拥有比RAID 0更高的数据保护性。
    
# 更多阅读

[理解 Linux 的硬链接与软链接](https://www.ibm.com/developerworks/cn/linux/l-cn-hardandsymb-links/)  
[共享文件：硬链接和软链接](http://c.biancheng.net/cpp/html/2622.html)  
[RAID 工作模式](http://www.nowcoder.com/test/question/done?tid=2502355&qid=14636#summary)  
[硬盘的读写原理](http://blog.csdn.net/hguisu/article/details/7408047)  
[RAID技术介绍和总结](http://blog.jobbole.com/83808/)  


[1]: http://7xrlu9.com1.z0.glb.clouddn.com/Linux_OS_FileSystem_1.jpg
[2]: http://7xrlu9.com1.z0.glb.clouddn.com/Linux_OS_FileSystem_2.jpg
[3]:http://7xrlu9.com1.z0.glb.clouddn.com/Linux_OS_FileSystem_3.jpg


