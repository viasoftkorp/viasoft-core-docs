Visão Geral
===========

Lista de issues nessa versão
----------------------------

RFEN-175_

SDKB-102_

.. _RFEN-175: https://portal.korp.com.br/projetos/c2fe24bf-5393-742e-e371-237e95e9b08f/lista?identificador=RFEN-175
.. _SDKB-102: https://portal.korp.com.br/projetos/935791cf-dbfc-6432-00ce-8a691531e47f/lista?identificador=SDKB-102
.. _PROP-1949: https://portal.korp.com.br/projetos/e528ec30-79a6-dead-9fa6-89f587ec761a/lista?tenantId=16a7571e-7ff6-479e-a6f5-3514414179dc&identificador=PROP-1949&primeiro=true

Melhorias
---------

* Agora é possível incrementar e obter o valor de um parâmetro inteiro pelo ``ILegacyParametrosService``, utilizando o método ``ReadAutoIncrementInteger``. Irá criar caso o parâmetro não exista, definindo o valor inicial como input.DefaultValue + 1.
* Adicionado novo método ``valorDecimal.ArredondarAbnt()`` (de ``Viasoft.Core.DecimalRounding``), permitindo arredonda um valor decimal para 2 casas decimais utilizando o método de arredondamento "para longe de zero", conforme a norma ABNT NBR 5891. Este método arredonda valores exatamente no meio (como x.xx5) sempre  na direção oposta ao zero (valores positivos são arredondados para cima e negativos para baixo).
* Agora o método ``TriggerRecurringJobAsync`` do serviço ``IBackgroundJobManager`` retorna o id do job que foi "Triggered".
* Agora é possível enviar ``CC`` ao enviar um email pelo serviço ``IEmailSender``. Utilize a propriedade ``Cc`` no ``EmailSenderInput``.
* Agora o proxy pra a controller do Authentication ``AuthenticationProxyController`` expõe a rota ``users/default-image`` responsável por pegar a imagem de placeholder para um usuário que não tem foto.
* Agora é possível configurar quando os valores de propriedades normalizadas para ``IMustHaveCompany`` e ``IMustHaveLegacyCompany`` podem ser sobrescritas :ref:`configurando-override-property-normalizer`.
* Agora é possível buscar parametros em batch utilizando o ``ILegacyParametrosService`` método ``ReadBatch``

Breaking Changes
----------------

* Filtros por TenantId (classes que implementam ``IMustHaveTenant``) e filtros por EnvironmentId (classes que implementam ``IMustHaveEnvironment``) agora avaliam se o schema já está separado por Tenant ou Environment, dessa forma evitando adicionar o WHERE de forma desnecessária (aumentando a performance). Essa breaking change não requer alteração de código.

Correções
---------

* Agora o ``NotificationUpdateId`` é conduzido corretamente para BackgroundJob, portanto enviar notificações pelo ``IPushNotification`` funcionará em jobs
