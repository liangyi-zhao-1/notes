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
