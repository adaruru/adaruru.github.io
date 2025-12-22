---
title: Infra Diagram Tool
menu:
  sidebar:
    name: Infra Diagram Tool
    identifier: InfraDiagram
    weight: 20
    parent: devops
tags:
- Infrastructure
- Mermaid
- Drawio
---

# Infra Diagram Tool

## Mermaid

- Service 關係示意
- Request flow
- CI/CD pipeline
- 小型單機或單服務結構
- ✅方便 diff 版控
- ❌不支援服務 icon 顯示
- ❌無法精準位置控制

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#fdecef"
  }
}}%%

graph LR
    %% ====== Global Style ======
    classDef default fill:#fdecef,stroke:#efb752,stroke-width:1px,color:#000;
    linkStyle default stroke:#333,stroke-width:1.5px;
    
    %% ====== Custom Styles ======
    classDef serverStyle fill:#99d9ea,stroke:#6c9aa5,stroke-width:1px,color:#000;
    classDef vmStyle fill:#f7cb9f,stroke:#f79c67,stroke-width:1px,color:#000;
    classDef dockerSwarmStyle fill:#81e1f7,stroke:#4a90e2,stroke-width:1px,color:#000;
    classDef containerEngStyle fill:#97f7df,stroke:#2fa4a9,stroke-width:1px,color:#000;
    classDef idle fill:#ffb3c3,stroke:#ff5a78,stroke-width:1px,color:#000;
    
    red[["(1) netplan 192.168.168.n <br>(2) red represent idle for manual stand by"]]:::idle
    
	client((client))
	client --> Nginx
    
    subgraph Server100["Server 100 windows"]
        nfsc100["nfs client<br>client conn 101"]
        mssql[("MSSQL 1436")]
    	subgraph Server101["VM 101 ubuntu"]
        	nfss["nfs server"]    
        	subgraph containerEng["101 Container Engine"]
            	Nginx("Nginx<br>dependent main proxy")
            	redis("redis")
            	seq("seq")
            	subgraph dockerSwarm["Docker Swarm"]
            		direction TB
                	StoreWeb("store web") --> StoreApp("store api")	
                	CustomerWeb("customer web") --> CustomerApp("customer api")
                	CRMBackend("CRMBackend")
        		end
        	end
        	Nginx --> StoreWeb
        	Nginx --> CustomerWeb
        	Nginx --> CRMBackend
    	end
    	StoreApp --> mssql
    	CustomerApp --> mssql
    	CRMBackend --> mssql
    end
    
    subgraph Server200["Server 200 windows"]
        nfsc200["nfs client<br>client conn 101"]
        mssql200[("MSSQL 1436")]:::idle
    	subgraph Server201["VM 201 ubuntu"]
    	    nfss201["nfs server"]:::idle
    	    nfsc201["nfs client<br>client conn 101"]
        	subgraph containerEng201["201 Container Engine"]
            	Nginx201("Nginx<br>dependent main proxy"):::idle
            	redis201("redis"):::idle
            	seq201("seq"):::idle
            	subgraph dockerSwarm201["Docker Swarm"]
                	StoreWeb201("store web") --> CustomerWeb201("customer web")
                	StoreApp201("store api") --> CustomerApp201("customer api")
                	CRMBackend201("CRMBackend")
        		end
        	end
        	Nginx --> StoreWeb201
        	Nginx --> CustomerWeb201
    	end
    	StoreApp201 --> mssql
    	CustomerApp201 --> mssql
    end
    %% ====== Apply Styles ======
    class Server101,Server201 vmStyle
    class dockerSwarm,dockerSwarm201 dockerSwarmStyle
    class containerEng,containerEng201 containerEngStyle
    class Server100,Server200 serverStyle
```

## Drawio

- 系統架構圖
- Infrastructure Diagram
- 高可用設計
- 安全 / Network / Trust Boundary
- 跨團隊、跨角色溝通
- ✅服務 icon 支援多元
- ❌可版控但 diff 不易閱讀 (xml)
- ❌無法整合 README.md (只能匯出 png/svg)

![ingress](attach/InfraDiagram/ingress.png)