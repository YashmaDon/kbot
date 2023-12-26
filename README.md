# Task18 Створення скрипту для git pre-commit hook з використанням gitleaks для перевірки наявності секретів у коді.

1. Клонуємо репозиторій готуємо нову гілку:
```sh
$ git clone https://github.com/vit-um/kbot.git
$ git checkout -b Task18
$ git tag git_hook
$ git push origin git_hook
$ git push --set-upstream origin Task18
```
2. Для зручної роботи з провідником в VSCode відкриємо видимість каталогу .git
- Тиснемо `Ctrl+,` 
- В пошук вводимо: `Search: Exclude`
- Видаляємо `**/.git`

3. Для початку встановимо [gitleaks](https://github.com/gitleaks/gitleaks?tab=readme-ov-file#pre-commit) локально.
```sh
$ cd ~
$ git clone https://github.com/gitleaks/gitleaks.git
$ cd gitleaks
$ make build
$ cp gitleaks /usr/local/bin 
$ gitleaks detect --source . --log-opts="--all"

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

10:50PM INF 99 commits scanned.
10:50PM INF scan completed in 4.01s
10:50PM INF no leaks found

```


4. Вивчаємо файл `kbot/.git/hooks/pre-commit.sample`
