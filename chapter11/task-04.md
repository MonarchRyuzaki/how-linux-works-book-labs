# Task 04: The Argument Eater (`shift` & `$@`)

**Scenario:**
An SRE deployment script accepts a variable number of service names to restart. However, the script is designed such that the **first** argument must ALWAYS be the environment name (e.g., `prod`, `staging`), and the rest are the services.
Example: `./deploy.sh prod auth-service billing-service`

The current script uses `$@` to loop over the arguments, which accidentally tries to restart a service named "prod", causing the deployment pipeline to fail.

```bash
#!/bin/sh
ENV=$1
echo "Deploying to environment: $ENV"

# FIX ME: This loops through ALL arguments, including the environment!
for service in $@; do
    echo "Restarting service: $service"
done
```

**Your Objective:**
Fix the script using the built-in `shift` command so that it correctly identifies the environment, and then uses a `for` loop to iterate over ONLY the remaining service arguments.

Provide your solution below:
### Solution
1. 
```bash 
#!/bin/sh
ENV=$1
echo "Deploying to environment: $ENV"
shift 
# FIX ME: This loops through ALL arguments, including the environment!
for service in $@; do
    echo "Restarting service: $service"
done
```
```

**Correction:**
**Grade: 100% (Excellent!)**

Perfect implementation of `shift`. By placing `shift` immediately after `ENV=$1`, the script literally shifts all arguments to the left. The old `$1` (the environment) is discarded, the old `$2` becomes the new `$1`, and so on. Now, when the `for service in $@` loop runs, `$@` correctly contains only the service names!
