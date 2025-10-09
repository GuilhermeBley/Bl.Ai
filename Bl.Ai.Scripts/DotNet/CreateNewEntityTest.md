Plase, Create a test for my application, I'll use it in my domain layer.

- Example of my entity:
```c#
/// <summary>
/// This entity represents a current payment that will be charged in a subscription.
/// </summary>
public class PaymentService
    : Entity,
    ITrackedSearchProvider
{
    public long Id { get; private set; }
    /// <summary>
    /// The ID of the subscription.
    /// </summary>
    /// <remarks>
    /// Relationship with <see cref="Subscription"/>
    /// </remarks>
    public long SubscriptionId { get; private set; }
    /// <summary>
    /// This field ensures that a payment will be charged only once a month.
    /// </summary>
    /// <remarks>
    /// Union of SubscriptionId, BaseDate, PaymentKey.
    /// </remarks>
    public string PartialKey => string.Concat(SubscriptionId, BaseDate.ToString("yyyy-MM"), PaymentKey);
    /// <summary>
    /// The ID of the service.
    /// </summary>
    /// <remarks>
    /// Relationship with <see cref="CommercialService"/>.
    /// </remarks>
    public int ServiceId { get; private set; }
    public string PaymentKey { get; private set; } = string.Empty;
    public BaseDate BaseDate { get; private set; }

    /// <summary>
    /// BRL value
    /// </summary>
    public decimal Value { get; private set; }
    public DateTime? UpdatedManuallyAt { get; private set; }
    public DateTime PaymentGeneratedAt { get; private set; }
    public DateTime InsertedAt { get; private set; }
    public string? Obs { get; private set; }

    private PaymentService() { }

    public override bool Equals(object? obj)
    {
        return obj is PaymentService service &&
               EntityId.Equals(service.EntityId) &&
               Id == service.Id &&
               SubscriptionId == service.SubscriptionId &&
               PartialKey == service.PartialKey &&
               ServiceId == service.ServiceId &&
               PaymentKey == service.PaymentKey &&
               BaseDate.Equals(service.BaseDate) &&
               Value == service.Value &&
               PaymentGeneratedAt == service.PaymentGeneratedAt &&
               Obs == service.Obs &&
               InsertedAt == service.InsertedAt;
    }

    public IEnumerable<TrackedSearchState> GetCurrentTrack()
    {
        return [
            new TrackedSearchState<PaymentService>(Id.ToString(), SubscriptionId, ServiceId, PaymentKey, BaseDate, Value, PaymentGeneratedAt)
        ];
    }

    public override int GetHashCode()
    {
        HashCode hash = new HashCode();
        hash.Add(EntityId);
        hash.Add(Id);
        hash.Add(SubscriptionId);
        hash.Add(PartialKey);
        hash.Add(ServiceId);
        hash.Add(PaymentKey);
        hash.Add(BaseDate);
        hash.Add(Value);
        hash.Add(Obs);
        hash.Add(PaymentGeneratedAt);
        hash.Add(InsertedAt);
        return hash.ToHashCode();
    }

    public static Result<PaymentService> Create(
        long id,
        long subscriptionId,
        int serviceId,
        string paymentKey,
        decimal value,
        string? obs = null,
        DateOnly? baseDate = null,
        DateTime? paymentGeneratedAt = null,
        DateTime? insertedAt = null,
        DateTime? updatedManuallyAt = null,
        IDateTimeProvider? dateProvider = null)
    {
        dateProvider = dateProvider ?? DateTimeProvider.Default;
        paymentKey = paymentKey.TrimEmptySpaces() ?? string.Empty;

        ResultBuilder<PaymentService> builder = new();

        builder.AddIf(paymentKey.Length < 2 || paymentKey.Length > 50, "Invalid payment key length.");

        builder.AddIf(value < 0, "Invalid value.");

        var parsedBaseDate = baseDate is null ? new BaseDate(dateProvider.UtcNow) : new BaseDate(baseDate.Value);
        var paymentGenerationDate = paymentGeneratedAt is null ? dateProvider.UtcNow : paymentGeneratedAt.Value;
        var parsedInsertedAt = insertedAt ?? dateProvider.UtcNow;

        builder.AddIf(!parsedBaseDate.Equals(new BaseDate(paymentGenerationDate)), "The base date should be the same of the payment generation date.");

        if (string.IsNullOrWhiteSpace(obs)) obs = null;
        else
            obs = string.Concat(obs.TrimEmptySpaces().Take(50)).ToUpperInvariant();

        return builder.CreateResult(() =>
        {
            return new PaymentService()
            {
                BaseDate = parsedBaseDate,
                PaymentGeneratedAt = paymentGenerationDate,
                Id = id,
                InsertedAt = parsedInsertedAt,
                PaymentKey = paymentKey,
                ServiceId = serviceId,
                SubscriptionId = subscriptionId,
                Value = value,
                UpdatedManuallyAt = updatedManuallyAt,
                Obs = obs
            };
        });
    }
}
```

