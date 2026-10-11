---
name: "RHTV 动作迁移"
description: "用 RunningHub 无限画布 RHTV + MiniMax H3 RH Enhanced 做短视频动作迁移（角色替换）。当用户需要生成角色替换视频、动作迁移时触发。"
metadata: { "includeInPrompt": true }
---

# RHTV 动作迁移

## Purpose
将源短视频按 5 秒切片，用 MiniMax H3 RH Enhanced 图生视频逐段替换角色，最后拼接并保留原声。适用于抖音收藏视频的角色替换二创。

## 核心规则（Jim 制定，通用）
1. **模型**：MiniMax H3 RH Enhanced（注意不是普通 H3，只有 Enhanced 版是每天 22:00–次日 10:00 免费）
2. **方法**：图生视频，首帧模式
3. **提示词必须先给 Jim 确认**，确认后才能生成（plan-first）
4. **RHTV 地址**：https://rhtv.runninghub.cn（cn 结尾，不是 ai；登录态与 www.runninghub.cn 通用）

## Workflow

### 1. 获取源视频
用「热门短视频解析」skill 解析下载，ffprobe 验证时长/分辨率。

### 2. 切片
按 5 秒一切（最后一段收尾）：
```bash
ffmpeg -i src.mp4 -ss 0 -t 5 -c:v libx264 -c:a aac seg1.mp4
# ... 依次类推
```
提取每段首帧：
```bash
ffmpeg -i seg1.mp4 -frames:v 1 seg1_first.jpg
```
再提取原视频末帧（给最后一段做尾帧参考）。

### 3. 写分段描述 + 提示词
仔细看每段内容，写清楚动作、场景、镜头。提示词模板：
> 以[首帧图]为视频首帧，生成[N]秒视频。[角色描述]（参考[角色图]）[动作描述]，[场景描述]，保持原动作。

**提示词必须发给 Jim 确认后才能下一步。**

### 4. 逐段生成（一条一条来，不要批量）
在 rhtv.runninghub.cn 无限画布中：
- 新建项目 → 视频生成节点 → 模型切到 MiniMax H3 RH Enhanced
- 模式用「首帧」（「首尾帧」选项卡点不亮是已知问题，不要纠结）
- 上传：角色参考图 + 该段首帧图
- 填确认过的提示词，设对应时长，生成
- 生成完下载视频，**用 ffmpeg 提取其末帧**，作为下一段的衔接参考

### 5. 拼接 + 保留原声
```bash
# 拼接（先无声拼视频）
ffmpeg -f concat -safe 0 -i list.txt -c copy silent.mp4
# 从原视频抽音频并覆盖
ffmpeg -i silent.mp4 -i src.mp4 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 -shortest final.mp4
```
校验 final.mp4 时长与原视频一致。

## 踩坑记录

### RHTV 界面
- 「首尾帧」选项卡无法激活（试过点击/双击/键盘/刷新，均无效）。改用「首帧」模式一条一条做，手动提末帧做衔接。
- 模型下拉里 H3 有多个版本，认准 **H3 RH Enhanced** 才是免费时段那个。
- 生成速度慢（约 1%/分钟，5 秒视频约 1 小时），做好心理预期。

### 首尾相接
- 不要指望模型自动衔接。每段生成完手动提末帧，下一段用该末帧+新段首帧做参考。
- 拼接前检查相邻两段的衔接帧是否跳变。

### 费用
- H3 RH Enhanced 免费时段：每天 22:00–次日 10:00（北京时间）。
- 非免费时段按量扣费，生成前看一眼账户余额。

## Operating Rules
1. 提示词先给 Jim 确认，再生成——没有例外。
2. 一次只生成一段，做完下载、提末帧，再做下一段。
3. 模型用 H3 RH Enhanced，不要用错版本。
4. 最终交付前校验时长、音画同步。
