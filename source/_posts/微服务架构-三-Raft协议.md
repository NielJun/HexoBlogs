---
title: '微服务架构[三] Raft协议'
date: 2020-02-10 11:08:33
tags: [Go,微服务,Micro Service]
categories: Go栈
---

## 微服务架构 之 Raft协议

### 使用场景

- ​	解决分布式系统一致性的问题
- ​    基于复制的方式

### 工作机制

- Leader选举
- 日志复制
- 安全性

### 角色

- Follower 角色
- Leader 角色
- Candicate 角色

### 任期( Term ) 概念

在raft协议中，将时间分为一个个 term（任期），每一个任期内都可正常操作。但是如果主节点挂掉则必须通过选举才可以进行Leader节点的选择。

![](/5.png)

![](/6.png)

- Raft 把时间分割成任意长度的任期（term），如图 5 所示。任期用连续的整数标记。
- 每一段任期从一次选举开始，一个或者多个 candidate 尝试成为 leader 。如果一个 candidate 赢得选举，然后他就在该任期剩下的时间里充当 leader 。在某些情况下，一次选举无法选出 leader 。在这种情况下，这一任期会以没有 leader 结束；一个新的任期（包含一次新的选举）会很快重新开始。
- Raft 保证了在任意一个任期内，最多只有一个 leader

个人理解：

- 在raft算法中，比较谁的数据最新有2个参考指标，任期和logIndex，任期大的节点，数据一定最新，任期一样的话，就要比较该任期内谁的MaxLogIndex最大了。引入任期的概念可以简化数据比较的精度。

### 任期的作用

- 不同的服务器节点观察到的任期转换的次数可能不同，在某些情况下，一个服务器节点可能没有看到 leader 选举过程或者甚至整个任期全程。
- 任期在 Raft 算法中充当逻辑时钟的作用，这使得服务器节点可以发现一些过期的信息比如过时的 leader 。
- 每一个服务器节点存储一个当前任期号，该编号随着时间单调递增。
- 服务器之间通信的时候会交换当前任期号；
- 如果一个服务器的当前任期号比其他的小，该服务器会将自己的任期号更新为较大的那个值。
- 如果一个 candidate 或者 leader 发现自己的任期号过期了，它会立即回到 follower 状态。（所以说老leader如果发生了网络分区，后来接收到新leader的心跳的时候，比拼完任期之后，会自动变成follower。
- 如果一个节点接收到一个包含过期的任期号的请求，它会直接拒绝这个请求

### 状态机复制

其实很简单，通过每个节点执行一样的命令序列，则最终会得到相同的状态，Raft就是通过状态机的方式来达到一致性的效果。

### 心跳和超时机制

raft使用两个timeout机制来控制leader的选举，当选举完成后，leader会向所有的从节点发心跳包确认存活，再Leader挂掉的时候，操作是不可用的。

Raft的Heartbeat是leader在选举成功后巩固自己地位和同步信息的一种方式，日志的复制大多数情况下适合Heartbeat是同步进行的。当一个raft节点选举成为leader后，该节点周期性执行的tick函数指向了raft.tickHeartbeat，对于leader而言主要是通知follower自己还活着，与此同时leader在收到follower对Heartbeat的响应之后，又会向日志比较落后的follower发送追加日志请求，进行log replication。

```go
func (r *raft) tickHeartbeat() {
    r.heartbeatElapsed++
    // ... 

    // 如果心跳计时超过了心跳包的发送间隔，就进入发送心跳包流程，并重置心跳计时
    if r.heartbeatElapsed >= r.heartbeatTimeout {
        r.heartbeatElapsed = 0 
        r.Step(pb.Message{From: r.id, Type: pb.MsgBeat})
    }   
}

func (r *raft) Step(m pb.Message) error {
    // ...

    // Step最终调用raft.step变量指向的函数
    // 现阶段当前节点处于leader状态，所以step指向stepLeader 
    r.step(r, m)
    return nil 
}
```

stepLeader中心跳发送代码相关执行流程如下：

