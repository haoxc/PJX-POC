<%*
// 1. 字典式维护领域知识（在此处轻松扩展数据）
const domainDict = {
    "哲学(Phi)": {cn: "哲学", en: "Philosophy", abbr: "PHIL", emoji: "🏛️" },
    "艺术(Art)": {cn: "艺术" en: "Art", abbr: "ART", emoji: "🎨" },
    "数学(Math)": {cn: "数学" en: "Mathematics", abbr: "MATH", emoji: "🔢" },
    "管理学(Mgr)": {cn: "管理学" en: "Management", abbr: "MGMT", emoji: "📊" }
};

// 2. 让使用者选择领域
const fields = Object.keys(domainDict);
const selectedField = await tp.system.suggester(fields, fields);
// 如果用户取消选择，则停止运行
if (!selectedField) return;
const info = domainDict[selectedField];
// 3. 交互获取术语具体内容
const termCn =  tp.file.title;
const termEn = await tp.system.prompt("请输入英文术语名");
const termAbbr =""
// 4. 重命名文件（可选，将文件名改为：术语名-领域）
//await tp.file.rename(`${termCn}`);
-%>
---
category: 术语笔记
domain: <% info.cn %>
domain_en: <% info.en %>
abbr: <% termAbbr %>
tags: [术语, <% selectedField %>]
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
---

# <% info.emoji %> <% termCn %>

## 📋 基本信息
- **中文全称**：<% termCn %>
- **英文全称**：<% termEn %>
- **缩写/代号**：<% termAbbr %>
- **所属领域**：[[<% selectedField %>]] (<% info.en %>)

## 🔍 定义与内涵
> 在 <% selectedField %> 语境下，该术语的核心定义是：
> （在此处输入定义...）

## 💡 沟通应用（教练建议）
- **使用场景**：
- **易混淆点**：

---
