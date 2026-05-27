# 建模思路

## 核心思想

业务系统 = 骨架（Spec）× 血肉（Impl）

### 骨架（Spec）

硬、可验证、定义了姿态（阶段）和活动范围（转移）。两个形态：

| 形态 | 用途 | 读者 |
|------|------|------|
| `spec.yaml` | 编辑和沟通 | 人 + AI |
| `spec.lean` | 形式化验证 | AI + Lean 类型检查 |

### 血肉（Impl）

软、灵活、持续新陈代谢。同一副骨架长出不同血肉：pr4xis 引擎、状态机图、文档、AI prompt。

## 形态结构

一个 spec 包含三层：

```
ontology      → 概念的类型定义（枚举、结构体）
stages        → 状态空间和合法转移
expand        → 迭代上限
```

### 例子

```yaml
# spec.yaml
ontology:
  ReleaseStatus:
    enum: [Staged, Published, Retired]

stages:
  unreleased: { to: [staged] }
  staged:     { to: [published, retired, self: restage] }
  published:  { to: [retired] }

expand:
  staged:
    restage: { max: 5 }
```

## 关键决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 事实源 | spec.yaml | 人和 AI 共读写，配形式化校验 |
| 形态翻译 | translate | YAML ↔ Lean 等价转换，不是生成 |
| 验证方式 | Lean 类型检查 | 封闭公理系统，AI 不能自由发挥 |
| 范畴结构 | 自由范畴 | 路径独立性，每条路径唯一 |
| 模型中心 | spec | 不是 contract，不是 code，是规格本身 |

## 范畴视角

外层是两个对象（spec, impl）间有双向非互逆边：

```
spec ──implement──→ impl
     ←──discover──
```

spec 内部有自态射族：

```
spec ──translate──→ spec（形态切换）
    ──validate────→ spec（自洽性校验）
```

自由范畴保留每条路径的独立身份，不假设等价关系。
