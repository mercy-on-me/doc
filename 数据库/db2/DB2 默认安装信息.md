# DB2 安装
- 下载
- 分别解压两个包
- 进入 expc 包
- 执行 ./db2setup 进行安装
- 默认安装信息 :

```properties
name : db2inst1
group : db2iadm1
passwod : Cy891026
home director : /home/db2inst1
response file name : /root/db2expc.rsp

Install Location:/opt/ibm/db2/V11.1

Instance Name:db2inst1

Fenced User Name:db2fenc1

 Product to install:                     	DB2 Express-C 
Installation type:                      	Custom 
                                        
Previously Installed Components:        
                                        
Selected Components:                    
    Base client support                 	
    Java support                        	
    SQL procedures                      	
    Base server support                 	
    DB2 data source support             	
    Spatial Extender server support     	
    DB2 LDAP support                    	
    DB2 Instance Setup wizard           	
    Integrated Flash Copy Support       	
    Spatial Extender client             	
    Communication support - TCP/IP      	
    Base application development tools  	
    DB2 Update Service                  	
    Sample database source              	
    DB2 Text Search                     	
    First Steps                         	
                                        
Languages:                              
    Chinese (Simplified)                	
        All Products                    	
    English                             	
        All Products                    	
                                        
Target directory:                       	/opt/ibm/db2/V11.1
                                        
Space required:                         	1382 MB 
                                        
New instances:                          
    Instance name:                      	db2inst1
        Start instance on reboot:       	Yes 
        TCP/IP configuration:           	
            Service name:               	db2c_db2inst1
            Port number:                	50000
        Instance user information:      	
            User name:                  	db2inst1
            Group name:                 	db2iadm1
            Home directory:             	/home/db2inst1
        Fenced user information:        	
            User name:                  	db2fenc1
            Group name:                 	db2fadm1
            Home directory:             	/home/db2fenc1
        DB2 Text Search:                	
            HTTP service name:          	db2j_db2inst1
            HTTP service port number:   	55000
```
