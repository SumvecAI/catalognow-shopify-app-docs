# Monitoring

## Application Logs

### Remix Server
Standard console output from the Node.js process includes:
- HTTP request logs
- Authentication events
- Webhook processing

### AI Core
Python FastAPI logs include:
- API endpoint calls
- LLM model selection and fallback events
- Agent reasoning traces
- Error details

## Audit Trail

The built-in audit trail (accessible at `/app/audit`) records:
- All product modifications (who, what, when)
- AI agent actions with reasoning
- Webhook events
- Backup/restore operations

## Agent Traces

For debugging AI behavior:
- Agent traces are stored in the `AgentTrace` table
- Each trace includes: thought steps, tool calls, observations
- View traces in the Audit Trail with the trace viewer component

## Health Checks

- App health: Check if the Remix server responds at the root URL
- Database: Prisma connection check
- AI Core: `GET http://localhost:8100/health`
