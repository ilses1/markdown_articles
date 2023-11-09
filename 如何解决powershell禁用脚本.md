获取当前策略：Get-ExecutionPolicy

设置当前策略：Set-ExecutionPolicy Unrestricted

Restricted——默认的设置， 不允许任何script运行
AllSigned——只能运行经过数字证书签名的script
RemoteSigned——运行本地的script不需要数字签名，但是运行从网络上下载的script就必须要有数字签名
Unrestricted——允许所有的script运行