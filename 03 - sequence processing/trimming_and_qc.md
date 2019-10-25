序列预处理：trimming和QC
================
Wei-Hua Chen (CC BY-NC 4.0)
25 October, 2019

## 学习要求

  - 操作系统为 Linux (Ubuntu或类似版本)，或 MacOS
  - 具有Linux系统操作基础

## 1\. 使用 `trimmomatic` 程序去除`接头、低质量碱基和短序列`

`rawdata` 在使用之前，需要去除`接头(adaptors)`，`低质量`和`短序列`。这些任务都可由 `trimmomatic`
程序完成。

### 1.1. 下载`trimmomatic` 程序

  - 在浏览器中输入地址： <http://www.usadellab.org/cms/?page=trimmomatic>

  - 选择并最新版的 `binary` 文件下载。目前最新版为 `Version 0.39` （2019年10月）

  - 下载文件为 `.zip` 格式，可用 `unzip` 命令解压。会得到：

`trimmomatic-0.39.jar` 文件和 `adaptors/` 目录。

### 1.2. 处理双端数据 (pair-end or PE data)

假设双端序列文件分别为（经gzip压缩）：

``` sh numberLines lineAnchors
SRR12345_1.fq.gz
SRR12345_2.fq.gz
```

其中`SRR12345_1.fq.gz`又称为 `forward` 文件，`SRR12345_2.fq.gz` 又称为 `reverse`
文件。详情见 [Illumina
官方说明](https://support.illumina.com/bulletins/2016/04/fastq-files-explained.html)。

`trimmomatic` 命令如下：

``` sh numberLines lineAnchors
java -jar /path/to/trimmomatic-0.39.jar PE \
    SRR12345_1.fq.gz SRR12345_2.fq.gz \
    SRR12345_1_paired.trimmed.fq.gz SRR12345_1_unpaired.trimmed.fq.gz \
    SRR12345_2_paired.trimmed.fq.gz SRR12345_2_unpaired.trimmed.fq.gz \
    ILLUMINACLIP:TruSeq3-PE.fa:2:30:10:2:keepBothReads LEADING:3 TRAILING:3 MINLEN:36
```

说明：

  - line 1, `/path/to/trimmomatic-0.39.jar` ： 在运行时需要被真实的
    `trimmomatic-0.39.jar` 的位置替代，比如它在磁盘上的位置为：
    `/home/wchen/Downloads/Trimmomatic-0.39/trimmomatic-0.39.jar`
    ，那么相应的命令为：

`java -jar /home/wchen/Downloads/Trimmomatic-0.39/trimmomatic-0.39.jar`

  - line 1, `PE` ：表示输入序列是双端序列

  - line 2, `SRR12345_1.fq.gz SRR12345_2.fq.gz` :
    两个输入文件，可以是压缩格式（.gz），也可以非压缩

  - line 3, `SRR12345_1_paired.trimmed.fq.gz
    SRR12345_1_unpaired.trimmed.fq.gz` : `SRR12345_1.fq.gz`文件 trim
    后的两个输出。

其中仍然成对的序列存入 `SRR12345_1_paired.trimmed.fq.gz`，不成对的存入
`SRR12345_1_unpaired.trimmed.fq.gz`。后者是指对应的`reverse`序列trim后不符合**筛选标准**而变成
`orphan reads` 。

文件名可以随意取名；以 `.gz` 以结尾时，输出压缩格式，否则输出 `plain text` 普通文本格式。

  - line 4, `SRR12345_2_paired.trimmed.fq.gz
    SRR12345_2_unpaired.trimmed.fq.gz` : `SRR12345_2.fq.gz`文件 trim
    后的两个输出。

\*\* 注意 \*\* ： 通常trim后成对合格的reads，即： `SRR12345_1_paired.trimmed.fq.gz`
`SRR12345_2_paired.trimmed.fq.gz` 两个文件才会用于后期分析。

  - line 5, `TruSeq3-PE.fa` 接头文件；此文件不在当前目录下时，需要指定路径，比如：

`ILLUMINACLIP:adapters/TruSeq3-PE.fa:2:30:10:2:keepBothReads`

其它接头文件在 `adapters/` 目录下。

  - line 5， 详解如下：
    
      - Remove adapters (ILLUMINACLIP:TruSeq3-PE.fa:2:30:10)
      - Remove leading low quality or N bases (below quality 3)
        (LEADING:3)
      - Remove trailing low quality or N bases (below quality 3)
        (TRAILING:3)
      - Scan the read with a 4-base wide sliding window, cutting when
        the average quality per base drops below 15 (SLIDINGWINDOW:4:15)
      - Drop reads below the 36 bases long
        (MINLEN:36)；去接口和低质量碱基后，如果长度小于36，则去除。**注意**
        这个长度通常限定在测序长度的一半。比如：读长为200bp时，`MINLEN:100`甚至更长比较合适。

### 1.3. 处理单端序列 （`single end, SE` 序列 ）

假设单端序列文件为： `SRR456789.fq.gz`：

``` sh numberLines lineAnchors
java -jar trimmomatic-0.35.jar SE \
     -phred33 \
     SRR456789.fq.gz \
     SRR456789.trimmed.fq.gz \
     ILLUMINACLIP:TruSeq3-SE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```

**说明**：

  - line 1, `SE` : 表示单端

  - line 2, `-phred33` : 表示质量文件为 `phred33` ，默认值

  - line 5，`TruSeq3-SE.fa` 接头文件；此文件不在当前目录下时，需要指定路径，比如：

`ILLUMINACLIP:adapters/TruSeq3-SE.fa:2:30:10:2:keepBothReads`

其它接头文件在 `adapters/` 目录下。

  - line 5 的其它参数意义见上面双端的解释。

## 2\. FASTQC工具进行序列质量评估