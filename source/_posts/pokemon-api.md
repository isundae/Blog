---
title: pokemon-api
date: 2022-07-18 10:14:29
tags: api
cover: https://img.isundae.cn/blog/202207181018500.png
---
# 宝可梦相关接口

## 宝可梦列表接口

### 接口地址

GET https://pokemon.fantasticmao.cn/pokemon/list

### 参数说明

| 字段名 | 类型  | 说明  | 示例  | 是否必填 | 默认值 |
| --- | --- | --- | --- | --- | --- |
| generation | int | 第几世代 | 1   | 否   | 0：表示获取全部世代的宝可梦 |
| eggGroup | String | 蛋组名称 | 植物群 | 否   | N/A：蛋组不参与过滤条件 |

### 返回结果

| 字段名 | 类型  | 说明  | 示例  |
| --- | --- | --- | --- |
| index | int | 全国图鉴 | 1   |
| nameZh | String | 中文名称 | 妙蛙种子 |
| nameJa | String | 日文名称 | フシギダネ |
| nameEn | String | 英文名称 | Bulbasaur |
| type1 | String | 属性 1 | 草   |
| type2 | String | 属性 2 | 毒   |
| ability1 | String | 特性 1 | 茂盛  |
| ability2 | String | 特性 2 |     |
| abilityHide | String | 隐藏特性 | 叶绿素 |
| generation | int | 第几世代 | 1   |

---

## 宝可梦详情接口

### 接口地址

GET https://pokemon.fantasticmao.cn/pokemon/detail

### 参数说明

| 字段名 | 类型  | 说明  | 示例  | 是否必填 |
| --- | --- | --- | --- | --- |
| index | int | 全国图鉴编号 | 1   | 否   |
| nameZh | String | 中文名称，支持模糊查询，例如「妙蛙」 | 妙蛙种子 | 否   |

### 返回结果

| 字段名 | 类型  | 说明  | 示例  |
| --- | --- | --- | --- |
| index | int | 全国图鉴 | 1   |
| nameZh | String | 中文名称 | 妙蛙种子 |
| nameJa | String | 日文名称 | フシギダネ |
| nameEn | String | 英文名称 | Bulbasaur |
| type1 | String | 属性 1 | 草   |
| type2 | String | 属性 2 | 毒   |
| ability1 | String | 特性 1 | 茂盛  |
| ability2 | String | 特性 2 |     |
| abilityHide | String | 隐藏特性 | 叶绿素 |
| generation | int | 第几世代 | 1   |
| baseStat.hp | int | HP  | 45  |
| baseStat.attack | int | 攻击  | 49  |
| baseStat.defense | int | 防御  | 49  |
| baseStat.spAttack | int | 特攻  | 65  |
| baseStat.spDefense | int | 特防  | 65  |
| baseStat.speed | int | 速度  | 45  |
| baseStat.total | int | 总合  | 318 |
| baseStat.average | int | 平均值 | 53  |
| detail.imgUrl | String | 预览图片 | https://s1.52poke.wiki/wiki/thumb/2/21/001Bulbasaur.png/300px-001Bulbasaur.png |
| detail.category | String | 分类  | 种子宝可梦 |
| detail.height | String | 身高  | 0.7m |
| detail.weight | String | 体重  | 6.9kg |
| detail.bodyStyle | String | 体型  | https://s1.52poke.wiki/wiki/c/cc/Body08.png |
| detail.catchRate | String | 捕获率 | 5.9% |
| detail.genderRatio | String | 性别比例，以逗号分隔 | 雄性 87.5%,雌性 12.5% |
| detail.eggGroup1 | String | 蛋组 1 | 怪兽  |
| detail.eggGroup2 | String | 蛋组 2 | 植物群 |
| detail.hatchTime | String | 孵化时间 | 5140 步 |
| detail.effortValue | String | 基础点数，以逗号分隔 | ＨＰ 0,攻击 0,防御 0,特攻 1,特防 0,速度 0 |
| learnSetByLevelingUp.level1 | String | 等级（太阳/月亮） | -   |
| learnSetByLevelingUp.level2 | String | 等级（究极之日/究极之月） | -   |
| learnSetByLevelingUp.move | String | 招式名称 | 撞击  |
| learnSetByLevelingUp.type | String | 属性  | 一般  |
| learnSetByLevelingUp.category | String | 分类  | 物理  |
| learnSetByLevelingUp.power | String | 威力  | 40  |
| learnSetByLevelingUp.accuracy | String | 命中  | 100 |
| learnSetByLevelingUp.pp | String | PP  | 35  |
| learnSetByTechnicalMachine.imgUrl | String | 招式学习器图片链接 | https://s1.52poke.wiki/wiki/e/e3/Bag_TM_%E4%B8%80%E8%88%AC_Sprite.png |
| learnSetByTechnicalMachine.technicalMachine | String | 招式学习器名称 | 招式学习器０１ |
| learnSetByTechnicalMachine.move | String | 招式名称 | 叫声  |
| learnSetByTechnicalMachine.type | String | 属性  | 一般  |
| learnSetByTechnicalMachine.category | String | 分类  | 变化  |
| learnSetByTechnicalMachine.power | String | 威力  | -   |
| learnSetByTechnicalMachine.accuracy | String | 命中  | 100 |
| learnSetByTechnicalMachine.pp | String | PP  | 40  |
| learnSetByBreeding.parent | String | 亲代  | 呆呆兽,呆壳兽,卡比兽 |
| learnSetByBreeding.move | String | 招式名称 | 瞬间失忆 |
| learnSetByBreeding.type | String | 属性  | 超能力 |
| learnSetByBreeding.category | String | 分类  | 变化  |
| learnSetByBreeding.power | String | 威力  | -   |
| learnSetByBreeding.accuracy | String | 命中  | -   |
| learnSetByBreeding.pp | String | PP  | 20  |