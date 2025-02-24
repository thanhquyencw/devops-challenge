## Troubleshooting Steps

1. **Check System Memory Usage**
   - Run `free -h` to get an overview of memory consumption.
   - Run `vmstat -s` to check swap usage and memory fragmentation.

2. **Inspect Running Processes**
   - Use `top` or `htop` to identify processes consuming the most memory.
   - Run `ps aux --sort=-%mem | head -10` to list top memory-consuming processes.

3. **Check NGINX Memory Usage**
   - Run `systemctl status nginx` to check service health.
   - Use `ps -eo pid,cmd,%mem --sort=-%mem | grep nginx` to determine if NGINX processes are using excessive memory.

4. **Review NGINX Configuration**
   - Inspect `/etc/nginx/nginx.conf` for:
     - `worker_processes auto;` (ensures optimal worker count)
     - `worker_connections` limit (too high can cause high memory usage)
     - `client_max_body_size` (large values may increase memory use)
     - `proxy_buffer_size` and `proxy_buffers` settings.

5. **Analyze Logs**
   - Check `/var/log/nginx/access.log` and `/var/log/nginx/error.log` for spikes in requests, slow upstream responses, or errors.
   - Use `journalctl -u nginx --since "1 hour ago"` to see recent service logs.

6. **Monitor Network Traffic**
   - Run `iftop` or `netstat -tunapl` to check for unexpected traffic spikes.
   - Check for potential DDoS attacks using `netstat -an | awk '/tcp/ {print $6}' | sort | uniq -c`.

7. **Check Swap and OOM Killer**
   - Run `dmesg | grep -i oom` to see if the Out-of-Memory (OOM) killer is terminating processes.
   - If swap is heavily used, memory pressure is high.

8. **Check System Load**
   - Run `uptime` or `top` to check load averages.
   - Use `sar -r 5 10` to monitor real-time memory usage.

| Root Cause                        | Expected Scenario                                                      | Impact                                      | Recovery Steps                                                                 |
|-----------------------------------|------------------------------------------------------------------------|---------------------------------------------|--------------------------------------------------------------------------------|
| NGINX Memory Leak                 | A bug or misconfiguration causes NGINX workers to consume excessive memory. | Degraded performance or crashes.            | Restart NGINX, update to latest stable version, optimize config.               |
| High Request Volume / DDoS        | Large traffic spikes overwhelm the server.                             | Increased latency, OOM killer terminating NGINX. | Rate-limit requests, enable fail2ban, use Cloudflare or AWS WAF.               |
| Misconfigured Buffers/Caches      | Large `proxy_buffers`, `client_body_buffer_size`, or `fastcgi_buffers` settings. | Increased memory footprint, risk of swap usage. | Reduce buffer sizes, restart NGINX.                                            |
| Upstream Latency Causing Queues   | Slow upstream responses cause NGINX to hold connections longer.        | Memory usage increases over time.           | Optimize upstream services, implement circuit breakers.                        |
| Log File Growth in RAM (tmpfs)    | If logs are stored in `/run` or another tmpfs mount, they consume RAM. | Unexpected high memory usage.               | Redirect logs to disk, clear logs periodically.                                |
| Swap Thrashing                    | Swap is overused, causing high memory pressure.                        | Server slowdown, potential OOM events.      | Reduce memory footprint, increase swap size as temporary fix.                  |
| Zombie Processes                  | NGINX workers not properly terminating.                                | Memory stays allocated, performance degrades. | Restart NGINX, check for stale worker processes.                               |