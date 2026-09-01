# English 复习

复习单位是整篇日期笔记，不分析笔记内容。

复习间隔：1、2、4、7、15、30 天。

```base
filters:
  and:
    - file.inFolder("05_life/hobbies/english/daily")
    - or:
        - 'date(file.name) == today() - "1d"'
        - 'date(file.name) == today() - "2d"'
        - 'date(file.name) == today() - "4d"'
        - 'date(file.name) == today() - "7d"'
        - 'date(file.name) == today() - "15d"'
        - 'date(file.name) == today() - "30d"'
formulas:
  interval_days: '(today() - date(file.name)) / 86400000'
properties:
  file.name:
    displayName: 学习日期
  formula.interval_days:
    displayName: 间隔天数
views:
  - type: table
    name: 今日复习
    order:
      - file.name
      - formula.interval_days
```

## 使用方式

1. 每天在 `daily/YYYY/` 下创建一篇 `YYYY-MM-DD.md`。
2. 打开本页，复习“今日复习”表格列出的整篇日期笔记。
3. 不需要手工更新本页；只有调整复习间隔时才修改上面的筛选条件。
