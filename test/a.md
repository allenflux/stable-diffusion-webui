graph LR
    subgraph 游戏引擎 / 逻辑层
        A1[🎮 游戏引擎<br>Crash / Rocket]
        A2[💰 下单服务<br>Bet Service]
        A3[⚙️ 结算 / 兑付服务<br>Settlement & Cashout]
    end

    subgraph Redis 存储层
        B1[(game:term:state:{cid})]
        B2[(game:term:push:{cid})]
        B3[(game:channel:config:{cid})]
        B4[(game:seed:{game}:{cid}:{tid})]
        B5[(lock:*)]
        B6[(stream:cashout:tasks:{cid})]
        B7[(zset:refund:retry)]
        B8[(game:global:stats)]
    end

    subgraph 消费层
        C1[🛰️ WebSocket 推送层]
        C2[📊 监控服务 / 数据统计]
        C3[👩‍💼 后台面板 / 控制台]
    end

    %% Data Flows
    A1 -->|写入/更新状态| B1
    A1 -->|生成推送数据| B2
    A1 -->|读取配置| B3
    A1 -->|生成随机种子| B4
    A1 -->|申请状态锁| B5
    A3 -->|任务写入| B6
    A3 -->|失败重试| B7
    A1 -->|更新全局统计| B8

    %% Consumers
    B2 -->|订阅推送| C1
    B8 -->|汇总指标| C2
    B3 -->|配置修改| C3
    C3 -->|热更新配置| B3

    %% Monitoring
    B5 -->|锁状态检查| C2
    B6 -->|任务消费| C2
