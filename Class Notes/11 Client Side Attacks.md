
## 11.1 Target Reconaissance 

##### Information Gathering 
- Need to gather information about target users to initiate a client side attack
	- Often, end devices are non routable machines
- Common approach is to inspect metadata tags of publicly available files

Steps :
1. Download publicly available documents 
2. Exiftool the document 

```
2. exiftool -a -u <downloaded-doc>
```


##### Client Fingerprinting
