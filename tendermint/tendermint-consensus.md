
consensusState是tendermint中最重要的一个文件，里面有状态切换的状态机存放在其中．
consensusState 会调用各种资源来完成状态的接受与转变及外部的交互．

peerMsgQueue : 外部节点收到的消息,主要包括三种:proposal,blockPartMsg,voteMsg
internalMsgQueue:自己内部给自己发送的消息,同样包括上述三种.
RoundState:记录当前自己的参与共识状态的必要数据.
PeerRoundState:记录当前自己连接的peer的参与共识状态的必要数据.