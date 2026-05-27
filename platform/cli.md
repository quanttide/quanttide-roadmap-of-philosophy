# CLI validate 命令

## 定位

`validate` 是 `qtcloud-meta` 的核心命令，用户唯一必须记住的命令。

## 流程

```
spec.yaml
  │ 自动 translate（如果是 .yaml）
  ▼
spec.lean
  │ 结构检查：inductive Stage + def category
  ▼
Lean 类型检查：lean spec.lean
  │
  ├─ ✅ 通过 → 输出验证报告
  └─ ❌ 失败 → 输出错误位置和类型不匹配
```

## 自动翻译

如果输入是 `.yaml`，先执行 `translate` 生成 `.lean`，再对 `.lean` 执行验证。用户不需要知道 `translate` 的存在——它只是 `validate` 的一个内部步骤。

## 结构检查

在调用 Lean 之前，做一次轻量文本检查：

1. 文件是否包含 `inductive Stage : Type`（状态空间定义）
2. 文件是否包含 `def category : Category :=`（范畴实例定义）

两者缺一不可。如果缺少，提示不是有效的规约文件，不浪费时间调用 Lean。

## Lean 验证

直接调用 `lean <file>` 执行类型检查：

- **通过**：Lean 的整个语法和类型系统都接受了这个文件。范畴律（结合律、单位律）在 `category` 实例的定义中被验证。
- **失败**：输出 Lean 报告的错误。常见原因：未定义的类型引用、字段类型歧义、命名冲突。

## 已知问题

1. **inductive 的 universe 级别** — Lean 4 中 `inductive` 类型用作结构体字段时可能触发 universe 级别元变量错误。当前用 `structure T : Type where` 语法绕过，基本覆盖
2. **未引用类型的 stub** — spec 中引用但未在 ontology 中定义的类型（如 `ExpandMap`）需要生成空的 `inductive` 声明。当前已处理
3. **错误信息过滤** — Lean 输出的错误路径包含本地路径，当前做了基础过滤但不够干净

## 下一步

- `validate` 完成后自动输出可读的业务报告（非 Lean 源码），让 PM 能读懂
- 支持 `validate --watch`：文件变更自动重新验证
