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

.. _injetando-servicos-customizados-dbcontext:

Injetando serviços customizados no DbContext
-------------------------------------------------

Realize a injeção das dependências no seu dbcontext:

.. code-block:: c#

    private TenantProperties _tenantProperties;
    
    public PostgreSqlDbProviderTestServiceDbContext(DbContextOptions options, ISchemaNameProvider schemaNameProvider, ILoggerFactory loggerFactory, IBaseDbContextConfigurationService configurationService, TenantProperties tenantProperties)
        : base(options, schemaNameProvider, loggerFactory, configurationService)
    {
        _tenantProperties = tenantProperties;
    }

Na classe que DbContextDesignTime que implementa a interface ``PostgreSqlBaseDesignTimeDbContextFactory`` ou ``SqlServerBaseDesignTimeDbContextFactory`` liste os tipos das dependências na propriedade ``AllowedParameters`` no contrutor da classe:

.. code-block:: c#

    public class DbProviderTestServiceDbContextDesignTime : PostgreSqlBaseDesignTimeDbContextFactory<PostgreSqlDbProviderTestServiceDbContext>
    {
        public DbProviderTestServiceDbContextDesignTime()
        {
            AllowedParameters.Add(typeof(ITenantProperties));
        }
    }


Mapeando as entidades para utilizarem ``VARCHAR`` ou ``DECIMAL``
----------------------------------------------------------------

Há disponível dois métodos de extensão para mapeamento da entidade no banco de dados SQL Server.

- Mapeamento de propriedades ``string`` para colunas ``VARCHAR``, utilize ``entityTypeBuilder.ApplyVarcharColumnTypeForEntity()``.
- Mapeamento de propriedades ``decimal`` para colunas ``DECIMAL(19,6)``, utilize ``entityTypeBuilder.ApplyDecimalMappingForEntity()``.

Caso a extensão ``ApplyVarcharColumnTypeForEntity`` não esteja disponível, instale o pacote ``Viasoft.Core.EntityFrameworkCore.SQLServer.Legacy``.

.. warning::

    A extensão que mapeia para ``VARCHAR`` somente deve se utilizada nas entidade que acessarem o banco de dados do ERP desktop.

No método ``OnModelCreating`` do seu ``DbContext``, busque a entidade a ser configurada utilizando o método ``Entity<>`` do ``ModelBuilder``.

.. code-block:: c#

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        var estoque = modelBuilder.Entity<Estoque>();
    }

Utilize os métodos de extensão ``ApplyVarcharColumnTypeForEntity`` ou ``ApplyDecimalMappingForEntity`` para configurar.

.. code-block:: c#

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        var estoque = modelBuilder.Entity<Estoque>();

        stoque.ToTable("ESTOQUE");
        
        estoque
            .Property(entity => entity.Codigo)
            .HasColumnName("CODIGO")
            .HasMaxLength(50);
        
        estoque
            .Property(entity => entity.Descricao)
            .HasColumnName("DESCRI")
            .HasMaxLength(200);

        estoque
            .Property(entity => entity.SaldoReal)
            .HasColumnName("SALDO");

        estoque.ApplyDecimalMappingForEntity();
        estoque.ApplyVarcharColumnTypeForEntity();
    }

Configurando as consultas do AdvancedFilter para case sensitive/insensitive
---------------------------------------------------------------------------

Para configurar as consultas do AdvancedFilter, utilize a flag ``BuildExpressionOptions.Default.CaseInsensitiveStringComparision`` no ``Startup``.

- Case sensitive

.. code-block:: c#

    public void ConfigureServices(IServiceCollection services)
    {
        BuildExpressionOptions.Default.CaseInsensitiveStringComparision = false;
    }

- Case insensitive

.. code-block:: c#

    public void ConfigureServices(IServiceCollection services)
    {
        BuildExpressionOptions.Default.CaseInsensitiveStringComparision = true;
    }

.. warning::

    Os serviços que acessam SQL Server devem utilizar a flag com o valor ``false`` pois a configuração de *sensitivity* está a nível de banco de dados.

