# ApiController
```c#
public class CustomerInstructionsHandler : ApiController, ICustomerInstructions
    {
        private readonly Lazy<IGetCustomerInstructionQueryHandler> _getCustomerInstructionQueryHandler;
        public CustomerInstructionsHandler(
            Lazy<IGetCustomerInstructionQueryHandler> getCustomerInstructionQueryHandler,
            Lazy<IGetCustomerInstructionByIdQueryHandler> getCustomerInstructionByIdQueryHandler,
            Lazy<ICreateCustomerInstructionWithCustomerResponseCommandHandler> createCustomerInstructionCommandHandler,
            Lazy<IUpdateCustomerInstructionCommandHandler> updateCustomerInstructionCommandHandler)
        {
            _getCustomerInstructionQueryHandler = getCustomerInstructionQueryHandler;
            _getCustomerInstructionByIdQueryHandler = getCustomerInstructionByIdQueryHandler;
            _createCustomerInstructionCommandHandler = createCustomerInstructionCommandHandler;
            _updateCustomerInstructionCommandHandler = updateCustomerInstructionCommandHandler;
        }

        public CustomerInstructionCollection Get([FromUri] CustomerInstructionQuery query)
        {
            var customerInstruction = _getCustomerInstructionQueryHandler.Value.Execute(new GetCustomerInstructionQuery { WorkItemId = query.WorkItemId });

            return Mapper.Map<IList<Fnz.OneX.Legacy.Api.Customers.Instructions.CustomerInstruction>, CustomerInstructionCollection>(customerInstruction);
        }
    }
```

# Validator
```c#
public class CreateCommandValidator: FluentValidator<CreateCommand> {
  public CreateCommandValidator(
    IValidator<CreateInstruction> createInstructionValidator
  )
  {
    RuleFor(x => x.Instruction)
    .Cascade(CascadeMode.StopOnFirstFailure)
    .NotNull()
    .WithErrorCodeAndMessage(ErrorCodes.RequiredParameter)
    .When()
    .SetValidator(customerInstructionValidator);
  }
}
```

# QueryHandler
```c#
internal class CustomerRoleTypesQueryHandler : ICustomerRoleTypesQueryHandler
  {
      private readonly ICustomerRoleTypesReader _customerRoleTypesReader;
      private readonly IUserContextProvider _userContextProvider;

      public CustomerRoleTypesQueryHandler(
          ICustomerRoleTypesReader customerRoleTypesReader,
          IUserContextProvider userContextProvider,
          )
      {
          _customerRoleTypesReader = customerRoleTypesReader;
          _userContextProvider = userContextProvider;
      }

      public List<CustomerRoleType> Execute(CustomerRoleTypesQuery query)
      {
          var wrapProvider = query.WrapProvider.IsNullOrEmpty() ? _userContextProvider.GetUserContext().WrapProvider : query.WrapProvider;
          var roles = GetCustomerRoleTypes(wrapProvider, query);
          var parentRoles = GetParentCustomerRoleTypes(wrapProvider, query);
          roles.AddRange(parentRoles);
          return roles;
      }
|
```

# CommandHandler
```c#
public class CreateCustomerInstructionWithCustomerResponseCommandHandler : ICreateCustomerInstructionWithCustomerResponseCommandHandler
  {
      private readonly Lazy<IApplicationQueryHandler> _applicationQueryHandler;
      private readonly ICustomerInstructionWriter _customerInstructionWriter;
      private readonly ITaxReferenceTypeMapper _taxReferenceTypeMapper;
      private readonly Lazy<ISubmitCustomerInstructionCommandHandler> _submitCustomerInstructionCommandHandler;
      private readonly ICustomerExternalIdentifierStrategy _customerExternalIdentifierStrategy;
      private readonly ICustomerStatusOverrideStrategy _customerStatusOverrideStrategy;
      private readonly IUserContextProvider _userContextProvider;
      private readonly Fnz.OneX.Utils.DataFlow.Events.Publisher.IEventPublisher _eventPublisher;

      public CreateCustomerInstructionWithCustomerResponseCommandHandler(
          Lazy<IApplicationQueryHandler> applicationQueryHandler,
          ICustomerInstructionWriter customerInstructionWriter,
          ITaxReferenceTypeMapper taxReferenceTypeMapper,
          Lazy<ISubmitCustomerInstructionCommandHandler> submitCustomerInstructionCommandHandler,
          ICustomerExternalIdentifierStrategy customerExternalIdentifierStrategy,
          ICustomerStatusOverrideStrategy customerStatusOverrideStrategy,
          IUserContextProvider userContextProvider,
          Fnz.OneX.Utils.DataFlow.Events.Publisher.IEventPublisher eventPublisher)
      {
          _applicationQueryHandler = applicationQueryHandler;
          _customerInstructionWriter = customerInstructionWriter;
          _taxReferenceTypeMapper = taxReferenceTypeMapper;
          _submitCustomerInstructionCommandHandler = submitCustomerInstructionCommandHandler;
          _customerExternalIdentifierStrategy = customerExternalIdentifierStrategy;
          _customerStatusOverrideStrategy = customerStatusOverrideStrategy;
          _userContextProvider = userContextProvider;
          _eventPublisher = eventPublisher;
      }
      public CustomerCreatedFromInstructionResponse Execute(CreateCustomerInstructionWithCustomerResponseCommand command)
      {
          Customer customer = null;

          _customerStatusOverrideStrategy.OverrideCustomerInstructionCustomerStatus(command.Instruction);

          using (var transaction = new TransactionAndConnectionScope())
          {
              _customerExternalIdentifierStrategy.ValidateExternalIdentifier(command.Instruction.ExternalCustomerId); // This needs to be inside th transaction scope to enforce uniqueness on the ExternalCustomerId and will rollback if the transaction fails
              _taxReferenceTypeMapper.SetTaxReferenceTypeId(command.Instruction.PersonalDetails);
              command.Instruction.Id = _customerInstructionWriter.Create(command.Instruction, _userContextProvider.GetUserContext().UserId);

              if (command.Instruction.IsSubmitted())
              {
                  var application = _applicationQueryHandler.Value.ExecuteWithFallbackResult(new ApplicationQuery { ApplicationName = command.ApplicationName });
                  customer = _submitCustomerInstructionCommandHandler.Value.Execute(new SubmitCustomerInstructionCommand
                  {
                      Instruction = command.Instruction, ApplicationId = application.ApplicationId
                  });
              }

              _eventPublisher.Publish(new InstructionCreatedEventArgs(command.Instruction));

              transaction.Complete();
          }

          var hierarchyId = new HierarchyId { EntityId = command.Instruction.Id.Value, Type = HierarchyLevelName.CustomerInstruction };

          return new CustomerCreatedFromInstructionResponse
          {
              InstructionHierarchyId = hierarchyId,
              Customer = customer
          };
      }
  }
```

