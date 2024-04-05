Visão Geral
===========

Lista de issues nessa versão
----------------------------

PROP-1536_

SDKBACK-103_

SDKBACK-108_

SDKBACK-116_

SDKBACK-119_

PROM-242_

.. _PROP-1536: http://jira.korp.com.br/browse/PROP-1536
.. _SDKBACK-103: https://korp.youtrack.cloud/issue/SDKBACK-103
.. _SDKBACK-108: https://korp.youtrack.cloud/issue/SDKBACK-108
.. _SDKBACK-116: https://korp.youtrack.cloud/issue/SDKBACK-116
.. _SDKBACK-119: https://korp.youtrack.cloud/issue/SDKBACK-119
.. _PROM-242: https://portal.korp.com.br/projetos/e261cc03-896b-2316-6d90-dd1c1dc363e6/lista/84ade6c7-d489-ebdf-c066-c2e07599eeaf

Melhorias
---------

* Agora é possível especificar uma Culture_ ao exportar um relatório pelo método ``ExportAsync`` da interface ``IReportingStore``.

* Agora é possível buscar connection strings de outros serviços pela ``IConnectionStringStore``, utilizando o método ``LoadConnectionStringsAsync`` e informando o nome do serviço desejado pelo parâmetro ``serviceName``.

* Agora é possível buscar companies de maneira paginada e filtrada  pelo ``ICompanyStore``, utilizando o método ``GetAllCompaniesAsync`` e informando os parâmetros ``PagedFilteredAndSortedRequestInput``.

* Agora é possível injetar serviços customizados no DbContext além dos já padrões, para isso foi disponibilizado a propriedade ``AllowedParameters`` na classe DbContextDesignTime que implementa a interface ``PostgreSqlBaseDesignTimeDbContextFactory`` ou ``SqlServerBaseDesignTimeDbContextFactory``.

* Agora é possível criar parametros no sistema legado por meio do serviço ``ILegacyParametrosService`` usando os métodos ``WriteInteger```, ``WriteString`` e ``WriteBoolean``

* Agora é possível configurar globalmente o tempo de timeout para todas as queries que são executadas pelo EF Core, utilizando a propriedade ``"DbContextOptions:CommandTimeout"`` pelo Consul.

* Agora é possível configurar globalmente erros detalhados do EF Core, utilizando a propriedade ``"DbContextOptions:EnableDetailedErrors"`` pelo Consul.

* Agora é possível configurar globalmente logs de dados sensíveis (parâmetros das queries) do EF Core, utilizando a propriedade ``"DbContextOptions:EnableSensitiveDataLogging"`` pelo Consul.

* Agora é possível configurar o grau máximo de paralelismo no ServiceBus, utilizando a propriedade ``"ServiceBus:MaxParallelism"`` pelo Consul.

* Agora é possível configurar o número máximo de "trabalhadores" (threads) no ServiceBus, utilizando a propriedade ``"ServiceBus:NumberOfWorkers"`` pelo Consul.

* Agora é possível configurar o número máximo de tentativas para mensagens que falham no ServiceBus, utilizando a propriedade ``"ServiceBus:MaxDeliveryAttempts"`` pelo Consul.

* Agora é possível configurar o PrefetchSize_ do broker rabbitmq, utilizando a propriedade ``"ServiceBus:Transport:PrefetchSize"`` pelo Consul.

* Agora é possível configurar facilmente que as propriedades ``string`` ou ``decimal`` para utilizaram ``VARCHAR`` ou ``DECIMAL`` no banco de dados SQL Server.

.. _Culture: https://learn.microsoft.com/pt-br/dotnet/api/system.globalization.cultureinfo?view=net-7.0
.. _PrefetchSize: https://www.rabbitmq.com/docs/consumer-prefetch

Breaking Changes
----------------

* A classe ``ReportingStoreExportInput`` teve seu nome alterado para ``ReportingStoreExportInput``.
* O método ``ExportAsync`` da interface ``IReportingStore`` agora aceita um ``ReportingExportInput``.
* O método ``LoadConnectionStringsAsync`` da interface ``IConnectionStringStore`` agora aceita um novo parâmetro do tipo string ``serviceName``. Para corrigir, injete ``IServiceConfiguration`` e utilize a propriedade ``ServiceName``.
* O método ``GetAllCompaniesAsync`` da interface ``GetAllCompaniesAsync`` agora aceita um novo parâmetro do tipo ``PagedFilteredAndSortedRequestInput``.
* O método ``GetAllCompaniesAsync`` da interface ``ICompanyStore`` agora aceita um novo parâmetro do tipo ``PagedFilteredAndSortedRequestInput``.
* O método ``GetAllCompaniesAsync`` da interface ``IAdministrationApi`` agora aceita um novo parâmetro do tipo ``PagedFilteredAndSortedRequestInput``.
* A classe ``UnitTestEfCoreUnitOfWorkMediator`` foi removida.
* O método ``BatchHardDeleteAsync`` da classe ``EntityFrameworkCoreRepositoryExtensions`` teve o tipo do primeiro parametro alterado de ``IRepository`` para ``IQueryable``. Essa breaking change não requer alteração de código.
* O método ``BatchUpdateAsync`` da classe ``EntityFrameworkCoreRepositoryExtensions`` teve o tipo do primeiro parametro alterado de ``IRepository`` para ``IQueryable``. Essa breaking change não requer alteração de código.
* O método ``GetMessageDetailsByMessageTye`` da interface ``IServiceBusMessageMapping`` foi renomeado para ``GetMessageDetailsByMessageType``.

Correções
---------

* Agora é possível utilizar transações em testes unitários utilizando ``IUnitOfWork``.
