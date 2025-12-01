Gunicorn enxuto
===============

Este snapshot mantém o mínimo para rodar o servidor WSGI em produção ou
desenvolvimento local. Não há arquivos de CI, docs geradas ou scripts auxiliares
sobrando: apenas código, licença e um guia completo para operar e entender o que
fica no ar.

Sumário
-------

1. `O que está aqui (e para que serve) <#o-que-esta-aqui-e-para-que-serve>`_
2. `Dependências e variantes <#dependencias-e-variantes>`_
3. `Como rodar diretamente do repositório <#como-rodar-diretamente-do-repositorio>`_
4. `Como rodar instalado <#como-rodar-instalado>`_
5. `Fluxo de execução detalhado <#fluxo-de-execucao-detalhado>`_
6. `Configuração e flags essenciais <#configuracao-e-flags-essenciais>`_
7. `Sinais e ciclo de vida <#sinais-e-ciclo-de-vida>`_
8. `Layout do código (mapa rápido) <#layout-do-codigo-mapa-rapido>`_
9. `O que foi removido <#o-que-foi-removido>`_
10. `Licença <#licenca>`_

O que está aqui (e para que serve)
----------------------------------

* ``gunicorn/`` — todo o código de produção do servidor WSGI: CLI, supervisor,
  workers, logging, HTTP parser, autoreload e integrações opcionais.
* ``pyproject.toml`` — metadados, dependências, extras, entrypoints e empacotamento
  (inclui ``LICENSE`` e ``NOTICE`` via ``license-files`` sem precisar de
  ``MANIFEST.in``).
* ``LICENSE`` e ``NOTICE`` — termos MIT e créditos de terceiros.
* ``README.rst`` — este guia detalhado de uso e operação.

Dependências e variantes
------------------------

* **Requisito mínimo:** Python 3.10+ e a dependência ``packaging`` (instalada
  automaticamente).
* **Extras de workers:**
  * ``gevent`` (``pip install .[gevent]``) — worker assíncrono baseado em greenlets.
  * ``eventlet`` (``pip install .[eventlet]``) — worker assíncrono alternativo.
  * ``tornado`` (``pip install .[tornado]``) — integra com o loop do Tornado.
  * ``gthread`` — worker baseado em threads (sem dependências extras).
  * ``setproctitle`` — opcional para renomear o processo.

Como rodar diretamente do repositório
-------------------------------------

1. Garanta que o diretório do repositório está no ``PYTHONPATH``.
2. Exporte uma callable WSGI simples, por exemplo em ``hello.py``::

       def app(environ, start_response):
           start_response("200 OK", [("Content-Type", "text/plain")])
           return [b"hello"]

3. Inicie o servidor apontando para ``MODULE:APP``::

       python -m gunicorn hello:app

4. Abra ``http://127.0.0.1:8000`` para ver a resposta.

Como rodar instalado
--------------------

Instalação local (modo editable opcional)::

    pip install .            # instalação padrão
    pip install -e .         # modo desenvolvimento

Depois execute::

    gunicorn módulo:variavel

*Use* ``--workers``, ``--bind``, ``--worker-class`` e demais flags para ajustar
concorrência, endereço de escuta e tipo de worker.

Fluxo de execução detalhado
---------------------------

1. **Entrada/CLI** — ``gunicorn.app.wsgiapp:run`` (exposto pelo script
   ``gunicorn`` e por ``python -m gunicorn``) analisa argumentos, aplica defaults
   de ``gunicorn/config.py`` e lê ``GUNICORN_CMD_ARGS`` quando presente.
2. **Configuração carregada** — opções são consolidadas em um objeto ``Config``
   (``gunicorn/app/base.py``), incluindo um eventual ``gunicorn.conf.py`` no
   diretório atual ou apontado por ``-c``.
3. **Supervisor (Arbiter)** — ``gunicorn/arbiter.py`` abre o socket de escuta,
   seta o ``pidfile``/``umask``/``user`` se configurados, e faz ``fork`` dos
   workers.
4. **Workers** — cada classe em ``gunicorn/workers/`` implementa o método
   ``run()`` para aceitar conexões, criar o ``environ`` WSGI e chamar a aplicação.
   O worker padrão é ``sync``; alternativas incluem ``gevent``, ``eventlet``,
   ``tornado`` e ``gthread``.
