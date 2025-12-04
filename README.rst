Gunicorn enxuto
===============

Este snapshot mantém apenas o necessário para rodar o servidor WSGI. O código
fonte permanece em ``gunicorn/`` e o comando ``gunicorn`` continua disponível
via entrypoint definido em ``pyproject.toml``.

Instalação mínima
-----------------

* Python 3.10 ou superior.
* Dependência direta: ``packaging`` (já declarada no ``pyproject.toml``).

Instale localmente::

    pip install .

Ou execute sem instalar (assumindo o diretório no ``PYTHONPATH``)::

    python -m gunicorn MODULE:APP

``MODULE:APP`` segue o padrão ``pacote.modulo:variavel`` onde ``variavel`` é
uma callable WSGI exportada pelo módulo indicado.

O que foi removido
------------------

Para deixar o repositório mais enxuto e ainda funcional, removemos:

* ``docs/`` e demais ativos de documentação.
* Scripts auxiliares (``scripts/``) usados para tarefas de manutenção.
* Arquivos de contribuição, segurança e agradecimentos (``CONTRIBUTING.md``,
  ``SECURITY.md``, ``THANKS``, ``MAINTAINERS``).
* Configuração de CI, tox e requisitos de teste/desenvolvimento
  (``appveyor.yml``, ``Makefile``, ``tox.ini``, ``requirements_*.txt``).

Licença
-------

Gunicorn é distribuído sob a licença MIT. Consulte ``LICENSE`` e ``NOTICE``
para detalhes adicionais de copyright.
