## Задача 1: Обеспечить разработку

Предложите решение для обеспечения процесса разработки: хранение исходного кода, непрерывная интеграция и непрерывная поставка. 
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- облачная система;
- система контроля версий Git;
- репозиторий на каждый сервис;
- запуск сборки по событию из системы контроля версий;
- запуск сборки по кнопке с указанием параметров;
- возможность привязать настройки к каждой сборке;
- возможность создания шаблонов для различных конфигураций сборок;
- возможность безопасного хранения секретных данных (пароли, ключи доступа);
- несколько конфигураций для сборки из одного репозитория;
- кастомные шаги при сборке;
- собственные докер-образы для сборки проектов;
- возможность развернуть агентов сборки на собственных серверах;
- возможность параллельного запуска нескольких сборок;
- возможность параллельного запуска тестов.

Обоснуйте свой выбор.

## Решение 1

Рассматриваются следующие хранилища кода:
- Gitlab
- Nexus OSS
- Gitea  
- Harbor 
 GitLab и Gitea являются хранилищами кода и поддерживают версионирование, автоматизацию CI\CD(в том числе в k8s), а также имеют прекрасные показатели по интеграции с другими CI\CD продуктами(Jenkins,ArgoCD) и средствами совместной работы(Slack, Discord, MS Teams). Остальные два являются чисто хранилищами артифактов.

| Правило сравнения         |          GitLab                            |               Nexus OSS             |         Gitea                    |          Harbor                   |
|---------------------------|--------------------------------------------|-------------------------------------|----------------------------------|-----------------------------------|
| Поддержка code review     |           Есть                             |            Нет                      |            Есть                  |            Нет                    |
| Назначение                |     Средний и крупный бизнес               | От малого до крупных корпораций     |     Малый и средний бизнес       | От малого до крупных корпораций   | 
| Поддержка CI\CD           |          Есть                              |              Нет                    |           Есть                   |             Нет                   |
| Версионирование           |          Есть                              |              Нет                    |           Есть                   |             Нет                   |
| Сканирование на уязвимости|       Нет из коробки(*1)                   |            Есть(SVS)                |            Нет                   |             Есть(*2)              |
| Поддержка различных СУБД  |       Средняя                              |             Низкая                  |          Высокая                 |    Низкая(только PostgreSQL)      |
| Репликация                |          Нет                               |            Платно                   |          Нет                     |             Есть                  |
| Типы хранимых артифактов  | Docker образы, пакеты , файлы кода         | Docker Образы, пакеты сборки        | Docker образы, пакеты,файлы кода |     Только Docker-образы          |
| Политики очистки          |         Нет                                |           Есть                      |         Нет                      |             Есть                  |
| Интеграция                |     большое к-во плагинов                  |  большое к-во плагинов              | Много плагинов, но нет для Jira  | Есть все необходимые плагины(*3)  | 
| Аутентификация            | 6 (LDAP,SAML,SCIM ,OAuth 2.0,Kerberos,JWT) | 4 (Atlassian Crowd, LDAP, SAML, RUT)|    4 (AD, LDAP, PAM, FreeIPA )   |   4 (AD, LDAP, RBAC, OIDC) (*4)   |
| Метрики Prometheus        |             Есть                           |              Нет                    |        Есть                      |         Есть                      |