# Writer
```C#
public class CustomerInstructionAmlDetailsWriter : DataAccessBase, ICustomerInstructionAmlDetailsWriter
{
    public CustomerInstructionAmlDetailsWriter(IDataAccess dal) : base(dal)
    {
    }

    public void Create(int customerInstructionId, CustomerAmlDetails amlDetails)
    {
        var recordset = new CustomerInstructionAmlDetailsMapper().CreateRecordset(amlDetails);
        recordset.Apply(row => row[CustomerInstructionAMLDetailsTable.Columns.CustomerInstructionId] = customerInstructionId);
        recordset.RowStatus = Recordset.RecordStatus.Added;

        DataAccess.SaveRecordset<CustomerInstructionAMLDetailsTable>(recordset, false);
    }
    public void Delete(int instructionId)
        {
            QueryFactory
                .Procedure<DeleteCustomerInstructionRetirementDetailsProcedure>()
                .WithParameters(instructionId)
                .Execute();
        }
}
```
# Reader
```C#
    internal class CustomerInstructionConsentsReader : DataAccessBase, ICustomerInstructionConsentsReader
    {
        private readonly IClassEnumParser _classEnumParser;
        private readonly IEnumCache _enumCache;

        public CustomerInstructionConsentsReader
            (IDataAccessBaseParameter dataAccessBaseParameter,
             IClassEnumParser classEnumParser,
             IEnumCache enumCache) : base(dataAccessBaseParameter)
        {
            _classEnumParser = classEnumParser;
            _enumCache = enumCache;
        }

        public List<Consent> Get(int customerInstructionId)
        {
            return QueryFactory
                .Select()
                .From<CustomerInstructionConsentsTable>()
                .Where(CustomerInstructionConsentsTable.Columns.CustomerInstructionId, SqlOperatorString.Equal,
                    customerInstructionId)
                .Execute(new CustomerInstructionConsentMapper(_classEnumParser, _enumCache))
                .ToList();
        }
    }
```
# Mapper
```C#
    public class CustomerInstructionAmlDetailsMapper : RecordsetMapper<CustomerAmlDetails>
    {
        public CustomerInstructionAmlDetailsMapper() : base(new ActivatorFactory())
        {
            Maps(x => x.NatureAndPurpose).To(CustomerInstructionAMLDetailsTable.Columns.NatureAndPurpose);
            Maps(x => x.AmlRiskLastReviewDate).To(CustomerInstructionAMLDetailsTable.Columns.AmlRiskLastReviewDate);
        }
    }
```


# UserProvider
(in ASP.NET web applications HttpContext.User is copied into Thread.CurrentPrincipal for each new request)
```C#
    public class UserProvider : IUserProvider
    {
        public int UserId
        {
            get
            {
                return Thread.CurrentPrincipal.GetUserId();
            }
        }

        public int OriginalUserId
        {
            get
            {
                return Thread.CurrentPrincipal.GetOriginalUserId();
            }
        }

        public string UserName
        {
            get
            {
                return Thread.CurrentPrincipal.Identity.Name;
            }
        }
    }
```


# Installer
```C#
    public class ServiceClientInstaller : IWindsorInstaller
    {
        public void Install(IWindsorContainer container, IConfigurationStore store)
        {
            container.Register(
               Classes.FromThisAssembly()
               .BasedOn(typeof(BaseClient<>))
               .LifestyleTransient()
               .WithServiceAllInterfaces()
               .WithServiceSelf());
        }
    }
```

# Keypoints
## Service lifetimes
Singletons - These may exist for the entire application wide configuration setting, for example a game manager that keeps track of players progress throughout the game.
Scoped - entity framework contexts are recommended to be scoped so that you can reuse the connection properties.
Transient - entity framework contexts can not be shared by 2 threads, so if you wanted to do any asynchronous work. You would use a transient so that a new instance of the context is created for every component. Otherwise you would have to wait for the scoped component to finish before it moves onto the next.
