Visão Geral
===========

Lista de issues nessa versão
----------------------------

PROP-1536_

SDKBACK-103_

SDKBACK-108_

SDKBACK-116_

SDKBACK-119_

WK-1_

.. _PROP-1536: http://jira.korp.com.br/browse/PROP-1536
.. _SDKBACK-103: https://korp.youtrack.cloud/issue/SDKBACK-103
.. _SDKBACK-108: https://korp.youtrack.cloud/issue/SDKBACK-108
.. _SDKBACK-116: https://korp.youtrack.cloud/issue/SDKBACK-116
.. _SDKBACK-119: https://korp.youtrack.cloud/issue/SDKBACK-119
.. _WK-1: https://portal.korp.com.br/projetos/7343c888-54d4-9ef3-3e57-4caba660f687/lista/e4b7502b-65c2-ea4d-83cf-22f04b9b5a5d

Melhorias
---------

* Agora é possível especificar uma Culture_ ao exportar um relatório pelo método ``ExportAsync`` da interface ``IReportingStore``.


* Agora é possível buscar connection strings de outros serviços pela ``IConnectionStringStore``, utilizando o método ``LoadConnectionStringsAsync`` e informando o nome do serviço desejado pelo parâmetro ``serviceName``.


* Agora é possível buscar companies de maneira paginada e filtrada  pelo ``ICompanyStore``, utilizando o método ``GetAllCompaniesAsync`` e informando os parâmetros ``PagedFilteredAndSortedRequestInput``.


* Agora é possível injetar serviços customizados no DbContext além dos já padrões, para isso foi disponibilizado a propriedade ``AllowedParameters`` na classe DbContextDesignTime que implementa a interface ``PostgreSqlBaseDesignTimeDbContextFactory`` ou ``SqlServerBaseDesignTimeDbContextFactory``.

* Agora é possível criar parametros no sistema legado por meio do serviço ``ILegacyParametrosService`` usando os métodos ``WriteInteger```, ``WriteString`` e ``WriteBoolean``

.. _Culture: https://learn.microsoft.com/pt-br/dotnet/api/system.globalization.cultureinfo?view=net-7.0

* ``ApplyAdvancedFilter`` agora suporta ``NodaTime.Instant`` e ``NodaTime.Instant?``

* ``WorkflowController.PublishEventAsync`` agora envia tanto para o ``Workflow.Orchestrator`` quanto para o ``Workflow.Elsa``

Breaking Changes
----------------

* A classe ``ReportingStoreExportInput`` teve seu nome alterado para ``ReportingStoreExportInput``.
* O método ``ExportAsync`` da interface ``IReportingStore`` agora aceita um ``ReportingExportInput``.
* O método ``LoadConnectionStringsAsync`` da interface ``IConnectionStringStore`` agora aceita um novo parâmetro do tipo string ``serviceName``. Para corrigir, injete ``IServiceConfiguration`` e utilize a propriedade ``ServiceName``.
* O método ``GetAllCompaniesAsync`` da interface ``GetAllCompaniesAsync`` agora aceita um novo parâmetro do tipo ``PagedFilteredAndSortedRequestInput``.
* O método ``GetAllCompaniesAsync`` da interface ``ICompanyStore`` agora aceita um novo parâmetro do tipo ``PagedFilteredAndSortedRequestInput``.
* O método ``GetAllCompaniesAsync`` da interface ``IAdministrationApi`` agora aceita um novo parâmetro do tipo ``PagedFilteredAndSortedRequestInput``.
* A classe ``UnitTestEfCoreUnitOfWorkMediator`` foi removida.
* Versão do ``Hangfire.Core`` trocada de ``1.7.30`` para ``1.7.28``, isto não deve causar problemas no código, mas a nível de ``.csproj`` pode haver conflitos, necessitando corrigir a versão.

Correções
---------

* Agora é possível utilizar transações em testes unitários utilizando ``IUnitOfWork``.
