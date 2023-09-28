Visão Geral
===========

Lista de issues nessa versão
----------------------------

SDKBACK-94_

SDKBACK-71_

SDKBACK-77_

SDKBACK-79_

SDKBACK-76_

SDKBACK-18_

.. _SDKBACK-94: https://korp.youtrack.cloud/issue/SDKBACK-94
.. _SDKBACK-71: https://korp.youtrack.cloud/issue/SDKBACK-71
.. _SDKBACK-77: https://korp.youtrack.cloud/issue/SDKBACK-77
.. _SDKBACK-79: https://korp.youtrack.cloud/issue/SDKBACK-79
.. _SDKBACK-76: https://korp.youtrack.cloud/issue/SDKBACK-76
.. _SDKBACK-18: https://korp.youtrack.cloud/issue/SDKBACK-18

Breaking Changes de tempo de execução
-------------------------------------

* A extensão ``GetEndpoint`` do ``HttpContext`` **sempre** retornará nulo em contributors de AmbientData. Essa breaking change afetará códigos que resolvem AmbientData para endpoints ou controllers específicas, por exemplo:

.. code-block:: c#

  public class DefaultEnvironmentResolveContributorErp : IEnvironmentResolveContributor
  {
      public int? Order { get; set; }
      private readonly IHttpContextAccessor _httpContextAccessor;
      private static readonly List<MethodInfo> Endpoints = new()
      {
          typeof(SimulationController).GetMethod(nameof(SimulationController.GetSimulationResultById))
      };
      
      public DefaultEnvironmentResolveContributorErp(IHttpContextAccessor httpContextAccessor)
      {
          _httpContextAccessor = httpContextAccessor;
      }
      
      public async ValueTask<AmbientDataResolveResult> TryResolveEnvironment()
      {
        // a partir da 4.10 a variável ``endpoint`` estará *sempre nula*
        var endpoint = _httpContextAccessor.HttpContext.GetEndpoint();
        if (endpoint is not null)
        {
            var controllerDescriptor = endpoint.Metadata.GetMetadata<ControllerActionDescriptor>();
            if (Endpoints.Contains(controllerDescriptor?.MethodInfo))
            {
              var currentTenant = _httpContextAccessor.HttpContext.RequestServices.GetRequiredService<ICurrentTenant>();
              var environmentStore = _httpContextAccessor.HttpContext.RequestServices.GetRequiredService<IEnvironmentStore>();
              var environmentId = await environmentStore.GetDefaultEnvironmentAsync(currentTenant.Id);
            
              return AmbientDataResolveResult<Guid>.Handle(environmentId);
            }
        }

        return AmbientDataResolveResult.NotHandled;
      }
  }

Para corrigir essa breaking change, será necessário fazer a verificação pela propriedade ``Path`` da requisição:

.. code-block:: c#

  private const string SimulationResultById = "/Production/APS/Simulation/Simulation/GetSimulationResultById";

  public async ValueTask<AmbientDataResolveResult> TryResolveEnvironment()
  {
    var context = _httpContextAccessor.HttpContext;
    if (context is not null) 
    {
      var isSimulationResultByIdPath = context.Request.Path.ToString().Contains(SimulationResultById, StringComparison.OrdinalIgnoreCase);
      if (isSimulationResultByIdPath)
      {
          var currentTenant = context.RequestServices.GetRequiredService<ICurrentTenant>();
          var environmentStore = context.RequestServices.GetRequiredService<IEnvironmentStore>();
          var environmentId = await environmentStore.GetDefaultEnvironmentAsync(currentTenant.Id);
      
          return AmbientDataResolveResult<Guid>.Handle(environmentId);
      }
    }

    return AmbientDataResolveResult.NotHandled;
  }    


* O AutoMapper não é mais adicionado mais nos testes unitários, com isso:

  - A propriedade ``Mapper`` foi removida
  - Os Profiles não são mais adicionados ao mapping do AutoMapper
  
  Se você necessita de AutoMapper no seu teste unitário, verifique :ref:`adicionando-automapper-em-testes-unitarios`.

* O mock do serviço ``ICompanyStore`` não é mais adicionado.

* Os testes unitários agora utilizam SQLite como provedor de banco de dados, antes era em memória. Nos nossos testes isso não gerou nenhum problema significativo, porém se encontrar dificuldades falar com a equipe de tecnologia.

  Problemas já conhecidos ao trocar para SQLite

    - A ordenação de queries sem order by pode ter sido alterada, fazendo com que acessos por indice de array não tragam o resultado esperado

Breaking Changes de compilação
------------------------------

