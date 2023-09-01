Visão Geral
===========

Lista de issues nessa versão
----------------------------

PROP-1536_

.. _PROP-1536: http://jira.korp.com.br/browse/PROP-1536

Melhorias
---------

* Agora é possível especificar uma Culture_ ao exportar um relatório pelo método ``ExportAsync`` da interface ``IReportingStore``.

.. _Culture: https://learn.microsoft.com/pt-br/dotnet/api/system.globalization.cultureinfo?view=net-7.0

Breaking Changes
----------------

* A classe ``ReportingStoreExportInput`` teve seu nome alterado para ``ReportingStoreExportInput``.
* O método ``ExportAsync`` da interface ``IReportingStore`` agora aceita um ``ReportingExportInput``.