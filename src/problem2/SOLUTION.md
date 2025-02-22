Provide your solution here:
#   CLI check process using hight memory:
top -o %MEM
# If nginx is the top memory consumer, we proceed to investigate its behavior.

# issue1:NGINX Worker Processes using Hight memory
#   Cause:
#       High traffic load is causing NGINX workers to use more memory.
#       Memory leaks in NGINX or a third-party module.
#       Improper worker configuration (too many workers consuming excessive memory).
#   Impact:
#       Service Degradation: Requests may slow down or fail.
#       Potential OOM-Killer Triggered: The Linux kernel might kill processes to free memory, affecting availability.

#   Recovery Steps:
#       1.Check NGINX Worker Memory Usage
           ps -eo pid,comm,%mem,%cpu --sort=-%mem | grep nginx
#       2.Restart NGINX (Quick Fix - Temporary)
           sudo systemctl restart nginx
#       3.Adjust Worker Processes & Connections
#           Check current config: 
                cat /etc/nginx/nginx.conf | grep worker
#            Reduce worker_processes and worker_connections based on system resources:
                worker_processes auto;
                worker_rlimit_nofile 100000;
                worker_connections 10240;
#       4.Monitor Memory After Fixes
            free -m
            vmstat 1 5

# issue2: Memory Leak in NGINX or External Modules
#   Possible Cause:
#       Memory leak in a third-party NGINX module (e.g., caching, Lua scripting, or custom plugins).
#       NGINX processes not releasing memory properly.
#   Impact:
#       Continuous memory growth leads to OOM issues.
#       Gradual degradation until system crash.
#   Recovery Steps:
#       1.Check for Leaking Worker Processes
          sudo pmap -x $(pgrep -u www-data nginx) | tail -10
#       2.Disable Unnecessary Modules
#           Check enabled modules:
                nginx -V 2>&1 | grep -- "--with"
#       3.Enable Debug Logging & Check Logs
#           Enable debug mode in /etc/nginx/nginx.conf:
               error_log /var/log/nginx/error.log debug;
#           Restart and monitor logs:
               sudo systemctl restart nginx
               tail -f /var/log/nginx/error.log
#       4.Upgrade or Reinstall NGINX
           sudo apt update && sudo apt upgrade nginx -y

# Once fixes are applied, monitor the system:
   watch -n 5 "free -m && ps aux --sort=-%mem | head -5"