- Expected test output:
```dotnet
namespace Smartec.Robos.Domain.Test.EntitiesTests.Financial;

public class PaymentServiceTest
{
    [Fact]
    public void Create_ShouldCreateAValidEntity()
    {
        var entity = Create();

        Assert.NotNull(entity);
    }

    [Theory]
    [InlineData("2024-02-01", "2024-01")]
    public void Create_ShouldFailIfPaymentGenerationIsDifferentFromPaymentBaseDate(string paymentDate, string baseDate)
    {
        var parsedBaseDate = DateOnly.ParseExact(baseDate, "yyyy-MM");
        var parsedPaymentDate = DateTime.ParseExact(paymentDate, "yyyy-MM-dd", System.Globalization.CultureInfo.InvariantCulture);

        var act = () => Create(baseDate: parsedBaseDate, paymentGeneratedAt: parsedPaymentDate);

        Assert.ThrowsAny<CoreException>(act);
    }

    [Theory]
    [InlineData(null, null)]
    [InlineData("", null)]
    [InlineData("my obs", "MY OBS")]
    [InlineData(" my obs ", "MY OBS")]
    [InlineData("1234567890123456789012345678901234567890123456789012345678901234567890", "12345678901234567890123456789012345678901234567890")] // max 50 chars
    public void Create_ShouldMatchObs(string input, string expected)
    {
        var entity = Create(obs: input);

        Assert.Equal(expected, entity.Obs);
    }

    [Theory]
    [InlineData(-1)]
    public void Create_ShouldFailIfValueIsInvalid(decimal value)
    {
        var act = () => Create(value: value);

        Assert.ThrowsAny<CoreException>(act);
    }

    [Fact]
    public void Equal_ShouldBeEqual()
    {
        var entity = Create();

        Assert.Equal(entity, entity);
    }

    [Fact]
    public void Equal_ShouldNotBeEqual()
    {
        var entity1 = Create();
        var entity2 = Create();

        Assert.NotEqual(entity1, entity2);
    }

    public static PaymentService Create(
        long id = 0,
        long subscriptionId = 1,
        int serviceId = 1, 
        string paymentKey = "my key",
        decimal value = 1,
        string? obs = null,
        DateOnly? baseDate = null,
        DateTime? paymentGeneratedAt = null,
        DateTime? insertedAt = null,
        IDateTimeProvider? dateProvider = null)
    {
        return PaymentService.Create(
            id: id,
            subscriptionId: subscriptionId,
            serviceId: serviceId,
            paymentKey: paymentKey,
            value: value,
            obs: obs,
            baseDate: baseDate,
            paymentGeneratedAt: paymentGeneratedAt ,
            insertedAt: insertedAt ,
            dateProvider: dateProvider)
            .RequiredResult;
    }
}

```

- Now Create a test based on this entity:

<add-your-code-here>