# 衡量指標
- Latency
	- 影響: 使用者體驗
	- 期望值: 越低越好
- Throughput
	- 影響: 同時間可供user使用的數量
	- 期望值: 越高越好，至少要高於request rate
- Errors
	- 影響: 功能正確性
	- 期望值: 應該要沒有，但timeout這類型的錯誤可以被允許
- Resource Saturation(飽和)
	- 影響: 硬體容量
	- 期望值: 資源應該有效率的被使用

# Tail Latency

![[Pasted image 20230916194438.png]]
> Latency最高的部分
- workload變重時會tail latency的高latency會變成問題
- Average latency會掩蓋tail latency
	- 衡量 99(或99.9)% 的latency


#SoftwareArchitecture #Performance 