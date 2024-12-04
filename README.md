# Collection of custom BloodHound queries
## Legacy stuff

```
Domain Admin to OU mapping
MATCH p = (n:Domain)-[:Contains*1..]->(u:User)-[:MemberOf*1..]->(g:Group)
WHERE g.name CONTAINS "DOMAIN ADMINS@"
RETURN p
```

## New Azure stuff

### General stuff

```
{
			"name": "Return all Members of the 'Global Administrator' Role",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p =(n)-[r:AZGlobalAdmin*1..]->(m) RETURN p"
				}
			]
		},
		{
			"name": "Return all Members of High Privileged Roles",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p=(n)-[:AZHasRole|AZMemberOf*1..2]->(r:AZRole) WHERE r.displayname =~ '(?i)Global Administrator|User Administrator|Cloud Application Administrator|Authentication Policy Administrator|Exchange Administrator|Helpdesk Administrator|PRIVILEGED AUTHENTICATION ADMINISTRATOR|Domain Name Administrator|Hybrid Identity Administrator|External Identity Provider Administrator|Privileged Role Administrator|Partner Tier2 Support|Application Administrator|Directory Synchronization Accounts' RETURN p"
"
				}
			]
		},
		{
			"name": "Return all Members of High Privileged Roles that are synced from OnPrem AD",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p=(n WHERE n.onpremisesyncenabled = true)-[:AZHasRole|AZMemberOf*1..2]->(r:AZRole WHERE r.displayname =~ '(?i)Global Administrator|User Administrator|Cloud Application Administrator|Authentication Policy Administrator|Exchange Administrator|Helpdesk Administrator|PRIVILEGED AUTHENTICATION ADMINISTRATOR') RETURN p"
				}
			]
		},
		{
			"name": "Return all Owners of Privileged Groups that are synced from OnPrem AD",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (u:AZUser)-[r:AZOwns]->(g:AZGroup) WHERE NOT (u)-[:AZMemberOf|AZHasRole*1..]->(:AZRole) AND (g)-[:AZMemberOf|AZHasRole*1..]->(:AZRole) AND u.onpremisesyncenabled RETURN p"
				}
			]
		},
        {
			"name": "Return all Azure Users that are synced from OnPrem AD",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZUser WHERE n.onpremisesyncenabled = true) RETURN n",
					"allowCollapse": true
				}
			]
		},
        {
			"name": "Return all Azure Groups that are synced from OnPrem AD",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (g:AZGroup {onpremsyncenabled: True}) RETURN g"
				}
			]
		},
        {
			"name": "Return all Owners of Azure Applications",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (n)-[r:AZOwns]->(g:AZApp) RETURN p"
				}
			]
		},
		{
			"name": "Return all Azure Subscriptions",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZSubscription) RETURN n"
				}
			]
		},
		{
			"name": "Return all Azure Subscriptions and their direct Controllers",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (n)-[r:AZOwns|AZUserAccessAdministrator]->(g:AZSubscription) RETURN p"
				}
			]
		},
		{
			"name": "Return all principals with the UserAccessAdministrator Role against Subscriptions",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (u)-[r:AZUserAccessAdministrator]->(n:AZSubscription) RETURN p"
				}
			]
		},
		{
			"name": "Return all prinicpals with the UserAccessAdministrator Role",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (u)-[r:AZUserAccessAdministrator]->(n) RETURN p"
				}
			]
		},
		{
			"name": "Return all Azure Users that DON'T hold an Azure Role but the RBAC Role \"User Access Administrator\"",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (u:AZUser) WHERE NOT EXISTS((u)-[:AZMemberOf|AZHasRole*1..]->(:AZRole)) AND EXISTS((u)-[:AZUserAccessAdministrator]->()) RETURN u"
				}
			]
		},
		{
			"name": "Return all Azure Principals that DON'T hold an Azure Role but the RBAC Role \"User Access Administrator\"",
			"category": "Azure - General",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (u) WHERE NOT EXISTS((u)-[:AZMemberOf|AZHasRole*1..]->(:AZRole)) AND EXISTS((u)-[:AZUserAccessAdministrator]->()) RETURN u"
				}
			]
		},
```

### Attack Paths

