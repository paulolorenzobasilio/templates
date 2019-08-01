Prerequisites
* [Husky](https://www.npmjs.com/package/husky)


```
#!/usr/bin/env bash
# Run `chmod +x post-merge` to make it executable then put it into `.git/hooks/`.

changed_files="$(git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD)"

changed() {
	echo "$changed_files" | grep --quiet "$1" && eval "$2"
}

if changed 'package.lock'; then 
	echo "[package.lock] changed: Installing npm dependencies..."
    /**
    * Execute `npm ci` when in continous integration
    **/
	if [ "$(whoami)" == "ci_username" ]; 
	then
		echo "executing npm ci..."
        npm ci
	else 
		echo "executing npm install..."
		npm install
	fi
fi

```