# blocking

## 资源

csapp-cache章节：http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/recitations/rec07.pdf



## 简要说明

对于访问一个矩阵数据，我们单独访问一行加一列会得到(n/(缓存行简称c))+n个，对于访问多个只需要叠加即可。

而blocking技术是同过在我们访问多行时，将我们读取列时得缓存抖动变为读取行时得缓存抖动，此时每个块为(B/c)*B

因此我们读相同多得行列得到得缓存不命中分别为((n/c)+n)B和nB/4因此我们可以得到(c+1)/c一定大于(1/4)，所以未命中率减少，blocking有用；

在做cache-lab第二部分十分有用