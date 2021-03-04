1. 对 Kubernetes 了解怎么样，看过源码吗？
    kubelet、apiserver、scheduler、controller-manager

2. Kubernetes 的 Service 是什么概念，怎么实现的？

3. Informer 是怎么实现的，有什么作用？

4. StatefulSet 用过吗？有什么特点？

5. StatefulSet 的滚动升级是如何实现的？

6. 现在我们希望只升级 StatefulSet 中的任意个节点进行测试, 可以怎么做? 
    这题没有思路，只好强答用”两个 StatefulSet”，后来一想起一个新的 StatefulSet 那 PV 里的数据就丢了，其实正确办法是利用 partition 机制，笑容渐渐消失。

7. Kubernetes 的所有资源约定了版本号, 为什么要这么做?
    

8. 假如有多几个版本号并存, 那么 K8S 服务端需要维护几套代码?
    
9. ADD 和 copy 的区别
     add会将压缩文件给解压缩,而copy不会;(add 貌似可以加载网络文件,但是这个做法并不推荐)

10. k8s 请求到达 APIserver 的整个流程

11. docker 的网络原理，底层实现