```go
func stepLeader(r *raft, m pb.Message) {
    switch m.Type {
    case pb.MsgBeat：
        // 把心跳包广播出去
        r.bcastHeartbeat()
    return
    // ...
}

// 在bcastHeartbeat中循环的把心跳包发送出去
func (r *raft) bcastHeartbeat() {
    for id := range r.prs {
        if id == r.id {
            continue
        }
        r.sendHeartbeat(id)
        r.prs[id].resume()
    }
}
func (r *raft) sendHeartbeat(to uint64) {
    // r.raftLog.committed 已经拷贝到大多数节点上的日志index
    // r.prs[to].Match拷贝到to这个节点的日志最大下标
    commit := min(r.prs[to].Match, r.raftLog.committed)
    m := pb.Message{
        To:     to,
        Type:   pb.MsgHeartbeat,
        Commit: commit,
    }
    //把心跳包发送出去
    r.send(m)
}
```

follower在收到心跳包之后，最终处理心跳的包的流程会通过Step->step->stepFollower:

```go
func stepFollower(r *raft, m pb.Message) {
    switch m.Type {
    case pb.MsgHeartbeat:
        r.electionElapsed = 0
        r.lead = m.From
        r.handleHeartbeat(m)
    case //...
    }
}

// 更新本地可以提交的日志的最大下标，然后返回响应
func (r *raft) handleHeartbeat(m pb.Message) {
    r.raftLog.commitTo(m.Commit)
    r.send(pb.Message{To: m.From, Type: pb.MsgHeartbeatResp})
}

```

leader的心跳响应收到之后处理流程依旧会通过Step->step->stepLeader:

```go
func stepLeader(r *raft, m pb.Message) {
    switch m.Type {
        // ...
    case pb.MsgHeartbeatResp:
        pr.RecentActive = true

        // free one slot for the full inflights window to allow progress.
        if pr.State == ProgressStateReplicate && pr.ins.full() {
            pr.ins.freeFirstOne()
        }
        // 检查是否还有日志没有拷贝到当前follower，如果有待发送的日志，通过sendAppend发送过去。
        if pr.Match < r.raftLog.lastIndex() {
            r.sendAppend(m.From)
        }
        case //...
    }
}
```

sendAppend函数的实现如下：

```go
func (r *raft) sendAppend(to uint64) {
    pr := r.prs[to]
    if pr.isPaused() {
        return
    }
    m := pb.Message{}
    m.To = to
    // 给该follower发送的最后一条日志的term
    term, errt := r.raftLog.term(pr.Next - 1)
    // 即将要发送给该follower的日志条目
    ents, erre := r.raftLog.entries(pr.Next, r.maxMsgSize)

    // 如果待发送给follower已经写到snap里面，在raftlog里面无法找到
    if errt != nil || erre != nil { // send snapshot if we failed to get term or entries
        if !pr.RecentActive {
            r.logger.Debugf("ignore sending snapshot to %x since it is not recently active", to)
            return
        }

        m.Type = pb.MsgSnap
        snapshot, err := r.raftLog.snapshot()
        if err != nil {
            if err == ErrSnapshotTemporarilyUnavailable {
                r.logger.Debugf("%x failed to send snapshot to %x because snapshot is temporarily unavailable", r.id, to)
                return
            }
            panic(err) // TODO(bdarnell)
        }
        if IsEmptySnap(snapshot) {
            panic("need non-empty snapshot")
        }
        m.Snapshot = snapshot
        sindex, sterm := snapshot.Metadata.Index, snapshot.Metadata.Term
        r.logger.Debugf("%x [firstindex: %d, commit: %d] sent snapshot[index: %d, term: %d] to %x [%s]",
            r.id, r.raftLog.firstIndex(), r.raftLog.committed, sindex, sterm, to, pr)
        pr.becomeSnapshot(sindex)
        r.logger.Debugf("%x paused sending replication messages to %x [%s]", r.id, to, pr)
    } else {
        m.Type = pb.MsgApp
        m.Index = pr.Next - 1
        m.LogTerm = term
        m.Entries = ents
        // leader记录的最大的已经copy到大多数节点上的日志下标
        m.Commit = r.raftLog.committed
        if n := len(m.Entries); n != 0 {
            switch pr.State {
            // optimistically increase the next when in ProgressStateReplicate
            case ProgressStateReplicate:
                last := m.Entries[n-1].Index
                pr.optimisticUpdate(last)
                pr.ins.add(last)
            case ProgressStateProbe:
                pr.pause()
            default:
                r.logger.Panicf("%x is sending append in unhandled state %s", r.id, pr.State)
            }
        }
    }
    r.send(m)
}
```

follower在收到leader发送过来的追加日志请求时，会通过handleAppendEntries去处理消息：

