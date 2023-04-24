
## 行存储方式

### Redundant行格式

<table border="none">
    <tbody>
        <td style="background:red;color:white;">所有字段长度列表</td>
        <td style="background:blue;color:white;">记录头信息</td>
        <td style="background:green;color:white;">隐藏列</td>
        <td style="background:pink;color:white;">不为NULL值的列值</td>
    </tbody>
</table>

### Compact行格式

<table border="none">
    <tbody>
        <td style="background:red;color:white;">变长字段长度列表</td>
        <td style="background:blue;color:white;">NULL值列表</td>
        <td style="background:green;color:white;">记录头信息</td>
        <td style="background:gray;color:white;">row_id</td>
        <td style="background:gray;color:white;">trx_id</td>
        <td style="background:gray;color:white;">roll_ptr</td>
        <td style="background:pink;color:white;">不为NULL值的列值</td>
    </tbody>
</table>

### DYNAMIC

### COMPRESSED

* [博客 - MySQL 的 NULL 值是怎么存储的？](https://www.cnblogs.com/xiaolincoding/p/16941244.html)
* [掘金 - InnoDB引擎-四种行记录存储](https://juejin.cn/post/7004459098682949645)
* [掘金 - InnoDB引擎-行记录存储 Redundant行格式](https://juejin.cn/post/6847902223817506829)
* [掘金 - InnoDB引擎-行记录存储 Compact行格式](https://juejin.cn/post/6844903470860861454)