1. Для проведения сканирования на уязвимости Net сборок в Gitlab рекомендуется использовать SAST(Static Application Security Testing) анализатор [Semgrep](https://gitlab.com/gitlab-org/security-products/analyzers/semgrep) 
2. Достаточно поставить специальный флаг при установке, чтобы был установлен пакет Trivy. В Nexus реализована только базовое сканирование. За дополнительный функционал придется заплатить. В Harbor этот функционал поставляется бесплатно.
3. Есть [плагин Jenkins для Harbor](https://plugins.jenkins.io/harbor/), из коробки можно настраивать реплакацию с Gitlab и Nexus
4. Есть интеграция с AD 

Таким образом для решения задачи подходят оба решения: Gitlab и Gitea. 
Соответственно рассмотрим условия для внедрения каждого из предложенных вариантов
1. Gitea
- Более легковестное(менее ресурсозатратное) 
- Небольшое количество репозиториев(до 1000)
- Персонал хорошо знаком с GitHub Actions ( CI\CD пишутся с использованием Github Actions)
- Требуется мультиплатформенность(Gitea написана на Go и можно легко получить дистрибутив для Linux, Windows, Mac OS)

2. Gitlab:
- Может потребоваться сканирование на уязвимости(пусть нет из коробки, но можно скачать готовое решение)
- Требуется интеграция с Jira
- Требуется размещение большого количество репозиториев(более 200)
- Требуется написание сложных CI\CD конвейеров(Gitea Action Runner появился относительно недавно и еще сыроват) 

### Используемые ссылки

1. [Harbor](https://goharbor.io/docs/2.10.0/administration/)
2. [Сравнение хранилищ от Bluelight](https://bluelight.co/blog/how-to-choose-a-container-registry)
3. [Аутентификация в Gitlab](https://docs.gitlab.com/ee/administration/auth/)
4. [Сравнительная таблица от разработчиков Gitea](https://docs.gitea.com/next/installation/comparison)


## Задача 2: Логи

Предложите решение для обеспечения сбора и анализа логов сервисов в микросервисной архитектуре.
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- сбор логов в центральное хранилище со всех хостов, обслуживающих систему;
- минимальные требования к приложениям, сбор логов из stdout;
- гарантированная доставка логов до центрального хранилища;
- обеспечение поиска и фильтрации по записям логов;
- обеспечение пользовательского интерфейса с возможностью предоставления доступа разработчикам для поиска по записям логов;
- возможность дать ссылку на сохранённый поиск по записям логов.

Обоснуйте свой выбор.

## Решение 2

Рассматриваются следующие системы сбора и анализа логов:
- Fluentd
- Graylog
- SigNoz 
- Logstash(ELK)

| Правило сравнения         |          Fluentd                            |               Graylog               |         SigNoz                  |          ELK              |
|---------------------------|---------------------------------------------|-------------------------------------|---------------------------------|---------------------------|
| Цитируемость на Github    |           12.6 тысячи                       |                7.1 тысячи           |           17.1 тысяча           |            14 тысяч       | 
| Плагины                   |              1121                           |                более 1000           |           Есть(1*)              |            274            | 
| Антироссийские санкции    |               Нет                           |                   Да                |           Нет                   |            Нет            |  
| Уведомления               |              Есть                           |                  Есть               |           Есть                  |            Есть           |
| Все-в-одном(2*)           |               Да                            |                  Нет                |             Да                  |             Нет           |

1. SigNoz осуществляет обмен информацией по стандарту OpenTelemetry. Это позволяет например использовать Prometheus для сбора метрик и передачи их в SigNoz
2. "Все-в-одном" это функция позволяет использовать единый интерфейс выбранной системы не только для поиска логов, ошибок и назначения уведомлений, но и формирования метрик и трассировки работы сервисов в одном продукте, используя единый синтаксис команды и API

Среди представленных решений наиболее частоиспользуемым и активно развиваемым решением являются ELK-стек(включающий Elasticsearch, Logstash и kibana) и SigNoz.
ELK имеет хорошую документацию чего нельзя сказать про SigNoz. Однако SigNoz хранит свои данные в БД Clickhouse, которая имеет в 5 раз более высокую производительность по сравнению с elasticsearch. Также SigNoz обеспечивает не только логирование но и мониторинг, и трассировку сервисов в одном месте.  Таким образом, для крупных предприятий рекомендуется рассмотреть возможность использования SigNoz.
На последок надо отметить что наиболее серьездная утечкая конфиденциальной информации произошла в 2019 году через систему ELK, т.к.  в comminity edition поддерживается базовая система аутентификации. Это в свою очередь требует обязательного SSL-канала. Данное требование по безопасности относится также к SigNoz и другим системам логироования(FluentD). 

*Ответ: ELK стек для малых и средних предприятий. SigNoz для крыпных предприятий.*

### Используемые ссылки:

1. [Статья с Хабр по выбору системы обработки логов](https://habr.com/ru/companies/1cloud/articles/420589/)
2. [Использование SigNoz с ApiSix](https://navendu.me/posts/apisix-metrics-signoz/)
3. [Данные по Community Edition SigNoz](https://signoz.io/pricing/)

## Задача 3: Мониторинг

Предложите решение для обеспечения сбора и анализа состояния хостов и сервисов в микросервисной архитектуре.
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- сбор метрик со всех хостов, обслуживающих систему;
- сбор метрик состояния ресурсов хостов: CPU, RAM, HDD, Network;
- сбор метрик потребляемых ресурсов для каждого сервиса: CPU, RAM, HDD, Network;
- сбор метрик, специфичных для каждого сервиса;
- пользовательский интерфейс с возможностью делать запросы и агрегировать информацию;
- пользовательский интерфейс с возможностью настраивать различные панели для отслеживания состояния системы.

Обоснуйте свой выбор.

## Решение 3
Есть два варианта:
- TICK
- Prometheus + Grafana

1. TICK

 - Работает по Push-модели(т.е. требует установки агента сбора метрик Telegraph на каждую машину с которой будет собирать метрики)
 - БД InfluxDB поддерживает работу с наносекундами
 - СУБД InfluxDB имеет большую аудиторию пользователей(27k)
 - СУБД InfluxDB является Time-Series СУБД

2. Prometheus+Grafana

 - Может работает по Pull-модели (не требует установки дополнительных агентов)  и по Pull-модели(возможна установка Node Exporter)
 - БД Prometheus поддерживает работу с миллесекундами
 - Имеет удобный интерфейс
 - Хорошо интегрируемое решение. Имеет большую аудиторию пользователей и последователей(53k).
 - Prometheus является не только СУБД, но и средством для формирования правил мониторинга 
 - Поддерживает создание выделенного шлюза

 ### Использованные ссылки

 1. [Сравнение Prometheus и InfluxDB](https://www.metricfire.com/blog/prometheus-vs-influxdb/)

 *Решением с самой качественной документацией, самой широкой аудиторией поддержки и высокой степенью масштабирования является Prometheus+Grafana*

 
