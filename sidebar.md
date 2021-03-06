﻿# Flow 中文参考指南
[Javascript静态类型检测器]

(v0.87.0)

很多错误或错别字，以供参考

## 目录
1. [导言](#README)
1. [入门指南](#docs/overview)
 - [安装](#docs/overview/install)
 - [用法](#docs/overview/usage)
1. [类型注解](#docs/Type-Annotations)
 - [原始类型](#docs/type/Primitive)
 - [字面量类型](#docs/type/Literal)
 - [混合类型](#docs/type/Mixed)
 - [任意类型](#docs/type/Any)
 - [可选类型](#docs/type/Maybe)
 - [变量类型](#docs/type/Variable)
 - [函数类型](#docs/type/Function)
 - [对象类型](#docs/type/Object)
 - [数组类型](#docs/type/Array)
 - [元组类型](#docs/type/Tuple)
 - [Class 类型](#docs/type/Class)
 - [类型别名](#docs/type/Aliases)
 - [不透明类型别名](#docs/type/Opaque)
 - [接口类型](#docs/type/Interface)
 - [泛型](#docs/type/Generic)
 - [联合类型](#docs/type/Union)
 - [交集类型](#docs/type/Intersection)
 - [Typeof Types](#docs/type/Typeof)
 - [类型转换表达式](#docs/type/Casting)
 - [工具类型](#docs/type/Utility)
 - [模块类型](#docs/type/Module)
 - [注释类型](#docs/type/Comment)
1. [类型系统](#docs/System)
 - [类型与表达式](#docs/System/Types-Expressions)
 - [子集与子类型](#docs/System/Subsets-Subtypes)
 - [类型变异](#docs/System/Variance)
 - [名义与结构类型](#docs/System/Nominal-Structural)
 - [深度子类型](#docs/System/Depth-Subtyping)
 - [宽度子类型](#docs/System/Width-Subtyping)
 - [类型细化](#docs/System/Type-Refinements)
 - [惰性模式](#docs/System/Lazy-Modes)
1. [Flow CLI](#docs/CLI)
1. [配置](#docs/Configuration)
1. [库定义](#docs/libdefs)
1. [Linting](#docs/linting)