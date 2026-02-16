# Claude Server

Standalone Express server running the Claude Agent SDK for the Bibby AI LaTeX Editor.

## 📋 Overview

This is a dedicated server that handles AI-powered LaTeX editing requests using Anthropic's Claude Agent SDK. It runs as a standalone Node.js service separate from the main Next.js application to avoid serverless timeout limitations.

### Architecture

```
User Browser
    ↓
Next.js App (Vercel)
    ↓ CLAUDE_AGENT_SERVICE_URL
Claude Server (DigitalOcean VPS)
    ↓ ANTHROPIC_API_KEY
Claude API (Anthropic)
```

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ installed on the server
- SSH access to the deployment server
- Anthropic API key

### Environment Variables

```bash
export ANTHROPIC_API_KEY="sk-ant-api03-..."
export CLAUDE_SERVER_HOST="root@161.35.138.83"  # Optional, defaults to this
```

### First-Time Setup

1. Set your Anthropic API key:
   ```bash
   export ANTHROPIC_API_KEY="your-key-here"
   ```

2. Run the setup script:
   ```bash
   cd claude_server/scripts
   ./setup.sh
   ```

This will:
- Create service user (`claude`)
- Install dependencies
- Set up systemd service
- Start the server

### Deploying Updates

To deploy code changes:

```bash
cd claude_server/scripts
./deploy.sh
```

This will:
- Create a backup of the current deployment
- Sync updated files to the server
- Install any new dependencies
- Restart the service
- Verify the deployment

## 📁 Directory Structure

```
claude_server/
├── agent-service.ts       # Main Express server
├── package.json           # Dependencies
├── tsconfig.json          # TypeScript configuration
├── env.example            # Environment template
├── .gitignore             # Git ignore rules
├── README.md              # This file
├── CHANGELOG.md           # Version history
│
├── lib/
│   └── octra-agent/       # Standalone copy of agent library
│       ├── ast-edits.ts           # Edit validation
│       ├── config.ts              # Configuration
│       ├── content-processing.ts  # Content formatting
│       ├── index.ts               # Module exports
│       ├── intent-inference.ts    # Intent detection
│       ├── stream-handling.ts     # SSE utilities
│       └── tools.ts               # MCP tool definitions
│
└── scripts/
    ├── deploy.sh          # Deployment script
    ├── setup.sh           # First-time setup script
    ├── logs.sh            # View live logs
    └── status.sh          # Check service status
```

## 🔧 Server Configuration

### Current Setup

- **Host**: 161.35.138.83 (DigitalOcean VPS)
- **Region**: NYC3
- **Resources**: 2 vCPU, 2GB RAM
- **OS**: Ubuntu 25.04
- **Port**: 8787
- **User**: octra (non-root)

### Systemd Service

Service file: `/etc/systemd/system/claude-server.service`

```ini
[Unit]
Description=Claude Server
After=network.target

[Service]
Type=simple
User=claude
Group=claude
WorkingDirectory=/srv/claude-server
Environment=NODE_ENV=production
Environment=PORT=8787
ExecStart=/usr/bin/npx tsx agent-service.ts
Restart=always
RestartSec=2

[Install]
WantedBy=multi-user.target
```

## 📡 API Endpoints

### POST /agent

Process LaTeX editing requests with AI assistance.

**Request:**
```json
{
  "messages": [
    {
      "role": "user",
      "content": "Add a title to the document"
    }
  ],
  "fileContent": "\\documentclass{article}...",
  "textFromEditor": "selected text",
  "selectionRange": {
    "startLineNumber": 5,
    "endLineNumber": 10
  }
}
```

**Response:** Server-Sent Events (SSE)

Events emitted:
- `status` - Service started/finished
- `assistant_partial` - Streaming text chunks
- `assistant_message` - Complete response
- `tool` - Tool usage (get_context, propose_edits)
- `edits` - Edit suggestions
- `done` - Final result with all edits
- `error` - Error occurred

## 🛠️ Utility Scripts

All scripts are located in `claude_server/scripts/`

### View Live Logs
```bash
cd claude_server/scripts
./logs.sh
```

### Check Service Status
```bash
cd claude_server/scripts
./status.sh
```

Shows:
- Service status
- Process information
- Memory/disk usage
- Recent logs
- Endpoint test

### Deploy Updates
```bash
cd claude_server/scripts
./deploy.sh
```

Deploys code changes with:
- Automatic backup
- Dependency installation
- Service restart
- Health check

## 🔍 Monitoring

### View Logs

```bash
# Live tail
ssh root@161.35.138.83 'sudo journalctl -u claude-server -f'

# Last 50 lines
ssh root@161.35.138.83 'sudo journalctl -u claude-server -n 50'

# Today's logs
ssh root@161.35.138.83 'sudo journalctl -u claude-server --since today'
```

### Service Management

```bash
# Check status
ssh root@161.35.138.83 'sudo systemctl status claude-server'

# Restart
ssh root@161.35.138.83 'sudo systemctl restart claude-server'

# Stop
ssh root@161.35.138.83 'sudo systemctl stop claude-server'

# Start
ssh root@161.35.138.83 'sudo systemctl start claude-server'
```

