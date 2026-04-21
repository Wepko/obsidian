Команда на изучение собирает все в одно очень классная вешь!!!
```sh
find docker -type f -exec echo "======= {} =======" \; -exec cat {} \; > all_docker_files.txt

find docker -type f ! -name "*.sql" -exec echo "======= {} =======" \; -exec cat {} \; > all_docker_files.txt
```

```sh
ollama launch claude --model minimax-m2.7:cloud --yes -- --dangerously-skip-permissions
```
