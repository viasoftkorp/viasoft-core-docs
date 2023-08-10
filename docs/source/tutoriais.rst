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