```go
func (r *raft) handleAppendEntries(m pb.Message) {
    // r.raftLog.committed实在当前节点已经提交的日志
    // m.Index为leader发送给该follower的上一条日志索引
    // m.Index < r.raftLog.committed情况多出现在刚刚选出的leader向follower拷贝日志，leader本地还未记录向该follower拷贝了多少日志
    // 出现这种请求leader会把自己已经提交的日志的最大下标发送给leader，下次leader就会从r.raftLog.committed + 1开始发送日志
    if m.Index < r.raftLog.committed {
        r.send(pb.Message{To: m.From, Type: pb.MsgAppResp, Index: r.raftLog.committed})
        return
    }
    // leader拷贝日志有可能在follower已经存在，所以要先找到相对与follower日志比较新的日志下标
    if mlastIndex, ok := r.raftLog.maybeAppend(m.Index, m.LogTerm, m.Commit, m.Entries...); ok {
        // 当leader发送过来的上次发送给该follower的最后一条日志信息与follower本地储存的无冲突时，
        // 返回follower本地已经写入的最大日志下标，下次leader就会从该下标发送日志，并会更新leader记录的该follower的日志发送信息
        r.send(pb.Message{To: m.From, Type: pb.MsgAppResp, Index: mlastIndex})
    } else {
        // 最后leader发送过来的最后一条日志的index，term等信息与follower本地记录的不一样
        r.send(pb.Message{To: m.From, Type: pb.MsgAppResp, Index: m.Index, Reject: true, RejectHint: r.raftLog.lastIndex()})
    }
}

func (l *raftLog) maybeAppend(index, logTerm, committed uint64, ents ...pb.Entry) (lastnewi uint64, ok bool) {
    lastnewi = index + uint64(len(ents))
    if l.matchTerm(index, logTerm) {
        // 找到leader发送给follower的第一条新日志的位置
        ci := l.findConflict(ents)
        switch {
        // 说明leader拷贝过来的日志都在follower的raftlog里面有记录
        case ci == 0:
        // 日志冲突的下标小于follower本地已经提交的日志，出现bug了
        case ci <= l.committed:
            l.logger.Panicf("entry %d conflict with committed entry [committed(%d)]", ci, l.committed)
        default:
           // 把leader拷贝过来的新日志写入follower本地
            offset := index + 1 
            l.append(ents[ci-offset:]...)
        }
        l.commitTo(min(committed, lastnewi))
        // 返回follower本地最新的日志记录index
        return lastnewi, true
    }   
    return 0, false
}
```

leader收到的follower对日志追加请求的响应主要有两种：follower拒绝追加日志，follower追加0-n条日志，相关响应的处理代码如下:

```go
func stepLeader(r *raft, m pb.Message) {
    // ...
    switch m.Type {
    case pb.MsgAppResp:
        pr.RecentActive = true

        if m.Reject {
            // 拒绝追加日志
            // follower拒绝追加日志时都会返回一个RejectHint，RejectHint表示当前follower最后一条日志的index
            // maybeDecrTo函数在follower为ProgressStateReplicate状态是会把pr[to].Next设置为pr[to].Match + 1，否者会设置为被设置为m.Index、m.RejectHint中较小的一个，然后继续尝试发送。
            if pr.maybeDecrTo(m.Index, m.RejectHint) {
                if pr.State == ProgressStateReplicate {
                    pr.becomeProbe()
                }
                r.sendAppend(m.From)
            }
        } else {
            oldPaused := pr.isPaused()
            // 如果又在follower上追加日志，更新以下两个下标:
            // （1）follower已经和leader一致的最大日志下标:pr[to].Match = m.Index
            // （2）下一条需要拷贝给该follower的日志下标:pr[to].Next = m.Index + 1
            if pr.maybeUpdate(m.Index) {
                switch {
                case pr.State == ProgressStateProbe:
                    pr.becomeReplicate()
                case pr.State == ProgressStateSnapshot && pr.maybeSnapshotAbort():
                    pr.becomeProbe()
                case pr.State == ProgressStateReplicate:
                    pr.ins.freeTo(m.Index)
                }
                // 如果该follower有追加成功的日志，可能会出现新的日志条目拷贝到大多数raft节点上，
                // 因此leader更新本地raftlog.commitid，然后通过bcastAppend把新更新的commit(如果follower日志落后于leader，可能携带日志)广播给follower，
                // follower更新本地已经提交到大多数机器上的日志下标，本地小于commit未apply的日志就可以apply了
                if r.maybeCommit() {
                    r.bcastAppend()
                } else if oldPaused {
                    // update() reset the wait state on this node. If we had delayed sending
                    // an update before, send it now.
                    r.sendAppend(m.From)
                }
            }
        }
    case // ...
}
```

