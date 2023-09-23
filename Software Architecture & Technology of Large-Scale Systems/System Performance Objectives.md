- 讓Request-Response間的延遲(Latency)最小
	- 用時間單位衡量
	- Depends on
		- Walt/Idle Time
		- Processing Time
- Maximize Throughput(在給定的時間內能處理多少request)
	- Throughput is measuted as ==Rate of Request processing==
	- Depends on
		- Latency
		- Capacity

- Batch Procssing
	- 例如從DB讀取資料產生report等
	- 這種行為不用看Request-Response Latency，只需要關注throughput



#SoftwareArchitecture #Performance 