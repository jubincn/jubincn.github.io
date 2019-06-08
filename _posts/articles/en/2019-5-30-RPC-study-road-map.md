# Why RPC is important?
Remote Method Call is to solve how to pass messages in distributed system. The stability and performance of RPC will affect the whole system.

# Components of RPC
1. Transport
- TCP
2. I/O
- Blocking I/O
- Non-blocking I/O
- I/O Multiplexing
- Asynchronous I/O
3. Process/Thread Model
- [Reactor Pattern](http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf)
- [Proactor Pattern](http://www.cs.wustl.edu/~schmidt/PDF/proactor.pdf)
4. Schema & Data Serialization
- Encoding format
- Schema declaration
- 语言平台的中立性
- 新老契约的兼容性
- 和压缩算法的契合度
- Performance
5. Wire Protocol (Wire Frame)
6. Reliability

# References
[体系化认识 RPC](https://www.infoq.cn/article/get-to-know-rpc)
