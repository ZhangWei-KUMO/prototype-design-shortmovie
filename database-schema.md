# 短剧AI管理平台数据库设计文档

## 1. 概述

本文档详细介绍了短剧AI管理平台的数据库设计，该平台用于创建、管理和生成AI辅助的短剧内容。数据库采用PostgreSQL实现，托管在Neon云数据库平台上。

## 2. 数据库信息

- **数据库名称**: short_drama_ai_platform
- **项目ID**: cool-smoke-87876279
- **连接方式**: PostgreSQL标准连接

## 3. 表结构设计

### 核心表

#### 3.1 用户表 (users)

存储系统用户信息。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| username | VARCHAR(50) | 用户名，唯一 |
| email | VARCHAR(100) | 电子邮箱，唯一 |
| password_hash | VARCHAR(255) | 加密后的密码 |
| full_name | VARCHAR(100) | 用户全名 |
| role | VARCHAR(20) | 用户角色，默认'writer' |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 更新时间 |

#### 3.2 剧本表 (scripts)

存储短剧剧本的核心信息。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| script_code | VARCHAR(20) | 剧本编号，如S00001，唯一 |
| title | VARCHAR(100) | 剧本标题 |
| description | TEXT | 剧本简介 |
| outline | TEXT | 剧本大纲 |
| status | VARCHAR(20) | 状态：draft(草稿)、review(审核中)、published(已发布)、offline(已下架) |
| author_id | INTEGER | 作者ID，外键关联users表 |
| ai_prompt | TEXT | 用于AI生成的全局提示词 |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 更新时间 |

#### 3.3 角色表 (characters)

存储剧本中的角色信息。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| script_id | INTEGER | 关联剧本ID |
| name | VARCHAR(100) | 角色名称 |
| description | TEXT | 角色描述 |
| role_type | VARCHAR(50) | 角色类型：protagonist(主角)、deuteragonist(女主角)、antagonist(反派)等 |
| order_index | INTEGER | 显示顺序 |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 更新时间 |

### 角色形象相关表

#### 3.4 角色形象表 (character_appearances)

存储角色的不同形象设定。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| character_id | INTEGER | 关联角色ID |
| appearance_name | VARCHAR(100) | 形象名称，如"日常装扮"、"战斗装扮" |
| appearance_type | VARCHAR(50) | 形象类型：daily(日常)、battle(战斗)、special(特殊能力)、chibi(Q版) |
| base_model | VARCHAR(100) | 基础AI模型，如"星流 Star-3" |
| prompt | TEXT | 生成此形象的AI提示词 |
| order_index | INTEGER | 显示顺序 |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 更新时间 |

#### 3.5 形象图片表 (appearance_images)

存储为角色形象生成的图片。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| appearance_id | INTEGER | 关联形象ID |
| image_url | VARCHAR(255) | 图片URL |
| is_primary | BOOLEAN | 是否为主要图片 |
| created_at | TIMESTAMP | 创建时间 |

#### 3.6 形象参考图表 (appearance_reference_images)

存储用户上传的参考图片。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| appearance_id | INTEGER | 关联形象ID |
| image_url | VARCHAR(255) | 参考图片URL |
| created_at | TIMESTAMP | 创建时间 |

#### 3.7 LoRA模型表 (lora_models)

存储增强模型信息。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| name | VARCHAR(100) | 模型名称 |
| description | TEXT | 模型描述 |
| preview_url | VARCHAR(255) | 预览图URL |
| created_at | TIMESTAMP | 创建时间 |

#### 3.8 形象LoRA关联表 (appearance_loras)

多对多关系表，连接形象和LoRA模型。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| appearance_id | INTEGER | 关联形象ID |
| lora_id | INTEGER | 关联LoRA模型ID |
| weight | NUMERIC(3,2) | 权重值，默认1.0 |
| created_at | TIMESTAMP | 创建时间 |

### 提示词和分镜相关表

#### 3.9 提示词模板表 (prompt_templates)

存储角色的核心提示词模板。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| character_id | INTEGER | 关联角色ID |
| template_name | VARCHAR(100) | 模板名称，如"日常装扮提示"、"战斗姿态提示" |
| prompt_text | TEXT | 提示词内容 |
| order_index | INTEGER | 显示顺序 |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 更新时间 |

#### 3.10 分镜表 (storyboards)

存储剧本的分镜信息。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| script_id | INTEGER | 关联剧本ID |
| scene_number | INTEGER | 场景编号 |
| scene_name | VARCHAR(100) | 场景名称 |
| description | TEXT | 场景描述 |
| order_index | INTEGER | 显示顺序 |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 更新时间 |

#### 3.11 分镜帧表 (storyboard_frames)

存储分镜中的每一帧。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| storyboard_id | INTEGER | 关联分镜ID |
| frame_number | INTEGER | 帧编号 |
| frame_description | TEXT | 帧描述 |
| prompt | TEXT | 生成此帧的AI提示词 |
| image_url | VARCHAR(255) | 生成图片URL |
| order_index | INTEGER | 显示顺序 |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 更新时间 |

### 辅助表

#### 3.12 检查点参数表 (checkpoint_params)

