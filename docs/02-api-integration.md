# 接口对接要点

## 对接前准备

- 获取商户号（MchID）、AppID、密钥
- 确认签名算法（MD5 / SHA256 / RSA）
- 配置回调白名单 IP
- 准备沙盒测试环境

## 签名机制

大多数支付接口使用 **参数字典序排列 + 密钥拼接** 的方式签名：

```python
import hashlib

def sign(params: dict, secret: str) -> str:
    # 过滤空值，按 key 字典序排列
    filtered = {k: v for k, v in params.items() if v}
    sorted_str = \"&\".join(f\"{k}={v}\" for k, v in sorted(filtered.items()))
    sign_str = sorted_str + \"&key=\" + secret
    return hashlib.md5(sign_str.encode()).hexdigest().upper()
```

## 回调处理要点

- 必须验签，防止伪造回调
- 回调处理要做**幂等**，同一订单多次回调只处理一次
- 返回明确的成功标识（如 `SUCCESS`），否则通道会重试
- 先更新数据库，再返回成功

## 常见坑

- 时间戳精度问题（秒 vs 毫秒）
- 金额单位问题（元 vs 分）
- 编码问题（UTF-8 vs GBK）
- SSL 证书验证（部分通道要求双向证书）
