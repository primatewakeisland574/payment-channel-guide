# 结算与对账逻辑

## 结算周期

| 类型 | 说明 |
|------|------|
| T+0 | 当日实时到账，费率较高 |
| T+1 | 次日到账，最常见 |
| D+1 | 自然日次日，含节假日顺延 |
| 周结/月结 | 部分平台采用 |

## 对账流程

1. 每日拉取通道对账文件
2. 与本地订单数据逐笔比对
3. 发现差异分类处理：
   - **多收少付**：通道多收费，发起退款申请
   - **漏单**：本地有订单，通道无记录，核查原因
   - **重复扣款**：立即冻结，联系通道处理

## 对账核心字段

- 商户订单号（本地唯一）
- 通道流水号
- 交易金额
- 手续费
- 实际到账金额
- 交易时间
- 交易状态

## 自动对账脚本思路

```python
def reconcile(local_orders, channel_records):
    local_map = {o['out_trade_no']: o for o in local_orders}
    channel_map = {r['trade_no']: r for r in channel_records}
    
    discrepancies = []
    for trade_no, record in channel_map.items():
        local = local_map.get(trade_no)
        if not local:
            discrepancies.append({'type': 'missing_local', 'trade_no': trade_no})
        elif abs(float(local['amount']) - float(record['amount'])) > 0.01:
            discrepancies.append({'type': 'amount_mismatch', 'trade_no': trade_no})
    return discrepancies
```
