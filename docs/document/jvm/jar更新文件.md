# jar命令替换jar包中的文件

## 一、jar命令用法

``` 
-c 创建新的归档文件
 
　-t 列出归档目录和文件
 
　-x 解压缩已归档的指定（或所有）文件
 
　-u 更新现有的归档文件
 
　-v 在标准输出中生成详细输出 / 提供更详细输出信息
 
　-f 指定归档文件名 / 为压缩包指定名字
 
　-m 包含指定清单文件中的清单信息 
 
　-e 为捆绑到可执行 jar 文件的独立应用程序
 
指定应用程序入口点
 
　-0 仅存储，不压缩，只是打包；不使用任何 ZIP 压缩
 
　-M 不创建条目的清单文件 META-INF/MANIFEST.MF
 
　-i 为指定的 jar 文件生成索引信息
 
　-C 更改为指定的目录并包含其中的文件
```

## 二、替换jar包中指定文件

#### 1、列出指定文件的路径

``` linux
jar -tvf xxx.jar |grep xxx.file
```

#### 2、解压指定路径下的文件

``` 
jar -xvf xxx.jar xxx/xxx.file
```

#### 3、删除需要替换的文件或修改文件

```
rm -rf xxx.file | vi xxx.file
```

#### 4、更新文件到jar包中

```
jar -uvf xxx.jar xxx/xxx.file
```