```
{
			"name": "Find all Azure Users with a Path to High Value Targets",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (m:AZUser),(n {highvalue:true}),p=shortestPath((m)-[r*1..]->(n)) WHERE NONE (r IN relationships(p) WHERE type(r)= \"GetChanges\") AND NONE (r in relationships(p) WHERE type(r)=\"GetChangesAll\") AND NOT m=n RETURN p",
					"allowCollapse": true
				}
			]
		},
                {
			"name": "Find OnPrem synced Users with Paths to High Value Targets",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (m:AZUser WHERE m.onpremisesyncenabled = true),(n {highvalue:true}),p=shortestPath((m)-[r*1..]->(n)) WHERE NONE (r IN relationships(p) WHERE type(r)= \"GetChanges\") AND NONE (r in relationships(p) WHERE type(r)=\"GetChangesAll\") AND NOT m=n RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Find shortest Paths to High Value Roles",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZRole WHERE n.displayname =~ '(?i)Global Administrator|User Administrator|Cloud Application Administrator|Authentication Policy Administrator|Exchange Administrator|Helpdesk Administrator|PRIVILEGED AUTHENTICATION ADMINISTRATOR'), (m), p=shortestPath((m)-[r*1..]->(n)) WHERE NOT m=n RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Find Azure Applications with Paths to High Value Targets",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (m:AZApp),(n {highvalue:true}),p=shortestPath((m)-[r*1..]->(n)) WHERE NONE (r IN relationships(p) WHERE type(r)= \"GetChanges\") AND NONE (r in relationships(p) WHERE type(r)=\"GetChangesAll\") AND NOT m=n RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Find shortest Paths from Azure Users to Subscriptions",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZUser) WITH n MATCH p = shortestPath((n)-[r*1..]->(g:AZSubscription)) RETURN p"
				}
			]
		},
		{
			"name": "Find all Paths to Azure VMs",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (n)-[r]->(g:AZVM) RETURN p"
				}
			]
		},
        {
			"name": "Find shortest Path from Owned Azure Users to VMs",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZVM) MATCH p = shortestPath((m:AZUser{owned: true})-[*..]->(n)) RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Find all Paths to Azure KeyVaults",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (n)-[r]->(g:AZKeyVault) RETURN p"
				}
			]
		},
		{
			"name": "Find all Paths to Azure KeyVaults from Owned Principals",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = ({owned: true})-[r]->(g:AZKeyVault) RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Find shortest Paths to Azure Subscriptions",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (n:AZSubscription), (m), p=shortestPath((m)-[r*1..]->(n)) WHERE NOT m=n RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Find the Paths to Resources from Azure Users that DON'T hold an Azure Role but the RBAC Role \"User Access Administrator\"",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p=(u:AZUser)-[:AZUserAccessAdministrator]->(target) WHERE NOT EXISTS((u)-[:AZMemberOf|AZHasRole*1..]->(:AZRole)) RETURN u, p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Find the Paths to Resources from Azure Principals that DON'T hold an Azure Role but the RBAC \"User Access Administrator\"",
			"category": "Azure - Paths",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p=(u)-[:AZUserAccessAdministrator]->(target) WHERE NOT EXISTS((u)-[:AZMemberOf|AZHasRole*1..]->(:AZRole)) RETURN u, p",
					"allowCollapse": true
				}
			]
		},
```

### MS Graph related


```
{
			"name": "Return all Service Principals with MS Graph AZMGGrantAppRoles rights -> PrivEsc Path to Global Admin",
			"category": "Azure - MS Graph",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p=(n)-[r:AZMGGrantAppRoles]->(o:AZTenant) RETURN p",
					"allowCollapse": true
				}
			]
		},
        {
			"name": "Return all Service Principals with MS Graph App Role Assignments",
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
			"name": "Return all direct Controllers of MS Graph",
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
			"name": "Find shortest Paths to MS Graph",
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
			"name": "Return all Azure Service Principals",
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
			"name": "Return all PRIVILEGED Azure Service Principals",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p=(n:AZServicePrincipal)-[:AZHasRole|AZMemberOf*1..2]->(r:AZRole) WHERE r.displayname =~ '(?i)Global Administrator|User Administrator|Cloud Application Administrator|Authentication Policy Administrator|Exchange Administrator|Helpdesk Administrator|PRIVILEGED AUTHENTICATION ADMINISTRATOR|Domain Name Administrator|Hybrid Identity Administrator|External Identity Provider Administrator|Privileged Role Administrator|Partner Tier2 Support|Application Administrator|Directory Synchronization Accounts' RETURN p",
					"allowCollapse": true
				}
			]
		},
        {
			"name": "Find all VMs with a tied Managed Identity",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p=(:AZVM)-[:AZManagedIdentity]->(n) RETURN p",
					"allowCollapse": true
				}
			]
		},
        {
			"name": "Return all Azure Service Principals that are Managed Identities",
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
			"name": "Return all Azure Service Principals that are tied to Apps",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (sp:AZServicePrincipal {serviceprincipaltype: 'Application'}) RETURN sp",
					"allowCollapse": true
				}
			]
		},
        {
			"name": "Find all Azure Privileged Service Principals",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p = (g:AZServicePrincipal)-[r]->(n) RETURN p"
				}
			]
		},
		{
			"name": "Find shortest Paths from Owned Azure Users to Azure Service Principals",
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
			"name": "Find shortest Paths from Owned Azure Users to Azure Service Principals that are Managed Identities",
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
			"name": "Find shortest Paths from all Azure Users to Azure Service Principals that are Managed Identities",
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
			"name": "Find all Service Principals that are Managed Identities an have a Path to an Azure Key Vault",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH (m:AZServicePrincipal {serviceprincipaltype: 'ManagedIdentity'})-[*]->(kv:AZKeyVault) WITH collect(m) AS managedIdentities MATCH p = (n)-[r]->(kv:AZKeyVault) WHERE n IN managedIdentities RETURN p",
					"allowCollapse": true
				}
			]
		},
		{
			"name": "Find Paths from Managed Identities tied to a VM with a path to a Key Vault",
			"category": "Azure - Service Principals",
			"queryList": [
				{
					"final": true,
					"query": "MATCH p1 = (:AZVM)-[:AZManagedIdentity]->(n) WITH collect(n) AS managedIdentities MATCH p2 = (m:AZServicePrincipal {serviceprincipaltype: 'ManagedIdentity'})-[*]->(kv:AZKeyVault) WHERE m IN managedIdentities RETURN p2",
					"allowCollapse": true
				}
			]
		},

```

### AADConnect related stuff

```
{
			"name": "Return all Users and Azure Users possibly related to AADConnect",
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
			"name": "Find all Sessions of possibly AADConnect related Accounts",
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
			"name": "Find all AADConnect Servers (extracted from the SYNC_ Account names)",
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
			"name": "Find shortest Paths to AADConnect Servers from Owned Users",
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
