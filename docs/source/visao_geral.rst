Visão Geral
===========

Lista de issues nessa versão
----------------------------

RFEN-175_
SDKB-102_

.. _RFEN-175: https://portal.korp.com.br/projetos/c2fe24bf-5393-742e-e371-237e95e9b08f/lista?identificador=RFEN-175
.. _SDKB-102: https://portal.korp.com.br/projetos/935791cf-dbfc-6432-00ce-8a691531e47f/lista?identificador=SDKB-102

Melhorias
---------

* Agora é possível incrementar e obter o valor de um parâmetro inteiro pelo ``ILegacyParametrosService``, utilizando o método ``ReadAutoIncrementInteger``. Irá criar caso o parâmetro não exista, definindo o valor inicial como input.DefaultValue + 1.

Breaking Changes
----------------

* Filtros por TenantId (classes que implementam ``IMustHaveTenant``) e filtros por EnvironmentId (classes que implementam ``IMustHaveEnvironment``) agora avaliam se o schema já está separado por Tenant ou Environment, dessa forma evitando adicionar o WHERE de forma desnecessária (aumentando a performance). Essa breaking change não requer alteração de código.

