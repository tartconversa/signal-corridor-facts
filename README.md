# 道路信号方案的生效与走廊服务事实仓

道路信号部署及可观测服务的离线资料。统一使用 UTC 秒，坐标不包含真实地点。 资料为本地合成样本。

## 源交付

`sources/` 存放 JSON 原件，文件名前两位是到达序号，同一序号内的各表属于同次交付。每个文件有 `intake`、`table`、`records`，记录 `id` 在所属表内唯一。`source-fields.json` 列出各表字段类型，空值用 `null`，空集合用 `[]`。业务时间按下方约定解释。

| 数据表 | 说明 |
| --- | --- |
| `plans` | 获批方案对路口及相位的要求。 |
| `launches` | 批次指定的发布时刻和实际切换回执。 |
| `lanes` | 运行状态所适用的车道通行关系。 |
| `observations` | 实际相位周期的观测信号。green 是该次全局 UTC 秒区间。 |
| `takeovers` | 临时接管与解除后的恢复凭证。 |
| `gaps` | 采集缺口。 |
| `routes` | 方向的经过关系和相邻行程。 |
| `counts` | 匿名通行观察按运动方向登记的时间跨度。 |
| `acceptance` | 运维处已登记的核对结果与资料截点。 |

## 来源约定

span 左闭右开，plan phase 中的 green 以完整周期的起点为相对零点。周期起点由控制器下发回执里的 cycle_zero 与 phase 实况给出，计划参考值本身不能作为已执行的观测。覆盖 spans 在 gap 中的部分代表确知采集缺口。takeovers 有开启、解除和后来收到的恢复记录；解除和恢复之间的状态未知。lane map 的 applies 只覆盖有效时间及所列 movement，一段路径依次经过 route 的 movement，travel_min/max 分别是从该动作到下一个动作的最短、最长行程秒，最后一项使用零。
可联程意为对可能的任意行程都在已核清的连续通行窗口内，若所有可能的行程都可以被证明受到中断，属于确知不连续；其余说明可用范围的上下界，区分确知混合与观测缺口。窗口边界适用半开，经整秒范围表达。counts 只覆盖指定 movement 和 span，粒度不足时采用有依据的界限。acceptance 以 scope 和收到的资料批次约束曾引用事实。

## 资料盘点

运行环境为 Python 3.11，盘点与资料校验仅使用标准库。盘点结果按到达批次显示表名及行数。

```sh
python source_inventory.py
python -m unittest discover -s tests -v
```
