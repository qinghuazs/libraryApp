# 微书包用户模块详细需求文档（UltraThink深度分析版）

> **文档版本**: V2.0（UltraThink深度分析版）
> **创建时间**: 2025-10-15
> **分析模式**: UltraThink 全方位需求整理
> **适用范围**: 微书包APP用户模块（移动端 + Web管理端）
> **核心目标**: 提供完整、可落地、高质量的用户模块实施方案

---

## 📋 目录

1. [概述与定位](#1-概述与定位)
2. [核心价值主张](#2-核心价值主张)
3. [用户角色与权限体系](#3-用户角色与权限体系)
4. [功能模块详细设计](#4-功能模块详细设计)
5. [数据库架构设计](#5-数据库架构设计)
6. [业务流程与交互](#6-业务流程与交互)
7. [安全与隐私保护](#7-安全与隐私保护)
8. [跨设备同步方案](#8-跨设备同步方案)
9. [成就与激励体系](#9-成就与激励体系)
10. [社交互动设计](#10-社交互动设计)
11. [API接口设计](#11-api接口设计)
12. [性能与扩展性](#12-性能与扩展性)
13. [测试验收标准](#13-测试验收标准)
14. [实施路线图](#14-实施路线图)

---

## 1. 概述与定位

### 1.1 用户模块在整体架构中的位置

```
┌─────────────────────────────────────────────────────────────────┐
│                        微书包应用系统                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│  │  用户模块      │  │  书籍模块      │  │  阅读模块      │   │
│  │  (本文档)      │  │                │  │                │   │
│  │                │  │  - 实体书      │  │  - 电子书      │   │
│  │  - 注册登录    │  │  - 电子书      │  │  - 阅读记录    │   │
│  │  - 个人资料    │  │  - 书单        │  │  - 笔记书签    │   │
│  │  - 成就体系    │  │  - 借阅管理    │  │  - 进度同步    │   │
│  │  - 社交关系    │  │                │  │                │   │
│  └────────┬───────┘  └────────┬───────┘  └────────┬───────┘   │
│           │                   │                   │           │
│           └───────────────────┼───────────────────┘           │
│                               │                               │
│  ┌────────────────────────────┴────────────────────────────┐  │
│  │              公共服务层 (认证/权限/存储/通知)              │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 核心设计原则

1. **安全第一**: 用户数据保护、隐私安全、防刷防攻击
2. **体验优先**: 流畅注册、快速登录、跨设备无缝同步
3. **激励导向**: 成就体系、等级勋章、社交互动驱动用户活跃
4. **扩展性强**: 支持第三方登录、多设备管理、权限细粒度控制
5. **数据驱动**: 完整的用户行为分析、阅读习惯追踪

---

## 2. 核心价值主张

### 2.1 对用户的价值

| 用户角色 | 核心需求 | 解决方案 | 价值点 |
|---------|---------|---------|--------|
| 阅读爱好者 | 管理个人藏书、记录阅读进度 | 多设备同步、云端存储 | 随时随地访问书架 |
| 知识工作者 | 做笔记、整理知识 | 笔记高亮、AI问答 | 高效知识管理 |
| 社交型读者 | 分享书评、交流心得 | 社区互动、书单分享 | 找到同好、扩大影响力 |
| 目标导向型 | 完成阅读计划、获得成就 | 目标设定、成就徽章 | 持续激励、可视化成长 |

### 2.2 对业务的价值

- **用户增长**: 低门槛注册、社交裂变
- **用户留存**: 成就体系、社交关系、跨设备粘性
- **商业变现**: 会员体系、增值服务、数据资产
- **数据价值**: 阅读偏好、用户画像、推荐优化

---

## 3. 用户角色与权限体系

### 3.1 角色定义

```sql
-- 用户角色枚举
ENUM role:
  - 'user'        -- 普通用户
  - 'vip'         -- VIP会员
  - 'admin'       -- 管理员
  - 'super_admin' -- 超级管理员
```

### 3.2 权限矩阵

| 功能/操作 | 普通用户 | VIP用户 | 管理员 | 超级管理员 |
|----------|---------|---------|--------|-----------|
| 注册登录 | ✅ | ✅ | ✅ | ✅ |
| 个人资料编辑 | ✅ | ✅ | ✅ | ✅ |
| 书籍录入（扫码/手动） | ✅ 限制50本/月 | ✅ 无限制 | ✅ 无限制 | ✅ 无限制 |
| 电子书上传 | ✅ 限制5本/月 | ✅ 无限制 | ✅ 无限制 | ✅ 无限制 |
| 创建书单 | ✅ 最多10个 | ✅ 无限制 | ✅ 无限制 | ✅ 无限制 |
| 创建群组 | ✅ 最多3个 | ✅ 最多20个 | ✅ 无限制 | ✅ 无限制 |
| 离线下载 | ✅ 最多5本 | ✅ 无限制 | ✅ 无限制 | ✅ 无限制 |
| AI问答 | ✅ 10次/天 | ✅ 100次/天 | ✅ 无限制 | ✅ 无限制 |
| 发布书评 | ✅ | ✅ | ✅ | ✅ |
| 查看他人私藏 | ❌ | ❌ | ❌ | ❌ |
| 删除他人评论 | ❌ | ❌ | ✅ | ✅ |
| 管理轮播图 | ❌ | ❌ | ✅ | ✅ |
| 用户封禁 | ❌ | ❌ | ✅ | ✅ |
| 系统配置 | ❌ | ❌ | ❌ | ✅ |

### 3.3 会员等级体系

```
Level 0: 普通会员 (0-99 XP)
  - 基础功能
  - 广告支持

Level 1: 铜牌会员 (100-299 XP)
  - 基础功能
  - 无广告
  - 9折购书

Level 2: 银牌会员 (300-699 XP)
  - Level 1权益
  - 优先客服
  - 8折购书
  - 高级统计报表

Level 3: 金牌会员 (700-1499 XP)
  - Level 2权益
  - 专属活动
  - 7折购书
  - AI高级功能

Level 4: 钻石会员 (1500+ XP)
  - 全部权益
  - 定制服务
  - 6折购书
  - 优先体验新功能
```

---

## 4. 功能模块详细设计

### 4.1 注册与登录

#### 4.1.1 注册方式

**方式一: 手机号 + 验证码注册**

```
用户操作流程:
1. 输入手机号
2. 点击"获取验证码"
3. 输入验证码
4. 自动完成注册并登录
5. (可选) 完善个人信息
```

**验证码规则:**
- 6位数字
- 有效期: 5分钟
- 单个手机号: 1次/60秒, 5次/天
- 单个IP: 10次/小时, 50次/天
- 验证码内容模板: "【微书包】您的验证码是{code},5分钟内有效,请勿泄露"

**方式二: 手机号 + 密码注册**

```
用户操作流程:
1. 输入手机号
2. 设置密码 (6-16位,含字母和数字)
3. 点击"注册"
4. 自动完成注册并登录
```

**密码规则:**
- 长度: 6-16位
- 必须包含: 字母 + 数字
- 可选包含: 特殊字符(!@#$%^&*)
- 不能是纯数字或纯字母
- 不能包含手机号
- 实时密码强度提示 (弱/中/强)

**方式三: 第三方快捷登录**

支持平台:
- 微信
- QQ
- Apple ID (iOS)
- 支付宝

流程:
```
1. 点击第三方登录按钮
2. 跳转第三方授权页
3. 用户授权
4. 获取用户信息 (openid, unionid, 昵称, 头像)
5. 检查是否已绑定账号
   - 已绑定: 直接登录
   - 未绑定: 创建新账号或绑定现有手机号
```

#### 4.1.2 登录方式

**方式一: 手机号 + 验证码登录**
- 适用场景: 忘记密码、快捷登录
- 安全性: 高

**方式二: 手机号 + 密码登录**
- 适用场景: 常规登录
- 记住密码功能 (7天免登录)

**方式三: 第三方登录**
- 一键登录,无需输入手机号密码

#### 4.1.3 异常处理

| 异常场景 | 错误提示 | 处理方式 |
|---------|---------|---------|
| 手机号已注册 | "该手机号已绑定账号,可直接登录" | 跳转登录页 |
| 验证码错误 | "验证码无效或已过期" | 重新获取 |
| 验证码发送过频繁 | "操作过于频繁,请60秒后再试" | 倒计时等待 |
| 密码格式错误 | "密码需包含数字和字母,长度6-16位" | 实时提示 |
| 密码错误 | "密码错误,还可尝试{n}次" | 记录失败次数 |
| 连续密码错误5次 | "账户已锁定30分钟,请稍后再试" | 锁定账户 |
| 账户已冻结 | "账户已被冻结,请联系客服" | 显示客服联系方式 |
| 网络异常 | "网络连接失败,请检查网络" | 重试按钮 |

#### 4.1.4 首次登录引导

```
首次登录弹窗:
┌────────────────────────────┐
│    完善个人信息 (可跳过)     │
├────────────────────────────┤
│                            │
│   [头像上传区域]            │
│   点击上传或拍照            │
│                            │
│   昵称: [_____________]    │
│                            │
│   性别: ○ 男  ○ 女  ○ 保密  │
│                            │
│   个性签名:                 │
│   [____________________]   │
│   [____________________]   │
│                            │
│   [稍后再说]    [完成]      │
│                            │
└────────────────────────────┘
```

**头像上传功能:**
- 支持格式: JPG, PNG, GIF
- 大小限制: ≤ 5MB
- 裁剪比例: 1:1正方形
- 实时预览
- 支持相机拍照和相册选择

### 4.2 个人信息管理

#### 4.2.1 个人资料

**基础信息:**
- 用户名 (不可修改,唯一标识)
- 昵称 (可修改,2-20字符)
- 头像 (可修改)
- 性别 (男/女/保密)
- 生日 (可选)
- 地理位置 (国家/省份/城市)
- 个性签名 (最多100字)

**社交信息 (可选):**
- 微信号
- QQ号
- 微博账号

**统计数据 (只读):**
- 粉丝数
- 关注数
- 已读图书数
- 累计阅读天数
- 累计阅读时长
- 发表书评数
- 创建书单数
- 阅读积分
- 账户余额

#### 4.2.2 账号与安全

**手机号管理:**
- 查看已绑定手机号 (脱敏显示: 138****5678)
- 更换手机号 (需验证原手机号)
- 绑定/解绑第三方账号

**密码管理:**
- 修改密码 (需验证原密码或验证码)
- 忘记密码 (通过手机验证码重置)

**设备管理:**
- 查看当前登录设备列表
- 远程登出其他设备
- 标记受信任设备

**登录历史:**
- 最近30天登录记录
- 显示: 时间、设备、IP、地理位置
- 异常登录提醒

#### 4.2.3 隐私设置

```
隐私控制面板:
┌────────────────────────────────────┐
│  个人资料可见性                      │
│  ☑ 允许其他用户查看我的个人资料       │
│  ☑ 允许其他用户在搜索中找到我         │
│                                    │
│  动态与内容                          │
│  ☑ 允许其他用户查看我的阅读动态       │
│  ☐ 允许其他用户查看我的书架          │
│  ☑ 允许其他用户查看我的书单          │
│                                    │
│  社交互动                           │
│  ☑ 允许任何人向我发送好友请求         │
│  ☐ 仅允许好友查看我的书评            │
│  ☑ 允许他人@我                       │
│                                    │
│  数据与推荐                          │
│  ☑ 允许基于我的阅读习惯推荐内容       │
│  ☐ 允许在推荐中展示我的活动          │
│                                    │
└────────────────────────────────────┘
```

#### 4.2.4 偏好设置

**阅读偏好:**
- 字体大小 (12-30px, 默认16px)
- 字体样式 (系统默认/宋体/黑体/楷体)
- 行间距 (1.0-2.5, 默认1.5)
- 翻页模式 (滚动/仿真翻页/滑动)
- 阅读背景 (白色/护眼绿/夜间模式)
- 自动保存书签 (开/关)
- 屏幕常亮 (开/关)

**通知设置:**
- 系统通知 (开/关)
- 邮件通知 (开/关)
- 短信通知 (开/关)
- 推送通知 (开/关)
- 消息免打扰时段设置

**语言与地区:**
- 语言偏好 (中文简体/中文繁体/English)
- 时区设置 (自动检测/手动选择)

### 4.3 成就与等级体系

#### 4.3.1 经验值(XP)获取规则

| 行为 | 获得XP | 频率限制 | 说明 |
|-----|--------|---------|------|
| 完成注册 | +20 | 一次性 | 新用户奖励 |
| 完善个人资料 | +10 | 一次性 | 上传头像+填写昵称 |
| 完成一本书 | +50 | 无限制 | 阅读进度达100% |
| 发表书评 (≥50字) | +10 | 10次/天 | 有效内容 |
| 书评被点赞 | +2 | 无限制 | 每个赞 |
| 创建书单 | +15 | 5次/天 | 至少包含5本书 |
| 书单被收藏 | +5 | 无限制 | 每次收藏 |
| 连续阅读7天 | +30 | 无限制 | 每个周期 |
| 连续阅读30天 | +150 | 无限制 | 每个周期 |
| 邀请好友注册 | +25 | 无限制 | 好友完成首次阅读 |
| 参与话题讨论 | +5 | 10次/天 | 有效回复 |
| 上传电子书 | +8 | 5次/天 | 完整书籍 |

#### 4.3.2 成就徽章系统

**阅读类成就:**

| 成就名称 | 解锁条件 | 徽章图标 | 奖励XP |
|---------|---------|---------|--------|
| 初出茅庐 | 完成第1本书 | 🌱 | +20 |
| 渐入佳境 | 完成第10本书 | 📚 | +50 |
| 阅读达人 | 完成第50本书 | 📖 | +100 |
| 阅读大师 | 完成第100本书 | 🏆 | +200 |
| 阅读狂人 | 完成第500本书 | 👑 | +500 |

**时长类成就:**

| 成就名称 | 解锁条件 | 徽章图标 | 奖励XP |
|---------|---------|---------|--------|
| 初识时光 | 累计阅读1小时 | ⏱️ | +10 |
| 时光旅人 | 累计阅读10小时 | ⏰ | +30 |
| 时光行者 | 累计阅读100小时 | ⌚ | +100 |
| 时光守护者 | 累计阅读1000小时 | 🕐 | +300 |

**坚持类成就:**

| 成就名称 | 解锁条件 | 徽章图标 | 奖励XP |
|---------|---------|---------|--------|
| 初心不改 | 连续阅读3天 | 🔥 | +15 |
| 持之以恒 | 连续阅读7天 | 🌟 | +30 |
| 铁杵成针 | 连续阅读30天 | ⭐ | +150 |
| 百折不挠 | 连续阅读100天 | 💫 | +500 |
| 恒心永固 | 连续阅读365天 | 🏅 | +2000 |

**社交类成就:**

| 成就名称 | 解锁条件 | 徽章图标 | 奖励XP |
|---------|---------|---------|--------|
| 书评新人 | 发表第1条书评 | ✍️ | +10 |
| 书评达人 | 发表第10条书评 | 📝 | +30 |
| 意见领袖 | 书评获赞100次 | 💬 | +100 |
| 书单专家 | 创建10个书单 | 📋 | +50 |
| 分享之星 | 书单被收藏100次 | ⭐ | +150 |
| 社交达人 | 拥有50个粉丝 | 👥 | +100 |

#### 4.3.3 等级称号

```
Level 0 (0-99 XP): 书童
Level 1 (100-299 XP): 书虫
Level 2 (300-699 XP): 书友
Level 3 (700-1499 XP): 书迷
Level 4 (1500-2999 XP): 书痴
Level 5 (3000-5999 XP): 书评家
Level 6 (6000-9999 XP): 阅读达人
Level 7 (10000-19999 XP): 阅读大师
Level 8 (20000-39999 XP): 文学博士
Level 9 (40000-79999 XP): 知识导师
Level 10 (80000+ XP): 智慧贤者
```

**等级权益:**

| 等级 | 书架容量 | 离线下载 | AI问答 | 专属标识 | 其他权益 |
|-----|---------|---------|--------|---------|---------|
| 0-1 | 100本 | 3本 | 5次/天 | 无 | 基础功能 |
| 2-3 | 300本 | 5本 | 10次/天 | 铜牌标识 | 去广告 |
| 4-5 | 500本 | 10本 | 20次/天 | 银牌标识 | 优先客服 |
| 6-7 | 1000本 | 20本 | 50次/天 | 金牌标识 | 高级统计 |
| 8-9 | 3000本 | 50本 | 100次/天 | 钻石标识 | 专属活动 |
| 10+ | 无限制 | 无限制 | 无限制 | 王冠标识 | 定制服务 |

### 4.4 社交关系管理

#### 4.4.1 好友系统

**添加好友方式:**
1. 搜索用户名/手机号
2. 扫描二维码
3. 从通讯录导入
4. 推荐好友 (可能认识的人)
5. 群组成员列表

**好友请求流程:**
```
用户A → 发送好友请求 (附带验证消息)
       ↓
用户B ← 收到请求通知
       ↓
       选择: [接受] [拒绝] [忽略]
       ↓
[接受] → 成为好友,双向关注
[拒绝] → 通知A "对方拒绝了你的好友请求"
[忽略] → 不做处理,请求保留
```

**好友管理功能:**
- 好友列表 (按最近聊天/字母排序)
- 好友分组 (自定义标签)
- 特别关注 (置顶显示)
- 好友备注 (仅自己可见)
- 删除好友
- 拉黑用户

#### 4.4.2 关注系统

**关注 vs 好友的区别:**
- 关注: 单向关系,可看到对方公开动态
- 好友: 双向关系,可私信互动

**关注功能:**
- 关注用户
- 取消关注
- 查看关注列表
- 查看粉丝列表
- 互相关注 (互粉)

#### 4.4.3 屏蔽与举报

**屏蔽功能:**
- 屏蔽用户 (对方无法查看你的动态、评论)
- 屏蔽列表管理
- 解除屏蔽

**举报功能:**
- 举报理由: 垃圾广告/恶意辱骂/色情低俗/侵犯隐私/其他
- 提交证据 (截图)
- 举报处理流程: 审核 → 警告/封禁/永久封号
- 举报反馈通知

---

## 5. 数据库架构设计

### 5.1 核心表结构

#### 5.1.1 用户基础信息表 (users)

```sql
CREATE TABLE users (
    -- 主键和标识
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '用户ID',
    uuid VARCHAR(36) UNIQUE NOT NULL COMMENT '用户UUID,全局唯一',
    username VARCHAR(50) UNIQUE NOT NULL COMMENT '用户名',

    -- 认证信息
    password_hash VARCHAR(255) COMMENT 'bcrypt密码哈希',
    phone_number VARCHAR(20) UNIQUE COMMENT '手机号(AES-256加密)',
    phone_verified BOOLEAN DEFAULT FALSE COMMENT '手机号已验证',
    email VARCHAR(100) UNIQUE COMMENT '邮箱',
    email_verified BOOLEAN DEFAULT FALSE COMMENT '邮箱已验证',

    -- 账户状态
    status TINYINT DEFAULT 1 COMMENT '1-正常 2-冻结 3-注销 4-待激活',
    is_verified BOOLEAN DEFAULT FALSE COMMENT '实名认证',

    -- 角色和等级
    role ENUM('user', 'admin', 'super_admin', 'vip') DEFAULT 'user',
    member_level TINYINT DEFAULT 0 COMMENT '会员等级0-4',
    member_expire_at DATETIME COMMENT '会员到期时间',

    -- 成就系统
    xp INT DEFAULT 0 COMMENT '经验值',
    level TINYINT DEFAULT 0 COMMENT '用户等级0-10',

    -- 安全信息
    failed_login_attempts INT DEFAULT 0 COMMENT '连续登录失败次数',
    locked_until DATETIME COMMENT '账户锁定截止时间',
    last_login_at DATETIME COMMENT '最后登录时间',
    last_login_ip VARCHAR(45) COMMENT '最后登录IP',

    -- 时间戳
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at DATETIME COMMENT '软删除时间',

    -- 索引
    INDEX idx_username (username),
    INDEX idx_phone (phone_number),
    INDEX idx_email (email),
    INDEX idx_status (status),
    INDEX idx_level (level),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户基础表';
```

#### 5.1.2 用户详细资料表 (user_profiles)

```sql
CREATE TABLE user_profiles (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT UNIQUE NOT NULL,

    -- 个人信息
    nickname VARCHAR(50) COMMENT '昵称',
    real_name VARCHAR(50) COMMENT '真实姓名(加密)',
    avatar_url VARCHAR(255) COMMENT '头像URL',
    gender TINYINT COMMENT '0-未知 1-男 2-女',
    birthday DATE COMMENT '生日',

    -- 地理位置
    country VARCHAR(50) COMMENT '国家',
    province VARCHAR(50) COMMENT '省份',
    city VARCHAR(50) COMMENT '城市',

    -- 个人描述
    bio TEXT COMMENT '个人简介',
    signature VARCHAR(255) COMMENT '个性签名',

    -- 社交账号
    wechat VARCHAR(50) COMMENT '微信号',
    qq VARCHAR(20) COMMENT 'QQ号',
    weibo VARCHAR(50) COMMENT '微博',

    -- 统计数据
    followers_count INT DEFAULT 0 COMMENT '粉丝数',
    following_count INT DEFAULT 0 COMMENT '关注数',
    books_read_count INT DEFAULT 0 COMMENT '已读书籍数',
    reading_days INT DEFAULT 0 COMMENT '累计阅读天数',
    reading_minutes INT DEFAULT 0 COMMENT '累计阅读分钟数',
    reviews_count INT DEFAULT 0 COMMENT '书评数',
    book_lists_count INT DEFAULT 0 COMMENT '书单数',
    points INT DEFAULT 0 COMMENT '积分',
    balance DECIMAL(10, 2) DEFAULT 0.00 COMMENT '余额',

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_nickname (nickname),
    INDEX idx_city (city)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 5.1.3 第三方登录绑定表 (user_oauth_bindings)

```sql
CREATE TABLE user_oauth_bindings (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,

    -- 第三方平台
    provider VARCHAR(20) NOT NULL COMMENT 'wechat/qq/apple/google/alipay',
    provider_user_id VARCHAR(100) NOT NULL COMMENT '第三方用户ID',
    openid VARCHAR(100) COMMENT 'OpenID',
    unionid VARCHAR(100) COMMENT 'UnionID',

    -- 第三方用户信息
    provider_nickname VARCHAR(100) COMMENT '第三方昵称',
    provider_avatar VARCHAR(255) COMMENT '第三方头像',

    -- 访问令牌
    access_token TEXT COMMENT '访问令牌(加密)',
    refresh_token TEXT COMMENT '刷新令牌(加密)',
    expires_at DATETIME COMMENT '令牌过期时间',

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY uk_provider_openid (provider, openid),
    INDEX idx_user_id (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 5.1.4 用户设备表 (user_devices)

```sql
CREATE TABLE user_devices (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,

    -- 设备信息
    device_id VARCHAR(100) UNIQUE NOT NULL COMMENT '设备唯一标识',
    device_name VARCHAR(100) COMMENT '设备名称',
    device_type VARCHAR(20) COMMENT 'ios/android/web/desktop',
    os_version VARCHAR(50) COMMENT '操作系统版本',
    app_version VARCHAR(20) COMMENT 'APP版本',

    -- 设备详情
    brand VARCHAR(50) COMMENT '设备品牌',
    model VARCHAR(50) COMMENT '设备型号',

    -- 令牌
    push_token VARCHAR(255) COMMENT '推送令牌',
    refresh_token VARCHAR(255) UNIQUE COMMENT '刷新令牌',

    -- 登录信息
    last_active_at DATETIME COMMENT '最后活跃时间',
    last_ip VARCHAR(45) COMMENT '最后IP',
    login_count INT DEFAULT 1 COMMENT '登录次数',

    -- 状态
    is_active BOOLEAN DEFAULT TRUE COMMENT '是否活跃',
    is_trusted BOOLEAN DEFAULT FALSE COMMENT '是否受信任',

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_device_id (device_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 5.1.5 用户偏好设置表 (user_preferences)

```sql
CREATE TABLE user_preferences (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT UNIQUE NOT NULL,

    -- 基础设置
    language VARCHAR(10) DEFAULT 'zh-CN',
    timezone VARCHAR(50) DEFAULT 'Asia/Shanghai',
    theme VARCHAR(20) DEFAULT 'light' COMMENT 'light/dark/auto',

    -- 通知设置
    notification_enabled BOOLEAN DEFAULT TRUE,
    email_notification BOOLEAN DEFAULT TRUE,
    sms_notification BOOLEAN DEFAULT FALSE,
    push_notification BOOLEAN DEFAULT TRUE,

    -- 隐私设置
    profile_visible BOOLEAN DEFAULT TRUE COMMENT '资料公开',
    show_online_status BOOLEAN DEFAULT TRUE COMMENT '显示在线状态',
    allow_search BOOLEAN DEFAULT TRUE COMMENT '允许被搜索',
    allow_friend_request BOOLEAN DEFAULT TRUE COMMENT '允许好友请求',

    -- 阅读偏好
    font_size TINYINT DEFAULT 16 COMMENT '字体大小',
    font_family VARCHAR(50) DEFAULT 'system',
    line_spacing DECIMAL(3, 1) DEFAULT 1.5 COMMENT '行间距',
    page_mode VARCHAR(20) DEFAULT 'scroll' COMMENT 'scroll/flip/slide',
    reading_background VARCHAR(20) DEFAULT 'white',
    auto_bookmark BOOLEAN DEFAULT TRUE,
    reading_reminder BOOLEAN DEFAULT FALSE,
    preferred_genres JSON COMMENT '偏好分类',

    -- 扩展设置
    extra_settings JSON,

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 5.1.6 成就记录表 (user_achievements)

```sql
CREATE TABLE user_achievements (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,

    -- 成就信息
    achievement_code VARCHAR(50) NOT NULL COMMENT '成就代码',
    achievement_name VARCHAR(100) COMMENT '成就名称',
    achievement_description TEXT COMMENT '成就描述',
    badge_icon_url VARCHAR(255) COMMENT '徽章图标',

    -- 成就等级
    achievement_level TINYINT DEFAULT 1,
    achievement_points INT DEFAULT 0 COMMENT '获得积分',

    -- 触发方式
    trigger_type VARCHAR(50) COMMENT 'read_books/reviews/streak',
    trigger_value INT COMMENT '触发数值',

    unlocked_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY uk_user_achievement (user_id, achievement_code),
    INDEX idx_user_id (user_id),
    INDEX idx_achievement_code (achievement_code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 5.1.7 经验值记录表 (user_xp_logs)

```sql
CREATE TABLE user_xp_logs (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,

    -- 行为信息
    action_type VARCHAR(50) NOT NULL COMMENT 'complete_book/review/streak',
    action_description VARCHAR(255) COMMENT '行为描述',

    -- XP变化
    xp_delta INT NOT NULL COMMENT 'XP变化量(可为负)',
    xp_before INT NOT NULL COMMENT '变化前XP',
    xp_after INT NOT NULL COMMENT '变化后XP',

    -- 等级变化
    level_before TINYINT COMMENT '变化前等级',
    level_after TINYINT COMMENT '变化后等级',

    -- 关联资源
    resource_type VARCHAR(50) COMMENT 'book/review/booklist',
    resource_id BIGINT COMMENT '资源ID',

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_action_type (action_type),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 5.1.8 用户关系表 (user_relationships)

```sql
CREATE TABLE user_relationships (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL COMMENT '用户A',
    target_user_id BIGINT NOT NULL COMMENT '用户B',

    -- 关系类型
    relationship_type ENUM('follow', 'friend', 'block') NOT NULL,

    -- 好友请求
    request_message VARCHAR(255) COMMENT '好友请求消息',
    request_status ENUM('pending', 'accepted', 'rejected') DEFAULT 'pending',

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (target_user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY uk_user_relationship (user_id, target_user_id, relationship_type),
    INDEX idx_user_id (user_id),
    INDEX idx_target_user_id (target_user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 5.1.9 登录日志表 (user_login_logs)

```sql
CREATE TABLE user_login_logs (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,

    -- 登录信息
    login_type ENUM('password', 'sms', 'email', 'oauth', 'qrcode') NOT NULL,
    login_status ENUM('success', 'failed', 'blocked') NOT NULL,
    fail_reason VARCHAR(255) COMMENT '失败原因',

    -- 设备和网络
    device_id VARCHAR(100),
    device_type VARCHAR(20),
    ip_address VARCHAR(45),
    location VARCHAR(100) COMMENT '地理位置',
    user_agent TEXT,

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at),
    INDEX idx_ip_address (ip_address)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 5.2 数据加密策略

#### 5.2.1 敏感字段加密

```python
# 使用AES-256-GCM加密敏感数据

from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os
import base64

class DataEncryption:
    """数据加密服务"""

    def __init__(self, master_key: bytes):
        self.master_key = master_key
        self.aesgcm = AESGCM(master_key)

    def encrypt_phone_number(self, phone: str) -> dict:
        """加密手机号"""
        # 生成随机IV(12字节)
        iv = os.urandom(12)

        # 加密
        ciphertext = self.aesgcm.encrypt(iv, phone.encode(), None)

        return {
            'encrypted': base64.b64encode(ciphertext).decode(),
            'iv': base64.b64encode(iv).decode()
        }

    def decrypt_phone_number(self, encrypted: str, iv: str) -> str:
        """解密手机号"""
        ciphertext = base64.b64decode(encrypted)
        iv_bytes = base64.b64decode(iv)

        plaintext = self.aesgcm.decrypt(iv_bytes, ciphertext, None)
        return plaintext.decode()

# 密码哈希
import bcrypt

def hash_password(password: str) -> str:
    """使用bcrypt哈希密码"""
    salt = bcrypt.gensalt(rounds=12)
    hashed = bcrypt.hashpw(password.encode(), salt)
    return hashed.decode()

def verify_password(password: str, hashed: str) -> bool:
    """验证密码"""
    return bcrypt.checkpw(password.encode(), hashed.encode())
```

---

## 6. 业务流程与交互

### 6.1 注册流程图

```
┌────────────────────────────────────────────────────────────────┐
│                       用户注册完整流程                          │
└────────────────────────────────────────────────────────────────┘

用户                    前端                    后端                    第三方
 │                      │                      │                      │
 │  1.选择注册方式       │                      │                      │
 ├─────────────────────>│                      │                      │
 │                      │                      │                      │
 │  [方式A: 手机号+验证码]                      │                      │
 │                      │                      │                      │
 │  2.输入手机号         │                      │                      │
 ├─────────────────────>│                      │                      │
 │                      │  3.请求验证码         │                      │
 │                      ├─────────────────────>│                      │
 │                      │                      │  4.发送短信          │
 │                      │                      ├─────────────────────>│
 │                      │                      │<─────────────────────┤
 │                      │  5.验证码已发送       │                      │
 │                      │<─────────────────────┤                      │
 │  6.倒计时60秒         │                      │                      │
 │<─────────────────────┤                      │                      │
 │                      │                      │                      │
 │  7.输入验证码         │                      │                      │
 ├─────────────────────>│                      │                      │
 │                      │  8.提交注册          │                      │
 │                      ├─────────────────────>│                      │
 │                      │                      │  9.校验验证码         │
 │                      │                      │  10.创建用户          │
 │                      │                      │  11.生成token        │
 │                      │  12.注册成功          │                      │
 │                      │<─────────────────────┤                      │
 │  13.跳转首次引导页     │                      │                      │
 │<─────────────────────┤                      │                      │
 │                      │                      │                      │
 │  14.完善个人信息       │                      │                      │
 │  (上传头像/昵称)       │                      │                      │
 ├─────────────────────>│                      │                      │
 │                      │  15.更新用户资料      │                      │
 │                      ├─────────────────────>│                      │
 │                      │  16.完成              │                      │
 │                      │<─────────────────────┤                      │
 │  17.进入首页          │                      │                      │
 │<─────────────────────┤                      │                      │
```

### 6.2 登录流程图

```
┌────────────────────────────────────────────────────────────────┐
│                       用户登录流程                              │
└────────────────────────────────────────────────────────────────┘

用户                    前端                    后端                    缓存
 │                      │                      │                      │
 │  1.输入手机号+密码     │                      │                      │
 ├─────────────────────>│                      │                      │
 │                      │  2.提交登录          │                      │
 │                      ├─────────────────────>│                      │
 │                      │                      │  3.查询用户           │
 │                      │                      │  (by phone)          │
 │                      │                      │                      │
 │                      │                      │  4.验证密码           │
 │                      │                      │  (bcrypt.verify)     │
 │                      │                      │                      │
 │                      │                      │  5.检查账户状态       │
 │                      │                      │  (status=1正常)      │
 │                      │                      │                      │
 │                      │                      │  6.检查失败次数       │
 │                      │                      ├─────────────────────>│
 │                      │                      │<─────────────────────┤
 │                      │                      │  (Redis: login_fail) │
 │                      │                      │                      │
 │                      │                      │  7.生成token         │
 │                      │                      │  8.记录登录日志       │
 │                      │                      │  9.更新设备表         │
 │                      │                      │  10.清除失败次数      │
 │                      │                      │                      │
 │                      │  11.返回token+用户信息│                      │
 │                      │<─────────────────────┤                      │
 │                      │                      │                      │
 │  12.保存token到本地   │                      │                      │
 │  13.跳转首页          │                      │                      │
 │<─────────────────────┤                      │                      │
```

### 6.3 跨设备同步流程

```
┌────────────────────────────────────────────────────────────────┐
│                   跨设备实时同步流程                             │
└────────────────────────────────────────────────────────────────┘

设备A(iPhone)         后端同步服务              设备B(iPad)
     │                      │                      │
     │  1.用户在设备A阅读    │                      │
     │  (翻到第50页)         │                      │
     │                      │                      │
     │  2.上报进度更新       │                      │
     ├─────────────────────>│                      │
     │  {book_id: 123,      │                      │
     │   page: 50,          │                      │
     │   progress: 45%}     │                      │
     │                      │                      │
     │                      │  3.保存到数据库        │
     │                      │  4.更新Redis缓存      │
     │                      │  5.检查该用户的其他设备│
     │                      │                      │
     │                      │  6.推送同步消息       │
     │                      ├─────────────────────>│
     │                      │  (WebSocket)         │
     │                      │                      │
     │                      │                      │  7.收到同步消息
     │                      │                      │  8.更新本地进度
     │                      │                      │  9.刷新UI显示
     │                      │                      │
     │  10.返回同步成功      │                      │
     │<─────────────────────┤                      │
```

### 6.4 成就解锁流程

```
┌────────────────────────────────────────────────────────────────┐
│                     成就解锁流程                                │
└────────────────────────────────────────────────────────────────┘

用户操作             触发事件             成就检测              通知服务
   │                    │                    │                    │
   │  完成一本书         │                    │                    │
   ├───────────────────>│                    │                    │
   │                    │  1.记录阅读完成     │                    │
   │                    │  2.更新统计数据     │                    │
   │                    │                    │                    │
   │                    │  3.触发成就检测     │                    │
   │                    ├───────────────────>│                    │
   │                    │                    │  4.查询用户已读数   │
   │                    │                    │  5.匹配成就规则     │
   │                    │                    │                    │
   │                    │                    │  [规则: 完成1本书]  │
   │                    │                    │  -> 解锁"初出茅庐"  │
   │                    │                    │                    │
   │                    │                    │  6.插入成就记录     │
   │                    │                    │  7.增加XP          │
   │                    │                    │  8.检查等级提升     │
   │                    │                    │                    │
   │                    │                    │  9.发送通知         │
   │                    │                    ├───────────────────>│
   │                    │                    │                    │
   │                    │                    │                    │  10.推送消息
   │  收到成就通知        │                    │                    │  "恭喜解锁成就"
   │<───────────────────────────────────────────────────────────────┤
   │                    │                    │                    │
   │  [弹窗展示徽章]     │                    │                    │
```

---

## 7. 安全与隐私保护

### 7.1 密码安全策略

**密码强度要求:**
- 最少6位,最多16位
- 必须包含字母和数字
- 推荐包含特殊字符
- 不能是纯数字或纯字母
- 不能包含手机号

**密码存储:**
```python
# 使用bcrypt,自动加盐
import bcrypt

def hash_password(password: str) -> str:
    # cost factor = 12
    salt = bcrypt.gensalt(rounds=12)
    return bcrypt.hashpw(password.encode(), salt).decode()
```

**密码策略:**
- 90天提醒修改密码
- 禁止使用最近5次使用过的密码
- 连续5次登录失败锁定30分钟

### 7.2 验证码安全

**验证码规则:**
- 6位数字
- 有效期5分钟
- 同一手机号: 1次/60秒, 5次/天
- 同一IP: 10次/小时, 50次/天

**防刷策略:**
```python
import redis
from datetime import timedelta

class SMSRateLimiter:
    """短信发送频率限制"""

    def __init__(self, redis_client):
        self.redis = redis_client

    def check_phone_limit(self, phone: str) -> bool:
        """检查手机号限制"""
        # 60秒限制
        key_60s = f'sms:phone:60s:{phone}'
        if self.redis.exists(key_60s):
            return False

        # 24小时限制(5次)
        key_24h = f'sms:phone:24h:{phone}'
        count = self.redis.get(key_24h) or 0
        if int(count) >= 5:
            return False

        return True

    def record_phone_send(self, phone: str):
        """记录发送"""
        # 60秒key
        key_60s = f'sms:phone:60s:{phone}'
        self.redis.setex(key_60s, 60, 1)

        # 24小时计数
        key_24h = f'sms:phone:24h:{phone}'
        self.redis.incr(key_24h)
        self.redis.expire(key_24h, 86400)

    def check_ip_limit(self, ip: str) -> bool:
        """检查IP限制"""
        # 1小时限制(10次)
        key_1h = f'sms:ip:1h:{ip}'
        count_1h = self.redis.get(key_1h) or 0
        if int(count_1h) >= 10:
            return False

        # 24小时限制(50次)
        key_24h = f'sms:ip:24h:{ip}'
        count_24h = self.redis.get(key_24h) or 0
        if int(count_24h) >= 50:
            return False

        return True
```

### 7.3 敏感数据保护

**数据脱敏:**
```python
def mask_phone(phone: str) -> str:
    """手机号脱敏: 138****5678"""
    if len(phone) == 11:
        return phone[:3] + '****' + phone[7:]
    return phone

def mask_email(email: str) -> str:
    """邮箱脱敏: u***@example.com"""
    username, domain = email.split('@')
    if len(username) <= 1:
        masked_username = username
    else:
        masked_username = username[0] + '***'
    return f'{masked_username}@{domain}'

def mask_id_card(id_card: str) -> str:
    """身份证脱敏: 3301**********1234"""
    if len(id_card) == 18:
        return id_card[:4] + '**********' + id_card[14:]
    return id_card
```

### 7.4 权限控制

```python
from functools import wraps
from flask import request, jsonify

def require_auth(f):
    """需要登录"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        token = request.headers.get('Authorization')
        if not token:
            return jsonify({'error': 'Unauthorized'}), 401

        user = verify_token(token)
        if not user:
            return jsonify({'error': 'Invalid token'}), 401

        return f(user, *args, **kwargs)
    return decorated_function

def require_role(role: str):
    """需要特定角色"""
    def decorator(f):
        @wraps(f)
        def decorated_function(user, *args, **kwargs):
            if user.role != role and user.role != 'super_admin':
                return jsonify({'error': 'Forbidden'}), 403
            return f(user, *args, **kwargs)
        return decorated_function
    return decorator

# 使用示例
@app.route('/admin/users')
@require_auth
@require_role('admin')
def admin_users(user):
    # 只有admin和super_admin可以访问
    pass
```

---

## 8. 跨设备同步方案

### 8.1 WebSocket实时同步

```python
import asyncio
import websockets
import json

class SyncGateway:
    """同步网关"""

    def __init__(self):
        self.connections = {}  # {user_id: {device_id: websocket}}

    async def handle_connection(self, websocket, user_id, device_id):
        """处理客户端连接"""
        # 注册连接
        if user_id not in self.connections:
            self.connections[user_id] = {}
        self.connections[user_id][device_id] = websocket

        try:
            # 发送初始同步数据
            await self.send_initial_sync(websocket, user_id, device_id)

            # 持续监听消息
            async for message in websocket:
                await self.handle_message(user_id, device_id, json.loads(message))

        except websockets.ConnectionClosed:
            pass
        finally:
            # 移除连接
            if user_id in self.connections:
                self.connections[user_id].pop(device_id, None)

    async def handle_message(self, user_id, device_id, message):
        """处理同步消息"""
        msg_type = message['type']

        if msg_type == 'sync_progress':
            # 同步阅读进度
            await self.sync_reading_progress(user_id, device_id, message)

        elif msg_type == 'sync_bookmark':
            # 同步书签
            await self.sync_bookmark(user_id, device_id, message)

        elif msg_type == 'sync_note':
            # 同步笔记
            await self.sync_note(user_id, device_id, message)

    async def broadcast(self, user_id, source_device_id, message):
        """广播到用户的其他设备"""
        if user_id not in self.connections:
            return

        for device_id, ws in self.connections[user_id].items():
            if device_id != source_device_id:
                await ws.send(json.dumps(message))
```

### 8.2 冲突解决策略

**策略1: 最新写入优先 (Last-Write-Wins)**

适用于:
- 阅读进度
- 用户偏好设置

```python
def resolve_latest_wins(data_a, data_b):
    """最新时间戳的数据获胜"""
    if data_a['timestamp'] > data_b['timestamp']:
        return data_a
    else:
        return data_b
```

**策略2: 合并 (Merge)**

适用于:
- 书签列表
- 笔记列表

```python
def resolve_merge(data_a, data_b, merge_type):
    """合并两边的数据"""
    if merge_type == 'bookmark':
        # 合并书签,去重
        bookmarks = data_a['bookmarks'] + data_b['bookmarks']
        unique_bookmarks = {b['id']: b for b in bookmarks}.values()
        return {'bookmarks': list(unique_bookmarks)}
```

---

## 9. 成就与激励体系

### 9.1 成就系统实现

```python
class AchievementSystem:
    """成就系统"""

    # 成就定义
    ACHIEVEMENTS = {
        'first_book': {
            'name': '初出茅庐',
            'description': '完成第1本书',
            'icon': '🌱',
            'xp': 20,
            'trigger': 'complete_book',
            'threshold': 1
        },
        'ten_books': {
            'name': '渐入佳境',
            'description': '完成第10本书',
            'icon': '📚',
            'xp': 50,
            'trigger': 'complete_book',
            'threshold': 10
        },
        'streak_7': {
            'name': '持之以恒',
            'description': '连续阅读7天',
            'icon': '🌟',
            'xp': 30,
            'trigger': 'reading_streak',
            'threshold': 7
        }
    }

    async def check_achievements(self, user_id: int, trigger: str, value: int):
        """检查是否解锁成就"""
        unlocked = []

        for code, achievement in self.ACHIEVEMENTS.items():
            if achievement['trigger'] == trigger:
                # 检查是否已解锁
                existing = await db.fetchone("""
                    SELECT id FROM user_achievements
                    WHERE user_id = %s AND achievement_code = %s
                """, (user_id, code))

                if not existing and value >= achievement['threshold']:
                    # 解锁成就
                    await self.unlock_achievement(user_id, code, achievement)
                    unlocked.append(achievement)

        return unlocked

    async def unlock_achievement(self, user_id: int, code: str, achievement: dict):
        """解锁成就"""
        # 1. 插入成就记录
        await db.execute("""
            INSERT INTO user_achievements
            (user_id, achievement_code, achievement_name, achievement_description,
             badge_icon_url, achievement_points, trigger_type, trigger_value, unlocked_at)
            VALUES (%s, %s, %s, %s, %s, %s, %s, %s, NOW())
        """, (
            user_id, code, achievement['name'], achievement['description'],
            achievement['icon'], achievement['xp'],
            achievement['trigger'], achievement['threshold']
        ))

        # 2. 增加用户XP
        await self.add_xp(user_id, achievement['xp'], f"解锁成就: {achievement['name']}")

        # 3. 推送通知
        await notification.send_push(
            user_id,
            title=f"🎉 解锁新成就: {achievement['name']}",
            body=achievement['description']
        )
```

---

## 10. 社交互动设计

### 10.1 好友系统

```python
class FriendshipService:
    """好友服务"""

    async def send_friend_request(self, from_user_id: int, to_user_id: int, message: str = None):
        """发送好友请求"""
        # 1. 检查是否已经是好友
        existing = await db.fetchone("""
            SELECT id FROM user_relationships
            WHERE user_id = %s AND target_user_id = %s
            AND relationship_type = 'friend'
        """, (from_user_id, to_user_id))

        if existing:
            raise ValueError("Already friends")

        # 2. 创建好友请求
        await db.execute("""
            INSERT INTO user_relationships
            (user_id, target_user_id, relationship_type, request_message, request_status)
            VALUES (%s, %s, 'friend', %s, 'pending')
        """, (from_user_id, to_user_id, message))

        # 3. 推送通知
        from_user = await self.get_user(from_user_id)
        await notification.send_push(
            to_user_id,
            title="新的好友请求",
            body=f"{from_user.nickname} 请求添加你为好友"
        )

    async def accept_friend_request(self, user_id: int, request_id: int):
        """接受好友请求"""
        # 1. 更新请求状态
        await db.execute("""
            UPDATE user_relationships
            SET request_status = 'accepted', updated_at = NOW()
            WHERE id = %s AND target_user_id = %s
        """, (request_id, user_id))

        # 2. 创建反向关系(双向好友)
        request = await db.fetchone("""
            SELECT user_id, target_user_id FROM user_relationships WHERE id = %s
        """, (request_id,))

        await db.execute("""
            INSERT INTO user_relationships
            (user_id, target_user_id, relationship_type, request_status)
            VALUES (%s, %s, 'friend', 'accepted')
        """, (request['target_user_id'], request['user_id']))
```

---

## 11. API接口设计

### 11.1 认证相关接口

#### POST /api/v1/auth/send-otp
发送验证码

**请求:**
```json
{
  "phone_number": "13800138000",
  "purpose": "register" // register | login | reset_password
}
```

**响应:**
```json
{
  "code": 200,
  "message": "验证码已发送",
  "data": {
    "expires_in": 300, // 5分钟
    "retry_after": 60  // 60秒后可重试
  }
}
```

#### POST /api/v1/auth/register
注册

**请求:**
```json
{
  "phone_number": "13800138000",
  "verification_code": "123456",
  "password": "abc123" // 可选
}
```

**响应:**
```json
{
  "code": 200,
  "message": "注册成功",
  "data": {
    "user_id": 12345,
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "...",
    "expires_in": 7200,
    "is_new_user": true
  }
}
```

#### POST /api/v1/auth/login
登录

**请求:**
```json
{
  "phone_number": "13800138000",
  "login_type": "password", // password | otp
  "password": "abc123",     // login_type=password时必需
  "verification_code": "123456", // login_type=otp时必需
  "device_id": "device_xxx"
}
```

**响应:**
```json
{
  "code": 200,
  "message": "登录成功",
  "data": {
    "user_id": 12345,
    "token": "...",
    "refresh_token": "...",
    "user": {
      "id": 12345,
      "username": "user123",
      "nickname": "书虫",
      "avatar_url": "...",
      "level": 3,
      "xp": 850
    }
  }
}
```

### 11.2 用户相关接口

#### GET /api/v1/users/me
获取当前用户信息

**响应:**
```json
{
  "code": 200,
  "data": {
    "id": 12345,
    "username": "user123",
    "nickname": "书虫",
    "avatar_url": "...",
    "phone_number": "138****5678",
    "level": 3,
    "xp": 850,
    "member_level": 2,
    "profile": {
      "gender": 1,
      "birthday": "1990-01-01",
      "city": "上海",
      "bio": "热爱阅读"
    },
    "stats": {
      "followers_count": 120,
      "following_count": 80,
      "books_read_count": 45,
      "reading_days": 230,
      "reading_minutes": 12500
    }
  }
}
```

#### PUT /api/v1/users/profile
更新个人资料

**请求:**
```json
{
  "nickname": "新昵称",
  "bio": "个人简介",
  "city": "上海",
  "avatar_url": "..."
}
```

---

## 12. 性能与扩展性

### 12.1 缓存策略

```python
import redis
import json

class UserCache:
    """用户缓存服务"""

    def __init__(self, redis_client):
        self.redis = redis_client

    async def get_user(self, user_id: int):
        """获取用户信息(优先从缓存)"""
        # 1. 尝试从缓存获取
        cache_key = f'user:{user_id}'
        cached = self.redis.get(cache_key)

        if cached:
            return json.loads(cached)

        # 2. 从数据库查询
        user = await db.fetch_user(user_id)

        # 3. 写入缓存(TTL: 1小时)
        self.redis.setex(cache_key, 3600, json.dumps(user))

        return user

    async def invalidate_user(self, user_id: int):
        """清除用户缓存"""
        cache_key = f'user:{user_id}'
        self.redis.delete(cache_key)
```

### 12.2 数据库优化

**索引策略:**
```sql
-- 高频查询字段建立索引
CREATE INDEX idx_username ON users(username);
CREATE INDEX idx_phone ON users(phone_number);
CREATE INDEX idx_user_level ON users(level);

-- 联合索引
CREATE INDEX idx_user_status_level ON users(status, level);

-- 外键索引
CREATE INDEX idx_profile_user_id ON user_profiles(user_id);
```

**分表策略:**
```sql
-- 登录日志按月分表
CREATE TABLE user_login_logs_202510 LIKE user_login_logs;
CREATE TABLE user_login_logs_202511 LIKE user_login_logs;

-- XP记录按用户ID范围分表
CREATE TABLE user_xp_logs_0 LIKE user_xp_logs; -- user_id: 0-999999
CREATE TABLE user_xp_logs_1 LIKE user_xp_logs; -- user_id: 1000000-1999999
```

---

## 13. 测试验收标准

### 13.1 功能测试

| 测试项 | 验收标准 | 优先级 |
|-------|---------|--------|
| 手机号注册 | 验证码正确发送,注册成功创建用户 | P0 |
| 密码登录 | 密码验证正确,登录成功返回token | P0 |
| 个人资料编辑 | 修改生效并实时更新 | P0 |
| 头像上传 | 支持裁剪,上传成功 | P1 |
| XP增加 | 完成书籍后XP正确增加 | P0 |
| 成就解锁 | 触发条件后成就正常解锁 | P1 |
| 跨设备同步 | 5秒内同步到其他设备 | P0 |
| 好友请求 | 请求发送,接受/拒绝正常 | P1 |

### 13.2 性能测试

| 指标 | 目标值 | 测量方法 |
|-----|-------|---------|
| 注册接口响应时间 | ≤ 1500ms | 并发100 |
| 登录接口响应时间 | ≤ 800ms | 并发200 |
| 用户信息查询 | ≤ 200ms | 并发500 |
| 跨设备同步延迟 | ≤ 5s | WebSocket |
| 数据库连接池 | 最少10,最大100 | 配置 |

### 13.3 安全测试

| 测试项 | 验收标准 |
|-------|---------|
| SQL注入 | 所有输入参数化查询,无SQL注入漏洞 |
| XSS攻击 | 所有用户输入转义,无XSS漏洞 |
| CSRF防护 | Token验证,无CSRF漏洞 |
| 密码强度 | 弱密码被拒绝 |
| 验证码防刷 | 频率限制生效 |
| 敏感数据加密 | 手机号/身份证加密存储 |

---

## 14. 实施路线图

### Phase 1: 基础功能 (2-3周)

**Week 1:**
- ✅ 数据库表设计与创建
- ✅ 用户注册(手机号+验证码)
- ✅ 用户登录(密码/验证码)
- ✅ 基础个人资料管理

**Week 2:**
- ✅ 第三方登录(微信/QQ)
- ✅ 头像上传与裁剪
- ✅ 设备管理
- ✅ 登录日志

**Week 3:**
- ✅ 密码修改/重置
- ✅ 隐私设置
- ✅ 偏好设置
- ✅ 单元测试

### Phase 2: 成就与社交 (2-3周)

**Week 4:**
- ✅ XP系统
- ✅ 等级系统
- ✅ 成就系统基础框架

**Week 5:**
- ✅ 好友系统
- ✅ 关注系统
- ✅ 举报屏蔽

**Week 6:**
- ✅ 成就自动解锁
- ✅ 推送通知集成
- ✅ 集成测试

### Phase 3: 跨设备同步 (2周)

**Week 7:**
- ✅ WebSocket同步服务
- ✅ 阅读进度同步
- ✅ 书签同步

**Week 8:**
- ✅ 冲突检测与解决
- ✅ 离线数据合并
- ✅ 性能优化

### Phase 4: 优化与上线 (1-2周)

**Week 9:**
- ✅ 性能测试与优化
- ✅ 安全加固
- ✅ 文档完善

**Week 10:**
- ✅ 灰度发布
- ✅ 监控与告警
- ✅ 正式上线

---

## 15. 附录

### 15.1 错误码定义

| 错误码 | 说明 | HTTP状态码 |
|-------|------|-----------|
| 1001 | 手机号已注册 | 400 |
| 1002 | 验证码错误 | 400 |
| 1003 | 验证码已过期 | 400 |
| 1004 | 密码格式错误 | 400 |
| 1005 | 用户名已存在 | 400 |
| 2001 | 用户不存在 | 404 |
| 2002 | 密码错误 | 401 |
| 2003 | 账户已冻结 | 403 |
| 2004 | 账户已锁定 | 403 |
| 3001 | Token无效 | 401 |
| 3002 | Token已过期 | 401 |
| 4001 | 权限不足 | 403 |
| 5001 | 服务器错误 | 500 |

### 15.2 环境变量配置

```bash
# 数据库
DB_HOST=localhost
DB_PORT=3306
DB_NAME=library_app
DB_USER=root
DB_PASSWORD=xxxxx

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=xxxxx

# JWT
JWT_SECRET_KEY=your-secret-key
JWT_EXPIRATION=7200  # 2小时

# 短信服务
SMS_PROVIDER=aliyun
SMS_ACCESS_KEY=xxxxx
SMS_ACCESS_SECRET=xxxxx

# 对象存储
OSS_ENDPOINT=oss-cn-shanghai.aliyuncs.com
OSS_BUCKET=library-app
OSS_ACCESS_KEY=xxxxx
OSS_ACCESS_SECRET=xxxxx

# 加密
ENCRYPTION_MASTER_KEY=32-byte-hex-key
```

---

## 📝 总结

本文档提供了微书包用户模块的完整设计方案,涵盖:
- ✅ 完整的功能需求与业务流程
- ✅ 详细的数据库设计(9张核心表)
- ✅ 安全与隐私保护方案
- ✅ 跨设备同步架构
- ✅ 成就与激励体系
- ✅ API接口设计
- ✅ 测试验收标准
- ✅ 实施路线图

**下一步行动:**
1. Review本文档,确认需求与设计
2. 搭建开发环境
3. 按Phase 1开始开发
4. 定期进行代码审查和测试

---

**文档维护:**
- 创建人: Claude (UltraThink Mode)
- 最后更新: 2025-10-15
- 版本: V2.0
