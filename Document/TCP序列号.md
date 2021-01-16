序列号分析

clinet                                    server
1   -->SYN        Seq=0         NextSeq=0       Ack=0      
2   <--SYN,ACK    Seq=0         NextSeq=0       Ack=1
3   -->ACK        Seq=1         NextSeq=1       Ack=1
4   -->Get        seq=1         NextSeq=628     Ack=1     len=627
5   <--ACK        seq=1         NextSeq=1       Ack=628   
6   <--ACK        seq=1         NextSeq=1407    Ack=628   len=1406
7   <--ACK        seq=1407      NextSeq=2813    Ack=628   len=1406
8   -->ACK        seq=628       NextSeq=628     Ack=2813  
9   <--ACK        seq=2813      NextSeq=4219    Ack=628   len=1406  
10  <--PUSH,ACK   seq=4219      NextSeq=4321    Ack=628   len=102  
