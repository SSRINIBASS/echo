# End-to-End Deployment Guide

Complete deployment guide for the Crypto Narrative Pulse Tracker - covering both frontend and backend deployment options.

## Architecture Overview

```
User → CDN/Proxy → Frontend (Next.js) → Backend API (Python/Pathway)
                                      ↓
                                   Telegram Bot
                                      ↓
                                   WebSocket
```

## Deployment Options

### Option 1: Cloud Services (Recommended)
- **Backend**: Railway (easiest) or AWS EC2 + Cloudflare
- **Frontend**: Vercel (recommended) or Netlify
- **Cost**: ~$0-20/month

### Option 2: Self-Hosted
- **Backend + Frontend**: Single VPS with Docker Compose
- **Cost**: ~$5-15/month

### Option 3: Local Development
- **Backend + Frontend**: Docker Compose locally
- **Cost**: Free

---

## Option 1A: Railway + Vercel (Easiest)

### Step 1: Deploy Backend to Railway

1. **Push to GitHub** (if not already done)
   ```bash
   git add .
   git commit -m "Deploy setup"
   git push origin main
   ```

2. **Deploy to Railway**
   - Go to [railway.app](https://railway.app)
   - Click "New Project" → "Deploy from GitHub"
   - Select your repository
   - Railway will automatically detect `railway.json` and use `Dockerfile.railway`

3. **Set Environment Variables in Railway**
   
   Go to your Railway project → Variables tab:
   ```env
   TELEGRAM_TOKEN=your_bot_token_here
   TELEGRAM_CHANNEL_ID=@your_channel_or_chat_id
   TRACKED_COIN=MEME
   COINGECKO_API_KEY=CG-your_key_here
   CORS_ORIGINS=https://your-app.vercel.app
   DATA_PERSISTENCE=memory
   ```

4. **Generate Railway Domain**
   - Settings → Networking → Generate Domain
   - You'll get: `your-app-production.up.railway.app`
   - Note this URL for frontend configuration

### Step 2: Deploy Frontend to Vercel

1. **Deploy Frontend**
   ```bash
   cd frontend
   npx vercel
   # Follow prompts, select your GitHub repo
   ```

2. **Set Environment Variables in Vercel**
   
   Go to Vercel Dashboard → Your Project → Settings → Environment Variables:
   ```env
   NEXT_PUBLIC_API_URL=https://your-app-production.up.railway.app
   NEXT_PUBLIC_WS_URL=wss://your-app-production.up.railway.app
   ```

3. **Update CORS in Railway**
   
   Back in Railway, update the CORS_ORIGINS variable:
   ```env
   CORS_ORIGINS=https://your-app.vercel.app,https://your-custom-domain.com
   ```

4. **Test Deployment**
   ```bash
   # Test backend
   curl https://your-app-production.up.railway.app/health
   
   # Test frontend
   open https://your-app.vercel.app
   ```

---

## Option 1B: AWS EC2 + Cloudflare + Vercel

### Step 1: Setup AWS EC2

1. **Launch EC2 Instance**
   - AMI: Ubuntu 22.04 LTS
   - Instance Type: t3.small (2GB RAM) minimum
   - Storage: 20GB gp3
   - Security Group: Allow ports 22, 80, 443

2. **SSH and Setup**
   ```bash
   ssh -i your-key.pem ubuntu@your-ec2-ip
   
   # Run setup script
   curl -O https://raw.githubusercontent.com/your-username/your-repo/main/deploy/ec2-setup.sh
   chmod +x ec2-setup.sh
   sudo ./ec2-setup.sh
   ```

3. **Configure Environment**
   ```bash
   cd /opt/your-repo
   sudo nano .env
   ```
   
   Add your environment variables:
   ```env
   TELEGRAM_TOKEN=your_bot_token
   TELEGRAM_CHANNEL_ID=@your_channel
   TRACKED_COIN=MEME
   COINGECKO_API_KEY=CG-your_key
   CORS_ORIGINS=https://your-domain.com
   WEBHOOK_PORT=8080
   PIPELINE_PORT=8000
   ```

4. **Start Services**
   ```bash
   sudo docker compose up -d pipeline
   ```

### Step 2: Setup Cloudflare

1. **Add Domain to Cloudflare**
   - Add your domain to Cloudflare
   - Update nameservers at your registrar

2. **Configure DNS**
   | Type | Name | Content | Proxy |
   |------|------|---------|-------|
   | A | api | your-ec2-ip | ✅ Proxied |
   | A | @ | your-ec2-ip | ✅ Proxied |

3. **SSL Settings**
   - SSL/TLS → Full (strict) or Flexible
   - Always Use HTTPS: On

### Step 3: Deploy Frontend to Vercel

Same as Option 1A Step 2, but use your custom domain:
```env
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
NEXT_PUBLIC_WS_URL=wss://api.yourdomain.com
```

---

## Option 2: Self-Hosted VPS (Single Server)

### Step 1: Setup VPS

1. **Get a VPS** (DigitalOcean, Linode, Vultr)
   - Ubuntu 22.04
   - 2GB RAM minimum
   - 20GB storage

2. **Initial Setup**
   ```bash
   ssh root@your-vps-ip
   
   # Update system
   apt update && apt upgrade -y
   
   # Install Docker
   curl -fsSL https://get.docker.com | sh
   
   # Install Nginx
   apt install -y nginx certbot python3-certbot-nginx
   
   # Clone repository
   cd /opt
   git clone https://github.com/your-username/your-repo.git
   cd your-repo
   ```

3. **Configure Environment**
   ```bash
   cp .env.example .env
   nano .env
   ```
   
   Set your variables:
   ```env
   TELEGRAM_TOKEN=your_bot_token
   TELEGRAM_CHANNEL_ID=@your_channel
   TRACKED_COIN=MEME
   COINGECKO_API_KEY=CG-your_key
   CORS_ORIGINS=https://yourdomain.com
   DASHBOARD_PORT=3000
   PIPELINE_PORT=8000
   WEBHOOK_PORT=8080
   ```

### Step 2: Configure Nginx

1. **Create Nginx Config**
   ```bash
   nano /etc/nginx/sites-available/crypto-tracker
   ```
   
   ```nginx
   server {
       listen 80;
       server_name yourdomain.com www.yourdomain.com;
       
       # Frontend
       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_cache_bypass $http_upgrade;
       }
       
       # Backend API
       location /api/ {
           proxy_pass http://localhost:8000/api/;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
       
       # WebSocket
       location /ws {
           proxy_pass http://localhost:8000/ws;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
       
       # Webhook
       location /webhook {
           proxy_pass http://localhost:8080/;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

2. **Enable Site**
   ```bash
   ln -s /etc/nginx/sites-available/crypto-tracker /etc/nginx/sites-enabled/
   rm /etc/nginx/sites-enabled/default
   nginx -t
   systemctl restart nginx
   ```

3. **Setup SSL**
   ```bash
   certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```

### Step 3: Start Services

```bash
cd /opt/your-repo

# Start full stack
docker compose --profile full up -d

# Check status
docker compose ps
```

---

## Option 3: Local Development

### Step 1: Clone and Setup

```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo

# Copy environment file
cp .env.example .env
nano .env
```

### Step 2: Start Backend

```bash
# Option A: Docker (Linux/WSL)
docker compose up pipeline

# Option B: Local Python (Windows)
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
python main.py
```

### Step 3: Start Frontend

```bash
cd frontend
npm install
npm run dev
```

### Step 4: Access Application

- Frontend: http://localhost:3000
- Backend API: http://localhost:8000
- WebSocket: ws://localhost:8000/ws

---

## Environment Variables Reference

### Required Variables

```env
# Telegram Bot
TELEGRAM_TOKEN=1234567890:ABCdefGHIjklMNOpqrsTUVwxyz
TELEGRAM_CHANNEL_ID=@your_channel_name

# Tracking
TRACKED_COIN=MEME
```

### Optional Variables

```env
# CoinGecko API (for price data)
COINGECKO_API_KEY=CG-your_api_key_here

# CORS (add your frontend domains)
CORS_ORIGINS=https://your-app.vercel.app,https://yourdomain.com

# Data Persistence
DATA_PERSISTENCE=memory  # or 'file' for persistence

# Ports (for self-hosted)
PIPELINE_PORT=8000
WEBHOOK_PORT=8080
DASHBOARD_PORT=3000

# LLM (optional)
OLLAMA_HOST=http://localhost:11434
```

---

## Testing Your Deployment

### Backend Health Check

```bash
# Test API health
curl https://your-backend-url/health

# Test metrics endpoint
curl https://your-backend-url/api/metrics

# Test WebSocket (using wscat)
npm install -g wscat
wscat -c wss://your-backend-url/ws
```

### Frontend Testing

1. Open your frontend URL
2. Check that metrics are loading
3. Verify WebSocket connection (real-time updates)
4. Test the RAG chat feature

### End-to-End Testing

```bash
# Send test message to webhook
curl -X POST https://your-backend-url/ \
  -H "Content-Type: application/json" \
  -d '{"text": "MEME coin is going to the moon! 🚀", "user": "test_user"}'

# Check if metrics updated
curl https://your-backend-url/api/metrics
```

---

## Monitoring and Maintenance

### Log Monitoring

```bash
# Railway: Check logs in dashboard
# EC2: 
sudo docker compose logs -f pipeline

# VPS:
docker compose logs -f
sudo tail -f /var/log/nginx/access.log
```

### Updates

```bash
# Pull latest changes
git pull origin main

# Rebuild and restart
docker compose down
docker compose up -d --build
```

### Backup (Self-hosted)

```bash
# Backup data directory
tar -czf backup-$(date +%Y%m%d).tar.gz data/

# Backup environment
cp .env .env.backup
```

---

## Troubleshooting

### Common Issues

**502 Bad Gateway**
- Check if backend container is running: `docker compose ps`
- Verify ports are not blocked by firewall
- Check container logs: `docker compose logs pipeline`

**CORS Errors**
- Ensure frontend domain is in `CORS_ORIGINS`
- Include both http and https versions if needed

**WebSocket Connection Failed**
- Check if WebSocket is enabled in proxy (Cloudflare/Nginx)
- Verify WebSocket endpoint is accessible

**Telegram Bot Not Working**
- Verify `TELEGRAM_TOKEN` is correct
- Check if bot is added to the channel
- Ensure `TELEGRAM_CHANNEL_ID` format is correct (@channel or -100123456789)

### Getting Help

1. Check container logs: `docker compose logs -f`
2. Test individual endpoints with curl
3. Verify environment variables are set correctly
4. Check firewall and security group settings

---

## Cost Estimates

| Option | Monthly Cost | Pros | Cons |
|--------|-------------|------|------|
| Railway + Vercel | $0-20 | Easiest, auto-scaling | Limited free tier |
| EC2 + Cloudflare + Vercel | $15-25 | More control, reliable | More setup |
| VPS Self-hosted | $5-15 | Full control, cheapest | Most maintenance |
| Local Development | $0 | Free, full control | Not accessible externally |

Choose based on your needs:
- **Prototype/Demo**: Railway + Vercel
- **Production**: EC2 + Cloudflare + Vercel  
- **Budget**: VPS Self-hosted
- **Development**: Local setup