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
* Adicionado novo método ``valorDecimal.ArredondarAbnt()`` (de ``Viasoft.Core.DecimalRounding``), permitindo arredonda um valor decimal para 2 casas decimais utilizando o método de arredondamento "para longe de zero", conforme a norma ABNT NBR 5891. Este método arredonda valores exatamente no meio (como x.xx5) sempre  na direção oposta ao zero (valores positivos são arredondados para cima e negativos para baixo).

* Agora o método ``TriggerRecurringJobAsync`` do serviço ``IBackgroundJobManager`` retorna o id do job que foi "Triggered".

Breaking Changes
----------------

* Filtros por TenantId (classes que implementam ``IMustHaveTenant``) e filtros por EnvironmentId (classes que implementam ``IMustHaveEnvironment``) agora avaliam se o schema já está separado por Tenant ou Environment, dessa forma evitando adicionar o WHERE de forma desnecessária (aumentando a performance). Essa breaking change não requer alteração de código.

Correções
---------

* Agora o ``NotificationUpdateId`` é conduzido corretamente para BackgroundJob, portanto enviar notificações pelo ``IPushNotification`` funcionará em jobs
