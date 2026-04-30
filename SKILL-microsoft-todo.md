# Microsoft To Do Integration Skill

## When to Use
- Reading, creating, updating, or deleting tasks in Microsoft To Do
- Assigning tasks to Scott (human assistant)
- Checking task lists when KL asks about "todo", "tasks", or "check task list"
- Refreshing expired access tokens (HTTP 401 from Graph API)
- Marking tasks as done by appending "(DONE)" to the title

## CRITICAL: Never Ask KL About Token Problems
Microsoft Graph API access tokens expire after ~1 hour. **NEVER ask KL to re-authenticate.** Always refresh the token yourself using the script in "Token Refresh" below. Only tell KL to re-authenticate if the refresh itself fails with 401 (meaning the 90-day refresh token expired).

## Token Refresh (MANDATORY -- Do This First on 401 Errors)

When Graph API returns HTTP 401, run this script:

```python
import json, urllib.request, urllib.parse, sys
sys.stdout.reconfigure(encoding='utf-8')

CLIENT_ID = 'b555eb09-6e28-4d34-b125-a184c9cc9dfa'
# Read CLIENT_SECRET from any project's .mcp.json (e.g., D:\Projects\VocabVista\.mcp.json)
# Look for the env.CLIENT_SECRET field
CLIENT_SECRET = '<read from .mcp.json>'

with open(r'D:/OneDrive/AI/Qoder Workspace/todoMCP/tokens.json') as f:
    tokens = json.load(f)

data = urllib.parse.urlencode({
    'client_id': CLIENT_ID,
    'client_secret': CLIENT_SECRET,
    'refresh_token': tokens['refreshToken'],
    'grant_type': 'refresh_token',
    'scope': 'Tasks.Read Tasks.ReadWrite Tasks.Read.Shared Tasks.ReadWrite.Shared'
}).encode()

req = urllib.request.Request(
    'https://login.microsoftonline.com/consumers/oauth2/v2.0/token',
    data=data,
    headers={'Content-Type': 'application/x-www-form-urlencoded'}
)

resp = json.loads(urllib.request.urlopen(req).read())

new_tokens = {
    'accessToken': resp['access_token'],
    'refreshToken': resp.get('refresh_token', tokens['refreshToken'])
}
with open(r'D:/OneDrive/AI/Qoder Workspace/todoMCP/tokens.json', 'w') as f:
    json.dump(new_tokens, f, indent=2)

print('Token refreshed OK')
```

**Key points:**
- Always save the new `refreshToken` back to `tokens.json` (it rotates on each refresh)
- If refresh fails with 401: Tell KL "Microsoft To Do token expired -- I need you to run the auth flow in `D:\OneDrive\AI\Qoder Workspace\todoMCP\README.md` Section 1."

## Task List Mapping

| Quest | To Do List Name |
|-------|-----------------|
| Qoder Admin | "Qoder PM (Quest)" |
| VocabVista | "Vocab Vista (Quest)" |
| VocabVista Product Manager | "Vocab Vista (Quest)" |
| Little Reader Storybook | "LR Storybooks (Quest)" |
| English Prep | "English Prep App (Quest)" |
| ZhongkaoPrep | "ZK Prep (Quest)" |
| Little Learner | "Learner Website (Quest)" |
| Little Learner PM | "Learner Website (Quest)" |
| **Scott (human assistant)** | "For Scott from Qoder Quests" |

## Scott's Task List
**List ID:** `AQMkADAwATM3ZmYAZS04OABjMS1mOTg2LTAwAi0wMAoALgAAA2g773rSOQhIms6qpVd3BLgBAPPHOT4NZ0tCnZHsl_2wI-EACZ7j-cgAAAA=`

When creating tasks for Scott:
1. Always include clear step-by-step instructions in the task body (HTML format)
2. Set `"importance": "high"` for urgent tasks
3. Ask Scott to send results/screenshots back to "Master KL"
4. Scott is non-technical -- be very specific with commands

## Task Workflow Rules
1. **Only check the relevant task list** for this project's quest
2. **Start executing immediately** when KL asks to check tasks -- don't ask for permission
3. **Always read the notes** (body content) and check for attachments
4. **Never mark as completed** -- append "(DONE)" to the title instead
5. **Skip draft tasks** -- ignore titles starting with "dft", "DFT", or "DRAFT"
6. **Starred/high-importance first**, then work top-to-bottom
7. **Notify KL when starred tasks are done**, then continue automatically

## Graph API Endpoints

### Get all tasks in a list
```
GET https://graph.microsoft.com/v1.0/me/todo/lists/{listId}/tasks
```

### Create a task
```
POST https://graph.microsoft.com/v1.0/me/todo/lists/{listId}/tasks
{
    "title": "Task title",
    "importance": "high",
    "body": {
        "content": "<p>HTML content with instructions</p>",
        "contentType": "html"
    }
}
```

### Update task title (to mark as done)
```
PATCH https://graph.microsoft.com/v1.0/me/todo/tasks/{taskId}
{
    "title": {"text": "Original title (DONE)"}
}
```

### Get task details (body/notes)
```
GET https://graph.microsoft.com/v1.0/me/todo/tasks/{taskId}
```

## MCP Server
If the `mstodo` MCP server is available in the session, use it directly instead of the Graph API. It handles token refresh automatically. The MCP server is configured in `.mcp.json` at the project root.

## Paths
- **Tokens file:** `D:\OneDrive\AI\Qoder Workspace\todoMCP\tokens.json`
- **MCP config:** `<project-root>/.mcp.json`
- **Setup docs:** `D:\Projects\qoder-fresh-setup-info.md`
