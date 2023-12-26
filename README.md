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

3. Для початку встановимо [gitleaks](https://github.com/gitleaks/gitleaks?tab=readme-ov-file#pre-commit) локально та перевіримо його роботу.
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

$ nano helm/values.yaml
$ git add .
$ git commit -m"add secret"

$ gitleaks detect --source . --verbose

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

Finding:     key: "************************************************"
Secret:      
RuleID:      telegram-bot-api-token
Entropy:     4.637586
File:        helm/values.yaml
Line:        15
Commit:      35bde2bb875a7ad294146787d5409dd4e5b7ab51
Author:      Vitalii Umanets
Email:       vit@i.ua
Date:        2023-12-26T21:02:37Z
Fingerprint: 35bde2bb875a7ad294146787d5409dd4e5b7ab51:helm/values.yaml:telegram-bot-api-token:15

11:09PM INF 100 commits scanned.
11:09PM INF scan completed in 4.02s
11:09PM WRN leaks found: 1
```

4. Встановимо пакет [pre-commit](https://pre-commit.com/#install)
```sh
$ sudo apt-get install pre-commit

$ pre-commit --version
pre-commit 2.17.0

$ touch .pre-commit-config.yaml

$ pre-commit install
pre-commit installed at .git/hooks/pre-commit

$ pre-commit run --all-files
Check Yaml...............................................................Failed
Fix End of Files.........................................................Failed
Trim Trailing Whitespace.................................................Failed

$ git add .
$ git commit -m"test"
Check Yaml...............................................................Failed
```
5. Як це працює зрозуміло, тепер реалізуємо перевірку репозиторію [gitleaks](https://github.com/gitleaks/gitleaks?tab=readme-ov-file#pre-commit) 
- Додаємо у файл `.pre-commit-config.yaml` наступний код:
```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.16.1
    hooks:
      - id: gitleaks
```
- Перевіряємо роботу в zsh:
```sh
$ pre-commit autoupdate
$ pre-commit install


```