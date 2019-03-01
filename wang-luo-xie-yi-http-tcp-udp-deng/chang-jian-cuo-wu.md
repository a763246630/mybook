#### connect  timed out  

 是`client`发出`sync`包，server端在指定的时间内没有回复`ack`导致的.没有回复`ack`的原因可能是[网络丢包](https://www.baidu.com/s?wd=%E7%BD%91%E7%BB%9C%E4%B8%A2%E5%8C%85&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)、防火墙阻止服务端返回syn的ack包等

服务器链接超时  原因1.服务器网络错误

#### read timed out 

表示已经连接成功(即三次握手已经完成)，但是服务器没有及时返回数据(没有在设定的时间内返回数据)，导致读超时

