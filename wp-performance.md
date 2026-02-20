# Contexto de Performance WordPress

Este arquivo consolida o conhecimento dos documentos de referência anexados sobre performance em WordPress, incluindo práticas, comandos, ferramentas e links úteis.

---

## 1. Autoloaded Options
- Opções autoload são carregadas em toda requisição; excesso prejudica performance.
- Comandos úteis:
  - Total de bytes autoload: `wp option list --autoload=on --format=total_bytes`
  - Maiores opções autoload: `wp option list --autoload=on --fields=option_name,size_bytes | sort -n -k 2 | tail`
- Fixes:
  - Não autoloadar blobs grandes; use transients/object cache.
  - Remova opções órfãs (confirme antes de deletar).
- Docs: https://wpcli.dev/docs/option/list
- WP Doctor: check `autoload-options-size`.

---

## 2. WP-Cron Performance
- Use quando cron causa picos ou lentidão.
- Comandos:
  - `wp cron test`, `wp cron event list`, `wp cron event run --due-now`
- Fixes:
  - Deduplica eventos, reduza frequência.
  - Tarefas devem ser idempotentes e rápidas.
  - Mova tarefas pesadas para fora do request.
- Docs: https://github.com/wp-cli/cron-command

---

## 3. Database / Query Performance
- Use ao identificar lentidão em queries ou alto número de queries.
- Fixes:
  - Evite N+1 queries, prefira batch.
  - Use `fields => 'ids'` quando possível.
  - Evite meta queries caras; considere índices.
  - Use object cache para leituras repetidas.
- Ferramentas:
  - Query Monitor (REST headers/envelope).
  - `wp db query` para SQL/explain.
- Docs: https://wordpress.org/plugins/query-monitor/

---

## 4. HTTP API (Remote Requests)
- Use ao identificar lentidão em requests externos (`wp_remote_get`, etc).
- Fixes:
  - Adicione timeouts e fail-fast.
  - Cacheie respostas (transients/object cache).
  - Faça batch de requests, evite em toda page load.
  - Mova requests pesados para async (cron/queue).
- Ferramenta: Query Monitor (timing de HTTP API via REST envelope).

---

## 5. Measurement (Profiling vs Benchmarking)
- Ferramentas backend:
  - `wp profile` (profiling de hooks/stages)
  - `wp doctor` (diagnóstico rápido)
  - Query Monitor via REST (headers x-qm-*)
  - Server-Timing (Performance Lab, via headers)
  - APM/profilers: New Relic, Datadog, Blackfire, Tideways, XHProf/Xdebug
- Boas práticas:
  - Sempre capture baseline.
  - Fixe cenário de teste.
  - Use múltiplas amostras e mediana.
- Docs:
  - https://make.wordpress.org/performance/handbook/measuring-performance/
  - https://make.wordpress.org/performance/handbook/measuring-performance/benchmarking-server-timing/

---

## 6. Object Caching
- Cache padrão do WP é por request.
- Cache persistente: `wp-content/object-cache.php` (Redis/Memcached).
- Comandos: https://wpcli.dev/docs/cache
- Guardrails: `wp cache flush` afeta todos sites em multisite.
- Fixes:
  - Cacheie resultados caros (transients/object cache) com invalidação explícita.
  - Evite caches sem limite (use expiração/invalidação).
  - Coordene com infra ao adicionar cache persistente.

---

## 7. Query Monitor (Headless/Backend)
- Query Monitor mostra queries, hooks, HTTP API, erros PHP.
- Pode expor dados via REST (headers x-qm-*, envelope `qm`).
- Docs: https://querymonitor.com/wordpress-debugging/rest-api-requests/
- Guardrails: não use em produção sem aprovação.

---

## 8. Server-Timing (Performance Lab)
- Permite inspeção backend via header `Server-Timing`.
- Plugin: https://wordpress.org/plugins/performance-lab/
- Benchmark: https://make.wordpress.org/performance/handbook/measuring-performance/benchmarking-server-timing/
- Guardrails: não habilite módulos experimentais em produção sem aprovação.

---

## 9. WP-CLI Doctor (`wp doctor`)
- Checks de prontidão para produção.
- Instalação: `wp package install wp-cli/doctor-command`
- Comandos:
  - `wp doctor check`
  - `wp doctor list`
- Checks relevantes: `autoload-options-size`, debug flags, cron.
- Docs: https://make.wordpress.org/cli/handbook/doctor-default-checks/

---

## 10. WP-CLI Profiling (`wp profile`)
- Profiling sem browser.
- Instalação: `wp package install wp-cli/profile-command`
- Comandos:
  - `wp profile stage --fields=stage,time,cache_ratio [--url=<url>]`
  - `wp profile hook --spotlight [--url=<url>]`
  - `wp profile hook init --spotlight [--url=<url>]`
  - `wp profile eval 'do_action("init");' --hook=init`
- Dicas:
  - Use `--url` para rotas específicas.
  - Use `--skip-plugins`/`--skip-themes` para isolar componentes.
- Docs:
  - https://wpcli.dev/docs/profile/stage
  - https://wpcli.dev/docs/profile/hook
  - https://wpcli.dev/docs/profile/eval

---

Este arquivo resume todos os pontos dos documentos de referência anexados para consulta rápida em diagnósticos e otimizações de performance WordPress.