Utilizando o System.Text.JSON nas chamadas HTTP
-----------------------------------------------

Para utilizar o serializdor ``System.Text.JSON.JsonSerializer`` utilize o método ``UseNewSerializer`` informando os dois parâmetros como ``true``.

.. code-block:: c#

    public Task<Cliente> RequisitarCliente(Guid id)
    {
       var call = _apiClientCallBuilder
            .WithEndpoint($"/clientes/{id}")
            .WithServiceName(Endpoints.ServiceName)
            .WithHttpMethod(HttpMethod.Get)
            .UseNewSerializer(true, true)
            .Build();
            
        var result = await call.ResponseCallAsync<Cliente>();
        return result;
    }

Adicionando um step customizado ao pipeline de execução do ServiceBus
---------------------------------------------------------------------

Ao registrar o ServiceBus no `IServiceCollection` é possível configurar um *step* customizado para ser executado no momento em que uma mensagem é recebida ou enviada. Esse passo permite alterar o comportamento do pipeline, incluindo bloqueio de execução de *handlers*, enriquecimento de contexto ou métricas.

Passos para configurar:

1. Crie o step herdando de ``CustomIncomingStepBase`` (ou ``IOutgoingStep`` para pipeline de saída) e implemente ``ExecuteAsync``.
2. Registre o step via ``AddServiceBus`` utilizando a opção ``PipelineConfigurer``.
3. Defina a posição relativa no pipeline com ``PipelineStepInjector.OnReceive(...)`` (ou ``PipelineStepInjector.OnSend(...)`` para pipeline de saída).

Exemplo de registro no ``Startup``:

.. code-block:: csharp

    serviceCollection.AddServiceBus(options =>
    {
        options.PipelineConfigurer = configurer =>
        {
            configurer.Decorate<IPipeline>(c =>
            {
                var pipeline = c.Get<IPipeline>();
                var step = new AmbientVersionServiceBusCustomIncomingStep(); // Seu step customizado

                return new PipelineStepInjector(pipeline)
                    .OnReceive(step, PipelineRelativePosition.Before, typeof(ProvisioningIncomingStep)); // Ou OnSend para pipeline de saída
            });
        };
    }, ServiceConfiguration, _configuration);

Implementação de um step customizado:

.. code-block:: csharp

    // Step customizado para ignorar handler de acordo com versão do banco
    public class AmbientVersionServiceBusCustomIncomingStep : CustomIncomingStepBase
    {
        protected override async Task ExecuteAsync(IncomingStepContext context, Func<Task> next)
        {
            var serviceProvider = context.GetServiceProvider();
            var ambientData = serviceProvider.GetRequiredService<IAmbientData>();
            var dbVersion = ambientData.GetEnvironmentDesktopDatabaseVersion();

            var handleType = context.GetHandleType();
            var handlerMethod = context.GetHandlerHandleMethod();
            
            // Aceita atributo na classe ou método
            var minVersionAttr = 
                handlerMethod.GetCustomAttribute<MinimumVersionAttribute>(inherit: true)
                ?? handleType.GetCustomAttribute<MinimumVersionAttribute>(inherit: true);
            
            if (minVersionAttr is not null)
            {
                if (string.IsNullOrEmpty(dbVersion))
                {
                    throw new ArgumentException(
                        $"A versão do banco não está disponível para prosseguir com o método {handlerMethod.Name} de {handleType.FullName}");
                }

                if (minVersionAttr.IsVersionGreaterThan(dbVersion))
                {
                    // Interrompe o dispatch do handler e os demais steps customizados
                    context.AbortDispatchAndRemainingCustomSteps();
                }
            }

            await next();
        }
    }

Observações:

* Utilize ``context.GetServiceProvider()`` para acessar serviços do DI no step.
* A ordem dos steps influencia o comportamento. Use ``PipelineRelativePosition.Before/After`` com o tipo de um step existente para posicionamento preciso.
* Para impedir a execução do handler e dos demais ``CustomIncomingStepBase`` seguintes, chame ``context.AbortDispatchAndRemainingCustomSteps()``.