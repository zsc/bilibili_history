# 第9章：推荐系统与算法

> 从人工编辑到AI驱动，B站推荐系统的技术演进之路

## 章节概览

B站的推荐系统经历了从纯人工运营到智能算法的完整进化历程。作为一个以用户生成内容（UGC）为核心的平台，推荐系统不仅决定了内容分发效率，更直接影响着创作者生态和用户体验。本章将深入剖析B站推荐系统的技术架构、算法演进和工程实践。

## 目录

1. [从编辑推荐到算法推荐](#从编辑推荐到算法推荐)
2. [个性化推荐架构](#个性化推荐架构)
3. [深度学习应用](#深度学习应用)
4. [用户画像系统](#用户画像系统)
5. [AB测试平台](#ab测试平台)

---

## 从编辑推荐到算法推荐

### 1.1 人工编辑时代（2009-2013）

#### 运营机制
```
┌──────────────────────────────────────────┐
│         早期推荐流程（2009-2013）         │
├──────────────────────────────────────────┤
│                                          │
│  UP主上传 → 审核员审核 → 编辑精选       │
│     ↓           ↓            ↓          │
│  普通区域    通过/拒绝    首页推荐      │
│                                          │
│  推荐位：                                │
│  - 首页Banner（3-5个）                   │
│  - 分区推荐（每区5-10个）                │
│  - 每日精选（10-20个）                   │
│                                          │
└──────────────────────────────────────────┘
```

#### 编辑团队构成
| 时期 | 团队规模 | 日处理视频量 | 推荐准确率 |
|------|----------|--------------|------------|
| 2009 | 2-3人 | 50-100 | 依赖个人品味 |
| 2010 | 5-8人 | 200-300 | ~40% |
| 2011 | 10-15人 | 500-800 | ~45% |
| 2012 | 20-30人 | 1000-1500 | ~50% |
| 2013 | 30-50人 | 2000-3000 | ~55% |

#### 问题与挑战
- **扩展性差**：人工成本随内容量线性增长
- **时效性低**：热点内容发现延迟（平均6-12小时）
- **个性化缺失**：千人一面的推荐结果
- **主观偏见**：编辑个人喜好影响推荐公平性

### 1.2 算法探索期（2014-2016）

#### 协同过滤初探
```python
# 早期协同过滤伪代码示例
class SimpleCollaborativeFilter:
    def __init__(self):
        self.user_item_matrix = {}  # 用户-视频交互矩阵
        self.similarity_cache = {}   # 相似度缓存
    
    def calculate_similarity(self, user1, user2):
        # 基于余弦相似度
        common_items = set(self.user_item_matrix[user1].keys()) & \
                      set(self.user_item_matrix[user2].keys())
        if not common_items:
            return 0
        
        # 简化的余弦相似度计算
        numerator = sum([self.user_item_matrix[user1][item] * 
                        self.user_item_matrix[user2][item] 
                        for item in common_items])
        denominator = sqrt(sum([v**2 for v in self.user_item_matrix[user1].values()])) * \
                     sqrt(sum([v**2 for v in self.user_item_matrix[user2].values()]))
        
        return numerator / denominator if denominator != 0 else 0
```

#### 混合推荐策略
```
┌─────────────────────────────────────────────┐
│          混合推荐架构（2014-2016）          │
├─────────────────────────────────────────────┤
│                                             │
│   热门推荐（30%）                           │
│      ├── 播放量排序                        │
│      ├── 弹幕数排序                        │
│      └── 硬币数排序                        │
│                                             │
│   协同过滤（40%）                           │
│      ├── User-Based CF                     │
│      ├── Item-Based CF                     │
│      └── Matrix Factorization              │
│                                             │
│   编辑推荐（20%）                           │
│      └── 人工精选内容                      │
│                                             │
│   随机探索（10%）                           │
│      └── 新内容曝光                        │
│                                             │
└─────────────────────────────────────────────┘
```

### 1.3 机器学习时代（2017-2019）

#### 特征工程体系
| 特征类别 | 具体特征 | 维度 | 重要性 |
|----------|----------|------|--------|
| 用户特征 | 年龄、性别、地域、设备 | ~50 | 高 |
| 视频特征 | 标题、标签、时长、分区 | ~200 | 高 |
| 统计特征 | CTR、完播率、互动率 | ~100 | 极高 |
| 时间特征 | 发布时间、观看时间、节假日 | ~30 | 中 |
| 创作者特征 | 粉丝数、更新频率、内容质量 | ~50 | 高 |

#### 排序模型演进
```
2017: Logistic Regression
     ↓
2018: GBDT (Gradient Boosting Decision Tree)
     ↓
2019: Wide & Deep Model
     ↓
     特征交叉 + 深度学习
```

### 1.4 深度学习革命（2020-至今）

#### 模型架构升级
```
┌──────────────────────────────────────────────┐
│         深度推荐模型架构（2020+）            │
├──────────────────────────────────────────────┤
│                                              │
│  召回层（Recall）                            │
│   ├── 协同过滤召回                          │
│   ├── 内容召回（BERT）                      │
│   ├── 图神经网络召回                        │
│   └── 实时热点召回                          │
│                    ↓                         │
│  粗排层（Pre-ranking）                       │
│   └── 轻量级DNN（~1000候选）                │
│                    ↓                         │
│  精排层（Ranking）                           │
│   └── 复杂DNN模型（~100候选）               │
│                    ↓                         │
│  重排层（Re-ranking）                        │
│   ├── 多样性优化                            │
│   ├── 业务规则                              │
│   └── 用户体验优化                          │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 用户画像系统

### 4.1 画像体系架构

#### 多层次用户画像
```
┌──────────────────────────────────────────────┐
│             B站用户画像体系                  │
├──────────────────────────────────────────────┤
│                                              │
│  基础属性层                                  │
│   ├── 人口统计：年龄、性别、地域            │
│   ├── 设备信息：机型、系统、网络            │
│   └── 账号信息：等级、大会员、注册时间      │
│                                              │
│  行为特征层                                  │
│   ├── 观看行为：时长、频次、完播率          │
│   ├── 互动行为：点赞、投币、收藏、弹幕      │
│   ├── 社交行为：关注、私信、动态            │
│   └── 创作行为：投稿、直播、专栏            │
│                                              │
│  兴趣标签层                                  │
│   ├── 内容偏好：分区、标签、UP主            │
│   ├── 时间偏好：活跃时段、观看节奏          │
│   └── 消费偏好：付费意愿、打赏习惯          │
│                                              │
│  价值分层                                    │
│   ├── 用户价值：LTV、活跃度、留存          │
│   ├── 内容价值：内容消费量、质量偏好        │
│   └── 社区价值：UGC贡献、社交影响力         │
│                                              │
└──────────────────────────────────────────────┘
```

### 4.2 标签体系建设

#### 标签分类架构
| 标签类型 | 数量级 | 更新频率 | 应用场景 |
|----------|--------|----------|----------|
| 基础标签 | 100+ | 每天 | 用户分群 |
| 兴趣标签 | 5000+ | 实时 | 内容推荐 |
| 行为标签 | 1000+ | 每小时 | 行为预测 |
| 预测标签 | 500+ | 每天 | 流失预警 |
| 圈层标签 | 200+ | 每周 | 社群运营 |

#### 标签生成Pipeline
```python
class TagGenerationPipeline:
    """用户标签生成流水线"""
    
    def __init__(self):
        self.rule_engine = RuleEngine()
        self.ml_models = {}
        self.tag_storage = TagStorage()
        
    def generate_interest_tags(self, user_id):
        """生成兴趣标签"""
        # 获取用户行为序列
        behaviors = self.get_user_behaviors(user_id, days=30)
        
        # 内容标签聚合
        content_tags = defaultdict(float)
        for behavior in behaviors:
            video_tags = self.get_video_tags(behavior.video_id)
            weight = self.calculate_weight(behavior)
            
            for tag in video_tags:
                content_tags[tag] += weight
        
        # TF-IDF标准化
        user_tags = self.tfidf_normalize(content_tags)
        
        # 标签衰减（时间衰减因子）
        decayed_tags = self.apply_time_decay(user_tags)
        
        # 阈值过滤
        final_tags = {
            tag: score 
            for tag, score in decayed_tags.items() 
            if score > 0.3
        }
        
        return final_tags
    
    def calculate_weight(self, behavior):
        """计算行为权重"""
        weights = {
            'view': 1.0,
            'like': 2.0,
            'coin': 3.0,
            'favorite': 3.0,
            'share': 4.0,
            'comment': 2.5
        }
        
        # 完播率加权
        completion_rate = behavior.watch_time / behavior.video_duration
        base_weight = weights.get(behavior.action_type, 1.0)
        
        return base_weight * (1 + completion_rate)
```

### 4.3 用户分群策略

#### 核心用户分群
```
┌──────────────────────────────────────────────┐
│              用户分群矩阵                    │
├──────────────────────────────────────────────┤
│                                              │
│  活跃度维度                                  │
│   高 ┌────────┬────────┬────────┐          │
│      │核心用户│活跃用户│普通用户│          │
│   中 ├────────┼────────┼────────┤          │
│      │忠实用户│常规用户│低频用户│          │
│   低 ├────────┼────────┼────────┤          │
│      │回流用户│沉睡用户│流失用户│          │
│      └────────┴────────┴────────┘          │
│        高      中      低                    │
│            价值度维度                        │
│                                              │
│  分群策略                                    │
│   • 核心用户：专属权益、优先体验            │
│   • 活跃用户：内容精推、社区互动            │
│   • 沉睡用户：召回策略、激励唤醒            │
│   • 新用户：引导教育、兴趣探索              │
│                                              │
└──────────────────────────────────────────────┘
```

#### RFM模型应用
| 维度 | 定义 | 计算方法 | 分层阈值 |
|------|------|----------|----------|
| R(Recency) | 最近活跃 | 距今天数 | 1/3/7/30天 |
| F(Frequency) | 活跃频次 | 月均活跃天数 | 1/5/15/25天 |
| M(Monetary) | 消费价值 | 月均消费金额 | ¥0/10/50/200 |

### 4.4 实时画像更新

#### 流式计算架构
```
┌──────────────────────────────────────────────┐
│           实时画像更新系统                   │
├──────────────────────────────────────────────┤
│                                              │
│  数据源                                      │
│   ├── 客户端埋点 → Kafka                    │
│   ├── 服务端日志 → Flume                    │
│   └── 数据库Binlog → Canal                  │
│                ↓                             │
│  实时计算层（Flink）                         │
│   ├── 事件清洗：去重、过滤、标准化          │
│   ├── 特征计算：滑动窗口聚合                │
│   └── 标签更新：增量计算                    │
│                ↓                             │
│  存储层                                      │
│   ├── Redis：热数据（1天内）                │
│   ├── HBase：温数据（30天内）               │
│   └── Hive：冷数据（历史归档）              │
│                ↓                             │
│  服务层                                      │
│   └── 画像查询API（QPS: 100K+）             │
│                                              │
└──────────────────────────────────────────────┘
```

#### 画像更新策略
```python
class ProfileUpdateStrategy:
    """画像更新策略"""
    
    def __init__(self):
        self.redis = RedisCluster()
        self.hbase = HBaseClient()
        
    def update_realtime_profile(self, user_id, event):
        """实时画像更新"""
        # 短期行为序列更新
        self.update_behavior_sequence(user_id, event)
        
        # 实时统计指标更新
        self.update_statistics(user_id, event)
        
        # 兴趣标签增量更新
        if event.type in ['view', 'like', 'favorite']:
            self.update_interest_tags(user_id, event)
        
        # 触发规则引擎
        self.trigger_rules(user_id, event)
    
    def update_behavior_sequence(self, user_id, event):
        """更新行为序列（保留最近1000条）"""
        key = f"user:behavior:{user_id}"
        
        # 使用Redis List存储
        self.redis.lpush(key, event.to_json())
        self.redis.ltrim(key, 0, 999)
        self.redis.expire(key, 86400)  # 1天过期
    
    def merge_profiles(self, user_id):
        """多时间尺度画像融合"""
        # 实时画像（5分钟）
        realtime = self.get_realtime_profile(user_id)
        
        # 近期画像（1天）
        recent = self.get_recent_profile(user_id)
        
        # 长期画像（30天）
        longterm = self.get_longterm_profile(user_id)
        
        # 加权融合
        merged = {
            'interests': self.merge_interests(
                realtime.interests * 0.5,
                recent.interests * 0.3,
                longterm.interests * 0.2
            ),
            'behaviors': self.merge_behaviors(
                realtime, recent, longterm
            )
        }
        
        return merged
```

### 4.5 画像应用场景

#### 个性化推荐
```
用户画像 → 召回策略
  ├── 兴趣标签 → 标签召回
  ├── 行为序列 → 协同过滤
  ├── 社交关系 → 社交推荐
  └── 消费能力 → 付费内容推荐
```

#### 精准营销
| 场景 | 画像维度 | 策略 | 效果 |
|------|----------|------|------|
| 拉新 | 相似人群 | Look-alike | CTR +30% |
| 促活 | 沉睡用户 | Push召回 | 唤醒率 15% |
| 付费转化 | 付费意愿 | 定向优惠 | 转化率 +50% |
| 防流失 | 流失预警 | 挽留策略 | 留存 +20% |

---

## 个性化推荐架构

### 2.1 系统整体架构

#### 四层架构设计
```
┌────────────────────────────────────────────────────┐
│              B站推荐系统架构全景                   │
├────────────────────────────────────────────────────┤
│                                                    │
│  数据层（Data Layer）                              │
│  ┌──────────────────────────────────────────────┐    │
│  │ • 用户行为日志（点击、播放、互动）        │    │
│  │ • 内容元数据（标题、标签、分类）          │    │
│  │ • 实时特征（热度、趋势、时效）            │    │
│  │ • 离线特征（用户画像、内容画像）          │    │
│  └──────────────────────────────────────────┘    │
│                        ↓                           │
│  召回层（Recall Layer）                            │
│  ┌──────────────────────────────────────────┐    │
│  │ • 多路召回：协同/内容/热门/标签/关注      │    │
│  │ • 候选集：10000+ → 1000                   │    │
│  │ • 响应时间：< 50ms                        │    │
│  └──────────────────────────────────────────┘    │
│                        ↓                           │
│  排序层（Ranking Layer）                           │
│  ┌──────────────────────────────────────────┐    │
│  │ • 粗排：1000 → 200（简单模型）            │    │
│  │ • 精排：200 → 50（复杂模型）              │    │
│  │ • 响应时间：< 100ms                       │    │
│  └──────────────────────────────────────────┘    │
│                        ↓                           │
│  策略层（Strategy Layer）                          │
│  ┌──────────────────────────────────────────┐    │
│  │ • 多样性控制                              │    │
│  │ • 新内容扶持                              │    │
│  │ • 创作者平衡                              │    │
│  │ • 商业化混排                              │    │
│  └──────────────────────────────────────────┘    │
│                                                    │
└────────────────────────────────────────────────────┘
```

### 2.2 召回策略详解

#### 多路召回架构
| 召回通道 | 算法类型 | 召回量 | 延迟 | 权重 |
|----------|----------|---------|------|------|
| 协同过滤 | UserCF/ItemCF | 300 | 20ms | 25% |
| 内容召回 | Embedding相似度 | 200 | 30ms | 20% |
| 标签召回 | 标签匹配 | 200 | 10ms | 15% |
| 热门召回 | 统计排序 | 100 | 5ms | 10% |
| 关注召回 | 社交关系 | 150 | 15ms | 15% |
| 历史召回 | 序列模型 | 150 | 25ms | 15% |

#### 向量化召回实现
```python
class EmbeddingRecall:
    """基于向量相似度的召回实现"""
    
    def __init__(self):
        self.user_embeddings = {}  # 用户向量
        self.item_embeddings = {}  # 内容向量
        self.index = FaissIndex()  # 向量索引
        
    def build_index(self):
        """构建向量索引"""
        # 使用Faiss构建高效的向量检索索引
        vectors = np.array(list(self.item_embeddings.values()))
        self.index.add(vectors)
        
    def recall(self, user_id, top_k=100):
        """召回Top-K相似内容"""
        user_vec = self.user_embeddings.get(user_id)
        if user_vec is None:
            return self.cold_start_recall(user_id)
        
        # 向量相似度检索
        distances, indices = self.index.search(user_vec, top_k)
        
        # 过滤已看过的内容
        candidates = self.filter_seen(user_id, indices)
        
        return candidates
```

### 2.3 排序模型架构

#### 特征处理流水线
```
┌─────────────────────────────────────────────────┐
│            特征处理Pipeline                     │
├─────────────────────────────────────────────────┤
│                                                 │
│  原始特征提取                                   │
│    ├── 实时特征（Kafka/Redis）                 │
│    └── 离线特征（Hive/HBase）                  │
│              ↓                                  │
│  特征工程                                       │
│    ├── 数值特征：标准化、离散化                │
│    ├── 类别特征：One-hot、Embedding            │
│    └── 交叉特征：自动特征交叉                  │
│              ↓                                  │
│  特征选择                                       │
│    ├── 重要性评分                              │
│    └── 在线特征监控                            │
│              ↓                                  │
│  模型输入                                       │
│    └── Dense Vector（~1000维）                 │
│                                                 │
└─────────────────────────────────────────────────┘
```

#### 深度排序网络
```
输入层（1000维）
    ↓
Embedding层
    ├── 用户ID Embedding（128维）
    ├── 视频ID Embedding（128维）
    └── 类别特征Embedding
    ↓
特征交叉层（Wide部分）
    └── 手工特征交叉
    ↓
深度网络层（Deep部分）
    ├── Hidden Layer 1（512维）+ ReLU + Dropout
    ├── Hidden Layer 2（256维）+ ReLU + Dropout
    └── Hidden Layer 3（128维）+ ReLU + Dropout
    ↓
融合层
    └── Wide & Deep 合并
    ↓
输出层
    └── Sigmoid（点击率预测）
```

### 2.4 实时推荐系统

#### 流式计算架构
```
┌──────────────────────────────────────────────┐
│           实时推荐数据流                     │
├──────────────────────────────────────────────┤
│                                              │
│  用户行为 → Kafka → Flink                    │
│                ↓                             │
│         实时特征计算                         │
│           ├── 5分钟时间窗口                 │
│           ├── 用户兴趣漂移检测              │
│           └── 热点发现                      │
│                ↓                             │
│         Redis特征存储                        │
│                ↓                             │
│         在线推理服务                         │
│                ↓                             │
│         推荐结果                             │
│                                              │
└──────────────────────────────────────────────┤
```

#### 性能指标
| 指标 | 目标值 | 实际值 | 说明 |
|------|--------|--------|------|
| 端到端延迟 | <200ms | 150ms | 从请求到返回 |
| 召回延迟 | <50ms | 35ms | 多路召回总耗时 |
| 排序延迟 | <100ms | 80ms | 模型推理时间 |
| QPS | 100K | 120K | 单机处理能力 |
| 缓存命中率 | >80% | 85% | Redis缓存 |

### 2.5 分布式训练架构

#### 参数服务器模式
```
┌────────────────────────────────────────────┐
│         Parameter Server架构                │
├────────────────────────────────────────────┤
│                                            │
│  Parameter Servers（参数服务器）           │
│     ├── PS1：存储Embedding参数             │
│     ├── PS2：存储Dense参数                 │
│     └── PS3：备份与容错                    │
│              ↓ ↑                           │
│                                            │
│  Worker Nodes（计算节点）                  │
│     ├── Worker1：训练Batch1                │
│     ├── Worker2：训练Batch2                │
│     ├── Worker3：训练Batch3                │
│     └── Worker4：训练Batch4                │
│              ↓                             │
│                                            │
│  Model Serving（模型服务）                 │
│     └── TensorFlow Serving集群             │
│                                            │
└────────────────────────────────────────────┘
```

---

## 深度学习应用

### 3.1 模型演进历程

#### 深度学习技术栈时间线
```
2017: DNN (Deep Neural Network)
      ├── 3层全连接网络
      └── 特征维度：500

2018: Wide & Deep
      ├── Wide部分：线性模型
      └── Deep部分：4层DNN

2019: DeepFM
      ├── FM特征交叉
      └── Deep网络学习高阶特征

2020: DIEN (Deep Interest Evolution Network)
      ├── 兴趣抽取层
      └── 兴趣演化层

2021: Multi-Task Learning
      ├── 点击率预测
      ├── 完播率预测
      └── 互动率预测

2022: Transformer应用
      ├── Self-Attention机制
      └── 长序列建模

2023: 大模型融合
      ├── LLM特征提取
      └── 多模态理解
```

### 3.2 核心模型详解

#### DIEN架构实现
```python
class DIEN(nn.Module):
    """深度兴趣演化网络"""
    
    def __init__(self, config):
        super().__init__()
        # 兴趣抽取层（GRU）
        self.interest_extractor = nn.GRU(
            input_size=config.embedding_dim,
            hidden_size=config.hidden_size,
            num_layers=2,
            batch_first=True
        )
        
        # 兴趣演化层（AUGRU）
        self.interest_evolution = AUGRU(
            input_size=config.hidden_size,
            hidden_size=config.hidden_size
        )
        
        # Attention机制
        self.attention = MultiHeadAttention(
            hidden_size=config.hidden_size,
            num_heads=config.num_heads
        )
        
        # 预测层
        self.prediction = nn.Sequential(
            nn.Linear(config.hidden_size * 3, 200),
            nn.ReLU(),
            nn.Dropout(0.2),
            nn.Linear(200, 80),
            nn.ReLU(),
            nn.Dropout(0.2),
            nn.Linear(80, 1),
            nn.Sigmoid()
        )
    
    def forward(self, user_seq, target_item, neg_items=None):
        # 兴趣抽取
        interests, _ = self.interest_extractor(user_seq)
        
        # 辅助损失（使用负样本）
        if neg_items is not None:
            aux_loss = self.auxiliary_loss(interests, neg_items)
        
        # 兴趣演化
        evolved_interests = self.interest_evolution(
            interests, target_item
        )
        
        # Attention pooling
        final_interest = self.attention(
            evolved_interests, target_item
        )
        
        # 预测
        output = self.prediction(final_interest)
        
        return output, aux_loss if neg_items else output
```

#### 多任务学习框架
```
┌──────────────────────────────────────────────┐
│           多任务学习架构（MMOE）             │
├──────────────────────────────────────────────┤
│                                              │
│  输入特征                                    │
│     ↓                                        │
│  共享底层（Shared Bottom）                   │
│     ├── Expert 1：用户行为专家              │
│     ├── Expert 2：内容理解专家              │
│     ├── Expert 3：上下文专家                │
│     └── Expert 4：统计特征专家              │
│            ↓                                 │
│  门控网络（Gating Network）                  │
│     ├── Gate 1 → 点击率任务                 │
│     ├── Gate 2 → 完播率任务                 │
│     └── Gate 3 → 互动率任务                 │
│            ↓                                 │
│  任务特定层（Task-specific）                 │
│     ├── Tower 1：CTR预测                    │
│     ├── Tower 2：完播率预测                 │
│     └── Tower 3：互动率预测                 │
│            ↓                                 │
│  多目标优化                                  │
│     └── Loss = α*L_ctr + β*L_finish + γ*L_interact │
│                                              │
└──────────────────────────────────────────────┘
```

### 3.3 特征工程创新

#### 多模态特征融合
| 模态类型 | 特征提取方法 | 维度 | 应用场景 |
|----------|-------------|------|----------|
| 视频帧 | ResNet/ViT | 2048 | 内容理解 |
| 音频 | Wav2Vec | 768 | 音乐/语音识别 |
| 文本 | BERT/RoBERTa | 768 | 标题/弹幕理解 |
| 用户行为 | RNN/Transformer | 512 | 序列建模 |
| 图结构 | GNN | 256 | 社交关系 |

#### 实时特征工程
```python
class RealtimeFeatureEngine:
    """实时特征计算引擎"""
    
    def __init__(self):
        self.redis_client = Redis()
        self.kafka_consumer = KafkaConsumer()
        
    def compute_user_features(self, user_id):
        features = {}
        
        # 短期兴趣（5分钟窗口）
        features['short_term'] = self.get_recent_behaviors(
            user_id, window='5m'
        )
        
        # 中期兴趣（1小时窗口）
        features['medium_term'] = self.get_recent_behaviors(
            user_id, window='1h'
        )
        
        # 实时统计特征
        features['realtime_stats'] = {
            'click_rate_5m': self.compute_ctr(user_id, '5m'),
            'watch_time_avg': self.compute_avg_watch_time(user_id),
            'interaction_rate': self.compute_interaction_rate(user_id)
        }
        
        # 上下文特征
        features['context'] = {
            'time_of_day': self.get_time_features(),
            'device_type': self.get_device_info(user_id),
            'network_type': self.get_network_info(user_id)
        }
        
        return features
```

### 3.4 模型优化技术

#### 模型压缩与加速
```
┌─────────────────────────────────────────────┐
│           模型优化Pipeline                  │
├─────────────────────────────────────────────┤
│                                             │
│  原始模型（1GB）                            │
│     ↓                                       │
│  知识蒸馏（Knowledge Distillation）         │
│     Teacher Model → Student Model           │
│     压缩率：4x                              │
│     ↓                                       │
│  量化（Quantization）                       │
│     FP32 → INT8                            │
│     压缩率：4x                              │
│     ↓                                       │
│  剪枝（Pruning）                            │
│     移除冗余连接                            │
│     压缩率：2x                              │
│     ↓                                       │
│  优化后模型（31.25MB）                      │
│     推理速度提升：10x                       │
│     精度损失：<1%                           │
│                                             │
└─────────────────────────────────────────────┘
```

#### 在线学习系统
| 组件 | 技术选型 | 功能 | 性能指标 |
|------|----------|------|----------|
| 特征更新 | Flink | 实时特征计算 | 延迟<100ms |
| 模型更新 | Parameter Server | 增量训练 | 5分钟更新周期 |
| 效果评估 | A/B Testing | 实时效果监控 | 秒级指标 |
| 回滚机制 | Model Registry | 版本管理 | 1分钟回滚 |

### 3.5 前沿技术探索

#### Transformer在推荐中的应用
```python
class BST(nn.Module):
    """Behavior Sequence Transformer"""
    
    def __init__(self, config):
        super().__init__()
        
        # Position encoding
        self.position_encoding = PositionalEncoding(
            d_model=config.d_model,
            max_len=config.max_seq_len
        )
        
        # Multi-head self-attention blocks
        self.transformer_blocks = nn.ModuleList([
            TransformerBlock(
                d_model=config.d_model,
                n_heads=config.n_heads,
                d_ff=config.d_ff,
                dropout=config.dropout
            ) for _ in range(config.n_blocks)
        ])
        
        # Target attention
        self.target_attention = TargetAttention(
            d_model=config.d_model
        )
        
    def forward(self, behavior_sequence, target_item):
        # 添加位置编码
        seq_emb = self.position_encoding(behavior_sequence)
        
        # Transformer blocks
        for block in self.transformer_blocks:
            seq_emb = block(seq_emb)
        
        # Target-aware attention
        user_interest = self.target_attention(
            seq_emb, target_item
        )
        
        return user_interest
```

#### 图神经网络应用
```
┌──────────────────────────────────────────────┐
│            图神经网络推荐架构                │
├──────────────────────────────────────────────┤
│                                              │
│  用户-物品二部图                             │
│     ├── 用户节点：100M+                     │
│     ├── 物品节点：10M+                      │
│     └── 边：交互关系                        │
│              ↓                               │
│  图卷积层（GCN）                             │
│     ├── Layer 1：邻居聚合                   │
│     ├── Layer 2：特征传播                   │
│     └── Layer 3：高阶关系                   │
│              ↓                               │
│  节点Embedding                               │
│     ├── 用户Embedding                       │
│     └── 物品Embedding                       │
│              ↓                               │
│  相似度计算                                  │
│     └── 内积/余弦相似度                      │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 用户画像系统

### 4.1 画像体系架构

#### 多层次用户画像
```
┌──────────────────────────────────────────────┐
│             B站用户画像体系                  │
├──────────────────────────────────────────────┤
│                                              │
│  基础属性层                                  │
│   ├── 人口统计：年龄、性别、地域            │
│   ├── 设备信息：机型、系统、网络            │
│   └── 账号信息：等级、大会员、注册时间      │
│                                              │
│  行为特征层                                  │
│   ├── 观看行为：时长、频次、完播率          │
│   ├── 互动行为：点赞、投币、收藏、弹幕      │
│   ├── 社交行为：关注、私信、动态            │
│   └── 创作行为：投稿、直播、专栏            │
│                                              │
│  兴趣标签层                                  │
│   ├── 内容偏好：分区、标签、UP主            │
│   ├── 时间偏好：活跃时段、观看节奏          │
│   └── 消费偏好：付费意愿、打赏习惯          │
│                                              │
│  价值分层                                    │
│   ├── 用户价值：LTV、活跃度、留存          │
│   ├── 内容价值：内容消费量、质量偏好        │
│   └── 社区价值：UGC贡献、社交影响力         │
│                                              │
└──────────────────────────────────────────────┘
```

### 4.2 标签体系建设

#### 标签分类架构
| 标签类型 | 数量级 | 更新频率 | 应用场景 |
|----------|--------|----------|----------|
| 基础标签 | 100+ | 每天 | 用户分群 |
| 兴趣标签 | 5000+ | 实时 | 内容推荐 |
| 行为标签 | 1000+ | 每小时 | 行为预测 |
| 预测标签 | 500+ | 每天 | 流失预警 |
| 圈层标签 | 200+ | 每周 | 社群运营 |

#### 标签生成Pipeline
```python
class TagGenerationPipeline:
    """用户标签生成流水线"""
    
    def __init__(self):
        self.rule_engine = RuleEngine()
        self.ml_models = {}
        self.tag_storage = TagStorage()
        
    def generate_interest_tags(self, user_id):
        """生成兴趣标签"""
        # 获取用户行为序列
        behaviors = self.get_user_behaviors(user_id, days=30)
        
        # 内容标签聚合
        content_tags = defaultdict(float)
        for behavior in behaviors:
            video_tags = self.get_video_tags(behavior.video_id)
            weight = self.calculate_weight(behavior)
            
            for tag in video_tags:
                content_tags[tag] += weight
        
        # TF-IDF标准化
        user_tags = self.tfidf_normalize(content_tags)
        
        # 标签衰减（时间衰减因子）
        decayed_tags = self.apply_time_decay(user_tags)
        
        # 阈值过滤
        final_tags = {
            tag: score 
            for tag, score in decayed_tags.items() 
            if score > 0.3
        }
        
        return final_tags
    
    def calculate_weight(self, behavior):
        """计算行为权重"""
        weights = {
            'view': 1.0,
            'like': 2.0,
            'coin': 3.0,
            'favorite': 3.0,
            'share': 4.0,
            'comment': 2.5
        }
        
        # 完播率加权
        completion_rate = behavior.watch_time / behavior.video_duration
        base_weight = weights.get(behavior.action_type, 1.0)
        
        return base_weight * (1 + completion_rate)
```

### 4.3 用户分群策略

#### 核心用户分群
```
┌──────────────────────────────────────────────┐
│              用户分群矩阵                    │
├──────────────────────────────────────────────┤
│                                              │
│  活跃度维度                                  │
│   高 ┌────────┬────────┬────────┐          │
│      │核心用户│活跃用户│普通用户│          │
│   中 ├────────┼────────┼────────┤          │
│      │忠实用户│常规用户│低频用户│          │
│   低 ├────────┼────────┼────────┤          │
│      │回流用户│沉睡用户│流失用户│          │
│      └────────┴────────┴────────┘          │
│        高      中      低                    │
│            价值度维度                        │
│                                              │
│  分群策略                                    │
│   • 核心用户：专属权益、优先体验            │
│   • 活跃用户：内容精推、社区互动            │
│   • 沉睡用户：召回策略、激励唤醒            │
│   • 新用户：引导教育、兴趣探索              │
│                                              │
└──────────────────────────────────────────────┘
```

#### RFM模型应用
| 维度 | 定义 | 计算方法 | 分层阈值 |
|------|------|----------|----------|
| R(Recency) | 最近活跃 | 距今天数 | 1/3/7/30天 |
| F(Frequency) | 活跃频次 | 月均活跃天数 | 1/5/15/25天 |
| M(Monetary) | 消费价值 | 月均消费金额 | ¥0/10/50/200 |

### 4.4 实时画像更新

#### 流式计算架构
```
┌──────────────────────────────────────────────┐
│           实时画像更新系统                   │
├──────────────────────────────────────────────┤
│                                              │
│  数据源                                      │
│   ├── 客户端埋点 → Kafka                    │
│   ├── 服务端日志 → Flume                    │
│   └── 数据库Binlog → Canal                  │
│                ↓                             │
│  实时计算层（Flink）                         │
│   ├── 事件清洗：去重、过滤、标准化          │
│   ├── 特征计算：滑动窗口聚合                │
│   └── 标签更新：增量计算                    │
│                ↓                             │
│  存储层                                      │
│   ├── Redis：热数据（1天内）                │
│   ├── HBase：温数据（30天内）               │
│   └── Hive：冷数据（历史归档）              │
│                ↓                             │
│  服务层                                      │
│   └── 画像查询API（QPS: 100K+）             │
│                                              │
└──────────────────────────────────────────────┘
```

#### 画像更新策略
```python
class ProfileUpdateStrategy:
    """画像更新策略"""
    
    def __init__(self):
        self.redis = RedisCluster()
        self.hbase = HBaseClient()
        
    def update_realtime_profile(self, user_id, event):
        """实时画像更新"""
        # 短期行为序列更新
        self.update_behavior_sequence(user_id, event)
        
        # 实时统计指标更新
        self.update_statistics(user_id, event)
        
        # 兴趣标签增量更新
        if event.type in ['view', 'like', 'favorite']:
            self.update_interest_tags(user_id, event)
        
        # 触发规则引擎
        self.trigger_rules(user_id, event)
    
    def update_behavior_sequence(self, user_id, event):
        """更新行为序列（保留最近1000条）"""
        key = f"user:behavior:{user_id}"
        
        # 使用Redis List存储
        self.redis.lpush(key, event.to_json())
        self.redis.ltrim(key, 0, 999)
        self.redis.expire(key, 86400)  # 1天过期
    
    def merge_profiles(self, user_id):
        """多时间尺度画像融合"""
        # 实时画像（5分钟）
        realtime = self.get_realtime_profile(user_id)
        
        # 近期画像（1天）
        recent = self.get_recent_profile(user_id)
        
        # 长期画像（30天）
        longterm = self.get_longterm_profile(user_id)
        
        # 加权融合
        merged = {
            'interests': self.merge_interests(
                realtime.interests * 0.5,
                recent.interests * 0.3,
                longterm.interests * 0.2
            ),
            'behaviors': self.merge_behaviors(
                realtime, recent, longterm
            )
        }
        
        return merged
```

### 4.5 画像应用场景

#### 个性化推荐
```
用户画像 → 召回策略
  ├── 兴趣标签 → 标签召回
  ├── 行为序列 → 协同过滤
  ├── 社交关系 → 社交推荐
  └── 消费能力 → 付费内容推荐
```

#### 精准营销
| 场景 | 画像维度 | 策略 | 效果 |
|------|----------|------|------|
| 拉新 | 相似人群 | Look-alike | CTR +30% |
| 促活 | 沉睡用户 | Push召回 | 唤醒率 15% |
| 付费转化 | 付费意愿 | 定向优惠 | 转化率 +50% |
| 防流失 | 流失预警 | 挽留策略 | 留存 +20% |

---

## AB测试平台

### 5.1 平台架构演进

#### 发展历程
```
2015: 手工AB测试
     ├── Excel记录实验
     └── 人工分析数据

2016: 初代AB平台
     ├── 简单分流系统
     └── 基础指标统计

2018: 平台化建设
     ├── 实验管理系统
     ├── 自动化分析
     └── 多层实验框架

2020: 智能化升级
     ├── 自动调参
     ├── 早停机制
     └── 因果推断

2023: 全域实验平台
     ├── 端到端实验
     ├── 多业务线支持
     └── 实时决策系统
```

#### 系统架构
```
┌──────────────────────────────────────────────┐
│            B站AB测试平台架构                 │
├──────────────────────────────────────────────┤
│                                              │
│  实验配置层                                  │
│   ├── 实验管理UI                            │
│   ├── 参数配置                              │
│   └── 审批流程                              │
│                ↓                             │
│  分流服务层                                  │
│   ├── 用户分流（哈希分桶）                  │
│   ├── 流量隔离（正交实验）                  │
│   └── 灰度发布                              │
│                ↓                             │
│  实验执行层                                  │
│   ├── 客户端SDK                             │
│   ├── 服务端SDK                             │
│   └── 配置中心                              │
│                ↓                             │
│  数据收集层                                  │
│   ├── 埋点采集                              │
│   ├── 日志聚合                              │
│   └── 实时流处理                            │
│                ↓                             │
│  分析决策层                                  │
│   ├── 统计分析                              │
│   ├── 效果评估                              │
│   └── 自动决策                              │
│                                              │
└──────────────────────────────────────────────┘
```

### 5.2 分流系统设计

#### 分流算法
```python
class TrafficSplitter:
    """流量分流器"""
    
    def __init__(self):
        self.experiments = {}
        self.user_buckets = 1000  # 用户分桶数
        
    def get_user_bucket(self, user_id):
        """计算用户所属桶"""
        # 使用MurmurHash保证分布均匀
        hash_value = mmh3.hash(str(user_id))
        return abs(hash_value) % self.user_buckets
    
    def assign_experiment(self, user_id, experiment_config):
        """分配实验组"""
        bucket = self.get_user_bucket(user_id)
        
        # 检查用户是否在实验流量中
        if bucket < experiment_config.traffic_start or \
           bucket >= experiment_config.traffic_end:
            return 'control'  # 对照组
        
        # 在实验流量中进一步分组
        exp_bucket = bucket - experiment_config.traffic_start
        bucket_size = (experiment_config.traffic_end - 
                      experiment_config.traffic_start)
        
        # 根据配置比例分配到不同实验组
        cumulative = 0
        for group, ratio in experiment_config.groups.items():
            cumulative += ratio * bucket_size
            if exp_bucket < cumulative:
                return group
        
        return 'control'
```

#### 正交实验框架
```
┌──────────────────────────────────────────────┐
│              正交实验设计                    │
├──────────────────────────────────────────────┤
│                                              │
│  Layer 1: 推荐算法实验                       │
│   ├── 实验A：新召回策略（10%流量）          │
│   ├── 实验B：排序模型v2（10%流量）          │
│   └── 基线：当前算法（80%流量）             │
│                                              │
│  Layer 2: UI实验                             │
│   ├── 实验C：新首页布局（20%流量）          │
│   └── 基线：当前UI（80%流量）               │
│                                              │
│  Layer 3: 业务策略实验                       │
│   ├── 实验D：激励策略（15%流量）            │
│   └── 基线：无激励（85%流量）               │
│                                              │
│  正交性保证：                                │
│   • 每层独立分流                            │
│   • 层间互不影响                            │
│   • 支持2^n种组合                           │
│                                              │
└──────────────────────────────────────────────┘
```

### 5.3 指标体系建设

#### 核心指标分类
| 指标类型 | 具体指标 | 计算方法 | 决策权重 |
|----------|----------|----------|----------|
| 北极星指标 | DAU | 日活跃用户数 | 40% |
| 用户体验 | 人均使用时长 | 总时长/DAU | 25% |
| 内容消费 | 人均播放量 | 总播放/DAU | 20% |
| 社区互动 | 互动率 | 互动用户/活跃用户 | 10% |
| 商业化 | ARPU | 总收入/活跃用户 | 5% |

#### 指标计算Pipeline
```python
class MetricsCalculator:
    """指标计算器"""
    
    def __init__(self):
        self.spark = SparkSession.builder.getOrCreate()
        
    def calculate_experiment_metrics(self, experiment_id, date):
        """计算实验指标"""
        
        # 获取实验用户
        exp_users = self.get_experiment_users(experiment_id, date)
        
        metrics = {}
        
        # 用户规模指标
        metrics['dau'] = exp_users.count()
        
        # 行为指标
        behaviors = self.get_user_behaviors(exp_users, date)
        metrics['avg_duration'] = behaviors.agg(
            F.avg('duration')
        ).collect()[0][0]
        
        metrics['avg_plays'] = behaviors.agg(
            F.avg('play_count')
        ).collect()[0][0]
        
        # 留存指标
        metrics['retention_1d'] = self.calculate_retention(
            exp_users, date, days=1
        )
        metrics['retention_7d'] = self.calculate_retention(
            exp_users, date, days=7
        )
        
        # 统计显著性检验
        control_metrics = self.get_control_metrics(date)
        metrics['p_value'] = self.statistical_test(
            metrics, control_metrics
        )
        
        return metrics
    
    def statistical_test(self, treatment, control):
        """统计显著性检验"""
        from scipy import stats
        
        # T检验
        t_stat, p_value = stats.ttest_ind(
            treatment['samples'],
            control['samples']
        )
        
        # 效应量计算（Cohen's d）
        effect_size = (treatment['mean'] - control['mean']) / \
                     np.sqrt((treatment['std']**2 + control['std']**2) / 2)
        
        return {
            'p_value': p_value,
            'effect_size': effect_size,
            'significant': p_value < 0.05
        }
```

### 5.4 实验分析系统

#### 自动化分析框架
```
┌──────────────────────────────────────────────┐
│           实验分析流程                       │
├──────────────────────────────────────────────┤
│                                              │
│  数据收集（T+0）                             │
│   └── 实时指标采集                          │
│            ↓                                 │
│  数据质量检查（T+1小时）                     │
│   ├── 数据完整性校验                        │
│   ├── 异常值检测                            │
│   └── 样本平衡性检查                        │
│            ↓                                 │
│  指标计算（T+2小时）                         │
│   ├── 核心指标计算                          │
│   ├── 分群指标计算                          │
│   └── 漏斗分析                              │
│            ↓                                 │
│  统计分析（T+3小时）                         │
│   ├── 显著性检验                            │
│   ├── 置信区间计算                          │
│   └── 多重检验校正                          │
│            ↓                                 │
│  效果评估（T+4小时）                         │
│   ├── 提升度计算                            │
│   ├── 分群效果分析                          │
│   └── 负向指标监控                          │
│            ↓                                 │
│  决策建议（T+6小时）                         │
│   └── 自动生成报告                          │
│                                              │
└──────────────────────────────────────────────┘
```

#### 因果推断应用
```python
class CausalInference:
    """因果推断分析"""
    
    def __init__(self):
        self.data = None
        
    def estimate_ate(self, treatment, outcome, confounders):
        """估计平均处理效应（ATE）"""
        
        # 倾向得分匹配
        ps_model = LogisticRegression()
        ps_model.fit(confounders, treatment)
        propensity_scores = ps_model.predict_proba(confounders)[:, 1]
        
        # 逆概率加权（IPW）
        weights = treatment / propensity_scores + \
                 (1 - treatment) / (1 - propensity_scores)
        
        # 加权回归
        model = LinearRegression()
        model.fit(
            np.column_stack([treatment, confounders]),
            outcome,
            sample_weight=weights
        )
        
        # ATE = 处理组效果 - 对照组效果
        ate = model.coef_[0]
        
        # Bootstrap计算置信区间
        ate_bootstrap = []
        for _ in range(1000):
            indices = np.random.choice(len(treatment), len(treatment))
            boot_ate = self.estimate_ate_single(
                treatment[indices],
                outcome[indices],
                confounders[indices]
            )
            ate_bootstrap.append(boot_ate)
        
        ci_lower = np.percentile(ate_bootstrap, 2.5)
        ci_upper = np.percentile(ate_bootstrap, 97.5)
        
        return {
            'ate': ate,
            'ci_lower': ci_lower,
            'ci_upper': ci_upper
        }
```

### 5.5 实验最佳实践

#### 实验设计原则
```
┌──────────────────────────────────────────────┐
│            实验设计检查清单                  │
├──────────────────────────────────────────────┤
│                                              │
│  ✓ 明确假设                                 │
│    • 实验目标清晰                           │
│    • 成功指标定义                           │
│    • 预期效果量                             │
│                                              │
│  ✓ 样本量计算                               │
│    • 统计功效 > 80%                         │
│    • 显著性水平 = 0.05                      │
│    • 最小可检测效应（MDE）                  │
│                                              │
│  ✓ 随机化设计                               │
│    • 分组随机性                             │
│    • 样本平衡性                             │
│    • 避免溢出效应                           │
│                                              │
│  ✓ 实验周期                                 │
│    • 最短：1周（快速迭代）                  │
│    • 标准：2周（常规实验）                  │
│    • 长期：4周+（重大改动）                 │
│                                              │
│  ✓ 风险控制                                 │
│    • 降级方案                               │
│    • 实时监控                               │
│    • 快速止损                               │
│                                              │
└──────────────────────────────────────────────┘
```

#### 典型实验案例

##### 案例1：推荐算法优化
| 维度 | 详情 |
|------|------|
| 背景 | CTR增长放缓，需要算法突破 |
| 假设 | 引入Transformer能提升推荐效果 |
| 实验设计 | 10%流量，运行14天 |
| 结果 | CTR +3.2%, 人均时长 +5.1% |
| 决策 | 全量上线，继续优化 |

##### 案例2：UI改版测试
| 维度 | 详情 |
|------|------|
| 背景 | 首页信息密度过高 |
| 假设 | 简化布局能提升用户体验 |
| 实验设计 | 20%流量，运行21天 |
| 结果 | 留存率 +1.5%, 但时长 -2% |
| 决策 | 部分采纳，保留核心改动 |

#### 实验文化建设
```
B站实验文化核心理念：

1. 数据驱动（Data-Driven）
   • 一切以数据说话
   • 拒绝拍脑袋决策

2. 快速迭代（Fail Fast）
   • 小步快跑
   • 及时止损

3. 科学严谨（Scientific）
   • 统计显著性
   • 因果关系验证

4. 全员参与（Inclusive）
   • 产品提假设
   • 技术做实验
   • 运营看数据

5. 持续优化（Continuous）
   • 复盘总结
   • 知识沉淀
```

---
