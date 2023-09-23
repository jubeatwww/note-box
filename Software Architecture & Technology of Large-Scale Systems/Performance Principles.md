# 兩種request種類

- serial requests
		- 依序執行的request
		- 在任何時期，系統內都只有==一個==request在執行
	- concurrent requests

# Principles

在處理效能問題的時候重要的是先辨別問題屬於哪類
## Efficiency

> 當提及efficiency的時候，都是在處理serial/single request processing

- Efficient Resource Utilization
	- IO - Memory, Network, Disk
	- CPU
- Efficient Logic
	- Algorithms
	- DB Queries
- Efficient Data Storage
	- Data structures
	- DB Schema
- Caching

## Concurrency

- Hardware
- Sofrware
	- Queuing
	- Coherence

## Capacity


#SoftwareArchitecture #Performance 