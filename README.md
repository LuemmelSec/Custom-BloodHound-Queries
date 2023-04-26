# Collection of custom BloodHound queries

## New Azure stuff

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
		}
