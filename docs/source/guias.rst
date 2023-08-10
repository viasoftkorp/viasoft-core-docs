Guias
=====

.. _implementando-um-ambientdata-contributor:

Implementando um AmbientData contributor
----------------------------------------

Os *contributors* servem para prover um AmbientData no fluxo atual.
Antes de implementar, você terá que visualizar na lista abaixo qual a interface correta para implementação

==================== ================================
AmbientData          Interface a ser implementada
==================== ================================
TenantId             ITenancyResolveContributor
UserId               IUserResolveContributor
CompanyId            ICompanyResolveContributor
EnvironmentId        IEnvironmentResolveContributor
LegacyCompanyId      ILegacyCompanyResolveContributor
NotificationUpdateId INotificationUpdateContributor
ApiVersion           IApiVersionResolveContributor
==================== ================================

Declare a classe herdando do *contributor* apropriado

.. code-block:: csharp

    public class TenantIdFromHeadersResolveContributor: ITenancyResolveContributor 
    {
        public ValueTask<AmbientDataResolveResult> TryResolveTenant()
        {               
        }    
    }

Como o retorno das interfaces sempre é ``AmbientDataResolveResult``, note que terá que especificar o tipo do AmbientData pelo *generic type* no método ``AmbientDataResolveResult<>.Handle``. Esse método automaticamente verifica se o valor informado no parâmetro ``value`` é vazio/nulo, potencialmente fazendo com que o seu *contributor* seja desconsiderado. O valor é considerado vazio/nulo quando:

* O seu tipo é ``Guid`` e o valor resolvido for ``00000000-0000-0000-0000-000000000000``
* O seu tipo é ``int`` e o valor resolvido for ``0``
* O seu tipo é ``string`` e o valor resolvido for "" ou ``null``

.. warning::

    Quando o tipo for ``string`` utilize o método ``HandleString``


Abaixo um exemplo de *contributor* que resolve o TenantId dos headers da requisição

.. code-block:: csharp

    public class TenantIdFromHeadersResolveContributor: ITenancyResolveContributor 
    {
        private readonly IHttpContextAccessor _httpContextAccessor;
        
        public RequestHeadersTenancyResolveContributor(IHttpContextAccessor httpContextAccessor)
        {
            _httpContextAccessor = httpContextAccessor;
        }
        
        public ValueTask<AmbientDataResolveResult> TryResolveTenant()
        {
            if (_httpContextAccessor.HttpContext is not null)
            {
                if (_httpContextAccessor.HttpContext.Request.Headers.TryGetValue(AmbientDataConsts.TenantId, out var tenantIdStr))
                {
                    if (Guid.TryParse(tenantIdStr, out var tenantId))
                    {
                        return ValueTask.FromResult(AmbientDataResolveResult<Guid>.Handle(tenantId));
                    }
                }
            }
            
            return ValueTask.FromResult(AmbientDataResolveResult.NotHandled);
        }   
    }

.. _desabilitando-obrigatoriedade-ambientdata:

Desabilitando a obrigatoriedade de um AmbientData
-------------------------------------------------

Para desablitar a obrigatorieade de um AmbientData, anote o método com um dos seguintes atributos

==================== ==========================
AmbientData          Atributo
==================== ==========================
Todos*               AmbientDataNotRequired
TenantId             TenantNotRequired
UserId               UserNotRequired
CompanyId            CompanyNotRequired
EnvironmentId        EnvironmentNotRequired
LegacyCompanyId      LegacyCompanyNotRequired
==================== ==========================

*Obs: Ao anotar a sua classe ou método com AmbientDataNotRequired, o SDK não obrigará nenhum AmbientData*

.. note:: 

   ``ApiVersion`` e ``NotificationUpdateId`` não são obrigatórios e portanto não geram erro.

Esses atributos podem ser utilizados em handlers de ServiceBus (``IHandleMessages<>``), controllers e background jobs (``IBackgroundJob<>``). Eles tem efeito quando anotados em método ou a nível de classe, nesse segundo caso valerá para todos os métodos da classe.

Para desabilitar a obrigatorieade **globalmente**, a configuração deverá ser feita a nível de ``Startup`` utilizando o parâmetro ``options``. Ajuste os requerimentos conforme a sua necessidade.

.. code-block:: c#

   public void ConfigureServices(IServiceCollection services)
   {
      services.AddMultiTenancy(MultiTenancyOptions.Default().CompanyNotRequired().EnvironmentNotRequired().TenantNotRequired())
              .AddUserIdentity(UserIdentityOptions.Default().UserNotRequired()) 
   }

Se a lógica para desabilitar a obrigatoriedade for complexa, será necessário implementar o método ``IsRequired``.
A interface a ser implementada varia conforme o AmbientData

==================== ============================================
AmbientData          Interface a ser implementada
==================== ============================================
TenantId             ITenantAmbientDataRequiredContributor
UserId               IUserAmbientDataRequiredContributor
CompanyId            ICompanyAmbientDataRequiredContributor
EnvironmentId        IEnvironmentAmbientDataRequiredContributor
LegacyCompanyId      ILegacyCompanyAmbientDataRequiredContributor
==================== ============================================

**Sempre** inicialize o retorno do método ``IsRequired`` com ``true``, pois desse jeito não correrá risco de desobrigar o AmbientData em um fluxo não esperado.

Os casos em que deverá retornar ``true`` por padrão são:

- a operação atual não tem relação ao que está sendo verificado (o *contributor* serve para requisições HTTP porém a ação atual é de um handler)
- a operação atual tem relação ao que está sendo verificado, porém a condição para desativação não é verdadeira

.. code-block:: c#

   public class ServiceBusCompanyAmbientDataRequiredContributor: ICompanyAmbientDataRequiredContributor
   {
        public Task<bool> IsRequired()
        {
            var isRequired = true;
    
            if (MessageContextExtensions.IsHandlingMessage())
            {            
                isRequired = VerificarObrigatoriedade()
            }
    
            return Task.FromResult(isRequired);
        }
   }