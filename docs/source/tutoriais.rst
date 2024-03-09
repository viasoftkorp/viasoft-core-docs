Tutoriais
#########

.. _adicionando-automapper-em-testes-unitarios:

Adicionando versionamento a API
-------------------------------

Para habilitar versionamento na API, você deve utilizar o método ``AddVersioning()``.
O parâmetro ``configuration`` deve ser alimentado com o campo ``_configuration`` disponível no ``Startup``.

Os parâmetros ``versioningSetupAction`` e ``mvcSetupAction`` são opcionais, e servem para configurar opções internas da biblioteca aspnet-api-versioning_.

.. warning::

   No caso de configurações customizadas, o SDK não aplicará nenhuma configuração nos tipos ``AspApiVersioningOptions`` e ``MvcApiVersioningOptions``.

Além de adicionar a biblioteca de versionamento, é necessário anotar **todas** controllers com as versões suportadas com o atributo ``ApiVersion``.

.. code-block:: c#

   [ApiVersion(2023.3)]
   [ApiVersion(2023.2)]
   [ApiVersion(2023.1)]
   public class CitiesController: BaseController
   {        
   }


Recuperando informações AmbientData
-----------------------------------

Todas as informações AmbientData estão centralizadas no serviço ``IAmbientData``.

Declare uma variável do tipo ``IAmbientData``

.. code-block:: c#

   private readonly IAmbientData _ambientData;

E injete no seu construtor.

Utilize as seguintes funções para recuperar o dado

==================== ==========================
AmbientData          Função
==================== ==========================
TenantId             GetTenantId()
UserId               GetUserId()
CompanyId            GetCompanyId()
EnvironmentId        GetEnvironmentIdOrThrow()
LegacyCompanyId      GetLegacyCompanyId()
NotificationUpdateId GetNotificationUpdateId()
ApiVersion           GetApiVersion()
==================== ==========================

Adicionando AutoMapper em testes unitários
------------------------------------------

Declare a função ``AddServices`` na sua classe de teste unitário e faça uma chamada para ``AddAutoMapper``.

.. code-block:: c#

   protected override void AddServices()
   {
     ServiceCollection.AddAutoMapper();  
     base.AddServices();
   }

O parâmetro ``assembliesToAdd`` é opcional porém recomendado. Nele você deverá passar em qual(is) assembly estão os Profiles do AutoMapper

.. code-block:: c#

   protected override void AddServices()
   {
     ServiceCollection.AddAutoMapper(new[] { typeof(CityMapperProfile).Assembly });  
     base.AddServices();
   }

Para utilizar uma instância de ``IMapper``, utilize o método ``GetService`` do ``ServiceProvider``

.. code-block:: c#

   var mapper = ServiceProvider.GetService<IMapper>();


.. _aspnet-api-versioning: https://github.com/dotnet/aspnet-api-versioning/

Aumentando o tempo de timeout de queries
----------------------------------------

Para aumentar o timeout de timeout das queries do EF Core, adicione a propriedade ``DbContextOptions:CommandTimeout`` ao Consul. Essa configuração é expressa em segundos, e seu valor padrão quando não informado é ``30``.

.. code-block:: json

  "DbContextOptions": {
    "CommandTimeout": 10
  }

Habilitando log de parâmetros de queries
----------------------------------------

Para habilitar o log de parâmetros de queries do EF Core, adicione a propriedade ``DbContextOptions:EnableSensitiveDataLogging`` ao Consul. Seu valor padrão quando não informado é ``false``.

.. code-block:: json

  "DbContextOptions": {
    "EnableSensitiveDataLogging": true
  }

Habilitando log detalhado de erros do EF Core
----------------------------------------------

Para habilitar o log detalhados de erros do EF Core, adicione a propriedade ``DbContextOptions:EnableDetailedErrors`` ao Consul. Seu valor padrão quando não informado é ``false``.

.. code-block:: json

  "DbContextOptions": {
    "EnableDetailedErrors": true
  }


Configurando o grau máximo de paralelismo no ServiceBus
-------------------------------------------------------

Para configurar o grau máximo de paralelismo no ServiceBus, adicione a propriedade ``ServiceBus:MaxParallelism`` ao Consul. Seu valor padrão quando não informado é ``5``.

.. code-block:: json

  "ServiceBus": {
    "MaxParallelism": 3
  }

Configurando o número de threads no ServiceBus
----------------------------------------------

Para configurar o número de threads no ServiceBus, adicione a propriedade ``ServiceBus:NumberOfWorkers`` ao Consul. Seu valor padrão quando não informado é ``1``.

.. code-block:: json

  "ServiceBus": {
    "NumberOfWorkers": 1
  }

Configurando o número máximo de tentativas para mensagens que falham no ServiceBus
----------------------------------------------------------------------------------

Para configurar o número máximo de tentativas para mensagens que falham no ServiceBus, adicione a propriedade ``ServiceBus:MaxDeliveryAttempts`` ao Consul. Seu valor padrão quando não informado é ``3``.

.. code-block:: json

  "ServiceBus": {
    "NumberOfWorkers": 1
  }

Configurando o PrefetchSize no ServiceBus
-----------------------------------------

Para configurar o PrefetchSize no ServiceBus, adicione a propriedade ``ServiceBus:PrefetchSize`` ao Consul. Seu valor padrão quando não informado é ``50``.

.. code-block:: json

  "ServiceBus": {
    "Transport": {
      "PrefetchSize": 100
    }    
  }