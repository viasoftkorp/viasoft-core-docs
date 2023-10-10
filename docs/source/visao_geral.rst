Visão Geral
===========

Lista de issues nessa versão
----------------------------

PROP-1536_

SDKBACK-103_

SDKBACK-108_

.. _PROP-1536: http://jira.korp.com.br/browse/PROP-1536
.. _SDKBACK-103: https://korp.youtrack.cloud/issue/SDKBACK-103
.. _SDKBACK-108: https://korp.youtrack.cloud/issue/SDKBACK-108

Melhorias
---------

* Agora é possível especificar uma Culture_ ao exportar um relatório pelo método ``ExportAsync`` da interface ``IReportingStore``.

.. _Culture: https://learn.microsoft.com/pt-br/dotnet/api/system.globalization.cultureinfo?view=net-7.0

* Agora é possível buscar connection strings de outros serviços pela ``IConnectionStringStore``, utilizando o método ``LoadConnectionStringsAsync`` e informando o nome do serviço desejado pelo parâmetro ``serviceName``.

* Agora é possível buscar companies de maneira paginada e filtrada  pelo ``ICompanyStore``, utilizando o método ``GetAllCompaniesAsync`` e informando os parâmetros ``PagedFilteredAndSortedRequestInput``.

Breaking Changes
----------------

* A classe ``ReportingStoreExportInput`` teve seu nome alterado para ``ReportingStoreExportInput``.
* O método ``ExportAsync`` da interface ``IReportingStore`` agora aceita um ``ReportingExportInput``.
* O método ``LoadConnectionStringsAsync`` da interface ``IConnectionStringStore`` agora aceita um novo parâmetro do tipo string ``serviceName``. Para corrigir, injete ``IServiceConfiguration`` e utilize a propriedade ``ServiceName``.
* O método ``GetAllCompaniesAsync`` da interface ``GetAllCompaniesAsync`` agora aceita um novo parâmetro do tipo ``PagedFilteredAndSortedRequestInput``.
* O método ``GetAllCompaniesAsync`` da interface ``ICompanyStore`` agora aceita um novo parâmetro do tipo ``PagedFilteredAndSortedRequestInput``.
* O método ``GetAllCompaniesAsync`` da interface ``IAdministrationApi`` agora aceita um novo parâmetro do tipo ``PagedFilteredAndSortedRequestInput``.
  