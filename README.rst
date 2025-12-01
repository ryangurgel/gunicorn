Gunicorn enxuto
===============

Este snapshot mantém apenas o necessário para rodar o servidor WSGI e foca em
instalação rápida. O código-fonte permanece em ``gunicorn/`` e o comando
``gunicorn`` continua disponível via entrypoint definido em ``pyproject.toml``.

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

* ``gunicorn/app/``: inicialização do servidor e CLI (``wsgiapp.py`` é o entrypoint).
* ``gunicorn/workers/``: implementações de workers síncronos e assíncronos.
* ``gunicorn/config.py``: opções de configuração e mapeamento de flags.
* ``gunicorn/http/``: parser/serializer HTTP.
* ``gunicorn/reloader.py``: autoreload para desenvolvimento.
* ``gunicorn/__main__.py``: permite ``python -m gunicorn``.

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
