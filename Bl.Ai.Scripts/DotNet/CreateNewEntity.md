Plase, Create a test for my application, I'll use it in my domain layer.

- Example of my entity:
```c#
/// <summary>
/// Represents costs associated with external resources consumed during a search operation.
/// </summary>
public class ExternalResourceConsumed : Entity
{
    public long SearchId { get; private set; }
    public string SearchName { get; private set; } = string.Empty;
    public string ExternalService { get; private set; } = string.Empty;
    public DateTime ExecutionTime { get; private set; }
    public decimal Cost { get; private set; }
    public string Currency { get; private set; } = "BRL";
    public ExternalResourceConsumedStatus Status { get; private set; }
    public string? ErrorMessage { get; private set; }
    public string? InitiatedBy { get; private set; }
    public string? ProjectCode { get; private set; }
    public object RequestedData { get; private set; } = new object();
    public object? ResultData { get; private set; }
    public DateTime UpdatedAt { get; private set; }
    public DateTime CreatedAt { get; private set; }

    private ExternalResourceConsumed()
    {
    }

    public override bool Equals(object? obj)
    {
        return obj is ExternalResourceConsumed search &&
               EntityId.Equals(search.EntityId) &&
               SearchId == search.SearchId &&
               SearchName == search.SearchName &&
               ExternalService == search.ExternalService &&
               ExecutionTime == search.ExecutionTime &&
               Cost == search.Cost &&
               Currency == search.Currency &&
               Status == search.Status &&
               ErrorMessage == search.ErrorMessage &&
               InitiatedBy == search.InitiatedBy &&
               ProjectCode == search.ProjectCode &&
               UpdatedAt == search.UpdatedAt &&
               RequestedData == search.RequestedData &&
               ResultData == search.ResultData &&
               CreatedAt == search.CreatedAt;
    }

    public override int GetHashCode()
    {
        HashCode hash = new();
        hash.Add(EntityId);
        hash.Add(SearchId);
        hash.Add(SearchName);
        hash.Add(ExternalService);
        hash.Add(ExecutionTime);
        hash.Add(Cost);
        hash.Add(Currency);
        hash.Add(Status);
        hash.Add(ErrorMessage);
        hash.Add(InitiatedBy);
        hash.Add(ProjectCode);
        hash.Add(UpdatedAt);
        hash.Add(CreatedAt);
        hash.Add(RequestedData);
        hash.Add(ResultData);
        return hash.ToHashCode();
    }

    public bool CanBeUpdated()
        => Status == ExternalResourceConsumedStatus.Inicialized;

    public Result SetAsError(object? Response, string? Message)
    {
        ResultBuilder<ExternalResourceConsumed> builder = new();

        builder.AddIf(!CanBeUpdated(), "This resource cannot be updated, it is not in the correct state.");

        return builder.CreateResult(() =>
        {
            ErrorMessage = Message?.TrimEmptySpaces();
            ResultData = Response;
            Status = ExternalResourceConsumedStatus.Failed;
            UpdatedAt = DateTime.UtcNow;
            return this;
        });
    }

    public Result SetAsSuccess(object? Response)
    {
        ResultBuilder<ExternalResourceConsumed> builder = new();

        builder.AddIf(!CanBeUpdated(), "This resource cannot be updated, it is not in the correct state.");

        return builder.CreateResult(() =>
        {
            ResultData = Response;
            Status = ExternalResourceConsumedStatus.Completed;
            UpdatedAt = DateTime.UtcNow;
            return this;
        });
    }

    public static Result<ExternalResourceConsumed> Create(
        string searchName,
        string externalService,
        string searchType,
        decimal cost,
        string initiatedBy,
        string projectCode,
        object requestedData)
        => Create(
            searchName: searchName,
            externalService: externalService,
            cost: cost,
            initiatedBy: initiatedBy,
            projectCode: projectCode,
            errorMessage: null,
            resultData: null,
            requestedData: requestedData,
            executionTime: DateTime.UtcNow,
            createdAt: DateTime.UtcNow,
            updatedAt: DateTime.UtcNow);

    public static Result<ExternalResourceConsumed> Create(
        string searchName,
        string externalService,
        decimal cost,
        string initiatedBy,
        string projectCode,
        string? errorMessage,
        object requestedData,
        object? resultData,
        DateTime executionTime,
        DateTime updatedAt,
        DateTime createdAt,
        long searchId = 0)
    {
        ResultBuilder<ExternalResourceConsumed> builder = new();

        searchName = searchName?.TrimEmptySpaces().RemoveAccents() ?? string.Empty;
        externalService = externalService?.TrimEmptySpaces().RemoveAccents() ?? string.Empty;
        initiatedBy = initiatedBy?.TrimEmptySpaces() ?? string.Empty;
        projectCode = projectCode?.TrimEmptySpaces() ?? string.Empty;

        builder.AddIf(string.IsNullOrWhiteSpace(searchName), "Search name is required.");
        builder.AddIf(searchName.Length > 200, "Search name is too long.");
        builder.AddIf(string.IsNullOrWhiteSpace(externalService), "External service is required.");
        builder.AddIf(externalService.Length > 100, "External service name is too long.");
        builder.AddIf(cost < 0, "Cost cannot be negative.");
        builder.AddIf(string.IsNullOrWhiteSpace(initiatedBy), "Initiated by is required.");
        builder.AddIf(initiatedBy.Length > 100, "Initiated by value is too long.");
        builder.AddIf(requestedData is null, "Requested data cannot be null, to start this entity you should have requested some data to consume the service.");

        return builder.CreateResult(() =>
        {
            var status = string.IsNullOrWhiteSpace(errorMessage)
                ? resultData is null ? ExternalResourceConsumedStatus.Inicialized : ExternalResourceConsumedStatus.Completed
                : ExternalResourceConsumedStatus.Failed;

            

            return new ExternalResourceConsumed
            {
                SearchId = searchId,
                SearchName = searchName,
                ExternalService = externalService,
                ExecutionTime = executionTime,
                Cost = cost,
                Status = status,
                ErrorMessage = errorMessage,
                InitiatedBy = initiatedBy,
                ProjectCode = projectCode,
                UpdatedAt = updatedAt,
                CreatedAt = createdAt,
                RequestedData = requestedData!,
                ResultData = resultData,
            };
        });
    }
}
```

The class should be named as: `_class_name`
The entity should have the follow method: 
```c#
public static Result<@ClassNameHere> Create(/*put all the properties here*/)
{
    // insert your code here
}
```
The entity should have the follow properties:
`_add_properties_here`