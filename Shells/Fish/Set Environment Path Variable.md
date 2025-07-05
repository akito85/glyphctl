### Check What Currently Sets
```bash
set -S fish_user_paths
```

### Remove Global Environment Path (If Needed)
```bash
set -e fish_user_paths
```

### Add New Global Environment Path without Overriding
```bash
set -U fish_user_paths <new_global_env_path> $fish_user_paths
```

