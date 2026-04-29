Permissions List: 

"permission": "job_titles",
        "actions": [
            {
                "name": "create",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            },
            {
                "name": "read",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            },
            {
                "name": "update",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            },
            {
                "name": "delete",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            }
        ]
    },
    {
        "permission": "competency_categories",
        "actions": [
            {
                "name": "read",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            },
            {
                "name": "update",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            }
        ]
    },
    {
        "permission": "competencies",
        "actions": [
            {
                "name": "read",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            }
        ]
    },
    {
        "permission": "competency_mappings",
        "actions": [
            {
                "name": "create",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            },
            {
                "name": "read",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            },
            {
                "name": "update",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            },
            {
                "name": "delete",
                "scopes": [
                    {
                        "scope_type": "ALL"
                    }
                ]
            }
        ]
    

Modules 
- BU : 
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/business_units: 403
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/business_units?page=1&per_page=10  :403
- SBU : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/business_units?page=1&per_page=1000 403
	- http://16.16.162.125:3000/api/v1/fetch_industries : 200
	- http://16.16.162.125:3000/api/v1/fetch_technologies : 200
	- http://16.16.162.125:3000/api/v1/fetch_certifications : 200
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/sub_business_units?page=1&per_page=10 : 403
- Department :
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/departments?page=1&per_page=1000 : 403
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/business_units : 403
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/sub_business_units?page=1&per_page=1000 : 403
	- http://16.16.162.125:3000/api/v1/fetch_technologies : 200
	- http://16.16.162.125:3000/api/v1/fetch_certifications : 200
- Job title :
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/job_titles?page=1&per_page=1000 : 200 
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/business_units : 403
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/sub_business_units?page=1&per_page=1000 : 403
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/departments?page=1&per_page=100 : 403
	-   http://16.16.162.125:3000/api/v1/fetch_jt_levels : 200
- Employee :
	 - http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/employees?page=1&per_page=10 : 401
	 - http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/business_units : 403 
	 - http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/sub_business_units?page=1&per_page=1000 : 403
	 - http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/departments?page=1&per_page=100 : 403 
	 - http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/job_titles?page=1&per_page=1000 : 200
	 - http://16.16.162.125:3000/api/v1/fetch_jt_levels : 200
	
- Teams : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/search_users?q= : 401 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/team_cohorts?type=team&page=1&per_page=10 : 404
- Projects : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/projects?page=1&per_page=1000&status=active : 403 
	- http://16.16.162.125:3000/api/v1/fetch_pro_types : 200
- Cohort : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/search_users : 403
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/team_cohorts?type=cohort&page=1&per_page=10 : 404 
- User Permissions : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/list_user_permission_sets : 304
- Domain : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/domains?page=1&per_page=1000 : 403 
- Knowledge Area : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/knowledge_areas?page=1&per_page=1000 : 403 
- Skill Category : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/skill_categories?page=1&per_page=10 : 403 
-  Skill Inventory : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/skills?page=1&per_page=1000 : 404
- Skill Groups : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/skill_groups?page=1&per_page=1000 : 403 
- Skill Mappings : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/importance_level : 401 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/fetch_skill_mappings?page=1&per_page=10 : 200 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/proficiency_level : 401 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/job_titles?page=1&per_page=10 : 200
- Skill Framework: 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/domains?page=1&per_page=10 : 403 
	-   http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/knowledge_areas?page=1&per_page=10 : 403 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/skill_categories?page=1&per_page=10 : 403 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/skill_groups?page=1&per_page=1000 : 403 
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/skills?page=1&per_page=10 : 404
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/fetch_skill_mappings?page=1&per_page=10 : 200
- Organizational Values: 
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/skills?page=1&per_page=1000 : 404
- Competency categories : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/competency_categories?page=1&per_page=1000 : 200
- All Competencies : 
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/competencies?page=1&per_page=1000&name= : 200 
- Competency Mappings : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/fetch_competency_mappings?page=1&per_page=10 : 200 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/job_titles?page=1&per_page=10 : 200 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/proficiency_level : 401 
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/importance_level : 401 
- Competency Framwork : 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/skills?page=1&per_page=1 : 404
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/competency_categories?page=1&per_page=1 : 200
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/competencies?page=1&per_page=1&name= : 200
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/fetch_competency_mappings?page=1&per_page=1 : 200
-  Task : 
	-  http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/tasks?page=1&per_page=10 : 403 
	- http://16.16.162.125:3000/api/v1/organizations/5699bfaf-31ed-4048-bec9-e2eac0fdb4e7/tasks?page=1&per_page=1000 : 403
 