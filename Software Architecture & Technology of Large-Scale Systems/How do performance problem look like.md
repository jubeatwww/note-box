- 每個performance problem的產生都是由於某處有佇列塞車(queue building)
	- Network socket queue
	- DB IO queue
	- OS run queue
	- ...
- 塞車(queue build-up)原則上可以歸納為以下三個原因
	- 糟糕的處理效能(Inefficient slow processing)
	- 存取必定要依次執行的資源 (serial resource access)
	- 資源限制(limited resource capacity)，例如CPU太少


#SoftwareArchitecture  #Performance