### Leader选举

#### 触发条件

- 一般条件下，追随者接到领导者的心跳的时候，把选举定时器清零，不会触发
- 最随着的选举定时器超时发生时（比如Leader故障了），那从节点会变成候选者，触发领导人选举

#### 选举过程

- 一开始，所有节点都是以Follow角色启动，同时启动选举定时器（时间随机，降低冲突概率）
- 当定时器到期的时候，转为候选人角色【candicate】
- 把当前任期+1并且为自己投票
- 发起RequestVote的RPC请求，要求其他节点为自己投票
- 如果得到办事以上的节点同意，自己称为Leader
- 如果选举超时，还没有Leader产生，则进入下一任期，重新选举

#### 限制条件

假定这样一个情况，1号节点作为Leader在同步完成3号以后，在对2号同步数据的时候挂掉了，那如果2号 [未携带合法数据] 参与选举则会引起错误。所以必须规避这些问题。

![](/7.png)

- 每个接待你在**一个任期里面只能投一个票**，采用先到先服务的原则
- 如果没有投过票，对比候选人节点的Log和当前节点的Log哪个更新，比较方式为**谁的lastLog的term越大谁越新**，如果term相同，**谁的lastLog的index谁越大谁越新**，如果当前节点更新，则拒绝投票。

### 日志复制

Raft状态机图示

![](/9.png)

 Leader将命令并发的复制给其他节点，并等待其他节点将命令写入到日志中，如果此时有些节点失败或者比较慢，Leader节点会一直重试，所有的节点都保存到日志中，之后Leader节点就提交命令， 在接下来的心跳包所有的从节点都会提交对应的修改，并将节点返回给客户端。

![](/8.png)

当客户端向服务器发送修改x的请求时，主节点接收到，并且修改掉自己的日志，但并未提交 ( comit )。

![](/10.png)

此时Leader会告诉所有的节点修改日志，但是都未提交。

![](/11.png)

从节点收到以后，回复Leader收到。

![](/12.png)

Leader节点Comit，然后通过心跳包所有的从节点提交。

![](/13.png)

### 安全性

一个作为候选人的节点要赢得选举，就需要喝网络中大部分节点进行通信，这就意味着已经提交的乳汁条目最少在其中一台服务器节点上出现，如果候选人的日志至少和大多数的服务器上的日志一样新，那么他一定包含全部已经提交的日志条目。

RequestVote RPC 实现了这个限制： RPC包括勾选人的日志信息，如果它自己的日志比候选人的日志要新，他会拒绝给候选人投票。

### 最新判断标准

- 如果两个日志的任期号不同，任期号打的日志内容更新
- 如果任期号一样大，则更具日志中最后一个命令的索引(Index)，谁大谁最新

### 数据安全性

在我们保证了选举时候选出的Leader必须是携带者最新的日志的同时，也需要保证数据的最新性。如下：

![](/15.png)

当我们的节点存在于多个路由器或者交换机下时，比如 D、E、F、节点处在交换机子网2中间，其中D为Leader节点。A、B、C是处于子网1中间。正常通信的时候通过分布式一致性协议下不会有任何问题。

但是当出现网络问题的时候，比如两个交换机之间出现网络分区的时候【老列】，我们仔细想一下这个问题。

- 正常通信的时候，分布式一致性协议会保证每个节点的数据一致

- 当网络分区出现时，A、B、C节点给D节点发送收不到回应，则会认为Leader D挂了

- 在A、B、C中间会进行Leader选举，假定A变成了Leader

- 与此同时，客户端发送数据修改请求，因为旧的Leader D节点收不到来自A、B、C、的修改确认回应，他会一直重复发心跳包而停止提交修改。
- A节点作为新的Leader接受了客户端的请求并且同步到了B、C节点。

当某一时刻，网络情况恢复了以后 :

- 新的Leader节点 A  携带的数据最新，D节点会发送心跳包给其他节点，D节点由于Term和数据携带不够新而被舍弃
- A发送心跳包同步到其他节点，当然也包括D节点，D节点发现A的数据包更新，于是丢弃Leader职位成为Follower。于是A节点成为Leader，一切正常

![](/16.png)

最后，推荐 [Raft](http://thesecretlivesofdata.com/raft/) 动画演示。可以很清楚了解raft协议的一些过程。

