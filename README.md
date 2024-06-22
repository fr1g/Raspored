# Raspored （已弃用）
 Web学生日程表, 毕业设计, 基于前端(vue2, tailwind, bootstrap)后端(springMVC, springboot), 感觉不错的话甚至毕业后还更新。

## 项目标志:

项目名来自塞尔维亚语的“日程”, 而产品logo为西里尔字母表示的该词。

# 这篇readme只是随笔。

## 目前计划:

### 前端:

使用vue2框架, 样式使用tailwind排版, 字体图标使用bootstrap icons@^

注意事项: 选择性地结合使用thymeleaf与jsp, 理论上可以只使用.vue来管理页面组件。

### 后端:

使用基于javaweb的系列技术, springboot; 视情况使用blazor

(! 对于不得不使用blazor的相关事项: 

​	如果出现了不得不使用blazor的情况, 细分为两种应用场景:

​		如果前端方面需要blazor则: 新开blazor wasm, 一般不会需要其他modify, 需要的地方直接通过<code><iframe></code>嵌入. 当然, 前端**可以考虑使用blazor wasm + vue2 + tailwindcss 的模式, 前提是使用静态的设计.**

​		如果后端方面需要blazor则: 0. 只能使用Blazor Server; 1. 明确上交作业是否可分为工程文件 + 实际发行版; 2. 要为两个不同的技术栈配合做好准备. 比如说, 在用jar打包springboot项目后留一个调用shell执行命令来启动blazor server app的方式; 3. blazor server必须只作为一个舒适的API(对于此项目可以是天气获取和转换API); 4. 需要解决跨域问题。同时, 后端应该尽可能避免使用blazor。

​	)

### 表设计:

对于本项目, 数据库用于保存用户以及用户课表信息。

| 表名          | 项目                                                         | 解释                                                     |
| ------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| User          | ID int auto_increment primary key, name varchar(30) not null unique, pwd varchar(50) not null | 用户基本信息                                             |
| UserSchedules | ID int auto_increment primary key, UserID int not null, ThisScheduleName varchar(50) not null, ThisScheduleUUID varchar(80) not null | 用户存储表名, 用户每创建一个表就在这里创建一条新的记录。 |
|               |                                                              |                                                          |

其中ThisScheduleName用特定格式保存着用户创建的表的名称, ThisScheduleUUID保存用于索引的对应新表的表名。不建议用户的表名中存在有半角的特殊字符。在为用户创建表时, 应该先创建一个UUID, 用于绑定表并且避免冲突。

UserSchedules保存了这个用户创建的表. 每个用户创造新的日程表后, 将会根据公共单表模版创建新的表。公共的单表模版如下:

| 表名       | 项目                                                         | 解释                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| %表的UUID% | ID int auto_increment primary key, ScheduleName varchar(50), Mon text, Tue text, Wed text, Thu text, Fri text, Sat text, Sun text, Ext text, config text | 保存了表中一周七天的事务, 每天使用一个统一的组件保存日程内容, 预留了一个带默认值的描述一天时段和一般日程时长的空间, 并且预留了一个空间标记周特殊事件。 |
| ————————   | ⬆️将多个日程项目组为一个数组。                                |                                                              |
|            |                                                              |                                                              |

ps: {day: “Mon”, items: [{},{},{},{},{}]}

上述的通用组件为一段JSON, 如下所示:

````json
{
  "isUnderSchedule": "默认为true, 控制这项日程是否跟随划定的日程格子",
  "partName": "这里是一段日程计划的名称",
  "fromTo": "如果isUnderSchedule为true, 这里将被忽略。否则这里表示起止时间, 同样有个通用格式, 比如「7:00-9:00」表示上午七点到上午九点。",
  "partDetail": "这里保存片段的附加信息",
  "avaliableDateRange": "这里用通用格式保存着此项日程有效的日期范围. 将保存所有它应该显示的日期, 或者特定的单词: 「once」代表本周的这一天, 「every」表示每周的这一天, 「」。",
  "occupyBlocks": "如果isUnderSchedule为false, 这里将被忽略。否则这里表示这个日程在当天的划定日程框架中占多少个格子。"
}
````

保存在

#### 如何处理时间冲突?

由于是分天存储的, 当新建日程时候: 1. 当天有至少一个日程; 2. 存在的最晚起始时间的日程(A)的起始时间比新日程的起始时间小; 3. 发现新日程选择的时间点起始部分**小于**A的结束时间, 则判断为时间冲突, 发出提示, 并按A的结束时间开启新日程。这个过程在后端静默进行检查和修正。
