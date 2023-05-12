# Collection of custom BloodHound queries

## New Azure stuff

### General stuff

```
{
			"name": "Shortest Path from Owned Azure Users to Azure VMs",
			"category": "Azure",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZVM) MATCH p = shortestPath((m:AZUser{owned: true})-[*..]->(n)) RETURN p",
					"allowCollapse": true
				}
			]
		},
```

### MS Graph related


```
{
			"name": "Return All Service Principals with MS Graph App Role Assignments",
			"category": "Azure - MS Graph",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p=(m:AZServicePrincipal)-[r:AZMGAppRoleAssignment_ReadWrite_All|AZMGApplication_ReadWrite_All|AZMGDirectory_ReadWrite_All|AZMGGroupMember_ReadWrite_All|AZMGGroup_ReadWrite_All|AZMGRoleManagement_ReadWrite_Directory|AZMGServicePrincipalEndpoint_ReadWrite_All]->(n:AZServicePrincipal) RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Return all direct controllers of MS Graph",
			"category": "Azure - MS Graph",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (n)-[r:AZAddOwner|AZAddSecret|AZAppAdmin|AZCloudAppAdmin|AZMGAddOwner|AZMGAddSecret|AZOwns]->(g:AZServicePrincipal {appdisplayname: \"Microsoft Graph\"}) RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Find shortest paths to MS Graph",
			"category": "Azure - MS Graph",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n) WHERE NOT n.displayname=\"Microsoft Graph\" WITH n MATCH p = shortestPath((n)-[r*1..]->(g:AZServicePrincipal {appdisplayname: \"Microsoft Graph\"})) WHERE n<>g RETURN p",
					"allowCollapse": true
				}
			]
		},
```


### Service Principal / Managed Identity related 

```
{
			"name": "Find all Privileged Service Principals",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (g:AZServicePrincipal)-[r]->(n) RETURN p"
				}
			]
		},
{
			"name": "Shortest path from Owned Azure Users to all Service Principals",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (u:AZUser {owned: true}), (m:AZServicePrincipal) MATCH p = shortestPath((u)-[*..]->(m)) RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Shortest path from Owned Azure Users to all Service Principals that are Managed Identities",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (u:AZUser {owned: true}), (m:AZServicePrincipal {serviceprincipaltype: 'ManagedIdentity'}) MATCH p = shortestPath((u)-[*..]->(m)) RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Shortest path from ALL Azure Users to all Service Principals that are Managed Identities",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (u:AZUser), (m:AZServicePrincipal {serviceprincipaltype: 'ManagedIdentity'}) MATCH p = shortestPath((u)-[*..]->(m)) RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "List all Azure Service Principals",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (sp:AZServicePrincipal) RETURN sp",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "List all Azure Service Principals that are Managed Identities",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (sp:AZServicePrincipal {serviceprincipaltype: 'ManagedIdentity'}) RETURN sp",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Shortest Path from owned Azure Users to Azure VMs",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZVM) MATCH p = shortestPath((m:AZUser{owned: true})-[*..]->(n)) RETURN p",
					"allowCollapse": true
				}
			]
		},

```

### AADConnect related stuff

```
{
			"name": "List all users possibly related to AADConnect",
			"category": "Azure - AADConnect",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (u) WHERE (u:User OR u:AZUser) AND (u.name =~ '(?i)^MSOL_|.*AADConnect.*' OR u.userprincipalname =~ '(?i)^sync_.*') OPTIONAL MATCH (u)-[:HasSession]->(s:Session) RETURN u, s",
					"allowCollapse": true
				}
			]
		},
{
			"name": "Return all Sessions of possibly AADConnect related Accounts",
			"category": "Azure - AADConnect",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p=(m:Computer)-[:HasSession]->(n) WHERE (n:User OR n:AZUser) AND ((n.name =~ '(?i)^MSOL_|.*AADConnect.*') OR (n.userPrincipalName =~ '(?i)^sync_.*')) RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Return all AADConnect Servers (extracted from the SYNC_ Account names)",
			"category": "Azure - AADConnect",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZUser) WHERE n.name =~ '(?i)^SYNC_(.*?)_(.*?)@.*' WITH n, split(n.name, '_')[1] AS computerNamePattern MATCH (c:Computer) WHERE c.name CONTAINS computerNamePattern RETURN c",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Shortest Path to AADConnect Servers from owned Users",
			"category": "Azure - AADConnect",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZUser) WHERE n.name =~ '(?i)^SYNC_(.*?)_(.*?)@.*' WITH n, split(n.name, '_')[1] AS computerNamePattern MATCH (c:Computer) WHERE c.name CONTAINS computerNamePattern WITH collect(c) AS computers MATCH p = shortestPath((u:User)-[*]-(c:Computer)) WHERE c IN computers AND length(p) > 0 AND u.owned = true RETURN u, p",
					"allowCollapse": true
				}
			]
		},
```
