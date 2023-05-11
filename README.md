# Collection of custom BloodHound queries

## New Azure stuff

### MS Graph related


```
{
			"name": "Return All Service Principals with MS Graph App Role Assignments",
			"category": "Azure",
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
			"category": "Azure",
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
			"category": "Azure",
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
			"name": "Shortest path from Owned Azure Users to all Service Principals",
			"category": "Azure",
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
			"category": "Azure",
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
			"category": "Azure",
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
			"category": "Azure",
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
			"category": "Azure",
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