* O método ``ResolveData`` da interface ``IAmbientDataResolverContributor`` teve seu tipo de retorno alterado para ``ValueTask``. Também houve uma mudança de comportamento em que as classes que implementam essa interface **não** devem mais validar a obrigatoriedade do dado pelo método ``ResolveData``. A obrigatoriedade deve ser feita implementando a interface ``IAmbientDataRequiredContributor``.

* O método ``TryResolveUser`` da interface ``IUserResolveContributor`` teve seu tipo de retorno alterado para ``ValueTask<AmbientDataResolveResult>``.

* O método ``TryResolveCompany`` da interface ``ICompanyResolveContributor`` teve seu tipo de retorno alterado para ``ValueTask<AmbientDataResolveResult>``.

* O método ``TryResolveEnvironment`` da interface ``IEnvironmentResolveContributor`` teve seu tipo de retorno alterado para ``ValueTask<AmbientDataResolveResult>``.

* O método ``TryResolveLegacyCompany`` da interface ``ILegacyCompanyResolveContributor`` teve seu tipo de retorno alterado para ``ValueTask<AmbientDataResolveResult>``.

* O método ``TryResolveNotificationUpdate`` da interface ``INotificationUpdateContributor`` teve seu tipo de retorno alterado para ``ValueTask<AmbientDataResolveResult>``.

.. note::

   Veja um exemplo de como implementar um `contributor` de AmbientData em :ref:`implementando-um-ambientdata-contributor`.

* O método ``ResolveAmbientData`` da interface ``IAmbientDataResolver`` teve seu tipo de retorno alterado para ``ValueTask``.   

* A extensão ``AddAmbientDataResolver`` foi renomeada para ``AddAmbientData``.

* A extensão ``GetIdOrThrow`` da interface ``ICurrentEnvironment`` foi removida. Para corrigir, injete ``IAmbientData`` e utilize a extensão ``GetEnvironmentIdOrThrow``,

* O construtor da classe ``AmbientDataCallOptionsResolver`` foi alterado e agora precisa de dois parâmetros do tipo ``IAmbientData`` e ``IHttpContextAccessor``. Com isso, a propriedade ``ServiceProvider`` deixou de existir.

* O método ``GetCompanyDetails`` da interface ``ICurrentCompany`` foi removido. Para corrigir, injete ``ICompanyStore`` e utilize o método ``GetCompanyDetailsAsync``.

* As opções para configuração de obrigatoriedade de AmbientData foram remodeladas

  - MustUseMultiTenancyPredicate removido
  - MustUseCompanyPredicate removido
  - MustUseEnvironmentPredicate removido
  - MustUseUserIdentityPredicate removido

  Se você utilizava os callbacks para desabilitar globalmente a obrigatoriedade, verifique :ref:`desabilitando-obrigatoriedade-ambientdata`.

* O método ``GetDbContextTypes()`` foi renomeado para ``GetDbContextType()`` e com isso seu retorno foi alterado de ``List<Type>`` para ``Type``.

.. note::

  Se você tiver um teste que utiliza mais que um ``DbContext``, terá que separar em duas classes de testes unitários.

* As seguintes classes foram *removidas*

  - CompanyHeaderExtension
  - CompanyResolveResult
  - EnvironmentHeaderExtensions
  - EnvironmentResolveResult
  - LegacyCompanyHeaderExtension
  - LegacyCompanyResolveResult
  - NotificationUpdateResolveResult
  - NotificationUpdateHeaderExtension
  - TenancyResolveResult
  - TenancyMessageHeadersExtensions
  - UserResolveResult
  - UserHeaderExtension
  - AuthorizationHttpHeaderStrategy

* As seguintes interfaces foram *removidas*

  - ICompany
  - ITenant
  - IEnvironment

* As seguinte classes de exceções foram removidas. Agora o SDK lançará as exceções pelo tipo ``MissingAmbientData``, preenchendo a propriedade ``AmbientDataName`` com o valor adequado.

  - CannotNotEvaluateUser
  - CannotNotEvaluateTenant
  - CannotNotEvaluateLegacyCompany
  - CannotNotEvaluateEnvironment
  - CannotNotEvaluateCompany

Correções
---------

* Proxy do FileProvider para arquivos com acentuação está funcionando corretamente.

Features
--------

- Adicionar ``Viasoft.Core.LiquidTemplate``, para renderização de templates utilizando a linguagem Liquid:
    - Para utilização, basta adicionar ``.AddLiquidTemplate()`` no ``Startup.cs`` e injetar ``ILiquidExpressionService`` onde necessário.
