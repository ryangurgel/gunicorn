Gunicorn enxuto
===============

Este snapshot mantém apenas o necessário para rodar o servidor WSGI e foca em
instalação rápida. O código-fonte permanece em ``gunicorn/`` e o comando
``gunicorn`` continua disponível via entrypoint definido em ``pyproject.toml``.

Visão geral rápida
------------------

* **Objetivo:** expor um servidor HTTP que recebe requisições, monta o objeto
  WSGI ``environ`` e delega o processamento para a sua aplicação, devolvendo a
  resposta HTTP resultante.
* **Entrada principal:** o pacote ``gunicorn`` é executável via ``python -m`` ou
  pelo script ``gunicorn`` criado pelo ``pyproject.toml``.
* **Processo em alto nível:**
  1. Leitura de configuração via CLI ou variáveis de ambiente.
  2. Criação do ``Arbiter`` (supervisor) que inicializa o socket de escuta.
  3. Fork dos workers (``sync`` por padrão ou outro worker class escolhido).
  4. Cada worker aceita conexões, chama sua aplicação WSGI e envia a resposta
     HTTP para o cliente.
  5. Sinais UNIX (por exemplo ``TERM``/``HUP``) controlam shutdown e reload.

O que ficou (e para que serve)
------------------------------

* ``gunicorn/``: todo o código de produção do servidor WSGI (workers,
  configurações, logger, reloader e CLI).
* ``pyproject.toml``: metadados do pacote, dependências opcionais, entrypoints e
  configurações do setuptools. Também garante que ``LICENSE`` e ``NOTICE`` vão
  para a distribuição sem depender de ``MANIFEST.in``.
* ``LICENSE`` e ``NOTICE``: termos da licença MIT e créditos de terceiros.
* ``README.rst``: este guia de uso.

Dependências e variantes
------------------------

* Requisito mínimo: Python 3.10+ e o pacote ``packaging`` (instalado
  automaticamente).
* Extras opcionais para escolher o tipo de worker:
  * ``gevent`` (``pip install .[gevent]``) — worker assíncrono baseado em greenlets.
  * ``eventlet`` (``pip install .[eventlet]``) — worker assíncrono alternativo.
  * ``tornado`` (``pip install .[tornado]``) — integra com o loop do Tornado.
  * ``setproctitle`` — opcional para personalizar o nome do processo.

Como rodar (sem instalar)
-------------------------

1. Garanta que o diretório do repositório está no ``PYTHONPATH``.
2. Exporte uma callable WSGI, por exemplo em ``hello.py``::

       def app(environ, start_response):
           start_response("200 OK", [("Content-Type", "text/plain")])
           return [b"hello"]

3. Inicie o servidor apontando para ``MODULE:APP``::

       python -m gunicorn hello:app

4. Abra ``http://127.0.0.1:8000`` para ver a resposta.

Como rodar instalado
--------------------

Instale localmente (modo editable opcional)::

    pip install .            # instalação padrão
    # ou
    pip install -e .         # desenvolvimento

Depois execute::

    gunicorn módulo:variavel

Use ``--workers``, ``--bind``, ``--worker-class`` e demais flags para ajustar
concorrência, endereço de escuta e tipo de worker.

Layout do código
----------------

**Núcleo de execução**

* ``gunicorn/app/``: inicialização do servidor e CLI (``wsgiapp.py`` é o entrypoint).
* ``gunicorn/workers/``: implementações de workers síncronos e assíncronos.
* ``gunicorn/config.py``: opções de configuração e mapeamento de flags.
* ``gunicorn/http/``: parser/serializer HTTP.
* ``gunicorn/reloader.py``: autoreload para desenvolvimento.
* ``gunicorn/__main__.py``: permite ``python -m gunicorn``.

**Componentes de suporte**

* ``gunicorn/util.py``: utilitários como import dinâmico de aplicações e
  manipulação de caminhos.
* ``gunicorn/arbiter.py``: supervisor responsável por criar workers, aplicar
  políticas de tempo limite e reagir a sinais.
* ``gunicorn/sock.py``: abstrações de socket usadas na criação de listeners.
* ``gunicorn/glogging.py``: configuração de logger (níveis, formato e rota para
  stdout/stderr).
* ``gunicorn/instrument/statsd.py``: integração opcional com StatsD para métricas.
* ``gunicorn/workers/base.py`` e ``base_async.py``: contratos e ciclo de vida
  comum a todas as classes de worker.
* ``gunicorn/workers/sync.py``: worker padrão síncrono (prefork + select/poll).
* ``gunicorn/workers/geventlet.py`` e ``tornado.py``: workers assíncronos baseados
  em greenlets ou no loop do Tornado.

**Pontos de extensão**

* **Configuração:** defina flags CLI (``--bind``, ``--workers``, ``--worker-class``)
  ou variáveis de ambiente (``GUNICORN_CMD_ARGS``). O parser de opções vive em
  ``gunicorn/config.py``.
* **Hooks:** implemente funções em ``gunicorn.conf.py`` (opcional) para eventos
  como ``post_fork`` e ``on_exit``. O carregamento desse arquivo ocorre em
  ``gunicorn/app/base.py``.
* **Apps WSGI:** qualquer callable ``(environ, start_response)`` é aceita. Use
  a opção ``--chdir`` se precisar apontar para outro diretório antes de importar
  sua aplicação.

Exemplos de execução e ciclo de vida
------------------------------------

* **Rodar rapidamente com app inline**::

      cat > hello.py <<'PY'
      def app(environ, start_response):
          start_response("200 OK", [("Content-Type", "text/plain")])
          return [b"hello"]
      PY
      python -m gunicorn hello:app

* **Escolher outro worker**::

      gunicorn --worker-class gevent --workers 4 hello:app

* **Reload em desenvolvimento (auto-recarrega código)**::

      gunicorn --reload hello:app

* **Binding em outro endereço ou porta**::

      gunicorn --bind 0.0.0.0:5000 hello:app

O ciclo de vida básico: o processo mestre (Arbiter) abre o socket definido em
``--bind``, cria os workers e monitora sinais. Se um worker morre, o arbiter o
substitui. Ao receber ``TERM`` ou ``INT``, inicia shutdown gracioso; ``HUP``
recarrega configuração e código quando possível.

Verificação rápida
------------------

Confirme a instalação e a versão esperada::

    python -m gunicorn --version

O que foi removido
------------------

Para deixar o repositório enxuto e ainda funcional, foram removidos:

* ``docs/`` e demais ativos de documentação.
* Scripts auxiliares de manutenção (``scripts/``) e arquivos de comunidade
  (``CONTRIBUTING.md``, ``SECURITY.md``, ``THANKS``, ``MAINTAINERS``).
* Configuração de CI e ferramentas de teste (``appveyor.yml``, ``Makefile``,
  ``tox.ini``, ``requirements_*.txt``).

Licença
-------

Gunicorn é distribuído sob a licença MIT. Consulte ``LICENSE`` e ``NOTICE``
para detalhes adicionais de copyright.
