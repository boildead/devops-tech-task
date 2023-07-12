2. Create a basic Dashboard (following USE method)
Увидел 'US', но не увидел 'E'. Можно пару панелей с примерами 'E'? 

- Добавил для сети, диска (из ошибок в syslog)

3. Write alerts to cover the primary VM resources
a. Нет доказательства работоспособности алертов. У prometheus'a есть для специальный инструмент для unit-тестов получившихся алертов.
b. Зачем нужны node-exporter.rules?

- Добавил тесты для некоторых алертов в `roles/common/files/node_alerts_test.yml`
- node-exporter.rules использую для графика в Grafana, чтобы считать в prometheus utilisation по CPU и RAM вместо Grafana. Уменьшает количество запросов к prometheus.

4. Everything must be set in docker containers under the control of dockerd. It should "survive" reboot
Почему не стал использовать docker-compose?

- Возможно стоило использовать для более структурированной конфигурации, когда у каждого сервиса был бы отдельный compose файл, легче читалась бы конфигурация, удобнее изменять. Так же в docker-compose есть опции, которые не поддерживаются модулем docker_container, но в данном случае они не используются.

6. Prometheus should scrape all containers via Docker SD
Не реализован функционал, впринципе. Речь про https://prometheus.io/docs/prometheus/latest/configuration/configuration/#docker_sd_config 

- Добавил конфигурацию, удалил metrics-addr из daemon.json

9. There should be infrastructure tests for resulted resources/stack
Пример тестов увидел, но использовать Ansible в качестве фреймворка для Unit-тестивроания и тестирования на регрессию, крайне нежелательный подход.
Можешь, пожалуйста, пару примеров тестов написать на pytest-testinfra?

- Добавил примеры тестов в tests/test_infra.py

Относительно общего подода, есть следующие комментарии.
1. Переменные в роли находятся в vars, что практически лишает возможности переопределить переменную где-либо (https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)

- Изменил на roles/common/defaults/main.yml, с самым низким приоритетом.

2. Отсутствует работа с inventory. Можешь накидать пример, как ты бы работал с inventory, учитывая предыдущий комментарий? Например, мы хотим на хосте использовать отличную версии Grafana или тот же пароль администратора Grafana

- Добавил пароль в roles/common/files/secrets_file.enc. Можно использовать тот же пароль с помощью `ansible-playbook ... -e secrets_file.enc --ask-vault-pass ...`
- При добавлении нового хоста можно например переопределить в host_vars переменную `grafana_version`

3. Что будет, если, например, Prometheus начнёт утилизировать весь доступный CPU или RAM? Либо любой другой сервис, используемый в твоём решении. Можешь обыграть этот момент?

- Добавил для prometheus ограничение CPU и RAM. Так же добавил переменную prometheus_query_max_samples, которая позволит соответсвенно уменьшить количество сэмплов, которые будут храниться в памяти. Переназначить можно например в host_vars в зависмости от ресурсов хоста. Соотвественно это можно сделать для всех остальных компонентов.

4. Если бы передавал секрет не через ENV, то как?
- В ansible-vault, как добавил в данном случае.
- Если есть Hashicorp Vault, то можно использовать его.

```
ansible.builtin.set_fact:
  secrets: "{{ lookup('community.hashi_vault.hashi_vault', 'secret/data/secrets', token=my_token_var, url='http://myvault_url:8200') }}"
  grafana_admin_password: {{ secrets[grafana_admin_password] }}
```

5. Отсутвует какое-либо руководство/readme к роли. 

- Добавил [README.md](README.md)

6. Отсутвует какая-либо параметризация роли. Если бы параметризировал, то что параметризировал в первую очередь?

- Добавил параметризацию в alertmanager
- Добавил префикс для контейнеров и volume
- Если бы параметризировал остальное, начал маппинга портов контейнеров, хостовых путей конфигов, volume mounts для сервисов которым нужен доступ к хостовой файловой системе.
- В docker_install.yml добавил бы параметризацию для возмонжности установки на системы, отличные от Ubuntu 20.04.
