# OpenClaw Railway — Security Hardening Guide

## 1. 🔒 Disable Insecure Auth (CRITICAL)

### Status
- **Default**: `ALLOW_INSECURE_AUTH=false` ✅ (SECURE)
- **Current**: Check your Railway Variables

### Action Required
**On Railway Dashboard:**
1. Go to your Railway project
2. **Variables** → Check `ALLOW_INSECURE_AUTH`
   - If it exists and is `true` → **immediately set to `false`** ⚠️
   - If it doesn't exist → you're already secure ✅

### Why This Matters
- `allowInsecureAuth` + `dangerouslyDisableDeviceAuth` allow token-only authentication to the control dashboard
- **On loopback (internal)**: Safe because the wrapper enforces `SETUP_PASSWORD` + HTTPS
- **On public domains**: CRITICAL SECURITY RISK — bypasses device pairing approval
- Even with the HTTPS wrapper, a compromised token grants full gateway access

### Recommended Timeline
1. **Initial setup** (if needed): Keep `ALLOW_INSECURE_AUTH=true` to quickly access `/lite` and configure channels/skills
2. **After setup is complete**: Disable immediately (`ALLOW_INSECURE_AUTH=false`)
3. **For subsequent management**: Use device pairing or secure authentication

### How to Access `/lite` After Disabling
1. SSH into your Railway instance: `railway ssh`
2. Navigate to `/data/.openclaw/`
3. Read `gateway.token` and use it in your requests, or approve device pairing prompts

---

## 2. 🔐 Secure secrets.json

### Problem
- The onboarding wizard may write `secrets.json` containing API keys to `/data/workspace`
- This file should never be committed or exposed

### Action Required
**Locate and protect:**
```bash
# Via Railway SSH or file manager
ls -la /data/workspace/secrets.json
```

**Options:**

**Option A: Move to secure location** (Recommended)
```bash
mv /data/workspace/secrets.json /data/.openclaw/secrets.json
chmod 600 /data/.openclaw/secrets.json
```

**Option B: Delete if not needed**
```bash
rm /data/workspace/secrets.json
```

**Option C: Use Railway Variables instead**
- Store API keys as Railway environment variables
- Reference them in `/data/.openclaw/openclaw.json` without writing them to disk

### Verify Security
```bash
# Ensure no secrets in workspace
grep -r "sk-\|sk_" /data/workspace/ 2>/dev/null
grep -r "Bearer\|Authorization" /data/workspace/ 2>/dev/null
```

---

## 3. 🧠 Configure Memory Embeddings

### Problem
- Memory backend defaults to builtin FTS (no external service)
- If using vector memory backends (Qdrant, Milvus, Pinecone), embeddings provider is required

### Action Required

**For Builtin FTS (no embeddings needed):**
- No action required — default is secure and self-contained

**For Vector Memory Backends:**

1. **Choose an embedding provider:**
   - OpenAI: `openai` (requires API key)
   - Ollama: `ollama` (local, no key required)
   - Cohere: `cohere` (requires API key)
   - Others: Refer to OpenClaw documentation

2. **Set Railway Variables:**
   ```
   MEMORY_EMBEDDING_PROVIDER=openai
   MEMORY_EMBEDDING_KEY=sk-your-openai-key-here
   ```

3. **Or set in `/lite` dashboard:**
   - Lite Panel → Memory → Configure embedding provider

### Example: Using Ollama
```bash
# Railway Variables
MEMORY_EMBEDDING_PROVIDER=ollama
# (No key needed for local Ollama)
```

### Example: Using OpenAI
```bash
# Railway Variables
MEMORY_EMBEDDING_PROVIDER=openai
MEMORY_EMBEDDING_KEY=sk-proj-...
```

---

## 4. 🛡️ Additional Security Recommendations

### SETUP_PASSWORD
- ✅ Required and enforced by the wrapper
- Keep it strong: `openssl rand -hex 24` (48 characters)
- Rotate periodically if exposed

### HTTPS on Public Domains
- ✅ Railway provides HTTPS by default
- Never disable HTTPS on public domains
- Verify certificate validity

### Gateway Token
- ✅ Auto-generated and stored securely
- Rotated on each gateway start
- Never commit to version control

### Rate Limiting
- Consider adding rate limiting to `/lite` and `/onboard` in your load balancer or reverse proxy
- Protects against brute-force attacks on `SETUP_PASSWORD`

### Audit Logging
- Enable Activity Log in `/lite` dashboard
- Monitor for suspicious device pairings or configuration changes
- Review gateway logs regularly

---

## 5. ✅ Security Checklist

Before deploying to production:

- [ ] `ALLOW_INSECURE_AUTH=false` (or not set)
- [ ] `secrets.json` removed or moved to secure location
- [ ] `SETUP_PASSWORD` is strong (48+ characters)
- [ ] HTTPS enforced on public domain
- [ ] Memory embeddings configured if using vector backend
- [ ] No API keys committed to version control
- [ ] No `.env` file in git (check `.gitignore`)
- [ ] Activity log reviewed for anomalies
- [ ] Device pairing approval process tested
- [ ] Backup and restore tested with encrypted secrets

---

## 6. 🚨 If You Suspect a Compromise

1. **Immediate:**
   - Set new `SETUP_PASSWORD` in Railway Variables
   - Rotate all API keys in your configured providers
   - Review recent activity log in `/lite`

2. **Short-term:**
   - Change `OPENCLAW_GATEWAY_TOKEN` (or restart gateway to auto-rotate)
   - Review all device pairings (revoke suspicious ones)
   - Check `/data/.openclaw/openclaw.json` for unauthorized changes

3. **Long-term:**
   - Implement rate limiting on `/lite` and `/onboard`
   - Set up alerts for failed login attempts
   - Regular security audits of your Railway setup

---

## Questions?

For OpenClaw security issues: [OpenClaw Security Docs](https://github.com/openclaw/openclaw)  
For Railway infrastructure: [Railway Documentation](https://docs.railway.app/)
