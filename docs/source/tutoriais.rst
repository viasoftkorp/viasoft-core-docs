Tutoriais
#########

.. _ef-property-versioning:

Adicionando versionamento em propriedades de entidades EF Core
--------------------------------------------------------------

Em cenários onde diferentes versões do banco de dados podem ou não conter determinadas colunas, é possível controlar o mapeamento de propriedades no EF Core utilizando o atributo ``[MinimumVersion("2025.1.0")]``.

O parâmetro ``minimumVersion`` refere-se a versão mínima necessária do banco de dados para que a propriedade seja considerada, caso contrário ela será ignorada.

.. code-block:: c#

   using Viasoft.Core.AmbientData.Attributes;
   using Viasoft.Core.MultiTenancy.Abstractions.Consts;
   
   [Table("Cities")]
   public class City
   {
      [MinimumVersion(DatabaseVersionConsts.V2024_3_0)]
      public string CityName { get; set; }

      [MinimumVersion(DatabaseVersionConsts.V2025_1_0)]
      public string Postcode { get; set; }

      public decimal? Population { get; set; }
   }

Versionamento em propriedades de entidades EF Core em testes unitários com UnitTestBase
--------------------------------------------------

Quando o versionamento de propriedades por versão de banco está habilitado, os testes baseados em ``UnitTestBase`` passam a depender de quando a versão desktop é definida.

O fluxo é controlado por três pontos:

- ``CreateDatabaseExplicitly``: define se o banco será criado automaticamente no construtor.
- ``DefaultDesktopDatabaseVersion``: versão usada na criação automática.
- ``SetVersionAndCreateDatabase(version)``: define a versão no AmbientData e cria o banco.

Regra importante: ``SetVersionAndCreateDatabase`` só pode ser chamado uma vez por instância de teste. Após a criação do banco, uma nova chamada lança ``InvalidOperationException``.

Compatibilidade e migração de testes existentes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Por padrão, testes que **não** precisam controlar explicitamente a versão do banco de dados **não precisam de nenhuma alteração**.

Para testes que antes definiam a versão dentro de ``ConfigureServices``, o padrão recomendado agora é sobrescrever ``DefaultDesktopDatabaseVersion``.

Antes:

.. code-block:: c#

  protected override void ConfigureServices()
  {
    var tenantProperties = ServiceProvider.GetService<ITenantProperties>();
    tenantProperties.SetDesktopDatabaseVersion("2024.3.0");
    base.ConfigureServices();
  }

Depois:

.. code-block:: c#

  protected override string DefaultDesktopDatabaseVersion => DatabaseVersionConsts.V2024_3_0;

Modo 1: criação automática no construtor (padrão)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use quando todos os testes da classe podem usar a mesma versão.

.. code-block:: c#

   public class LegacyCityUnitTest : UnitTestBase
   {
     protected override string DefaultDesktopDatabaseVersion => "2025.2.0";

     protected override Type GetDbContextType()
     {
       return typeof(LegacyTesteEfCoreContext);
     }
   }

Nesse modo, o construtor de ``UnitTestBase`` já executa internamente:

.. code-block:: c#

   SetVersionAndCreateDatabase(DefaultDesktopDatabaseVersion);

Portanto, chamar ``SetVersionAndCreateDatabase`` novamente dentro do teste gera exceção.

Modo 2: criação explícita por método de teste
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use quando cada teste precisa validar comportamentos com versões diferentes.

.. code-block:: c#

   public class EntityPropertyVersioningTestBase : UnitTestBase
   {
     protected override bool CreateDatabaseExplicitly => true;

     protected override Type GetDbContextType()
     {
       return typeof(PropertyVersioningDbContext);
     }
   }

Com ``CreateDatabaseExplicitly => true``, cada teste define a versão desejada antes de usar o repositório/DbContext:

.. code-block:: c#

   [Theory]
   [InlineData(DatabaseVersionConsts.V2023_4_0)]
   [InlineData(DatabaseVersionConsts.V2024_1_0)]
   [InlineData(DatabaseVersionConsts.V2025_1_0)]
   public void ShouldRespectPropertyVersioning(string version)
   {
     SetVersionAndCreateDatabase(version);

     // Executa asserções considerando o mapeamento da versão informada.
   }

Resumo prático
^^^^^^^^^^^^^^

- Se o teste não depende de uma versão específica de banco: nenhuma alteração é necessária.
- Se a versão é fixa na classe de teste: sobrescreva apenas ``DefaultDesktopDatabaseVersion``.
- Se a versão muda por cenário: sobrescreva ``CreateDatabaseExplicitly`` como ``true`` e chame ``SetVersionAndCreateDatabase`` no início do teste.
- Nunca chame ``SetVersionAndCreateDatabase`` duas vezes na mesma instância de teste.

Adicionando versionamento a API
-------------------------------
.. _aspnet-api-versioning: https://github.com/dotnet/aspnet-api-versioning/

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


.. _retrieving-ambient-data:

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
Environment          GetEnvironment()
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
    "MaxDeliveryAttempts": 7
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

.. _configurando-override-property-normalizer:

Configurando quando as propriedades normalizadas devem ser sobrescritas
-----------------------------------------------------------------------

Para configurar quando os valores de propriedades normalizadas devem ser sobrescritos, configure as propriedades ``AlwaysOverrideCompanyValue`` e ``AlwaysOverrideCreationTimeValue`` ao utilizar o método ``AddDomainDrivenDesign``. O valor padrão das duas propriedades é ``true``.

- AlwaysOverrideCompanyValue terá impacto nas interfaces ``IMustHaveCompany`` e ``IMustHaveLegacyCompany``
- AlwaysOverrideCreationTimeValue terá impacto na interface ``ICreationAuditedEntity``.

.. code-block:: c#
  
  .AddDomainDrivenDesign(options =>
  {
      options.CompanyNormalizerOptions.AlwaysOverrideCompanyValue = false;
      options.CreationAuditOptions.AlwaysOverrideCreationTimeValue = false;
  })