Prerequisites
* [composer-git-hooks](https://packagist.org/packages/brainmaestro/composer-git-hooks)


```
composer.json

"hooks": {
      "pre-commit": [
        "echo committing as $(git config user.name)",
        "./scripts/lint"
      ],
      "post-merge": [
        "echo 'post-merge started'",
        "./scripts/post-merge",
        "echo 'post-merge finished';"
      ]
    }

```

### COMPOSER INSTALL ON POST MERGE
```
#!/usr/bin/env bash
# MIT © Sindre Sorhus - sindresorhus.com

# git hook to run a command after `git pull` if a specified file was changed
# Run `chmod +x post-merge` to make it executable then put it into `.git/hooks/`.

changed_files="$(git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD)"

changed() {
	echo "$changed_files" | grep --quiet "$1" && eval "$2"
}

if changed 'composer.lock'; then 
	echo "[composer.lock] changed: Installing composer dependencies..."
	composer install
fi

if changed 'database/migrations'; then
  echo "Migrations are changed: Migrating..."
  phpdbg -qrr artisan migrate
fi

if changed 'database/seeds'; then
  echo "Seeds are changed: Seeding new data..."
  phpdbg -qrr artisan db:seed
fi

```