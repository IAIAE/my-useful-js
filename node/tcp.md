#tcp特性
##tcp的慢开始
发送方维持一个叫做拥塞窗口cwnd（congestion window）的状态变量。拥塞窗口的大小取决于网络的拥塞程度，并且动态地在变化。发送方让自己的发送窗口等于拥塞窗口，另外考虑到接受方的接收能力，发送窗口可能小于拥塞窗口。

慢开始算法的思路就是，不要一开始就发送大量的数据，先探测一下网络的拥塞程度，也就是说由小到大逐渐增加拥塞窗口的大小。

这里用报文段的个数的拥塞窗口大小举例说明慢开始算法，实时拥塞窗口大小是以字节为单位的。如下图：
![](https://github.com/IAIAE/my-useful-js/blob/master/img/20170102_1.jpg)
当然收到单个确认但此确认多个数据报的时候就加相应的数值。所以一次传输轮次之后拥塞窗口就加倍。这就是乘法增长，和后面的拥塞避免算法的加法增长比较。

为了防止cwnd增长过大引起网络拥塞，还需设置一个慢开始门限ssthresh状态变量。ssthresh的用法如下：

当cwnd\<ssthresh时，使用慢开始算法。
当cwnd>ssthresh时，改用拥塞避免算法。
当cwnd=ssthresh时，慢开始与拥塞避免算法任意。

拥塞避免算法让拥塞窗口缓慢增长，即每经过一个往返时间RTT就把发送方的拥塞窗口cwnd加1，而不是加倍。这样拥塞窗口按线性规律缓慢增长。

无论是在慢开始阶段还是在拥塞避免阶段，只要发送方判断网络出现拥塞（其根据就是没有收到确认，虽然没有收到确认可能是其他原因的分组丢失，但是因为无法判定，所以都当做拥塞来处理），就把慢开始门限设置为出现拥塞时的发送窗口大小的一半。然后把拥塞窗口设置为1，执行慢开始算法。如下图：
![](https://github.com/IAIAE/my-useful-js/blob/master/img/20170102_2.jpg)
再次提醒这里只是为了讨论方便而将拥塞窗口大小的单位改为数据报的个数，实际上应当是字节。


#启动一个tcp服务

```javascript
const app = (conn) => {
    conn.setEncoding('utf-8')
    let nickname;
    pleaseInput(conn);
    
    conn.on('data', (data)=>{
        if(!nickname){
            nickname = data.replace(/\r\n/,'');
            users[nickname] = conn;
            taleAll('in', nickname);
        }else{
            taleAll('tale', nickname, data);
        }
    });

    conn.on('close',()=>{
        delete users[nickname];
        taleAll('out', nickname);
    });
}

let net =  require('net'),
    server = net.createServer(app);
server.listen(3000,()=>{
    console.info('server listening 3000');
});
```





