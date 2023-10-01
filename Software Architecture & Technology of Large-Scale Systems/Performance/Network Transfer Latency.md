
# 網路延遲種類

- 三種網路延遲
	- Data transfer
	- Creation of connection (TCP Connection)
	- SSL/TLS Connection

# 最小化網路延遲

- Connection Creation Latency
	- Connection Pool
		- Restful api server
		- Database
	- Persistent Connections
		- Browser
- Data Transfer Overhead
	- Session/Data Caching(降低或根本上去除資料傳輸)
		- 內部server彼此的溝通
		- server到database
	- Static Data Caching
		- Browser
	- Efficient Data Format & Compression(e.g. binary protocol)
		- 任何可以自訂資料格式的server間的資料交換
			- 例如RPC, gRPC而不用HTTP
			- gzip等壓縮資料
		- 通常跟網路傳輸比起來，解壓縮資料產生的CPU overhead比較不那麼嚴重
- SSL/TLS Connection
	- SSL Session Caching

#SoftwareArchitecture #Performance 