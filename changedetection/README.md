# changedetection.io - Website Change Monitoring

A powerful, self-hosted website change detection and monitoring tool that tracks changes on web pages and sends notifications when updates occur.

## 🚀 Features

- **Website Monitoring**: Track changes on any website or web page
- **Browser Automation**: Full Chrome/Playwright support for JavaScript-heavy sites
- **Smart Filtering**: CSS selectors, XPath, JSON path, and visual selectors
- **Multiple Notifications**: Email, Discord, Slack, Telegram, and 80+ services via Apprise
- **Price Monitoring**: Track product prices and stock availability
- **API Monitoring**: Monitor JSON APIs and REST endpoints
- **Proxy Support**: Built-in proxy configuration for privacy and access
- **Interactive Browser Steps**: Automate login flows and complex interactions
- **Conditional Triggers**: Only notify when specific conditions are met

## 🏃‍♂️ Quick Start

### Prerequisites

- Docker and Docker Compose installed on your system

### 1. Start the Service

```bash
cd changedetection
docker compose up -d
```

This will start:

- **changedetection.io** on http://localhost:5000
- **Chrome browser service** for JavaScript support
- **Proxy support** via host.docker.internal:1080 (if configured)

### 2. Access the Web Interface

Open your browser and navigate to: http://localhost:5000

### 3. Add Your First Monitor

1. Click **"Add new"**
2. Enter a URL to monitor (e.g., `https://example.com`)
3. Click **"Save"**
4. Click **"Check now"** to run your first check

## 📋 Complete Example Workflow

Let's set up a comprehensive monitoring example that demonstrates all key features:

### Example: Amazon Product Price Monitoring with Email Notifications

#### Step 1: Choose Target Website

**Target**: Amazon product page
**URL**: `https://www.amazon.com/dp/B08N5WRWNW` (Echo Dot example)

#### Step 2: Create Monitor with Browser Steps

1. **Add New Monitor**:

   - URL: `https://www.amazon.com/dp/B08N5WRWNW`
   - Monitor Type: **Browser steps**

2. **Configure Browser Steps** (for login and dynamic content):

   ```
   Step 1: Click on "Sign in"
   - CSS: a[data-nav-role="signin"]

   Step 2: Enter email
   - CSS: input[name="email"]
   - Text: your-email@example.com

   Step 3: Click Continue
   - CSS: input#continue

   Step 4: Enter password
   - CSS: input[name="password"]
   - Text: your-password

   Step 5: Sign in
   - CSS: input#signInSubmit

   Step 6: Wait for page load
   - Wait: 3 seconds
   ```

#### Step 3: Filter Dynamic Content

**CSS Selector for Price**:

```css
.a-price .a-offscreen,
.a-price-whole;
```

**Alternative XPath**:

```xpath
//span[contains(@class, 'a-price-whole')]
```

**Include Filters** (only monitor these elements):

- Price information
- Availability status
- Prime shipping details

**Ignore Text**:

```
Amazon's Choice
Sponsored
Advertisement
```

#### Step 4: Set Up Email Notifications

1. **Go to Settings → Notifications**

2. **Email Configuration**:

   ```
   SMTP Server: smtp.gmail.com
   Port: 587
   Username: your-email@gmail.com
   Password: your-app-password
   From: your-email@gmail.com
   To: your-email@gmail.com
   ```

3. **Notification URL** (using Apprise format):

   ```
   mailto://your-email:app-password@gmail.com
   ```

4. **Custom Notification Message**:

   ```
   🔔 Price Alert: {{watch_url}}

   💰 Current Price: {{current_price}}
   📊 Change Detected: {{diff}}
   🕒 Last Check: {{current_timestamp}}

   View full changes: {{base_url}}/preview/{{watch_uuid}}
   ```

#### Step 5: Advanced Filtering & Triggers

**Trigger Conditions**:

- Only notify if price decreases by >5%
- Alert when "Currently unavailable" changes to "In Stock"
- Skip notifications for minor formatting changes

**Regular Expression Filter**:

```regex
\$[\d,]+\.?\d*
```

This extracts only price information like `$49.99`, `$1,299.00`

## ⚙️ Configuration Options

### Environment Variables

The docker-compose.yml includes these key configurations:

```yaml
environment:
  # Browser automation
  - PLAYWRIGHT_DRIVER_URL=ws://browser-sockpuppet-chrome:3000

  # Proxy settings (pre-configured)
  - HTTP_PROXY=http://host.docker.internal:1080
  - HTTPS_PROXY=http://host.docker.internal:1080

  # Performance tuning
  - FETCH_WORKERS=10 # Concurrent checks
  - MINIMUM_SECONDS_RECHECK_TIME=3 # Min check interval
  - SCREENSHOT_MAX_HEIGHT=16000 # Screenshot size

  # Privacy & security
  - HIDE_REFERER=true # Hide referer header
  - DISABLE_VERSION_CHECK=true # Disable update checks

  # Timezone
  - TZ=America/New_York # Your timezone
```

### Port & Access Configuration

- **Default Port**: 5000 (bound to localhost:5000)
- **Data Storage**: Persistent Docker volume `changedetection-data`
- **Browser Service**: Internal network communication

## 🔧 Advanced Features

### 1. Visual Selectors

Click and select elements directly in the browser:

1. Go to your monitor's **"Visual Selector"** tab
2. Take a screenshot of the current page
3. Click on elements you want to monitor
4. Fine-tune the selection area

### 2. JSON API Monitoring

Monitor REST APIs and JSON endpoints:

**URL**: `https://api.github.com/repos/dgtlmoon/changedetection.io/releases/latest`

**JSONPath Filter**:

```json
$.tag_name
$.published_at
$.assets[*].download_count
```

### 3. Conditional Notifications

**Trigger Text**: Only notify when these words appear:

```
sale, discount, available, in stock, price drop
```

**Ignore Text**: Skip notifications for these:

```
advertisement, sponsored, cookie notice
```

### 4. Multiple Notification Channels

**Discord Webhook**:

```
discord://webhook_id/webhook_token
```

**Slack**:

```
slack://botname@token/channel
```

**Telegram**:

```
tgram://bot_token/chat_id
```

**Custom Webhook**:

```
json://webhook.site/your-unique-url
```

## 🛠 Troubleshooting

### Common Issues

**1. Browser Steps Not Working**

- Ensure the `browser-sockpuppet-chrome` service is running
- Check that `PLAYWRIGHT_DRIVER_URL` is correctly configured
- Verify CSS selectors are valid and unique

**2. Notifications Not Sent**

- Test notification URLs in Settings → Send test notification
- Check firewall settings for outbound connections
- Verify email credentials and SMTP settings

**3. Frequent False Positives**

- Add ignore filters for dynamic content (ads, timestamps)
- Use more specific CSS selectors
- Enable "Ignore text" for common changing elements
- Increase check interval for stable content

**4. Memory Usage High**

- Reduce `FETCH_WORKERS` if monitoring many sites
- Lower `SCREENSHOT_MAX_HEIGHT` for image-heavy pages
- Restart services periodically: `docker compose restart`

## 🔒 Security Considerations

1. **Access Control**: Bind to localhost (`127.0.0.1:5000`) by default
2. **Proxy Usage**: Pre-configured proxy for additional privacy
3. **File Access**: `ALLOW_FILE_URI=False` prevents local file access
4. **SSL Support**: Configure SSL certificates for HTTPS access
5. **Referer Headers**: `HIDE_REFERER=true` for privacy

## 📁 File Structure

```
changedetection/
├── docker-compose.yml          # Main configuration
├── README.md                   # This file
└── data/                       # Created by Docker volume
    ├── datastore.db           # Monitor configurations
    ├── screenshots/           # Captured screenshots
    └── logs/                  # Application logs
```

## 🚀 Getting Started Checklist

- [ ] Start services: `docker compose up -d`
- [ ] Access web interface: http://localhost:5000
- [ ] Add your first monitor
- [ ] Configure notifications in Settings
- [ ] Test with "Check now" button
- [ ] Set up email alerts
- [ ] Create browser steps for complex sites
- [ ] Configure filtering for clean notifications

## 📖 Additional Resources

- **Official Documentation**: https://github.com/dgtlmoon/changedetection.io
- **Apprise Notification Formats**: https://github.com/caronc/apprise
- **CSS Selector Guide**: https://www.w3schools.com/cssref/css_selectors.asp
- **XPath Tutorial**: https://www.w3schools.com/xml/xpath_intro.asp

## 🆘 Support

- **Issues**: Report bugs at https://github.com/dgtlmoon/changedetection.io/issues
- **Discussions**: Community support and feature requests
- **Docker Hub**: https://hub.docker.com/r/dgtlmoon/changedetection.io

---

_Happy monitoring! 🔍_