### Resource Monitoring

```bash
# Memory usage
ssh root@161.35.138.83 'free -h'

# Disk usage
ssh root@161.35.138.83 'df -h'

# Process info
ssh root@161.35.138.83 'ps aux | grep agent'

# Port status
ssh root@161.35.138.83 'ss -tlnp | grep 8787'
```

## 🔒 Security

### Current Implementation

✅ **Implemented:**
- Non-root user execution
- Systemd service isolation
- Working directory permissions
- Auto-restart on failure

⚠️ **Needed:**
- [ ] API authentication
- [ ] Rate limiting
- [ ] HTTPS/TLS
- [ ] Request validation
- [ ] Edit limit checking
- [ ] Secure API key storage

### Recommended Improvements

1. **Add Authentication**
   - JWT tokens or API keys
   - Validate requests from Next.js app only

2. **Rate Limiting**
   - Per-IP rate limits
   - Per-user request quotas

3. **HTTPS Setup**
   - Reverse proxy (Caddy/nginx)
   - SSL certificates

4. **Monitoring**
   - Error tracking (Sentry)
   - Performance monitoring
   - Usage analytics

## 🚨 Troubleshooting

### Service Won't Start

```bash
# Check logs
ssh root@161.35.138.83 'sudo journalctl -u claude-server -n 50'

# Check file permissions
ssh root@161.35.138.83 'ls -la /srv/claude-server'

# Verify user exists
ssh root@161.35.138.83 'id claude'
```

### Port Already in Use

```bash
# Check what's using port 8787
ssh root@161.35.138.83 'sudo ss -tlnp | grep 8787'

# Kill process if needed
ssh root@161.35.138.83 'sudo kill -9 <PID>'
```

### API Key Issues

```bash
# Check environment variables
ssh root@161.35.138.83 'sudo systemctl show claude-server | grep Environment'

# Update API key
ssh root@161.35.138.83 'sudo vi /etc/systemd/system/claude-server.service.d/override.conf'
# Then: sudo systemctl daemon-reload && sudo systemctl restart claude-server
```

### Memory Issues

```bash
# Check memory usage
ssh root@161.35.138.83 'free -h'

# If low, restart service
ssh root@161.35.138.83 'sudo systemctl restart claude-server'

# Consider upgrading server if persistent
```

## 🔄 Rollback

If a deployment causes issues:

```bash
# The deploy script creates automatic backups
# Restore from backup:
ssh root@161.35.138.83 'sudo systemctl stop claude-server && \
  sudo rm -rf /srv/claude-server && \
  sudo mv /srv/claude-server-backup-YYYYMMDD-HHMMSS /srv/claude-server && \
  sudo systemctl start claude-server'
```

## 📊 Performance

### Typical Resource Usage

- **Memory**: ~90-100MB
- **CPU**: <1% idle, 5-20% during requests
- **Network**: Minimal (SSE streams)
- **Disk**: ~200MB (node_modules)

### Response Times

- **Cold start**: ~500ms
- **Warm request**: ~100ms initial response
- **Full AI generation**: 5-30s (depends on Claude)
- **Streaming**: Real-time chunks via SSE

## 🔗 Integration

### Next.js Configuration

Set environment variable in Vercel:

```bash
CLAUDE_AGENT_SERVICE_URL=http://161.35.138.83:8787/agent
```

### Local Development

For local testing, run the server:

```bash
cd server
npm install
npm run dev
```

Then in your Next.js `.env.local`:

```
CLAUDE_AGENT_SERVICE_URL=http://localhost:8787/agent
```

## 📝 Development

### Running Locally

```bash
cd claude_server
npm install
npm run dev  # Uses tsx watch for hot reload
```

### Building

```bash
cd claude_server
npm run build  # Compiles TypeScript to dist/
```

### Testing

```bash
# Test endpoint
curl -X POST http://localhost:8787/agent \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "test"}],
    "fileContent": "\\documentclass{article}"
  }'
```

## 📚 Dependencies

### Production

- `@anthropic-ai/claude-agent-sdk` - Claude Agent SDK
- `express` - Web server
- `cors` - CORS middleware
- `zod` - Schema validation

### Development

- `tsx` - TypeScript execution
- `typescript` - TypeScript compiler
- `@types/express` - Express type definitions
- `@types/cors` - CORS type definitions

## 📄 License

ISC

## 🤝 Contributing

When making changes:

1. Test locally first
2. Update documentation
3. Use `cd claude_server/scripts && ./deploy.sh` for deployment
4. Monitor logs after deployment
5. Keep backups before major changes

## 📞 Support

For issues:
- Check logs: `cd claude_server/scripts && ./logs.sh`
- Check status: `cd claude_server/scripts && ./status.sh`
- Review this README
- Check systemd service status

---

**Last Updated**: October 8, 2025  
**Version**: 1.0.0  
**Maintainer**: Bibby AI Team

