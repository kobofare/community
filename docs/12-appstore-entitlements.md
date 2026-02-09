# 应用商店权益与计费接口（规范草案）

> 目的：统一应用商店内权益（UC）与试用配置、计费扣减、风控与审计流程，保证“可用、可控、可追溯”。

## 1. 适用范围
- 适用于所有接入应用商店的应用
- 适用于 UC 的发放、消耗、退款与审计

## 2. 关键概念
- **UC（使用权益）**：仅可在应用商店内消耗，不可提现/转让
- **试用权益**：限定额度+有效期的 UC
- **权益包**：面向新用户/活动/邀请等场景的 UC 批量发放

## 3. 配置结构
应用级别配置（YAML/JSON），建议路径：`configs/entitlements/<app_id>.{yaml|json}`。

示例：
- YAML：`docs/templates/entitlements-config.yaml`
- JSON：`docs/templates/entitlements-config.json`

核心字段说明：
- `pricing.unit`：计费单位（request/minute/gb_month 等）
- `pricing.unit_price`：单位消耗 UC
- `trial.quota_uc`：试用额度
- `trial.expires_in_days`：试用有效期
- `consumption_order`：消耗顺序
- `risk_controls.daily_uc_cap`：日消耗上限

## 4. 接口规范（建议）
OpenAPI 草案见 `docs/15-entitlements-openapi.yaml`。

### 4.1 配置接口
- `GET /apps/{app_id}/entitlements`
- `PUT /apps/{app_id}/entitlements`

### 4.2 权益发放
- `POST /entitlements/grants`

请求示例：
```json
{
  "user_id": "u_123",
  "app_id": "knowledge",
  "grant_type": "trial",
  "quota_uc": 50,
  "expires_in_days": 14,
  "reason": "new_user"
}
```

### 4.3 权益消耗（计费）
- `POST /entitlements/consume`

请求示例：
```json
{
  "user_id": "u_123",
  "app_id": "knowledge",
  "usage_units": 3,
  "idempotency_key": "consume_20260207_001",
  "metadata": { "request_id": "req_abc" }
}
```

响应示例：
```json
{
  "consumed_uc": 3,
  "balance_uc": 120,
  "source": "trial"
}
```

### 4.4 权益回滚
- `POST /entitlements/refund`

### 4.5 账户查询
- `GET /users/{user_id}/entitlements`

## 5. 数据模型（建议）

**entitlements_config**
- `app_id`
- `pricing_unit`
- `pricing_unit_price`
- `trial_quota_uc`
- `trial_expires_in_days`
- `consumption_order`
- `risk_daily_uc_cap`

**entitlements_ledger**
- `id`
- `user_id`
- `app_id`
- `type`（trial/new_user/campaign/purchased）
- `delta_uc`
- `balance_uc`
- `expires_at`
- `idempotency_key`
- `created_at`

## 6. 计费与安全
- 计费请求必须提供 `idempotency_key`
- 超过 `daily_uc_cap` 进入限速/二次验证
- 消耗失败必须回滚，保证余额正确
- 重要操作记录审计日志

## 6.1 错误码（建议）
- `ENTITLEMENT_INSUFFICIENT`：余额不足
- `ENTITLEMENT_EXPIRED`：权益已过期
- `ENTITLEMENT_RATE_LIMIT`：触发风控限流
- `ENTITLEMENT_DUPLICATE`：重复请求（幂等冲突）
- `ENTITLEMENT_INVALID_CONFIG`：配置无效

## 7. 运营面板字段（对齐经济模型）
- 权益包配置：名称、额度、有效期、目标人群
- 风控配置：日限额、异常阈值、黑白名单
- 报表：发放量、核销量、转化率、留存率

## 8. 接入流程（建议）
1) 提交应用接入申请
2) 创建并发布权益配置
3) 接入计费与消耗接口
4) 完成试用权益联调
5) 上线与监控

## 9. 与现有应用商店代码对齐（当前实现）
现有应用商店服务（`node` 仓库）已提供**应用配置**接口，可作为权益配置的落地位置。
### 9.1 现有接口与存储
- 接口：`GET/PUT /api/v1/public/applications/:uid/config`
- 数据结构：`config` 是 `{ code, instance }` 数组，存储在 `application_configs.config_json`
- 相关文件：
  - `src/routes/public/applications.ts`
  - `src/domain/model/applicationConfig.ts`
  - `src/domain/mapper/entity.ts`（`ApplicationConfigDO`）

### 9.2 权益配置落地方式（建议）
将权益配置作为一条 `config` 记录：
```json
{
  "config": [
    {
      "code": "entitlements",
      "instance": "{...JSON 配置字符串...}"
    }
  ]
}
```
其中 `instance` 为 `docs/templates/entitlements-config.json` 序列化后的 JSON 字符串。完整包装示例见 `docs/templates/application-config.entitlements.json`。

### 9.3 需要的补充开发（最小改动）
- 在应用商店服务中新增权益计费接口（参见 `docs/15-entitlements-openapi.yaml`）
- 解析 `config` 中 `code=entitlements` 的配置并缓存
- 在调用计费接口时执行 UC 扣减与幂等校验
> 若需要强类型配置，可将 `instance` 改为结构化 JSON，并更新 `normalizeApplicationConfig` 的解析逻辑。
