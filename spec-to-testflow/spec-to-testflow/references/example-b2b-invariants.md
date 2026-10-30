# 硬性不变量章节写作粒度示例（取自 B2B 库存→退款链路）

展示"硬性不变量"该写到什么粒度：每条都是**可执行/可断言的结论**，不是泛泛原则；每条带变更标记出处，违反即测本错误。

---

1. 发车 `depart` 只作用于正向行；退货行 refund 保持 7，由 `completeDelivery` 兜底置 8（D34/D38/D41 再修订）。
2. 退货派车**不选仓库、不传 deliveryDate**——服务端强制当天（D30/D37）；禁止对历史配送日的旧派车单发车。
3. 库存导入响应**无 batchId**——用 `GET /store/storeInventory/batchList?skuId=` 反查（导入返回值只有成功/失败行数与错误 Excel 链接）。
4. `saveRoute` 只返回 Boolean——用 `queryRouteList` 按名称反查 routeId。
5. `confirmReceipt` 直达 50 并起算 `afterSalesDeadline`——下单节点完成后**立即**申请售后，防 CI 跨天越过窗口。
6. `confirmRefundPickedBack` 仅 refundStatus=8 可调；状态 7 调用必须失败（负向断言）。
7. 回库断言只看**最终合理回升**（完成配送可能已入库，幂等），不断言"退款成功时刻再加一次"。
8. 真付真退。支付/退款回调不可达、万能码被关、夹具缺失 → **环境阻塞**，标 skip/block，不报业务缺陷。

## 失败三分法（配套示例）

| 类型 | 示例 | 处理 |
|------|------|------|
| 环境阻塞 | 夹具 SKU 不存在；支付/退款回调不可达；万能码被关闭 | 标 skip/block，不报业务缺陷 |
| 业务失败 | 审核后非 7；发车后误变 8；状态 7 确认已取回成功；回库 batchId 错 | 失败用例 |
| 实现前探测 | passwordLogin、shopCart、confirmOrder、orderPay、接单/发货、汇付 URL | 首次接入环境探针固化字段 |
