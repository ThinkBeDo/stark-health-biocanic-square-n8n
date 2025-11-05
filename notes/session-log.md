# Session Notes

## 2025-11-05 - Initial Setup & Documentation Migration

### Discussion Points
- Explored options for shared AI editing (Google Drive vs GitHub vs ClickUp)
- Determined GitHub provides only platform with write access for both Claude and ChatGPT
- Established markdown as standard format for documentation
- Migrated comprehensive existing pillar document from Claude Projects knowledge base to GitHub

### Technical Decisions
- Repository name: `stark-health-biocanic-square-n8n`
- Primary document: `project_pillar.md`
- Structure optimized for non-code documentation projects
- Integrated latest session updates including Router node fixes and current blockers

### Current Project Status
- **Phase 4:** Production Deployment (95% complete)
- **Subscription Path:** ✅ 100% Complete
- **Order Path:** 🔄 60% Complete (Router logic fixed, testing needed)
- **Overall MVP:** 85% Complete

### Active Blockers (Nov 5, 2025)
1. **Router Node:** ✅ RESOLVED - Boolean routing working after removing `{{ }}` brackets
2. **Map Subscription Node:** 🔴 ACTIVE - Unknown subscription plan ID (plan_variation_id is null)
3. **Customer Email:** 🔴 ACTIVE - Square: Get Customer Details fetching all customers instead of specific one

### Key Lessons Added
- Router node configuration in n8n Switch (Rules Mode) requires direct expression evaluation without `{{ }}` brackets
- Boolean operator must be set to "true" for conditional logic
- Template literal syntax returns strings, not booleans in Rules mode

### Next Steps
1. Push repository to GitHub
2. Connect to ChatGPT via GitHub MCP
3. Connect to Claude Projects via GitHub integration
4. Test read/write access from both AI platforms
5. Fix plan_variation_id null value in Prepare Subscription Data node
6. Fix Square customer details API call to include customer_id parameter
7. Complete end-to-end subscription path testing

---

*Add new session notes below with date headers*