存储AI模型的检查点参数设置。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| appearance_id | INTEGER | 关联形象ID（可为空） |
| frame_id | INTEGER | 关联分镜帧ID（可为空） |
| param_name | VARCHAR(100) | 参数名称 |
| param_value | TEXT | 参数值 |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 更新时间 |

#### 3.13 生成积分表 (generation_credits)

跟踪AI生成功能的使用情况。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| user_id | INTEGER | 关联用户ID |
| credits_used | INTEGER | 使用的积分数量 |
| generation_type | VARCHAR(50) | 生成类型：character(角色)、storyboard(分镜)等 |
| related_entity_id | INTEGER | 关联实体ID |
| related_entity_type | VARCHAR(50) | 关联实体类型 |
| created_at | TIMESTAMP | 创建时间 |

#### 3.14 活动日志表 (activity_logs)

记录系统中的所有活动。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | SERIAL | 主键 |
| user_id | INTEGER | 关联用户ID |
| action_type | VARCHAR(50) | 操作类型：create、update、delete、generate等 |
| entity_type | VARCHAR(50) | 实体类型：script、character、appearance等 |
| entity_id | INTEGER | 实体ID |
| description | TEXT | 操作描述 |
| created_at | TIMESTAMP | 创建时间 |

## 4. 数据关系

```
users
 ↑
 |
scripts
 ↑
 |
characters ←→ prompt_templates
 ↑
 |
character_appearances ←→ lora_models (通过appearance_loras关联)
 ↑
 ↙          ↘
appearance_images    appearance_reference_images
 
scripts
 ↑
 |
storyboards
 ↑
 |
storyboard_frames

character_appearances ↖
                      | 
storyboard_frames ↗   ↓
                 checkpoint_params

users ←→ generation_credits
users ←→ activity_logs
```

## 5. 索引设计

以下索引已添加以优化查询性能：

- `idx_scripts_author_id` - 优化按作者查询剧本
- `idx_scripts_status` - 优化按状态筛选剧本
- `idx_characters_script_id` - 优化查询某剧本的所有角色
- `idx_appearances_character_id` - 优化查询某角色的所有形象
- `idx_storyboards_script_id` - 优化查询某剧本的所有分镜
- `idx_activity_logs_user_id` - 优化按用户查询活动日志
- `idx_activity_logs_entity` - 优化按实体类型和ID查询活动日志

## 6. 示例查询

### 6.1 获取所有已发布剧本

```sql
SELECT * FROM scripts WHERE status = 'published' ORDER BY updated_at DESC;
```

### 6.2 获取某剧本的所有角色

```sql
SELECT c.* 
FROM characters c
WHERE c.script_id = 1
ORDER BY c.order_index;
```

### 6.3 获取角色的所有形象及其生成的图片

```sql
SELECT ca.*, ai.image_url
FROM character_appearances ca
LEFT JOIN (
    SELECT appearance_id, image_url
    FROM appearance_images
    WHERE is_primary = true
) ai ON ca.id = ai.appearance_id
WHERE ca.character_id = 1
ORDER BY ca.order_index;
```

### 6.4 获取角色形象使用的所有LoRA模型

```sql
SELECT lm.name, lm.description, al.weight
FROM appearance_loras al
JOIN lora_models lm ON al.lora_id = lm.id
WHERE al.appearance_id = 1
ORDER BY al.weight DESC;
```

### 6.5 获取剧本的完整结构（包括角色和分镜）

```sql
WITH script_data AS (
    SELECT s.* FROM scripts s WHERE s.id = 1
),
characters_data AS (
    SELECT c.* FROM characters c WHERE c.script_id = 1
),
storyboards_data AS (
    SELECT sb.* FROM storyboards sb WHERE sb.script_id = 1
)
SELECT 
    json_build_object(
        'script', (SELECT row_to_json(s) FROM script_data s),
        'characters', (SELECT json_agg(c) FROM characters_data c),
        'storyboards', (SELECT json_agg(sb) FROM storyboards_data sb)
    ) AS script_complete;
```

## 7. TypeScript数据模型

```typescript
// 短剧AI管理平台 - TypeScript 数据模型定义

// 用户表
export interface User {
  id: number;
  username: string;
  email: string;
  passwordHash: string;
  fullName?: string;
  role: 'admin' | 'writer' | 'viewer';
  createdAt: Date;
  updatedAt: Date;
}

// 剧本表
export interface Script {
  id: number;
  scriptCode: string; // e.g., S00001
  title: string;
  description?: string;
  outline: string;
  status: 'draft' | 'review' | 'published' | 'offline';
  authorId: number;
  aiPrompt?: string;
  createdAt: Date;
  updatedAt: Date;
  
  // 关联
  author?: User;
  characters?: Character[];
  storyboards?: Storyboard[];
}

// 角色表
export interface Character {
  id: number;
  scriptId: number;
  name: string;
  description?: string;
  roleType?: 'protagonist' | 'deuteragonist' | 'antagonist' | 'supporting';
  orderIndex: number;
  createdAt: Date;
  updatedAt: Date;
  
  // 关联
  script?: Script;
  appearances?: CharacterAppearance[];
  promptTemplates?: PromptTemplate[];
}

// 更多接口定义...
```

## 8. 连接信息


推荐使用加密的环境变量存储连接信息。

## 9. 版本信息

- 数据库版本: 1.0
- 设计日期: 2025-05-14
- 最后更新: 2025-05-14 