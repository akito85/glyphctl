```
module.exports = {
  apps : [{
    // Name
    name: "bridex-api-ndp",
    // Init script
    script: "server.js",
    // Specify which folder to watch
    watch: ["app", "constants"],
    // Specify delay between watch interval
    watch_delay: 999,
    // Specify which folder to ignore
    ignore_watch: ["node_modules", "logs"],
    // Set autorestart when crash
    autorestart: true,
    // logs files
    out_file: "logs/console.log",
    error_file: "logs/error.log",
    combine_logs: true,
    log_date_format: "YYYY-MM-DD HH:mm:ss",
    // Variables
    env_production: {
       NODE_ENV: "production"
    },
    env_development: {
       NODE_ENV: "development"
    }
  }]
}

// CMD ["pm2-runtime", "pm2.conf.js"]

```