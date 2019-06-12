### 死锁

### 字节or字符
    char_length计算的是字符长度，而length计算的是字节长度，刚好我使用的是utf8，一个汉字占3个字节，占一个字符。
    select char_length('你你你你你'); 结果为5
    select length('你你你你你'); 结果为15

    在我们新建表的时候，会指定varchar(20)，在mysql4.x之前，指定的是字节的长度。在mysql5.x之后，指定的是字符的长度。