5. **Ciclo vivo** — o Arbiter monitora workers, reinicia os que morrem, aplica
   timeouts e responde a sinais (``TERM``, ``INT``, ``HUP``, ``USR2`` etc.).
6. **Shutdown gracioso** — ao receber ``TERM``/``INT``, o Arbiter fecha o socket,
   envia ``SIGTERM`` para os workers e aguarda conclusão, forçando ``SIGKILL``
   se o timeout expira.

Configuração e flags essenciais
-------------------------------

* **Bind/endereço**: ``--bind 0.0.0.0:8000`` (ou variável ``GUNICORN_CMD_ARGS``).
* **Workers**: ``--workers 4`` define quantos processos atenderão conexões; use
  ``--worker-class gevent`` para IO assíncrono.
* **Threads (worker gthread)**: ``--threads 8`` define o número de threads.
* **Timeouts**: ``--timeout 30`` (tempo máximo por requisição) e
  ``--graceful-timeout`` para shutdown.
* **Reload**: ``--reload`` ativa o autoreload em desenvolvimento
  (implementado em ``gunicorn/reloader.py``).
* **Config file**: ``gunicorn.conf.py`` ou ``-c caminho`` com Python simples, por
  exemplo::

      bind = "0.0.0.0:8000"
      workers = 3
      worker_class = "gthread"
      threads = 8

Sinais e ciclo de vida
----------------------

* ``TERM``/``INT`` — desligamento gracioso (encerra socket, sinaliza workers e
  aplica timeout).
* ``HUP`` — recarrega configuração e código da aplicação quando possível.
* ``USR2`` — hot reload binário: lança um novo mestre, depois substitui o antigo
  após transição bem-sucedida.
* ``TTIN``/``TTOU`` — adiciona ou remove um worker por sinal.

Layout do código (mapa rápido)
------------------------------

**Núcleo**

* ``gunicorn/app/wsgiapp.py`` — ponto de entrada da CLI.
* ``gunicorn/app/base.py`` — carregamento de config, aplicação WSGI e hooks.
* ``gunicorn/arbiter.py`` — supervisor mestre (fork, monitoramento, sinais).
* ``gunicorn/workers/`` — classes de worker: ``sync.py`` (padrão),
  ``gthread.py``, ``geventlet.py`` (gevent/eventlet), ``tornado.py``.
* ``gunicorn/http/`` — parsing/serialização HTTP de baixo nível.

**Suporte**

* ``gunicorn/config.py`` — definição das opções de CLI e defaults.
* ``gunicorn/glogging.py`` — configuração de logging estruturado no stdout/stderr.
* ``gunicorn/reloader.py`` — autoreload baseado em watchers de arquivo.
* ``gunicorn/instrument/statsd.py`` — envio opcional de métricas para StatsD.
* ``gunicorn/util.py`` e ``gunicorn/sock.py`` — utilitários de import dinâmico e
  criação de sockets.

Exemplos rápidos
----------------

* **App inline**::

      cat > hello.py <<'PY'
      def app(environ, start_response):
          start_response("200 OK", [("Content-Type", "text/plain")])
          return [b"hello"]
      PY
      python -m gunicorn hello:app

* **Worker assíncrono**::

      gunicorn --worker-class gevent --workers 4 hello:app

* **Auto-reload em desenvolvimento**::

      gunicorn --reload hello:app

* **Alterar bind**::

      gunicorn --bind 0.0.0.0:5000 hello:app

Verificação rápida
------------------

Confirme a instalação e a versão esperada::

    python -m gunicorn --version

O que foi removido
------------------

Para manter o repositório pequeno sem perder funcionalidade:

* ``docs/`` e ativos estáticos de documentação.
* Scripts auxiliares (``scripts/``), arquivos de comunidade e políticas
  (``CONTRIBUTING.md``, ``SECURITY.md``, ``THANKS``, ``MAINTAINERS``).
* Configuração de CI/build e ferramentas de teste
  (``appveyor.yml``, ``Makefile``, ``tox.ini``, ``requirements_*.txt``).

Licença
-------

Gunicorn é distribuído sob a licença MIT. Consulte ``LICENSE`` e ``NOTICE`` para
detalhes adicionais.
