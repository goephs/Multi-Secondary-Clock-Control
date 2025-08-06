# Multi-Secondary-Clock-Control
Using a wemos mini d1 for each slave clock is controlled by NTP locally and wifi manager, time adjustments are controlled by a raspberry pi with Mosquitto MQTT and website controlled by NGINX server. Minute plus &amp; minus web buttons and Hour plus &amp; minus buttons help users adjust the clock display for any time differences.

# 🛠 Raspberry Pi Setup: Mosquitto MQTT + NGINX Web Interface

This guide documents how to configure a Raspberry Pi to run a local MQTT broker using Mosquitto and serve a web-based clock control interface using NGINX.

---

## 📦 1. Install Required Packages

### Update & Upgrade

```bash
sudo apt update
sudo apt upgrade
```
### Install Mosquitto & clients
```bash
sudo apt install mosquitto mosquitto-clients
```
### Install NGINX
```bash
sudo apt install nginx
```
## 📦 2. Configure Mosquitto
### Edit Mosquitto Config
```bash
sudo nano /etc/mosquitto/mosquitto.conf
```
### Example Configurationn
```Conf
listener 1883
allow_anonymous true
persistence true
persistence_location /var/lib/mosquitto/
log_dest file /var/log/mosquitto/mosquitto.log
```
You can later add TLS or authentication if needed.
### Enable & Start Mosquitto
```bash
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```
## 🌐 3. Configure NGINX to Serve Web Interface
### Place Web Files
Put your Put your index.html and assets in:
```bash
/home/username/MultiClockControl/
```
### Fix Permissions
```bash
chmod o+x /home
chmod o+x /home/username
chmod o+x /home/username/MultiClockControl
chmod o+r /home/username/MultiClockControl/index.html
```
### Edit NGINX Site Config
```bash
sudo nano /etc/nginx/sites-available/default
```
### Updated Config Snippet
```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /home/username/MultiClockControl;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
}
```
### Reload NGINX
```bash
sudo nginx -t
sudo systemctl reload nginx
```
## 🧪 4. Test Access
- Visit: http://<your-pi-ip>/
- Confirm index.html loads
- Confirm MQTT broker is reachable
```bash
mosquitto_sub -h localhost -t test/topic
mosquitto_pub -h localhost -t test/topic -m "Hello MQTT"
```
## 🖥️ 5. Web Interface Features
### Clock Selector
These clock names are my initial choices.  Supply your own names.
This code resides in index.html (see code example).
```html
<select id="clock-selector">
  <option value="clock1">E Howard Clock</option>
  <option value="clock2">Standard Electric Clock 1</option>
  <option value="clock3">Standard Electric Clock 2</option>
</select>
```
### Info Section
```html
<div class="info-box">
  <strong>Connected to Raspberry Pi MQTT</strong><br>
  <span id="selectedClock">No clock selected</span>
</div>
```
### Publish Button
```html
<button id="publishBtn">Publish Clock Selection</button>
<div id="infoSpace"></div>
```
### JavaScript Logic
```javascript
document.addEventListener('DOMContentLoaded', () => {
  const selector = document.getElementById('clock-selector');
  const selectedClockSpan = document.getElementById('selectedClock');
  const publishBtn = document.getElementById('publishBtn');
  const infoSpace = document.getElementById('infoSpace');

  selectedClockSpan.textContent = `Selected Clock: ${selector.value}`;

  selector.addEventListener('change', () => {
    selectedClockSpan.textContent = `Selected Clock: ${selector.value}`;
  });

  publishBtn.addEventListener('click', () => {
    const selectedClock = selector.value;
    const topicPath = `clock/${selectedClock}/selected`;

    mqttClient.publish(topicPath, selectedClock); // Requires MQTT.js setup

    infoSpace.innerHTML = `
      <strong>Published to:</strong> ${topicPath}<br>
      <strong>Payload:</strong> ${selectedClock}<br>
      <strong>Time:</strong> ${new Date().toLocaleTimeString()}
    `;
  });
});
```
### Javascript Web Publishing
```html
<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
<script>
  const mqttClient = mqtt.connect('ws://192.168.x.x:9001'); // WebSocket port
  mqttClient.on('connect', () => console.log('MQTT connected'));
</script>
```
## Code Examples
### index.html
Code includes CSS, HTML, Javascript
### Wemos mini D1 ino
Code includes wifi manager, NTP, and button logic publishing.

