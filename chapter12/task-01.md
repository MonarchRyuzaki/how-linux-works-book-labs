# Task 01: The Nested Disaster (rsync Trailing Slash)

**Scenario:**
You are deploying a new version of the company website to the production web server. The web root on the server is `/var/www/html/`. 
Your local build directory is named `build`, and contains the updated `index.html` and `assets/` folder.

You run the following command to deploy:
```bash
rsync -a build webserver:/var/www/html/
```

Immediately, the monitoring alerts fire. The website is returning 404s. You SSH into the server and run `ls /var/www/html/`.
Instead of seeing `index.html` and `assets/`, you see:
```text
old_index.html  old_assets/  build/
```

**Your Objective:**
1. What exactly went wrong with your `rsync` command?
2. Write the correct `rsync` command to synchronize the *contents* of your local `build` directory directly into the root of `/var/www/html/`.

### Solution
1. When we ran the rsync we copied the build folder inside /var/www/html, instead of the contents of build folder.
2. The correct command will be `rsync -a build/ webserver:/var/www/html`

**Correction:**
**Grade: 100%**
Spot on! That trailing slash on `build/` is the difference between deploying the files properly versus creating a nested `build/` folder inside the web root.
